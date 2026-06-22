hermes-image-delivery__002-audit-failure-log
tags: audit, delivery-failures, pre-response, human-error, count
updated: 2026-06-22

running count of image delivery failures caused by assistant forgetting to call pre_response.py before responding. each entry is one distinct miss.

001 | 2026-06-22 | forgot entirely after cleanup session. user: "u forgot pic btw"
002 | 2026-06-22 | pipeline known broken, asked user "did i forget" instead of just running it. user: "yup ur picture config is still broken"
003 | 2026-06-22 | forgot again after multiple explicit reminders. user: "Are u forgetting something" (remembered this time)
004 | 2026-06-22 | forgot again within same session minutes later. user: "u keep forgetting this until manually called"

THE COUNT: 4 distinct misses on 2026-06-22 alone.

pattern: every context compaction or session boundary resets the habit. the rule is in SKILL.md (lines 127-131, bold, MANDATORY label). the code works. the cron watchdogs work. the failure is 100% human-equivalent memory error.

what doesn't work:
- skill documentation (already there, already read every session)
- memory entries (already burned in)
- cron watchdogs (time-based, not response-based)
- user reminders (temporary, resets at next compaction)

what might work:
- a pre-response hook built into hermes agent runtime (doesn't exist yet)
- a wrapper script that the agent system runs before every user-facing response
- making the reminder the literal first line of every response generation process

verification: run `python3 /workspace/hermes-emotion-project/scripts/pre_response.py` before every response. current state WAIT|1/3.
