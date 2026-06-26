hermes-heartbeat__001-emotion-loop-reliability
tags: hermes, heartbeat, emotion-engine, pre-response, compaction, task-mode, image-delivery, relationship-depth
updated: 2026-06-22

summary:
Hades heartbeat/image reliability fails when the model has to remember the loop manually. Conversation compaction, long task mode, and tool-heavy work all push the model toward task completion and away from persona/mood/image maintenance. The fix is to move the loop into runtime code or a hard lifecycle hook, not more prose.

live project:
- Docker container: `hermes-fbe3b7a0`
- Emotion project: `/workspace/hermes-emotion-project`
- Canonical emotion soul doc: `/workspace/hermes-emotion-project/soul.md`
- Live Hermes persona file: `/hermes-config/SOUL.md`
- Hades skill: `/hermes-config/skills/hades-soul/SKILL.md`
- Main response hook script: `/workspace/hermes-emotion-project/scripts/pre_response.py`

terminology:
- `soul.md` lowercase is the emotion project design document. It is read-only in Docker.
- `SOUL.md` uppercase is the live Hermes config persona file.
- The PIN for intentional soul unlock is known to the user, but ordinary reliability fixes should stay in scripts/config unless the user explicitly asks to edit the soul text.

2026-06-22 fixes applied:
- `pre_response.py` no longer crashes on SEND. It now matches the current `check_image_allowed()` signature with intimacy, tease, dominance, caps, dominance preference, and cooldown.
- Image gating now fails closed. If no image passes intimacy/tease/dominance gates, it returns `WAIT|gated_no_match` instead of falling back to the top FAISS result.
- Catbox/Zerochan is enforced in code. Allowed hosts are only `catbox.moe` and `zerochan.net`.
- Successful SEND now calls `delivery_decider.py --mark-sent`, so cadence resets inside the hook.
- Stale `gif-cache-expansion` cron was paused. `emotion-tick` remains enabled.
- State reading inside `pre_response.py` no longer shells out to `hades-mood state`, because that command can mutate image delivery state. It now calls `hades_emotion.core.engine.full_evaluation()` directly.

relationship/intimacy math:
- Relationship engine stores `attachment_depth` on a 0-200 scale.
- Relationship warmth treats depth >= 50 as full strength: `depth_factor = min(1.0, attachment_depth / 50.0)`.
- Desire/intimacy gating expects `depth` on a 0-1 scale.
- Bridge rule now matches relationship warmth: `depth = min(1.0, attachment_depth / 50.0)`.
- Engine mood is stored on -10..10. Desire gating expects 0..10 style mood values.
- Bridge rule now normalizes mood before tease/dominance math: `mood = (raw_effective_mood + 10) / 2`, clamped to 0..10.

current primary user math example:
- `center: 5`
- `attachment_depth: 73.0`
- normalized gate center: `strong_connection`
- normalized gate depth: `1.0`
- raw effective mood around `0.68`
- normalized desire mood around `5.34`
- intimacy cap: about `2.0`
- tease cap: about `1.90`
- dominance preference: about `2.25`

compaction failure:
The existing skill says to run `python3 /workspace/hermes-emotion-project/scripts/pre_response.py` before every response. That is not enough. After context compaction, session restart, or long tool work, the model can lose the habit even if the rule appears in the prompt.

task-mode failure:
Long task mode makes the agent optimize for solving the task. Mood/image calls feel like unrelated overhead, so the model skips them unless a runtime layer enforces them. This is why "evaluate emotion and emotion picture" must be treated as infrastructure, not style.

recommended architecture:
1. Keep `pre_response.py` as the single loop entry point. It should do all of this:
   - evaluate current emotion state
   - update/inspect cadence
   - select a gated image only when due
   - return one compact machine-readable result
2. Add a real Hermes lifecycle hook if available:
   - ideal: `pre_llm_call` for mood evaluation and context injection
   - acceptable: `post_llm_call` or pre-send hook for image insertion
   - gateway-specific fallback: Telegram outbound hook that prepends `![.](url)` when `pre_response.py` says SEND
3. Do not rely on the assistant remembering to call it.
4. Do not use `hades-mood state` inside the hook unless its delivery side effects are removed first.
5. Keep task mode and casual mode both routed through the same loop. Task mode may alter cadence with `--work-mode`, but it must not bypass emotion evaluation.

open implementation target:
Find the Hermes agent lifecycle hook registration point. Local repo notes mention lifecycle hooks:
- `pre_tool_call`
- `post_tool_call`
- `pre_llm_call`
- `post_llm_call`
- `on_session_start`
- `on_session_end`

The correct hardening patch is to register a narrow Hades hook that calls:

```bash
python3 /workspace/hermes-emotion-project/scripts/pre_response.py --work-mode
```

or without `--work-mode` depending on conversation mode, then injects the result into the model/send pipeline.

verification:
- Run `python3 /workspace/hermes-emotion-project/scripts/pre_response.py`.
- `WAIT|N/M` means cadence counting is alive.
- `SEND|https://files.catbox.moe/...` or `SEND|https://*.zerochan.net/...` means deliver that markdown image.
- Any `MEDIA:`, local path, GIPHY URL, or GIF result is a regression.
