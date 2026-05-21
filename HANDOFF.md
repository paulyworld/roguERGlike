# HANDOFF — roguERGlike (umbrella)

> The current state of the project across all four repos. Updated at the end of every session. Read this first.

**Last updated:** 2026-05-21
**Last session log:** `docs/sessions/2026-05-21-hardware-validation-and-merges.md`
**Current branch:** `docs/post-phase-2-merge` (PR pending); all feature work merged to develop
**Current focus:** Bike integration **complete and live-validated against a real KICKR CORE 1003**. Next: pick HRS profile (chest-strap HR), distance deriver, or resume the MVP playable loop.

## Where we are

Phase 1 (mock mode) and Phase 2 (real FTMS BLE) are both merged on the sidecar's `develop`. The engine's handshake test scene is merged on its `develop`. End-to-end real BLE → engine works against a KICKR CORE 1003: with `roguerglike-sidecar --mode live --device-bike "KICKR"` running, the engine's `test_main` scene logs `device_connected kind=bike_trainer name=KICKR CORE 1003` and real `power_changed` / `cadence_changed` events as you pedal. A Windows-specific Bleak scan-filter bug surfaced during validation and is fixed (`BleakScanner.discover(service_uuids=)` silently drops scan-response advertisements on Windows; we now scan unfiltered and post-filter in Python).

## What's next (immediate)

Choose one of three concrete directions for next session — all unblocked:

1. **HRS profile** (`feat/ble-hrs`) — chest-strap heart rate. One more `BleProfile` module (service `0x180D`, characteristic `0x2A37`), a `--device-hr` flag, and an HR-source dedup policy (default: prefer standalone chest strap over bike-embedded HR; `--prefer-bike-hr` override). Apply the `bleak-windows-scan-filter-bug` pattern (always scan unfiltered + post-filter) from the start.
2. **Distance deriver** (`feat/distance-deriver`) — small follow-up so the FTMS decoder can emit `DistanceData`. The pure decoder already consumes `meters_total` bytes; this branch adds the stateful `last_total → meters_delta` logic *outside* the decoder and emits the events.
3. **Resume the MVP playable loop** — in `repos/engine/`, `git stash list` then `git stash pop` to restore the WIP edits on `feat/mvp-playable-loop`. Continue building the HIIT card-game prototype, now driven by real bike telemetry. Draft engine PR #4 already exists for this branch.

## Open threads

- **Engine PR #4 (draft)** — MVP HIIT playable loop. Land when ready.
- **Stashed engine WIP** in `repos/engine/` (`git stash list` shows "mvp-playable-loop WIP"). Recoverable; don't lose it.
- **`card.gd`** still references undefined `Effect` and `CombatContext`. Not fatal (no autoload depends on `Card`), but blocks the card system. Land alongside MVP loop work.
- **CardRegistry autoload is a no-op stub** — flesh out alongside card-system work.
- **Engine derived signals** (`effort_surge_*`, `hr_zone_changed`, `effort_pulse`) — wired through `effort_bridge.gd` but no producer emits them yet. Defer until at least one rider session has been recorded.
- **Pairing UI** (~Phase 3 per `device-pairing-ux-model` memory) — Zwift-style web pairing screen + persistent config. CLI flags (`--device-bike`, future `--device-hr`) are the bootstrap UX.
- **First CI runs** — `statusCheckRollup` was empty on the merged PRs; matches the `github-actions-first-push-quirk` memory. Should fire on the next docs PR push.
- **Mechanic exploration ideas** in `repos/game/IDEAS.md` (Zone 2 + cognitive load inversion, modifying-vs-charging axis) — not yet promoted to experiments.
- **Sub-title for the bike-themed first game** — still TBD.
- **Server architecture (Nakama)** deferred to Phase 5.
- **Mobile / tablet build path** acknowledged as long-term but not designed.

## Repo state

| Repo | State | Branch | Notes |
|---|---|---|---|
| umbrella | docs refresh in flight | `docs/post-phase-2-merge` | this session log + HANDOFF refresh; PR pending |
| sidecar  | Phase 1 + Phase 2 merged + live-validated | develop (clean) | `feat/ble-ftms` merged via PR #4; `docs/handoff-refresh` and `docs/handoff-phase-2-ble` both merged |
| engine   | handshake test scene merged | develop (clean) | PR #1, #2 merged; PR #3 closed (wrong base); draft PR #4 tracks `feat/mvp-playable-loop` |
| game     | bootstrapped, pushed | develop | no source yet, by design |
| server   | not yet `git init` | n/a | placeholder only, defer to Phase 5 |

## Entry point for next session

> "Pick one: (a) HRS profile on `feat/ble-ftms`-style `feat/ble-hrs` branch in sidecar for chest-strap HR; (b) distance deriver on `feat/distance-deriver` so FTMS emits DistanceData; (c) restore the stashed engine WIP (`git stash list` in repos/engine/, `git stash pop`) and continue the MVP HIIT playable loop with real bike data wired in."
