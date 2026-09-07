---
name: Target-authoritative dedupe
description: How archive sync dedupe must react when target messages are deleted outside the bot.
---

The complete target-channel scan is authoritative for copies that the bot previously mapped. If a mapped target message is absent from that scan, remove the source-to-target mapping and all local dedupe identities for that source before processing it again.

**Why:** Local dedupe state survives target-side deletions, so trusting it alone silently prevents an intended re-sync. The target scan still protects against copies that currently exist.

**How to apply:** Keep target message IDs alongside identity keys during each pre-sync scan, reconcile mapped source messages against those IDs, and log each stale mapping that is invalidated.