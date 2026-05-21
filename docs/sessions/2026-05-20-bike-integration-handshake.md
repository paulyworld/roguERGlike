# Session: 2026-05-20 — Bike integration handshake

**Repo(s):** sidecar + engine + umbrella
**Interface:** Claude Code
**Branch(es):** sidecar `feat/mock-mode`, sidecar `fix/replay-session-state`, engine `feat/handshake-test-scene`, umbrella `docs/handoff-bike-integration-handshake`
**Duration:** ~2 hours

## Context loaded

- Umbrella `HANDOFF.md` (stale — last updated 2026-05-12; bootstrap-complete state)
- Umbrella `INSTRUCTIONS.md` + `CLAUDE.md` + `HOW-TO-UPDATE-HANDOFF.md` — conventions
- `repos/sidecar/CLAUDE.md` + `HANDOFF.md` + `docs/event-schema.md` — Phase 1 scope and the wire contract
- `repos/engine/CLAUDE.md` + `HANDOFF.md` + `src/effort/effort_bridge.gd` — what the engine already consumes
- Auto-memory: `setup-branch-protection-bug`, `github-actions-first-push-quirk`

## Decisions made

- **Stayed on Python for the sidecar.** Considered Rust/Go/Node/.NET at session start. Bleak (Python) is the only actively-maintained cross-platform BLE library with solid Windows support, and BLE is the gnarliest part of the project. Sidecar's perf envelope (≤10 Hz events) is nowhere near Python becoming a bottleneck. Existing bootstrap (pyproject, pre-commit, CI) reinforced the choice.
- **aiohttp for the mock-mode slider UI** (over plain `websockets` + stdlib http.server, or a CLI REPL). One process, no extra runtime, shares the asyncio loop with the WS broadcaster, and the slider page is a single static HTML string with no build step.
- **Mock-mode-first.** Phase 1 ships a slider-driven mock event source; Phase 2 swaps the source for real BLE/FTMS. Lets the whole pipeline (schema → broadcaster → engine bridge) be developed and validated without the trainer powered up.
- **Replay session-state events** (`session_start`, `device_connected`, `device_disconnected`) to late subscribers, but never per-tick telemetry. Replaying telemetry would deliver stale values to the engine as if they were live.
- **Widened publish's lock** to include subscriber snapshot, closing the race where a subscriber landing between state update and fan-out could receive the same envelope twice (once via replay, once live).
- **Stacked the fix PR on `feat/mock-mode`** (sidecar PR #2 → feat/mock-mode → develop) rather than waiting for mock-mode to merge. GitHub auto-retargets to develop on parent merge.
- **Bundled card.gd lint fix into the engine PR.** The handshake PR needed CI to be green; `card.gd`'s `class-definitions-order` failures pre-existed on develop. Trivial reorder (no behaviour change), clearly noted in the PR body.
- **Stubbed `CardRegistry` (not Card)** to unblock the engine project boot. Untyped `Dictionary` for now so the stub doesn't depend on `Card`'s parse state — `card.gd` still references undefined `Effect` and `CombatContext`, which we left as warnings rather than scope-creeping into the card system.

## Work done

### Sidecar — `feat/mock-mode` → PR #1
- `src/roguerglike_sidecar/events.py` — Pydantic v2 envelope + per-type data models (power, cadence, heart_rate, speed, distance, device_connected/disconnected, session_start/end). Strict, frozen, value-range validated.
- `src/roguerglike_sidecar/ws_server.py` — `EventBus` (assigns monotonic `seq` + stable `session_id`) and `run_ws_server` on `localhost:8421` using `websockets`.
- `src/roguerglike_sidecar/mock.py` — `MockState` (slider state with clamping) and `run_mock_loop` at 1 Hz emitting power/cadence/heart_rate.
- `src/roguerglike_sidecar/web_ui.py` — aiohttp server on `localhost:8422`; single static slider page; `/control` POST mutates `MockState`.
- `src/roguerglike_sidecar/cli.py` — Click entry point; `--mode mock` boots all three in one event loop.
- `tests/` — 7 unit tests: event round-trip, range validation, extra-field rejection, bus seq monotonicity, subscriber fan-out, unsubscribe lifecycle, mock state clamping + producer event types.
- `scripts/e2e_smoke.py` — drives a real WS client + UI control round-trip; confirms slider updates reach the next `power` envelope.
- `pyproject.toml` — added `aiohttp>=3.9` runtime dep.
- Commit: `feat(sidecar): implement Phase 1 mock mode` (signed `0e8a462`).

### Sidecar — `fix/replay-session-state` → PR #2 (stacked on #1)
- `src/roguerglike_sidecar/ws_server.py` — `EventBus` now tracks latest envelope per session-state type and prefills new subscriber queues in seq order before live forwarding. Publish's lock widened to include subscriber snapshot.
- `tests/test_ws_server.py` — 3 new tests: late subscriber receives replay; early subscriber no duplicate; latest-state wins on republish. 13/13 total tests pass.
- Commit: `fix(sidecar): replay session-state events to late subscribers` (signed `25b27b8`).

### Engine — `feat/handshake-test-scene` → PR #1
- `tests/test_main.tscn` + `tests/test_main.gd` — Control with labels for connection, device, power, cadence, heart rate, surge. Subscribes to every `EffortBridge` signal and mirrors each event to stdout for headless verification.
- `src/cards/card_registry.gd` — minimal stub Node so the `CardRegistry` autoload declared in `project.godot` actually loads.
- `src/cards/card.gd` — reorder definitions (class_name before extends, enum before exports) to satisfy `gdlint`'s `class-definitions-order`. No behaviour change.
- Commit: `feat(engine): add sidecar handshake test scene` (signed `d813a7d`).

### End-to-end verification
With sidecar mock mode running and sliders driven to `watts=312 rpm=95 bpm=154` via `/control`:
```
[test_main] connection_state_changed connected=true
[test_main] device_connected kind=mock name=Mock Bike    # ← fixed by sidecar PR #2
[test_main] power_changed watts=312
[test_main] cadence_changed rpm=95
[test_main] heart_rate_changed bpm=154
```
Schema, wire format, broadcaster, and engine bridge are compatible.

### Umbrella — `docs/handoff-bike-integration-handshake`
- This session log.
- `HANDOFF.md` refresh.

## Open threads

- **Three PRs awaiting review/merge:** sidecar #1 (mock mode), sidecar #2 (replay fix, stacked), engine #1 (handshake scene). Merge order: sidecar #1 → sidecar #2 → engine #1.
- **`card.gd` parse errors** still present — references undefined `Effect` and `CombatContext`. Not fatal (no autoload depends on `Card`), but should be resolved when the card system is fleshed out.
- **First CI runs** on both public repos will fire on these PR pushes — per the `github-actions-first-push-quirk` memory, this is expected and we should watch for any unexpected failures (gitleaks env, matrix runners, etc.).
- **Leftover sidecar processes can hold port 8421/8422** if `roguerglike-sidecar` is killed roughly during local testing. Found one during this session (PID had to be force-killed). Worth keeping in mind for next session.
- **Sidecar Phase 2 — real BLE** (`feat/ble-ftms`): Bleak scanner + FTMS bike characteristic decoder feeding the same `EventBus`. Mock and live should coexist behind `--mode`.
- **Engine derived signals** (`effort_surge_*`, `hr_zone_changed`, `effort_pulse`) — wire signals exist on the bridge but no producer emits them yet; defer until a real telemetry source exists to derive from.
- **CardRegistry stub is a no-op** — flesh out when the card system needs it.
- **Sub-title for the bike-themed game** — still TBD.
- **Server (Phase 5)** — untouched.
- **Mobile/tablet build path** — long-term.

## Next session entry point

> "Review and merge the three open PRs (sidecar #1 → #2 → engine #1), then start sidecar Phase 2 in repos/sidecar/ on a `feat/ble-ftms` branch: Bleak-based BLE scanner + FTMS bike characteristic decoder feeding the existing EventBus, with `--mode live` selecting it."

## Loose notes

- `ruff` (local CLI) classifies `roguerglike_sidecar` as first-party and wants imports in a separate block; the pre-commit hook (also ruff 0.5) classifies it as third-party and removes the blank line. Pre-commit always wins because it's the merge gate — defer to its formatting on commit.
- Pre-commit-hook auto-fixes ALWAYS abort the commit (`files were modified by this hook`); re-stage and re-commit. Counted twice in this session.
- The new `websockets.asyncio.server` API (replaces deprecated `websockets.server`) uses a single-arg handler `async def(connection)`, not the old `(websocket, path)` signature. Stable in `websockets>=13`.
- Stacked PRs in `gh`: pass `--base feat/mock-mode --head fix/replay-session-state`. GitHub auto-retargets the child PR to develop when the parent merges.
- Engine CI workflow currently has `|| true` on the headless test step — it's effectively only checking that Godot boots, not that any assertion passes. Worth tightening once we have a real test runner (GUT or hand-rolled).
- Today's date format reminder: HANDOFF + session-log dates are absolute (`YYYY-MM-DD`), per memory convention.
