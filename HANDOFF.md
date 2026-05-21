# HANDOFF — roguERGlike (umbrella)

> The current state of the project across all four repos. Updated at the end of every session. Read this first.

**Last updated:** 2026-05-20
**Last session log:** `docs/sessions/2026-05-20-phase-2-ble-ftms.md` (Phase 2 BLE work, immediately following `docs/sessions/2026-05-20-bike-integration-handshake.md` earlier the same day)
**Current branch:** `docs/handoff-bike-integration-handshake` in umbrella (PR #4 extended); sidecar `feat/ble-ftms` (PR #4) + new `docs/handoff-phase-2-ble` (PR pending); engine PR #1 + #2 still open
**Current focus:** Phase 2 sidecar (FTMS bike, live mode) shipped behind unit tests; **hardware validation is the next thing the next session must do** — pair the trainer to Windows Bluetooth, scan, connect, confirm engine sees real telemetry.

## Where we are

The full telemetry pipeline now exists in code for both mock and live sources. Mock mode is merged and working end-to-end against the engine. Phase 2 introduces a tiny "profile registry" architecture (one `BleProfile` per device type + a `BleSource` per profile-and-address feeding the shared `EventBus`) and ships the first real BLE source — `FtmsBikeProfile` for FTMS-compatible bike trainers — behind 30/30 unit tests including handcrafted FTMS packet fixtures. The pure-decoder design means the HRS chest-strap profile lands later as one more module of the same shape, not a refactor.

What has NOT been done yet: connecting to an actual trainer over BLE. The hardware is available but wasn't paired this session, so live mode is unit-tested only. That's the immediate next step.

## What's next (immediate)

1. **Live hardware validation.** Pair the FTMS trainer to Windows Bluetooth. From `repos/sidecar/` on the `feat/ble-ftms` branch: `roguerglike-sidecar --scan` to confirm discovery, then `roguerglike-sidecar --mode live --device-bike "<name-substring>"`. In parallel, run engine `test_main` (mock-mode-style) and confirm `power_changed` / `cadence_changed` fire with realistic values while you pedal. Fix any surprises in PR #4 before merge.
2. **Merge the open PRs** in this order: sidecar #4 (Phase 2 BLE, after validation) → engine #1 (handshake scene) → engine #2 (engine HANDOFF) → umbrella #4 (umbrella HANDOFF + both session logs) → sidecar `docs/handoff-phase-2-ble` PR.
3. **HRS profile** on a new `feat/ble-hrs` branch in sidecar. Adds `HrsProfile` + an `--device-hr` CLI flag + an HR de-dup policy (default: prefer standalone chest strap over bike-embedded HR).

## Open threads

- **Hardware validation pending for Phase 2** — see #1 above.
- **Three other PRs open across the project** (engine #1, engine #2, umbrella #4) — see Repo state table.
- **`card.gd`** still references undefined `Effect` and `CombatContext`. Not fatal (no autoload depends on `Card`), but blocks the card system. Next engine work after handshake merges.
- **CardRegistry autoload is a no-op stub** — flesh out alongside card-system work.
- **Distance derivation** — FTMS decoder consumes the field but emits no `DistanceData` because `meters_delta` needs a stateful deriver. Small follow-up PR (`feat/distance-deriver`).
- **`--mode live` does not start the slider UI** — by design. Consider a small status/diagnostic page later (connection state + latest packet ts + RSSI), reusing the aiohttp server.
- **Engine derived signals** (`effort_surge_*`, `hr_zone_changed`, `effort_pulse`) are wired through `effort_bridge.gd` but no producer emits them yet; defer until real telemetry exists to derive from.
- **Mechanic exploration ideas** in `repos/game/IDEAS.md` (Zone 2 + cognitive load inversion, modifying-vs-charging axis) — not yet promoted to experiments.
- **Sub-title for the bike-themed first game** — still TBD (working title: `roguERGlike-game`).
- **Server architecture (Nakama)** deferred to Phase 5.
- **Mobile / tablet build path** acknowledged as long-term but not designed.

## Repo state

| Repo | State | Branch | Notes |
|---|---|---|---|
| umbrella | docs refresh extended for Phase 2 | `docs/handoff-bike-integration-handshake` | PR #4 — will land both session logs + this HANDOFF in one go |
| sidecar  | Phase 1 merged; Phase 2 in flight | `feat/ble-ftms` (PR #4) + `docs/handoff-phase-2-ble` (PR pending) | 30/30 unit tests passing; hardware validation pending |
| engine   | handshake test scene in flight | `feat/handshake-test-scene` (PR #1) + `docs/handoff-refresh` (PR #2) | scene validated end-to-end against sidecar mock mode |
| game     | bootstrapped, pushed | develop | no source yet, by design |
| server   | not yet `git init` | n/a | placeholder only, defer to Phase 5 |

## Entry point for next session

> "Pair the FTMS trainer to Windows Bluetooth, then from repos/sidecar/ on feat/ble-ftms: `roguerglike-sidecar --scan` to confirm discovery, then `roguerglike-sidecar --mode live --device-bike '<name-substring>'`. In parallel run the engine's `test_main` headless and confirm real `power_changed`/`cadence_changed` events fire with realistic values. After validation, merge sidecar PR #4 + the remaining engine/umbrella PRs, then start HRS profile on `feat/ble-hrs`."
