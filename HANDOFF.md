# HANDOFF — roguERGlike (umbrella)

> The current state of the project across all four repos. Updated at the end of every session. Read this first.

**Last updated:** 2026-05-21
**Last session log:** `docs/sessions/2026-05-21-cadence-bailout-paused-mid-validation.md` (fourth same-day log; follows `-ftms-erg-end-to-end.md`)
**Current branch:** umbrella `docs/post-cadence-bailout-paused` (PR pending). Engine PR #4 (MVP HIIT loop) still Ready for Review.
**Current focus:** **Cadence-bailout safety feature shipped and merged, but live validation paused mid-test.** Two real bugs surfaced during the attempted validation ride that need to land before the next attempt. Whoop's broadcast-HR mode was also stuck off due to a Whoop-app-side quirk that's environmental, not project-side.

## Where we are

Three more PRs landed on develop this session and the whole trainer-control-plus-bailout loop is *on paper* complete:

- **sidecar #14** — cadence-based ERG bailout. Drops target to floor when cadence stays below ~30 rpm for an intensity-scaled wait window (90s at recovery, 15s at peak); ramps back up over an inverse window when cadence resumes. Also bundles a fix that makes the previously-silent "no `--allow-trainer-control` flag" failure visible as a typed `target_power_set accepted=false reason="trainer-control disabled"` envelope.
- **engine #9** — `EffortBridge` gains `cadence_bailout_engaged` / `_disengaged` signals + `is_cadence_paused: bool` state so game UI can show a "PAUSED — start pedalling" overlay.

The intent was to live-validate the bailout end-to-end against the KICKR + Whoop. **That test didn't get a green run.** Three things stacked: (1) the launch command lost `--allow-trainer-control` on PowerShell paste; (2) the retry hit the Whoop broadcast quirk (broadcast mode resets per-activity and the rider hadn't restarted an activity in the Whoop app); (3) during diagnosis we discovered a real timing bug in the new bailout — its idle window starts at sidecar startup rather than at first meaningful ERG target, so a >60-second setup delay between sidecar launch and pressing Start Workout would cause the bailout to fire immediately upon the first warmup target.

The session paused with no green live-ride validation but with a clear, small follow-up plan to fix the bugs and retry.

## What's next (immediate)

1. **One small sidecar PR** (`fix/bailout-startup-and-scan-ux`, ~45 min):
   - **Bug A**: `CadenceBailout.handle_set_target_power` should reset `_last_active_ts` when a non-floor target passes through. The bailout's idle window then measures "time since first meaningful ERG target / last cadence event" — correct semantic.
   - **Bug B**: replace `_run_live`'s fail-once scan with a forgiving retry loop. Default 30s total with per-5s "still waiting for X" prints; succeed on full match, fail only if total elapses. Optional `--scan-timeout-s` flag.
   - **Bug C**: emit `INFO ... claimed control of <device> — set_target_power now active` from `FtmsControl.request_control_and_start` so the operator has a clear positive signal that control claim succeeded (independent of the trainer LED state, which is misleading).
2. **Retry engine-driven ERG live validation** against KICKR. Whoop is optional — bike-only is sufficient to validate the bailout. Both the cadence-bailout (fixed) and the existing trainer-control writes should work end-to-end via the MVP scene's phase transitions.
3. **Merge engine PR #4** (MVP HIIT playable loop) once the validation pass is green.

Alternative / parallel work:
- **Engine card-system foundation** (`Effect: Resource` + `CombatContext: RefCounted`) on `feat/card-system-foundation`. Unblocks card variety beyond the MVP's two hard-coded cards.
- **Sidecar `--record <path>` first-class flag** to replace the external `scripts/record_session.py` (currently still pending in docs PR #13).

## Open threads

- **Three small sidecar bugs queued** — see #1 above.
- **Engine PR #4** Ready for Review, awaiting the validation pass.
- **Docs PR #13 sidecar (HANDOFF refresh + `scripts/record_session.py`)** still open and unmerged from a previous wrap.
- **Whoop broadcast UX is fragile** — broadcast mode resets per-activity / per-app-state. Worth documenting in `docs/ble-profiles.md`'s HRS section so future readers don't re-learn the lesson.
- **PowerShell long-line paste is fragile** — multi-flag commands keep losing flags. Switch to a small `start_test.ps1` script for the next live test.
- **KICKR LED state is misleading** — solid blue = any BLE GATT connection, NOT "FTMS control claimed". Don't rely on it for verification; use the sidecar log + bridge's `control_acquired` signal once bug C lands.
- **Mechanic exploration ideas** in `repos/game/IDEAS.md` (Zone 2 + cognitive load inversion, modifying-vs-charging axis) — not yet promoted to experiments.
- **Sub-title for the bike-themed first game** — still TBD.
- **Server architecture (Nakama)** deferred to Phase 5.
- **Mobile / tablet build path** acknowledged as long-term but not designed.

## Repo state

| Repo | State | Branch | Notes |
|---|---|---|---|
| umbrella | docs refresh in flight | `docs/post-cadence-bailout-paused` | session log + this HANDOFF; PR pending |
| sidecar  | bailout + silent-rejection fix merged; three small bugs queued | develop has the feature; `fix/bailout-startup-and-scan-ux` not yet started | 105/105 unit tests passing; live ERG ride still unvalidated |
| engine   | bridge bailout signals merged; MVP loop with ERG wiring on `feat/mvp-playable-loop` | develop has the bridge; PR #4 ready for review (awaits live validation) | mvp lives at `repos/engine-mvp/` worktree |
| game     | bootstrapped, pushed | develop | no source yet, by design |
| server   | not yet `git init` | n/a | placeholder only, defer to Phase 5 |

## Entry point for next session

> "Open `fix/bailout-startup-and-scan-ux` on the sidecar. Three small fixes: (1) reset `_last_active_ts` in `CadenceBailout.handle_set_target_power` so the bailout's idle window starts on first meaningful ERG target rather than sidecar startup; (2) forgiving scan loop in `_run_live` (default 30s total, per-5s 'still waiting for X' prints, succeed when complete); (3) explicit `INFO ... claimed control of <device>` log line in `FtmsControl.request_control_and_start`. After merging, retry engine-driven ERG live validation against KICKR — Whoop optional. Then merge engine PR #4 (MVP HIIT loop)."
