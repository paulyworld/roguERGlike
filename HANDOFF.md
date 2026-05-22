# HANDOFF — roguERGlike (umbrella)

> The current state of the project across all repos. Updated at the end of every session. Read this first.

**Last updated:** 2026-05-22
**Last session log:** `docs/sessions/2026-05-22-first-live-ride-and-pause-architecture.md`
**Current branch:** umbrella `docs/2026-05-22-first-live-ride` (PR pending)
**Current focus:** **First fully end-to-end live ride completed.** Trainer-control loop (sidecar ↔ KICKR + Whoop) validated; cadence bailout confirmed working in practice; architectural separation between safety bailout and game-flow pause documented. Engine MVP HIIT loop validated but not yet promoted. A second MVP (concert-mvp) shipped in parallel.

## Where we are

The single biggest milestone: **a real ERG-controlled bike ride against the engine MVP worked end-to-end.** The sidecar drove the KICKR, the engine MVP set targets through phase transitions, the cadence bailout fired and recovered correctly during natural between-round pauses, and the telemetry recorder captured everything.

Three sidecar PRs landed today:

- **#16/#17** (squash-bundled): three bug fixes from the first attempted live ride (bailout `_last_active_ts` reset on first ERG target; forgiving scan loop with `--scan-timeout-s`; explicit `claimed control of <device>` log). Plus the `run-live-test.ps1` launcher.
- **#18**: `docs/architecture/safety-vs-pause.md` — documents that the cadence bailout is safety, not game flow. Two valid pause patterns (client-side soft target, sidecar-side suspend) chosen by client context.
- **#19**: `record_session.py` + PowerShell wrapper. JSONL recorder for ride telemetry.

The first attempted live ride yesterday (2026-05-21) didn't get a green run — three real bugs stacked with PowerShell paste fragility. The second attempt today (2026-05-22) succeeded end-to-end on the first try after the fixes landed. Three cadence bailouts fired during the ride; the timing matched the intensity-aware formula exactly (74-78s at ~177W with FTP 250 → predicted 74.4s). But all three fired during between-round UI pauses, not real walk-aways — which surfaced the architectural separation between safety bailout and game-flow pause.

A second MVP repo also shipped today: **`repos/concert-mvp`** — a browser-based YouTube-driven ERG controller. Sibling to engine-mvp, not a replacement. It implements Pattern A (client-side soft pause) and validated the principle in practice.

## What's next (immediate)

1. **Decide engine PR #4's fate.** `feat/mvp-playable-loop` was live-validated. Promote it to develop, or keep it as a side experiment while concert-mvp explores the alternate path. Belongs to the user.
2. **If engine #4 promoted: ship Pattern B pause command.** Sidecar gets `PauseCommand`/`ResumeCommand` + handler that toggles `CadenceBailout.set_paused(...)` + emits `paused`/`resumed` envelopes. Engine emits pause/resume at phase boundaries.
3. **Sidecar side-tracks** (any one, all small):
   - Distance deriver (`feat/distance-deriver`) — emit `DistanceData` from FTMS `meters_total`.
   - CSCS profile (`feat/ble-cscs`) — older trainers / power meters with cadence outside FTMS.
   - First-class `--record <path>` CLI flag — replace the external `scripts/record_session.py`.

## Open threads

- **Engine PR #4** open, validated, awaiting promotion decision.
- **Concert-mvp HANDOFF** notes an open hardening note: `normalizeProfile` doesn't gracefully handle a profile file where every cue has an invalid timestamp. Fine for the bundled profile, must fix before loading external/user-authored profiles.
- **Whoop broadcast UX is fragile** — broadcast HR mode resets per-activity in the Whoop app. Worth documenting in `repos/sidecar/docs/ble-profiles.md`'s HRS section.
- **PowerShell long-line paste fragility** — solved for the live test by `run-live-test.ps1` + `record-session.ps1` splatting-based launchers. Use them; don't paste multi-line backtick commands.
- **Mechanic exploration ideas** in `repos/game/IDEAS.md` (Zone 2 + cognitive load inversion, modifying-vs-charging axis) — not yet promoted to experiments.
- **Sub-title for the bike-themed first game** — still TBD.
- **Server architecture (Nakama)** deferred.
- **Mobile / tablet build path** acknowledged as long-term but not designed.

## Repo state

| Repo | State | Branch | Notes |
|---|---|---|---|
| umbrella | docs refresh in flight | `docs/2026-05-22-first-live-ride` | session log + this HANDOFF; PR pending |
| sidecar | trainer-control loop live-validated end-to-end | develop has everything; docs branch in flight | 113/113 unit tests; ruff + mypy clean |
| engine | bridge fully wired; MVP loop on `feat/mvp-playable-loop` (PR #4 open) | develop is bridge-only | MVP worktree at `repos/engine-mvp/` was the live-ride engine surface |
| engine-mvp | live-ride engine surface (worktree of engine `feat/mvp-playable-loop`) | feat/mvp-playable-loop | tied to engine PR #4's fate |
| concert-mvp | browser-based YouTube ERG controller; shipped Pattern A pause | develop | 7/7 controller tests; static HTML + ES modules |
| game | bootstrapped, no source yet | develop | by design |
| server | not yet `git init` | n/a | placeholder, defer |

## Entry point for next session

> "First fully end-to-end live ERG ride is done — sidecar trainer-control loop is validated. Decide engine PR #4: promote `feat/mvp-playable-loop` to develop or keep as side experiment? If promoting, ship Pattern B pause command (sidecar `PauseCommand`/`ResumeCommand` + engine phase-boundary emits; contract in `repos/sidecar/docs/architecture/safety-vs-pause.md`). Sidecar alt tracks: distance deriver, CSCS profile, first-class `--record` flag. Concert-mvp lives at `repos/concert-mvp/` as a parallel exploration — don't conflate with engine-mvp."
