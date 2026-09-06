---
name: Telegram media thumbnail limits
description: Telegram does not provide an in-place thumbnail edit for an existing channel video post.
---

Bulk thumbnail changes must re-upload replacement video messages with the new thumbnail rather than silently deleting or mutating the original post.

**Why:** Telegram's media edit behavior does not expose a safe in-place thumbnail replacement for existing video messages, and deleting originals would be destructive.

**How to apply:** Keep originals by default, report that replacements were created, and only add an explicit deletion option after the user confirms the destructive behavior.