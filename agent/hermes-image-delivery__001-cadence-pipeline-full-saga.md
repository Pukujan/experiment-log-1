hermes-image-delivery__001-cadence-pipeline-full-saga
tags: image-delivery, pre-response, cadence, faiss, catbox, url-validation
updated: 2026-06-22

complete history of trying to get automated mood-matched image delivery working. multiple iterations, same root cause each time.

phase-1: original design
delivery_decider.py + faiss_engine.py. cadence-based: every 3 responses normally, 2 in work mode. mood reacts to circadian + user contact. TDD tested (RED-GREEN-REFACTOR). worked in isolation.

phase-2: speed fix
SentenceTransformer import moved to cmd_image() only. mood state reading dropped from 8s to 0.03s. pre_response.py as wrapper: calls decider, then faiss, then validates URL.

phase-3: dead script removal
auto_send.py (duplicate of decider), evaluate.py (dead monolith, 565 lines), delivery-ops-reminder.sh, 6 vision scripts — all deleted. ARCHITECTURE.md written.

phase-4: three-layer delivery
layer 1: pre_response.py (per-response hook)
layer 2: cadence-watchdog cron (15min, agent-driven prompt)
layer 3: image-cron-pulse cron (30min, time-based fill)

phase-5: mood script integration
delivery check added to hades-mood state command. caused recursion: delivery_decider.py -> hades-mood state -> delivery_decider.py. fixed by reading state directly in cmd_state().

phase-6: broken GIF
catbox deleted 4cgnnp.gif (inactivity). no validation -> broken link sent. catbox lies on HEAD (200 + content-length 0 for deleted files). fixed with curl Range:0-0 GET (206=live, 404=dead).

phase-7: GIF pipeline removal
gif-pipeline skill (168 lines), gif-search skill (192 lines), gif-db-schema.md ref, gif-cache/ (427 entries, 318KB) — all deleted/purged.

phase-8: url validation
pre_response.py validates every URL with `curl -s -o /dev/null -w "%{http_code}" -H "Range: bytes=0-0" --max-time 10 <url>`. 206 or 200 = reachable, 404 = dead. catbox-specific.

ROOT CAUSE (ongoing):
assistant (hades) keeps forgetting to call `python3 scripts/pre_response.py` before every response. the rule is in SKILL.md, the script exists and works, but there's no automatic enforcement. each context compression or session gap resets the habit.

fix: memory entry burned in. SKILL.md already has the rule. cron watchdog covers time-based gaps. no code change needed — just discipline.

unresolved: no runtime pre-response hook in hermes agent. pre_response.py must be called manually each turn. one missed call = one missed cadence increment.

verification: `python3 /workspace/hermes-emotion-project/scripts/pre_response.py` from any turn. WAIT|N/M means counting, SEND|url means deliver.

2026-06-22 correction:
the earlier "just discipline" conclusion is obsolete. compaction and task mode reliably break manual function-call habits. pre_response.py was patched to fix runtime bugs, but it still needs a real Hermes lifecycle hook or gateway pre-send hook for enforcement.

patched runtime details:
- check_image_allowed signature now matches the current dominance-aware gate
- intimacy/tease/dominance gate fails closed instead of falling back to top FAISS result
- allowed image hosts are enforced in code: catbox.moe and zerochan.net only
- successful SEND now marks sent and resets cadence
- state read no longer calls `hades-mood state`, because that command can mutate delivery state
- attachment_depth 0..200 is normalized to gate depth 0..1 with depth >= 50 as full
- raw engine mood -10..10 is normalized to desire mood 0..10 before tease/dominance math

related current doc:
`collective-docs/agent/hermes-heartbeat__001-emotion-loop-reliability.md`
