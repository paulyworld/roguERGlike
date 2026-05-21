# HANDOFF — roguERGlike (umbrella)

> The current state of the project across all four repos. Updated at the end of every session. Read this first.

**Last updated:** 2026-05-20
**Last session log:** `docs/sessions/2026-05-20-bike-integration-handshake.md`
**Current branch:** `docs/handoff-bike-integration-handshake` (PR pending) — three feature PRs open across sidecar + engine
**Current focus:** Bike integration pipeline proven end-to-end (mock telemetry → sidecar WS → engine bridge → typed signals); awaiting PR review before starting Phase 2 (real BLE).

## Where we are

The whole telemetry pipeline now works end-to-end with mock data: the sidecar's slider UI drives a Pydantic-validated event stream over WebSocket, the engine's `EffortBridge` consumes it, and the `test_main` scene renders the live values. The schema and wire format are wire-compatible with the engine as committed. Three signed PRs are open and awaiting review.

## What's next (immediate)

1. **Review + merge the three PRs in order:** sidecar #1 (mock mode) → sidecar #2 (replay fix, stacked) → engine #1 (handshake scene). Watch the first CI runs — per `github-actions-first-push-quirk` memory, this is the first real CI on both public repos.
2. **Sidecar Phase 2 — real BLE.** In `repos/sidecar/` on `feat/ble-ftms`: Bleak-based scanner + FTMS bike characteristic decoder feeding the existing `EventBus`. `--mode live` selects it; mock stays available.
3. **Engine v0.1.0 tag** — once Phase 2 has produced one real ride end-to-end, tag the engine so the game repo can pin it as a submodule.

## Open threads

- **Three open PRs across the project** (see `docs/sessions/2026-05-20-bike-integration-handshake.md`): sidecar #1, sidecar #2, engine #1.
- **`card.gd` parse errors** — references undefined `Effect` and `CombatContext` types. Not fatal (no autoload depends on `Card`), but the card system can't grow until they're defined.
- **Engine derived signals** (`effort_surge_*`, `hr_zone_changed`, `effort_pulse`) are wired through `effort_bridge.gd` but no producer emits them yet; defer until real telemetry exists to derive from.
- **CardRegistry autoload is a no-op stub** — flesh out when the card system needs it.
- **Mechanic exploration ideas** in `repos/game/IDEAS.md` (Zone 2 + cognitive load inversion, modifying-vs-charging axis) — not yet promoted to experiments.
- **Sub-title for the bike-themed first game** — still TBD (working title: `roguERGlike-game`).
- **Server architecture (Nakama)** deferred to Phase 5.
- **Mobile / tablet build path** acknowledged as long-term but not designed.

## Repo state

| Repo | State | Branch | Notes |
|---|---|---|---|
| umbrella | docs refresh in flight | `docs/handoff-bike-integration-handshake` | session log + this HANDOFF; PR pending |
| sidecar  | Phase 1 mock mode complete; replay fix in flight | `feat/mock-mode` + `fix/replay-session-state` | PR #1 (mock mode), PR #2 (stacked replay fix); 13/13 tests passing |
| engine   | handshake test scene in flight | `feat/handshake-test-scene` | PR #1 (test_main + CardRegistry stub + card.gd lint fix); validated against sidecar end-to-end |
| game     | bootstrapped, pushed | develop | no source yet, by design |
| server   | not yet `git init` | n/a | placeholder only, defer to Phase 5 |

## Entry point for next session

> "Review and merge the three open PRs (sidecar #1 → #2 → engine #1), then start sidecar Phase 2 in repos/sidecar/ on a `feat/ble-ftms` branch: Bleak-based BLE scanner + FTMS bike characteristic decoder feeding the existing EventBus, with `--mode live` selecting it."
