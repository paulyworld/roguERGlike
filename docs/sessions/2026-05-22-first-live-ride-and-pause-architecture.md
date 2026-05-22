# 2026-05-22 — First fully end-to-end live ride + pause architecture

## The headline

A real ERG-controlled bike ride against the engine MVP worked end-to-end. The sidecar drove a KICKR CORE 1003, the engine MVP set targets through phase transitions, the cadence bailout fired and recovered correctly, and the telemetry recorder captured everything. This is the first time every layer of the system worked together against real hardware.

## What shipped

### Sidecar

- **PR #16 → PR #17 (squash-bundled)** — three bug fixes from yesterday's failed first attempt at a live ride:
  - **Bug A:** `CadenceBailout.handle_set_target_power` now resets `_last_active_ts` when a non-floor target passes through. Without this fix, the bailout's "no cadence" timer started at sidecar launch — so if the operator took longer than the bailout window to start the engine and trigger the first ERG write, the bailout could fire on a target that had only just been issued.
  - **Bug B:** Replaced the fail-once scan in `_run_live` with `_scan_with_retry` + `_resolve_matches`. Re-scans until every required profile is matched (or the deadline elapses). Preserves already-matched devices between cycles. Logs what's still missing. Added `--scan-timeout-s` (default 30s).
  - **Bug C:** `FtmsControl.request_control_and_start` now emits an explicit `INFO ... claimed control — set_target_power now active` on success. The KICKR's solid-blue LED is a misleading proxy for control claim (it shows BLE GATT connection only). The log line is now the authoritative "writes work now" signal.
  - **Bonus:** `scripts/run-live-test.ps1` PowerShell launcher with splatting-based args. Solves the multi-line backtick fragility that bit us twice yesterday.

- **PR #18** — `docs/architecture/safety-vs-pause.md`. The cadence bailout is a safety mechanism, not a game-flow mechanism. Two valid patterns for the pause primitive:
  - **Pattern A — Client-side soft pause** (concert-mvp, shipped today). Client sends low `set_target_power` on its own pause condition. No sidecar contract change.
  - **Pattern B — Sidecar-side suspend** (engine-mvp, planned). Client sends `pause`/`resume` commands; sidecar suspends the cadence watcher.
  - Selection rule: "is my pause a known phase boundary in code, or a user gesture?"

- **PR #19** — `scripts/record_session.py` + `record-session.ps1` wrapper. WS subscriber writing every envelope to `docs/recordings/<timestamp>.jsonl` (gitignored). Continuously flushed.

### Concert-mvp (Codex)

- Codex implemented Pattern A pause in the concert-mvp browser app. App sends low target on YouTube pause; the bailout watcher stays running; overlap case (soft-pause target landing during active bailout) handled by surfacing `bailout-pending` rejection as "queued by cadence bailout" in UI.

## The live ride

Second attempt (yesterday's failed). Today's first attempt succeeded.

Sidecar log highlights:
- `matched FTMS Bike → KICKR CORE 1003 (F9:18:EE:27:40:02)`
- `matched HR Sensor → MUDRAT-DETECTOR (DA:53:53:D8:6B:46)`
- `BLE source FTMS Bike connected to KICKR CORE 1003`
- `KICKR CORE 1003: claimed control — set_target_power now active` ← Bug C log line confirmed in the wild
- ERG targets reached the KICKR; resistance changed as the engine MVP transitioned phases
- Three bailouts fired: 74s at 177W, 78s at 164W, 75s at 176W
- Two brief Whoop HR drops mid-ride; both auto-recovered

## The architectural insight

The three bailouts were all timing-correct per the intensity-aware formula:

> For pct_ftp 0.71: t = (0.71 − 0.5) / (1.5 − 0.5) = 0.21 → bailout_s = 90 − 0.21 × 75 = 74.4s

But they all fired during legitimate between-round UI pauses, not real walk-aways. The math was right; the question was wrong. The bailout was being asked to do double duty as "the rider stopped pedalling" *and* "the game is in a UI phase" — different concerns with different time scales.

Resolution: keep the bailout as-is for safety. Add a separate pause primitive for game-flow pauses. Two valid patterns documented (see PR #18).

The same day, concert-mvp validated Pattern A in practice — Codex shipped client-side soft pause without any sidecar contract change.

## Repo state at session end

| Repo | State |
|---|---|
| umbrella | docs refresh in flight (this branch) |
| sidecar | all trainer-control work shipped + validated; 113 tests passing |
| engine | bridge fully wired on develop; PR #4 (MVP HIIT loop) open and live-validated, awaiting promotion decision |
| engine-mvp | live-ride engine surface, tied to engine #4 |
| concert-mvp | new sibling MVP; shipped Pattern A pause |
| game | bootstrapped, no source yet (by design) |
| server | not yet init |

## PRs touched this session

| Action | PR | Repo |
|---|---|---|
| Created → squash-merged | sidecar #17 (bundled #16 fixes) | sidecar |
| Created → merged | sidecar #18 | sidecar |
| Created → merged | sidecar #19 | sidecar |
| Closed superseded | sidecar #13, #15 | sidecar |
| Closed superseded | engine #8, #10 | engine |
| Closed superseded | umbrella #7, #8 | umbrella |
| Left open (promotion decision) | engine #4 | engine |

## Memories saved this session

- `bailout-is-safety-not-pause` — feedback memory establishing the architectural principle
- `concert-mvp-repo` — project memory; fifth repo identification + pause pattern + run commands
- `branch-off-develop-not-feature-branches` — feedback memory; PR #17 accidentally bundled PR #16 because the chore branch was cut from #16's feature branch instead of develop

## Open questions for next session

1. **Promote engine PR #4 to develop, or keep as side experiment?** Today's live ride validated it works. The question per its own HANDOFF is whether the HIIT-shaped loop feels worth continuing as the canonical engine surface.
2. **If promoted: ship Pattern B pause command.** Small sidecar PR + matching engine emits at phase boundaries.
3. **Concert-mvp's `normalizeProfile` hardening** — must fix before external/user-authored profiles.

## Entry point

> "First fully end-to-end live ERG ride is done — sidecar trainer-control loop is validated. Decide engine PR #4: promote `feat/mvp-playable-loop` to develop or keep as side experiment? If promoting, ship Pattern B pause command (sidecar `PauseCommand`/`ResumeCommand` + engine phase-boundary emits; contract in `repos/sidecar/docs/architecture/safety-vs-pause.md`). Sidecar alt tracks: distance deriver, CSCS profile, first-class `--record` flag."
