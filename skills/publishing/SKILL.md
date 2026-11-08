---
name: publishing
description: Prepare and upload YouTube videos. Handles OAuth2 (refresh-token flow) and resumable uploads via the YouTube Data API. DRY RUN BY DEFAULT — actually uploads only with --confirm/explicit user approval. Use when asked to upload, publish, schedule or go live with a video.
compatibility: Agent with shell/curl access. Requires GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET, GOOGLE_REFRESH_TOKEN (OAuth scope youtube.upload) and YOUTUBE_API_KEY.
---

# Publishing

Upload videos to YouTube — safely.

## Inputs

- Video file (`./project/<slug>/final.mp4`) and metadata (`./project/<slug>/seo.json`)
- Thumbnail (`./project/<slug>/thumbnail.jpg`) — optional but strongly default
- Settings: `privacy` = `private` | `unlisted` | `public` (default `private`), `categoryId` (default `22`), `madeForKids` (default false), `publishAt` (optional RFC3339 scheduled time), `playlistId` optional
- **Explicit confirmation is mandatory before any real upload.**

## Requirements

- `YOUTUBE_API_KEY` — quota/metrics.
- OAuth refresh token with scope `https://www.googleapis.com/auth/youtube.upload` (+ `youtube.readonly` for status checks, `yt-analytics.readonly` for the analytics skill). If absent: stop and walk the user through Google Cloud console steps (create project → enable YouTube Data API v3 → OAuth consent screen → Desktop client → exchange auth code at https://oauth2.googleapis.com/token). Never improvise tokens.

## Workflow

1. **Pre-flight** (always run first): file exists and is a valid MP4 (`ffprobe`), `seo.json` parses, title ≤ 100 chars, description ≤ 5000, tags ≤ 500 total, madeForKids is a boolean, quota remaining estimate (warn if a publish is attempted today after heavy API use).
2. **Dry run** (default): print the full metadata payload + what would be uploaded + privacy status. Stop here unless the user explicitly confirms with the file path typed back or `--confirm` semantics.
3. **Get access token**:

   ```bash
   ACCESS_TOKEN=$(curl -s -X POST https://oauth2.googleapis.com/token \
     -d client_id=$GOOGLE_CLIENT_ID -d client_secret=$GOOGLE_CLIENT_SECRET \
     -d refresh_token=$GOOGLE_REFRESH_TOKEN -d grant_type=refresh_token | jq -r .access_token)
   ```

4. **Resumable upload** — step 1: initiate session; step 2: PUT bytes:

   ```bash
   curl -s -X POST "https://www.googleapis.com/upload/youtube/v3/videos?uploadType=resumable&part=snippet,status" \
     -H "Authorization: Bearer $ACCESS_TOKEN" -H "Content-Type: application/json; charset=UTF-8" \
     -H "X-Upload-Content-Type: video/mp4" -H "X-Upload-Content-Length: $(stat -c%s final.mp4)" \
     -d '{"snippet":{"title":"...","description":"...","tags":["..."],"categoryId":"22"},
          "status":{"privacyStatus":"private","selfDeclaredMadeForKids":false}}' -D - -o /dev/null
   # then: curl -X PUT "$SESSION_URI" -H "Content-Type: video/mp4" --data-binary @final.mp4
   ```

   Quota cost ≈ 1600 units — warn user of daily budget (10,000 default).
   If `publishAt` is set, pass `"publishAt":"..."` in status and note scheduling requires the video to remain private until then.
5. **Thumbnail upload** (after upload, get `videoId`):

   ```bash
   curl -s -X POST "https://www.googleapis.com/upload/youtube/v3/thumbnails/set?videoId=$VID&uploadType=media" \
     -H "Authorization: Bearer $ACCESS_TOKEN" -H "Content-Type: image/jpeg" --data-binary @thumbnail.jpg
   ```

6. **Playlist add** (optional): `POST .../playlistItems?part=snippet` with the playlistId.
7. **Post-upload**: report videoId, URL, privacy, quota consumed; write `./project/<slug>/publish.json` with the videoId (the analytics skill consumes it).

## Safety rails (non-negotiable)

- Default privacy is `private`. Never default to `public`.
- No upload without explicit, informed user confirmation in this session.
- `publishAt` must be ≥ 15 min in the future; invalid times get rejected, not coerced.
- On any non-2xx response: stop, print the response body, do not retryBlindly (retries only on 5xx/network, max 2).

## Quality checks

- All uploaded metadata comes from `seo.json` — no silent changes.
- madeForKids must match the content; defaulting to true on content not made for kids violates COPPA/YouTube policy — ask when unclear.
