# HANDOFF — roguERGlike (umbrella)

> The current state of the project across all four repos. Updated at the end of every session. Read this first.

**Last updated:** 2026-05-21
**Last session log:** `docs/sessions/2026-05-21-ftms-erg-end-to-end.md` (third of three same-day logs; follows `-hardware-validation-and-merges.md` and `-hrs-validation-and-mvp-promotion.md`)
**Current branch:** umbrella `docs/post-trainer-control-end-to-end` (PR pending). Engine MVP loop with ERG wiring is in flight on `feat/mvp-playable-loop` (PR #4 ready for review).
**Current focus:** **Trainer control loop is shipped end-to-end and ready for the engine-driven live test.** Sidecar capabilities + writes + replay fix all on develop; engine bridge on develop; MVP scene wires ERG to phase transitions. Next concrete action: ride the full warmup → recovery → interval loop against the KICKR + Whoop and confirm resistance tracks the engine's targets.

## Where we are

Full trainer-control feature shipped across both code repos this session: sidecar reads FTMS Feature characteristics + capabilities event (PR #9), claims control + accepts `set_target_power` over a bidirectional WS (PR #11, re-landed after a GitHub stacked-PR retarget snafu), replays control-lifecycle events to late subscribers (PR #12), engine's `EffortBridge` gained the matching write-side API + capability gating (engine PR #7), and the MVP HIIT scene now calls `EffortBridge.set_target_power(...)` on each phase transition with all calls gated on `supports_target_power`. KICKR was live-validated against the sidecar-side writes via a Python WS probe (100 → 250 W ramp, all `accepted=true`, rider reported the resistance steps felt clean); engine-as-driver validation is the **immediate next action** — three-window setup, ride, observe the trainer responding to the loop's intended phases in real time.

Also new this session: `scripts/record_session.py` in the sidecar — a small WS subscriber that dumps every envelope to gitignored JSONL for diagnostic capture. Pairs with the screenshot pattern from previous live tests.

## What's next (immediate)

1. **Engine-driven ERG live test against KICKR + Whoop.** Three PowerShell windows (sidecar with `--allow-trainer-control --disconnect-bailout-s 900`, JSONL recorder, Godot pointed at the `engine-mvp` worktree). Configure rider settings (suggest Warmup=2min for a fast first pass). Press Start Workout. Trainer should ramp during warmup, drop to ~55% FTP for recovery, slam to ~120% FTP for intervals. HR streams from the Whoop concurrently. Save the screenshot + JSONL recording as the validation artifact.
2. **Merge engine PR #4 (MVP HIIT playable loop)** once the live test validates. PR body needs a body refresh first — current text predates the session's substantial UX additions (settings panel, per-metric charts, big readouts, target meters, warmup phase, ERG wiring).
3. **Card-system foundation** (`Effect: Resource` with `apply(context)`, `CombatContext: RefCounted` with hand/draw/discard piles) on `feat/card-system-foundation` in the engine. Currently the MVP loop has two hard-coded cards (Power Strike, Cadence Guard); real variety needs the foundation.

Alternative next steps once PR #4 lands:
- Sidecar `--record <path>` as a first-class flag (replacing the script).
- FTMS SIM mode (slope/wind/CRR) — gated on `indoor_bike_simulation: true`. Useful when a non-HIIT mechanic wants the trainer to feel like a hill.
- Distance deriver (sidecar) so FTMS emits `DistanceData`.
- CSCS profile (sidecar) for older trainers / power meters with cadence outside FTMS.

## Open threads

- **Engine PR #4 ready for review**; awaiting live validation pass before merge.
- **`card.gd` parse errors** — references undefined `Effect` / `CombatContext`. Lands with the foundation PR.
- **Disconnect-bailout default** (`--disconnect-bailout-s 10`) is friendly to "trainer connected to one app at a time" but unfriendly to "set up sidecar, then engine, then ride" — tests have been bumping it to 600–900s. Might revisit default to ~30s. Not urgent.
- **MVP scene lives under engine `tests/`** — once a real test runner lands and the loop is more than a prototype, it should move to `repos/game/` per the engine/game boundary in the per-repo CLAUDE.md docs.
- **Engine CI's `godot-headless-tests` runs with `|| true`** — boots Godot but doesn't enforce assertions. Plumbing for a real runner (GUT or hand-rolled) still pending.
- **Stacked-PR retarget gotcha** (lesson from this session): GitHub does NOT auto-retarget the child PR when the parent merges if the parent branch isn't deleted. Always re-base + re-PR if the stacked merge goes sideways.
- **Mechanic exploration ideas** in `repos/game/IDEAS.md` (Zone 2 + cognitive load inversion, modifying-vs-charging axis) — not yet promoted to experiments.
- **Sub-title for the bike-themed first game** — still TBD.
- **Server architecture (Nakama)** deferred to Phase 5.
- **Mobile / tablet build path** acknowledged as long-term but not designed.

## Repo state

| Repo | State | Branch | Notes |
|---|---|---|---|
| umbrella | docs refresh in flight | `docs/post-trainer-control-end-to-end` | session log + this HANDOFF; PR pending |
| sidecar  | Full trainer-control loop merged + KICKR live-validated (write side) | develop (clean) | PRs #9, #11, #12 all merged; `scripts/record_session.py` for diagnostics |
| engine   | Bridge write-API merged; MVP loop wires ERG to phases | develop has bridge; `feat/mvp-playable-loop` has the MVP loop + ERG wiring (PR #4 ready for review) | mvp lives in a separate worktree at `repos/engine-mvp/` |
| game     | bootstrapped, pushed | develop | no source yet, by design |
| server   | not yet `git init` | n/a | placeholder only, defer to Phase 5 |

## Entry point for next session

> "Resume the engine-driven ERG live test. Three windows: sidecar with `--mode live --device-bike KICKR --device-hr mudrat --allow-trainer-control --disconnect-bailout-s 900`; `python scripts/record_session.py` once both devices show connected; Godot on the engine-mvp worktree, ▶ Play, ride through warmup → recovery → interval. Confirm KICKR resistance tracks engine-driven targets; HR flows from the Whoop concurrently. Screenshot + JSONL go in `docs/screenshots/` and `docs/recordings/`; engine PR #4 (MVP HIIT loop) is ready to merge if green."
