---
name: Telegram package collision
description: Python dependency maintenance can install the unrelated telegram package alongside python-telegram-bot.
---

When maintaining this project, keep `python-telegram-bot` as the Telegram Bot API dependency and do not add the unrelated `telegram` distribution. After package-tool operations, verify both the requirements file and `from telegram import Update`.

**Why:** Both distributions provide a top-level `telegram` package; the unrelated one can replace the correct package initializer and make imports such as `Update` fail at startup.

**How to apply:** After package changes, verify `from telegram import Update` before restarting the workflow and remove any standalone `telegram` requirement. If a package helper installs the collision as a side effect, reinstall the pinned bot package without that distribution, then recheck the import.