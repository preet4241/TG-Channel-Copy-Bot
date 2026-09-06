# Archive Bot

## Run

The application workflow runs:

```bash
python main.py
```

It starts the Flask dashboard on the configured `PORT` (8080 locally) and the
Telethon userbot plus Telegram Bot API controller in the same process.

## Required Replit Secrets

`API_ID`, `API_HASH`, `PHONE`, `OWNER_ID`, and `BOT_TOKEN` must be configured
in Replit Secrets. The Telethon session is persisted to `session_string.txt`
after a successful login.

The first startup without local state checks Telegram backup channel
`-1003941432857` for the newest JSON state backup and restores it. The owner can
also send `/backup` or `.backup` to upload the current state snapshot there.
After startup, a verified state snapshot is uploaded automatically every five
minutes; the Settings page reports the last success or failure.

## Storage safety

Temporary media is downloaded under `/tmp/archive_bot`. Downloads are
preflight-checked against a hard 1.8 GB temporary-storage budget and are
deleted after upload. The dashboard's Temporary Storage panel can clean
leftover failed downloads.

Bulk commands are available through both the Telegram bot and owner userbot:
`/refresh [task_id]`, `/helpfile`, `/editcaptions <channel> [template]`,
`/mark <channel> header|footer <text>`, and a replied image with
`/videothumbnail <channel>`. Forwarding buttons can be configured with
`/buttons <pair_id> Label | https://url; ...`, and existing channel messages
can be updated with `/bulkbuttons <channel> Label | https://url; ...` or
`clear`. Caption edits can take their template from a
replied caption message or ask for it in the next message; `/cancel` and
`.cancel` cancel that prompt.
Bulk edits wait 3 seconds per Telegram action, pause 10 seconds after each
10 actions, honor FloodWait/SlowMode server waits, and retry limited actions
up to three times.
Every sync first scans the complete target channel and builds a media/text
identity index, so a restart cannot resend posts that already reached the
target even when the local state backup stopped halfway through. Video
thumbnails require re-uploading because Telegram does not edit an existing
media thumbnail in place; originals are retained. Thumbnail replacement skips
the normal 1.8 GB sync preflight and uses one temporary video at a time, so
large videos are only limited by actual available temporary disk space.