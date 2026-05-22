# Session: 2026-05-21 — FTMS ERG end-to-end (capability → control → bridge → MVP wiring)

**Repo(s):** sidecar + engine + umbrella
**Interface:** Claude Code
**Branch(es):** sidecar `feat/ftms-capabilities` (merged), `feat/ftms-trainer-control` (re-landed via PR #11), `fix/replay-control-lifecycle-events` (merged); engine `feat/trainer-control-bridge` (merged), `feat/mvp-playable-loop` (in flight, PR #4)
**Duration:** ~3 hours (third session of the day, follows the HRS validation + MVP promotion session)

## Context loaded

- Umbrella + per-repo HANDOFFs from the morning's refresh
- Memories: `device-pairing-ux-model`, `bleak-windows-scan-filter-bug`, `hr-source-breadth`
- Bluetooth SIG FTMS v1.0 §4.3.2 (Feature characteristic) and §4.16.2 (Control Point opcodes)
- Engine PR #4 (MVP HIIT playable loop) ready-for-review state
- The user's preference per memory: aggressive default safety policy on trainer control (raw trainer feel over conservative smoothing)

## Decisions made

- **Capture aggressive-default safety as a memory** (`trainer-control-aggressive-defaults`) before code. The choice influences this PR's defaults and future intensity features; it's a durable preference, not a one-shot.
- **Three-PR sequence** for trainer control: PR1 capability discovery (informational, no behaviour change), PR2 sidecar control writes (the meaty one), PR3 engine bridge write-side API. Smaller diffs, intermediate safe states.
- **`device_capabilities` as a new event type**, not an extension of `device_connected`. Forward-compatible for additional capability flags without mutating an existing event.
- **HRS profile gets `feature_char_uuid = None`**. The Heart Rate Service has a *Body Sensor Location* characteristic but nothing useful for the schema's capability flags. Explicit None > implicit absence.
- **Bidirectional WS over the existing connection**, not a separate command channel. Engine sends `Command` envelopes; sidecar parses + dispatches via an injected `on_command` handler. One transport for both directions.
- **Command schema is a discriminated union** on `type`. Pydantic v2 handles this cleanly; no manual dispatch needed.
- **Outbound `device_capabilities` is NOT a Command** — it's an event. Inbound commands are deliberately a separate schema (no `ts`/`seq`/`session_id`) so they're plainly distinguishable.
- **`target_power_set` outbound event is intentionally NOT replayed** to late subscribers. It's an ack of a specific command, not ambient state. Engine can re-issue `set_target_power` if it needs to assert the current value.
- **Envelope round-trip discriminator bugfix.** Identically-shaped data models (`DeviceConnectedData` vs `ControlAcquiredData`, both `{kind, name}`) collapsed onto the first union member during parse. Added a `model_validator` that uses the outer `type` field to pick the right data class. JSON wire format unchanged; Python type identity restored.
- **Closed engine PR #3** (mis-targeted against `main`) and re-opened the trainer-control work as engine PR #7 against develop with the right base.
- **Re-landed sidecar PR #10 as PR #11** after GitHub failed to auto-retarget the stacked PR when its base (`feat/ftms-capabilities`) was merged but not deleted. Rebased the branch onto develop (git skipped the duplicated squashed commit automatically) and re-merged into develop.
- **Live KICKR validation against PR #11 happened via a Python WS probe**, not through the engine bridge. The probe drove the ramp; the engine ran the MVP scene as a passive observer concurrently. Validating engine-as-driver is a follow-up live test once the MVP scene's ERG wiring (this session) has been smoked.
- **MVP scene's ERG wiring is per-phase**, not continuous. `set_target_power(...)` fires once at each phase entry plus per-tick during the warmup ramp. The trainer interpolates internally between set points — no need for the sidecar to ramp on its own.
- **All MVP ERG writes gate on `EffortBridge.supports_target_power`**. A non-controllable device, mock-mode-without-flag, or no sidecar = silent no-ops. Game still plays; resistance just isn't enforced.
- **Disconnect-bailout grace window stays on with aggressive defaults** (10s default, but bumped to 600–900s during long-duration tests via `--disconnect-bailout-s`). This is a don't-burn-down-the-trainer concern, not a feel concern, so it's the one safety knob that's on by default.

## Work done

### Sidecar — `feat/ftms-capabilities` → PR #9, merged (`4e0a7ea`)
- `events.py`: new `DeviceCapabilitiesData` + `"device_capabilities"` event type. All flags default False.
- `ble/ftms_bike.py`: `parse_target_setting_features` decodes the 8-byte FTMS Feature characteristic (`0x2ACC`), upper-4-bytes Target Setting Features. Total; ignores reserved bits.
- `FtmsBikeProfile.feature_char_uuid` + `parse_features()` — opt-in hook the source inspects via `getattr`.
- `BleSource._read_and_publish_capabilities` post-connect. Soft failure: trainers that advertise FTMS but reject the feature read are logged + treated as no capabilities; the source continues.
- `HrsProfile.feature_char_uuid = None` (explicit).
- 10 new tests; docs/event-schema.md updated.

### Sidecar — `feat/ftms-trainer-control` → PR #10 → re-landed as PR #11, merged (`0e9c883`)
- `ble/ftms_control.py` (new): `FtmsControl` state machine. Request Control → Start (request_control_and_start); Set Target Power with int16 LE + clamp; Stop with subparameter `0x01`; idempotent `release(reason)`. Response indications correlated by request opcode via `asyncio.Future` per opcode.
- `BleSource.control_factory` (optional): post-capability-read, source invokes the factory with the live `BleakClient`; if it returns an `FtmsControl`, source attaches it and tries `request_control_and_start`. Failures don't crash the source — read path continues.
- `ws_server.py` bidirectional: connection handler runs send + recv loops concurrently. Recv parses `parse_command_json` and dispatches via injected `on_command` callback. Malformed messages logged + dropped.
- New `Command` discriminated union: `set_target_power` / `start` / `stop` / `release_control`.
- New outbound events: `control_acquired`, `control_released` (with reason), `target_power_set` (carries post-clamp value + accepted + reason).
- CLI flags: `--allow-trainer-control` (opt-in), `--max-target-power 800`, `--min-target-power 0`, `--disconnect-bailout-s 10`.
- Disconnect-bailout watcher polls `bus.subscriber_count` at 1 Hz; when all clients drop while controlling, waits the grace window then issues Stop; reconnect disarms.
- Mock-mode ERG simulation: `MockState.erg_target_watts` overrides slider; `mock_set_target_power` publishes wire-equivalent envelopes; mock auto-acquires control at startup when `--allow-trainer-control` is set.
- Envelope schema-discriminator bugfix (see Decisions).
- 24 new tests on top of capability discovery's 10. Total 90 → 94 over the session (with the replay-fix's three).
- **Live KICKR CORE 1003 validation 2026-05-21**: drove `100 → 250 → 100 W` ramp via Python WS probe; every command came back `accepted=true`; trainer's resistance tracked targets cleanly through the ramp. Reconnect-restore feature incidentally validated. Screenshot committed at `docs/screenshots/20260521_ftms_write_test.jpg` with a README captioning the convention.

### Sidecar — `fix/replay-control-lifecycle-events` → PR #12, merged
- Widened `SESSION_STATE_TYPES` to include `device_capabilities`, `control_acquired`, `control_released`. Engines that attach mid-session now learn the device's capabilities and current control state.
- `target_power_set` intentionally NOT replayed (it's an ack, not state).
- Caught while smoke-testing engine PR #7 against mock-mode — an engine attaching after `mock_acquire_control` had fired missed `control_acquired` entirely. Bridge code was correct; replay set needed widening.
- 4 new tests + a `_next_of_type` helper in `test_ftms_control.py` to drain replayed events in the 5 existing tests that subscribed AFTER `request_control_and_start`.

### Engine — `feat/trainer-control-bridge` → PR #7, merged
- `EffortBridge` gained write API: `set_target_power(watts)`, `start()`, `stop()`, `release_control()` — each sends a JSON command over the WS via a small `_send_command` helper.
- New signals: `device_disconnected`, `device_capabilities_changed`, `control_acquired`, `control_released`, `target_power_set`.
- New state: `supports_target_power: bool` driven by `device_capabilities` events; reset to `false` on `device_disconnected`. Game UI should gate ERG features on this.
- `tests/test_main.gd` (handshake scene): hooked the new signals up and mirrors them to stdout. Reflects control state in the "Derived" label.
- gdlint clean. Headless boot without a sidecar prints "ready; waiting for sidecar" and survives. Boot against mock-mode `--allow-trainer-control` shows telemetry flowing.

### Engine — `feat/mvp-playable-loop`, in flight (PR #4)
- Merged `develop` in (`86e0c7b`) to pick up the bridge write-API. Three-way conflicts in HANDOFF.md / tests/test_main.gd / tests/test_main.tscn resolved by keeping the MVP branch's versions; `src/effort/effort_bridge.gd` auto-merged cleanly.
- `tests/test_main.gd` (`01b68d7`): wired ERG writes to phase transitions.
  - Warmup entry → `set_target_power(40% FTP)`.
  - Warmup ticks (chart-sample cadence ~2 Hz + every 5s explicit) → update to lerped current target.
  - Recovery entry (`_begin_player_turn`) → `set_target_power(55% FTP)`.
  - Interval entry (`_start_enemy_interval`) → `set_target_power(120% FTP)`.
  - Reset / victory / defeat → `release_control()`.
  - All gated on `EffortBridge.supports_target_power`; no-op if the bridge hasn't seen `device_capabilities` with `target_power=true`.
- The user's parallel commits over the session evolved the MVP scene substantially: settings panel (weight, FTP, age, HR zones, warmup minutes), per-metric charts (split from a single chart), big readouts, target feedback meters, turn-scoped energy, sidecar URL override, "Start Workout" + warmup phase. The ERG wiring lands on top of all of that.

### Sidecar — diagnostics
- `scripts/record_session.py` (new): WS subscriber that dumps every envelope to a timestamped JSONL under `docs/recordings/` (gitignored). Flushes on every write so a Ctrl+C mid-test leaves a usable file. Intended as a portable diagnostic for paired-hardware test sessions — pairs with the screenshot pattern from earlier validation work.

### Live validation matrix (so far)
| Layer | Validation |
|---|---|
| Sidecar FTMS read (PR Phase 2) | KICKR CORE 1003 ✓ |
| Sidecar HRS read (PR #7) | Whoop MG5 ✓ |
| Concurrent bike + HR | KICKR + Whoop ✓ |
| Sidecar capability discovery (PR #9) | not yet exercised against KICKR (trainer asleep at commit time; will surface during PR #10 validation) |
| Sidecar trainer-control writes (PR #11) | KICKR via Python WS probe ✓ (this session) |
| Sidecar replay fix (PR #12) | unit-tested; will be exercised by any engine that connects mid-session |
| Engine bridge (PR #7) | mock-mode smoke ✓; live against KICKR pending |
| Engine-driven ERG end-to-end | **pending — the immediate next test** |

## Open threads

- **Engine-driven ERG live validation pending.** Three-window setup (sidecar / JSONL recorder / Godot), KICKR + Whoop, MVP scene's Start Workout button drives the phases and the trainer should respond. Will produce a screenshot + JSONL recording artifact. This is the next concrete action.
- **Engine PR #4 (MVP HIIT loop, Ready for Review)**. Once engine-driven ERG validates against KICKR, this PR is the right place to land. PR body needs another pass — current body was written before the session's substantial UX additions.
- **Card-system foundation** (`Effect` + `CombatContext` base classes in engine). Still pending; the MVP loop's two hard-coded cards (Power Strike, Cadence Guard) need this to grow.
- **`--record` as a first-class sidecar flag** (vs the current `scripts/record_session.py` external tool). Natural follow-up; small.
- **FTMS SIM mode** (slope/wind/CRR via opcode `0x11`) — gated on `indoor_bike_simulation: true`. Out of scope for now; useful when a non-HIIT mechanic wants the trainer to feel like a hill.
- **Distance deriver** (sidecar) — FTMS decoder consumes `meters_total` but emits no `DistanceData`. Small stateful module outside the pure decoder. Still pending.
- **CSCS profile** — older trainers / power meters with cadence outside FTMS. Still pending.
- **`card.gd` parse errors** — references undefined `Effect` / `CombatContext`. Lands with the foundation PR.
- **Engine CI**: `godot-headless-tests` runs `|| true`. Not enforcing assertions. Plumbing for a real test runner (GUT or hand-rolled) is still pending.
- **MVP scene living under `tests/`** — once a real test runner lands and the loop is more than a prototype, it should move to `repos/game/` (per the existing engine/game boundary in the per-repo CLAUDE.md docs).

## Next session entry point

> "Resume the engine-driven ERG live test. Three windows: (1) sidecar with `--mode live --device-bike KICKR --device-hr mudrat --allow-trainer-control --disconnect-bailout-s 900`; (2) `python scripts/record_session.py` once both devices show connected; (3) `godot --path C:\dev\roguERGlike\repos\engine-mvp` then ▶ Play. Configure rider settings (suggest Warmup=2min for a quick first pass), press Start Workout, ride through warmup → recovery → interval cycle. Confirm KICKR resistance tracks the engine-driven targets; HR flows from the Whoop concurrently. If green, screenshot + JSONL go in `docs/screenshots/` and `docs/recordings/`, and engine PR #4 (MVP HIIT loop) is ready to merge. If anything's off, fix in MVP branch before merging."

## Loose notes

- GitHub does **not** auto-retarget stacked PRs when the parent branch is merged but not deleted. PR #10 → `feat/ftms-capabilities`, then PR #9 squashed into develop, then PR #10's "merge" went into the stale `feat/ftms-capabilities` branch (which still existed) rather than develop. Workaround was to open a new PR (#11) directly against develop and rebase the branch onto develop's tip (git skipped the duplicated squashed commit automatically). Worth setting the org's "delete branch on merge" preference, or always opening stacked PRs with `--base develop` and just letting GitHub show the conflict until the parent merges.
- Pydantic v2's discriminated-union dispatch via `Field(discriminator="type")` only works when the discriminator is on the union member itself. For `Envelope`'s `data` field, the discriminator (`type`) is on the OUTER model — Pydantic can't see it from the field. Hence the `model_validator(mode="before")` that explicitly picks the data class based on the outer `type` field. JSON wire format unchanged.
- `git rebase origin/develop` after a stacked PR's parent has squash-merged: git detects the duplicate commit by patch content and skips it ("previously applied commit ... skipped"). Cleaner than cherry-picking.
- The recording script gates output under `docs/recordings/` which the sidecar's existing `.gitignore` already matches (the `recordings/` pattern matches at any depth). HR / power telemetry is PII per project conventions; this keeps it out of the repo without an additional ignore rule.
- Aggressive-default safety surprised exactly once during validation: a 10s disconnect-bailout fired when I'd booted the sidecar but hadn't yet connected a WS client. Bumped to 120s / 600s / 900s during subsequent tests. The behaviour is correct (don't keep the trainer captured forever if no engine is listening), the default just needs to be friendly to "set up sidecar, then engine, then ride" workflows. May revisit the default later — could be 30s instead of 10s without changing the safety intent.
- F5 in Godot was still flaky on the first launch attempt today. The ▶ Play button in the editor toolbar remains the reliable way.
- KICKR LED diagnostic stays valuable: solid blue = sidecar has it; blinking = available. Quick eyeball check before any test.
