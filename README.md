# Telegram Channel Archive Bot

Owner-only Telegram archive and channel-copy bot. It can copy channel history,
keep source-to-target routes, resume interrupted work, detect duplicates, and
manage bulk caption, marking, backup, and thumbnail operations.

The project runs three pieces in one Python process:

- **Telethon userbot** — reads source channels and transfers media.
- **Telegram Bot API controller** — accepts owner commands and sends alerts.
- **Flask dashboard** — manages pairs, tasks, limits, logs, and live progress.

## Contents

- [What it does](#what-it-does)
- [Run locally or on Replit](#run-locally-or-on-replit)
- [First-time setup](#first-time-setup)
- [Commands](#commands)
- [Bulk operation safety](#bulk-operation-safety)
- [Caption templates](#caption-templates)
- [Dashboard](#dashboard)
- [Backup and restore](#backup-and-restore)
- [Files and storage](#files-and-storage)
- [HTTP API](#http-api)
- [Troubleshooting](#troubleshooting)
- [Security](#security)

## What it does

### Channel copying

- Full history sync, sync from a message ID, or sync the last N messages.
- Multiple independent source → target channel pairs.
- Channel references by `@username`, `-100...` ID, numeric ID, Telegram link,
  or a forwarded message for private channels.
- Text, photos, videos, documents, audio, GIFs, voice messages, and other
  supported Telegram media.
- Album/grouped-media upload where Telegram allows it.
- Original filenames, document attributes, video streaming, and audio metadata
  are preserved when possible.
- Protected-content policy can either download/re-upload the post or skip it.
- Failed transfers are recorded with error details and an available message link.

### Duplicate protection

The bot uses more than one identity when deciding whether a message was already
copied:

- Source message ID mapping.
- Stable text identity.
- Telegram document/photo metadata fingerprint.
- Pair-specific dedupe records.

The dashboard also provides a **Copy again** action that clears dedupe
identities for a selected pair.

### Pair-level rules

Each source → target pair can have its own:

- Allowed message types.
- Include and exclude keyword filters.
- Caption prefix, suffix, and caption-removal rules.
- Caption template and parse mode.
- Video thumbnail.
- Rate profile or custom delay.
- Per-run and daily message/media limits.
- Maximum posts per hour.
- Schedule window and quiet hours.
- Protected-content behavior.
- Auto-forward setting.

### Live monitoring

The dashboard and status commands expose:

- Source status.
- Scanned, pending, transferred, duplicate, skipped, and failed counts.
- Current transfer and task queue state.
- Channel access and target write-permission health.
- FloodWait and storage-limit warnings.
- Searchable live logs.
- Temporary storage usage and cleanup.

## Run locally or on Replit

### Requirements

- Python 3.12 recommended.
- A Telegram account with access to the source channel.
- Posting permission in the target channel.
- Dependencies from `requirements.txt`.

Install dependencies:

```bash
pip install -r requirements.txt
```

Start the application:

```bash
python main.py
```

On Replit, the configured **Start application** workflow runs the same command.
The Flask dashboard listens on `PORT` and normally uses port `8080` locally.

## Required Replit Secrets

Set these in Replit Secrets. Never paste them into source code, README files, or
chat messages.

| Secret | Purpose |
|---|---|
| `API_ID` | Telegram API ID from `my.telegram.org` |
| `API_HASH` | Telegram API hash |
| `PHONE` | Telethon user account phone number |
| `OWNER_ID` | Numeric Telegram user ID allowed to control the bot |
| `BOT_TOKEN` | BotFather token |
| `DASHBOARD_PASSWORD` | Password for the Flask dashboard |

Optional values:

| Variable | Purpose |
|---|---|
| `SESSION_STRING` | Reuse an existing Telethon session |
| `FLASK_SECRET_KEY` | Keep dashboard sessions valid across restarts |
| `PORT` | Dashboard port; Replit supplies this automatically |

After a successful login, the Telethon session is persisted locally in
`session_string.txt`. Keep that file private.

## First-time setup

1. Add the required Replit Secrets.
2. Start `python main.py` and complete the Telethon login if requested.
3. Open the dashboard and sign in with `DASHBOARD_PASSWORD`.
4. Set a source channel and target channel.
5. Create a source → target pair.
6. Configure message types, filters, caption rules, thumbnail, and limits.
7. Run **Dry run** first to review the estimated messages and duplicates.
8. Start the sync and monitor live progress and logs.
9. Enable auto-forward only after the initial history sync is behaving correctly.

## Commands

Commands are owner-only. Use slash commands through the Telegram Bot API bot,
or the equivalent dot commands from the logged-in Telethon userbot.

### Userbot commands

| Command | Description |
|---|---|
| `.help` | Open the interactive category help menu. |
| `.helpfile` | Receive the complete command guide as `archive_bot_commands.txt`. |
| `.setsource <channel>` | Set the source channel. |
| `.settarget <channel>` | Set the target channel. |
| `.info` | Show current source, target, pairs, and configuration. |
| `.sync` | Queue a full sync. |
| `.syncfrom <id>` | Queue a sync starting from a source message ID. |
| `.synclast <n>` | Queue the last N source messages. |
| `.refresh [task_id]` | Rescan the complete source history; existing copies are counted as duplicates. |
| `.backup` | Upload the current state snapshot to the Telegram backup channel. |
| `.status` | Show live or last-sync status. |
| `.pause` | Pause the active sync. |
| `.resume` | Resume a paused sync. |
| `.stop` | Stop active and queued sync work. |
| `.reset` | Reset runtime configuration to defaults. |
| `.cancel` | Cancel a pending caption prompt. |

For private channels, `.setsource` and `.settarget` can also use a forwarded
message when an ID or username is not available.

### Telegram bot commands

| Command | Description |
|---|---|
| `/start` | Show a short bot introduction. |
| `/help` | Open the interactive category help menu. |
| `/helpfile` | Receive the complete command guide as a TXT file. |
| `/setsource <channel>` | Set the source channel. |
| `/settarget <channel>` | Set the target channel. |
| `/info` | Show current configuration. |
| `/status` | Show live or last-sync status. |
| `/sync` | Queue a full sync. |
| `/force_sync` | Queue a full sync while bypassing normal daily limits. Use carefully. |
| `/syncfrom <id>` | Queue a sync starting from a message ID. |
| `/synclast <n>` | Queue the last N messages. |
| `/refresh [task_id]` | Rescan the complete source history. |
| `/tasks` | Show the task queue. |
| `/autoforward on\|off` | Enable or disable global auto-forwarding. |
| `/caption <pair_id> on\|off [template]` | Configure a pair caption rule. |
| `/setthumbnail <pair_id>` | Save a replied photo as a pair video thumbnail. |
| `/setthumbnail <pair_id> off` | Disable a pair thumbnail. |
| `/backup` | Upload the current state snapshot. |
| `/editcaptions <channel> [template]` | Bulk-edit captions in an existing channel. |
| `/mark <channel> header\|footer <text>` | Add a header or footer to channel messages. |
| `/videothumbnail <channel>` | Re-upload channel videos with a replied image as thumbnail. |
| `/cancel` | Cancel a pending caption prompt. |
| `/pause`, `/resume`, `/stop` | Control active sync work. |
| `/reset` | Reset runtime configuration to defaults. |

### Bulk caption editing

The caption template can be supplied in any of these ways:

```text
/editcaptions @channel 🎬 {filename}
```

Reply to a message containing the desired caption and send:

```text
/editcaptions @channel
```

Or send only the channel. The bot will ask for the caption in your next
message:

```text
/editcaptions @channel
```

The same three flows work with `.editcaptions`. Use `/cancel` or `.cancel` if
you change your mind. Bulk caption editing changes captions in place; it skips
messages without editable media and does not delete messages.

### Bulk marking

```text
/mark @channel header 📚 Archive
/mark @channel footer — Saved by Archive Bot
```

The same syntax works with `.mark`. Repeating the command adds the mark again,
so review the first report before running it a second time.

### Bulk video thumbnails

Reply to a photo or image document:

```text
/videothumbnail @channel
```

The same syntax works with `.videothumbnail`. Telegram does not provide a safe
in-place thumbnail edit for an existing video post, so the bot uploads
replacement videos and retains the original posts. Repeating this operation
creates another set of replacements.

## Bulk operation safety

Bulk operations are deliberately slower than a normal loop:

- **3 seconds** after every Telegram API action.
- An additional **10-second pause** after every 10 actions.
- `FloodWait` and `SlowMode` use Telegram's suggested wait plus a 5-second
  safety margin.
- A limited action is retried up to three times.
- The final report shows scanned, changed/re-uploaded, skipped, and failed
  counts, plus the first errors.

FloodWait can still be returned by Telegram when account or channel limits
change, but the bot will wait and retry instead of hammering the API.

Normal syncs additionally use each pair's rate profile, custom delay, daily
quotas, hourly cap, schedule window, and quiet hours.

## Caption templates

Templates are supported by pair caption rules and bulk caption editing.

| Placeholder | Value |
|---|---|
| `{caption}` | Original message text or caption |
| `{filename}` | Document filename, when available |
| `{filesize}` | Human-readable file size |
| `{filesize_mb}` | File size in MB |
| `{message_id}` | Telegram message ID |
| `{source}` | Source channel title |
| `{date}` | Processing date (`YYYY-MM-DD`) |
| `{time}` | Processing time (`HH:MM`) |
| `{mime}` | Media MIME type |
| `{type}` | Detected message type |
| `{duration}` | Media duration, when available |
| `{resolution}` | Video/photo resolution, when available |

Example:

```text
🎬 {filename}
Size: {filesize}
Source: {source}

{caption}
```

Supported parse modes are Markdown, HTML, and plain text. Plain text is safest
when source captions contain characters such as `*`, `_`, `[`, `<`, or `&`.

## Dashboard

The dashboard provides:

- Source and target channel configuration.
- Pair create, edit, and delete.
- Message-type and keyword filters.
- Rate, quota, schedule, and quiet-hour controls.
- Caption template, parse mode, and selected caption types.
- Per-pair thumbnail upload and removal.
- Auto-forward controls.
- Full sync, sync-from-ID, last-N sync, and dry-run preview.
- Pause, resume, stop, queue priority, reorder, and bulk task actions.
- Task reports, searchable logs, live status, and channel health.
- Temporary storage usage and cleanup.
- Pair dedupe reset for a deliberate copy-again operation.

Important dashboard routes:

| Page | Purpose |
|---|---|
| `/dashboard` | Live overview and current task |
| `/pairs` | Source → target pair management |
| `/tasks` | Queue, task reports, and task controls |
| `/settings` | Global limits, notifications, and behavior |
| `/help` | Browser-based command and formatting guide |

## Backup and restore

The bot uses JSON state plus a local SQLite snapshot for runtime persistence.
The state contains pairs, tasks, progress, configuration, and dedupe identities;
it does not contain Telegram secrets.

- Backup channel: `-1003941432857`.
- On first startup without local state, the newest matching JSON backup is
  restored from that channel.
- Completed or partially completed syncs upload a backup with a five-minute
  throttle.
- `/backup` and `.backup` force a current backup upload.
- `sync_state.json.bak` is the previous valid local JSON snapshot and is used
  for recovery if the main JSON file is damaged.

## Files and storage

| Path | Purpose |
|---|---|
| `main.py` | Telethon, Bot API, Flask, sync, bulk-operation, and state logic |
| `templates/` | Dashboard HTML templates |
| `static/` | Dashboard JavaScript and styles |
| `requirements.txt` | Python dependencies |
| `sync_state.json` | Current JSON state |
| `sync_state.json.bak` | Previous valid JSON state |
| `archive_state.sqlite3` | Local SQLite snapshot |
| `sync.log` | Detailed application log |
| `session_string.txt` | Persisted Telethon session |
| `thumbnails/` | Saved per-pair and temporary thumbnail files |
| `/tmp/archive_bot/` | Temporary downloaded media |

Temporary media is checked against a hard **1.8 GB** working budget before
download and removed after upload. Use the dashboard cleanup action if an
interrupted transfer leaves temporary files behind.

## HTTP API

The dashboard uses same-origin Flask endpoints:

| Endpoint | Method | Purpose |
|---|---|---|
| `/health` | GET | Health check |
| `/api/status` | GET | Current dashboard status |
| `/api/bootstrap` | GET | Initial dashboard data |
| `/api/events` | GET | Live Server-Sent Events stream |
| `/api/pairs` | GET/POST | List or create pairs |
| `/api/pairs/<pair_id>` | PATCH/DELETE | Update or delete a pair |
| `/api/pairs/<pair_id>/thumbnail` | POST/DELETE | Add or remove a thumbnail |
| `/api/pairs/<pair_id>/dedupe` | POST | Clear pair dedupe identities |
| `/api/tasks` | GET/POST | List or create tasks |
| `/api/tasks/<task_id>` | PATCH/DELETE | Update or delete a task |
| `/api/tasks/<task_id>/report` | GET | View a task report; use `?format=csv` for CSV |
| `/api/tasks/bulk` | POST | Apply a bulk queue action |
| `/api/tasks/reorder` | POST | Reorder queued tasks |
| `/api/tasks/dry-run` | POST | Preview filters and duplicates |
| `/api/sync` | POST | Start a full sync |
| `/api/syncfrom` | POST | Start from a message ID |
| `/api/synclast` | POST | Sync the last N messages |
| `/api/autoforward` | POST | Toggle global auto-forwarding |
| `/api/pause`, `/api/resume`, `/api/stop` | POST | Control sync work |
| `/api/reset` | POST | Reset runtime state |
| `/api/logs` | GET | Read live logs |
| `/api/logs/search` | GET | Search live logs with `?q=...` |

## Troubleshooting

### Dashboard does not open

Confirm that the **Start application** workflow is running and that the
configured `PORT` is available. On Replit, use the preview rather than a
hardcoded localhost URL.

### Source is inaccessible

The Telethon user account must be able to read the source channel. For a
private channel, use its `-100...` ID or reply to a forwarded private-channel
message with `.setsource` or `/setsource`.

### Target write error

Give the Telethon account permission to post in the target channel. Check the
Channel Health panel and confirm that the target is not read-only.

### FloodWait

Do not repeatedly restart the bot or resend the same command. The bot respects
Telegram's suggested wait for normal and bulk operations. For normal syncs,
choose the `very_safe` rate profile or increase the pair delay, and review the
daily and hourly limits.

### Storage limit

Run dashboard temporary-storage cleanup, reduce parallel work, and review daily
media limits. Large videos can require significant temporary disk space.

### Thumbnail operation does not change old posts

This is expected: Telegram does not safely edit an existing video's thumbnail
in place. The bulk command creates replacement posts and keeps originals.
Confirm that the replied file is a valid image and that the target account can
post videos.

### Caption template error

Use only the documented placeholders. For source captions containing Markdown
or HTML special characters, choose plain text or escape the template
appropriately.

## Security

- Store Telegram credentials, bot tokens, and dashboard passwords only in
  Replit Secrets.
- Keep `session_string.txt`, state files, logs, and thumbnails private.
- The userbot and Bot API paths both enforce `OWNER_ID`.
- Review a dry run before copying a large history.
- Treat `/force_sync`, bulk marking, and thumbnail replacement as deliberate
  high-impact operations.
- The bundled Flask server is suitable for the Replit workflow; use a proper
  production WSGI setup if deploying it outside this environment.

## License

Project owner / PR Bot Service.