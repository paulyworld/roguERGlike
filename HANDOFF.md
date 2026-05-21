# HANDOFF — roguERGlike (umbrella)

> The current state of the project across all four repos. Updated at the end of every session. Read this first.

**Last updated:** 2026-05-21
**Last session log:** `docs/sessions/2026-05-21-hrs-validation-and-mvp-promotion.md` (follows `2026-05-21-hardware-validation-and-merges.md` earlier the same day)
**Current branch:** `docs/post-hrs-mvp-promotion` (PR pending); sidecar BLE pipeline fully merged on develop
**Current focus:** **Cycling BLE pipeline complete and live-validated** — KICKR CORE 1003 (FTMS) + Whoop MG5 (HRS) both running concurrently into the engine's MVP HIIT loop. Next: review/merge MVP PR #4 + pick card-system / distance / CSCS as the next code branch.

## Where we are

The full cycling-side pipeline is shipped: mock mode for off-bike dev, FTMS bike trainer source, HRS heart-rate source, all merged on the sidecar's `develop`. Both real BLE sources have been validated end-to-end against the engine — a KICKR CORE 1003 streams power+cadence, a Whoop MG5 (broadcast HR enabled in the Whoop app) streams real heart rate, and the engine's MVP HIIT loop on `feat/mvp-playable-loop` renders and responds to all three live signals in real time.

The MVP playable loop (PR #4, now promoted to **ready for review**) is the first complete vertical slice of the game: alternating player-turn (recovery + card play) and enemy-interval (high-intensity W/kg push), FTP- and age-derived phase targets, configurable rider profile, live telemetry chart, HP/energy combat skeleton. Single hard-coded strike for now; real card variety is blocked on the card-system foundation (`Effect` + `CombatContext` base classes).

## What's next (immediate)

1. **Review + merge engine PR #4** (MVP HIIT playable loop). Then decide whether to iterate the loop further or branch `feat/card-system-foundation` in engine.
2. **Card-system foundation** (engine `feat/card-system-foundation`) — define minimal `Effect: Resource` with `apply(context)` and `CombatContext: RefCounted` with hand/draw/discard piles. Unblocks `card.gd` parse errors and real card variety.
3. **Distance deriver** (sidecar `feat/distance-deriver`) — small follow-up. The FTMS decoder already consumes `meters_total` bytes; this branch adds the stateful `last_total → meters_delta` logic outside the decoder and emits `DistanceData` events.

Alternative directions (not blocking the above):
- **CSCS profile** (sidecar `feat/ble-cscs`) — for older trainers / power meters that expose cadence outside FTMS.
- **Session recording** (sidecar) — append-only JSONL writer subscribed to `EventBus`; foundation for replay mode and FIT export.
- **Pairing UI** (sidecar, ~Phase 3) — Zwift-style web pairing screen + persistent device config.

## Open threads

- **Engine PR #4** (Ready for Review) — MVP HIIT loop awaiting review pass and merge.
- **Stale stash on `feat/mvp-playable-loop`** in engine: `git stash list` shows "mvp-playable-loop WIP — saved before connection-test branch switch 2026-05-21". The branch has progressed past it (two new feature commits since); likely obsolete. `git stash drop` once you confirm.
- **`card.gd` parse errors** — still references undefined `Effect` and `CombatContext`. Not fatal (no autoload depends on `Card`), but blocks card variety. Land alongside the foundation branch.
- **CardRegistry autoload** is a no-op stub. Flesh out alongside card-system work.
- **Engine derived signals** (`effort_surge_*`, `hr_zone_changed`, `effort_pulse`) are wired through `effort_bridge.gd` but no producer emits them yet. Defer until at least one full ride has been recorded.
- **Pairing UI** as a near-term project per memory `device-pairing-ux-model` — bootstrap UX (CLI flags) is sufficient for solo dev.
- **Reconnect-on-drop chaos test** — `BleSource.run`'s reconnect loop is unit-tested but not stress-tested (yank trainer power mid-stream).
- **First CI runs** still subject to the `github-actions-first-push-quirk` memory — the next push to each repo should fire CI cleanly.
- **`fit-tool` runtime dep** is carried but unused; for eventual FIT export.
- **Mechanic exploration ideas** in `repos/game/IDEAS.md` (Zone 2 + cognitive load inversion, modifying-vs-charging axis) — not yet promoted to experiments.
- **Sub-title for the bike-themed first game** — still TBD.
- **Server architecture (Nakama)** deferred to Phase 5.
- **Mobile / tablet build path** acknowledged as long-term but not designed.

## Repo state

| Repo | State | Branch | Notes |
|---|---|---|---|
| umbrella | docs refresh in flight | `docs/post-hrs-mvp-promotion` | session log + this HANDOFF; PR pending |
| sidecar  | Full cycling BLE pipeline merged + live-validated | develop (clean) | mock + FTMS + HRS all on develop; KICKR + Whoop MG5 confirmed working |
| engine   | Handshake merged; MVP HIIT loop ready for review | `feat/mvp-playable-loop` (PR #4) | hardware-validated against bike + HR concurrently |
| game     | bootstrapped, pushed | develop | no source yet, by design |
| server   | not yet `git init` | n/a | placeholder only, defer to Phase 5 |

## Entry point for next session

> "Review and merge engine PR #4 (MVP HIIT playable loop). After merge, pick one of: (a) `feat/card-system-foundation` in engine — minimal Effect + CombatContext base classes so `card.gd` parses and real card variety can land on top of the MVP loop; (b) `feat/distance-deriver` in sidecar — stateful FTMS distance deriver outside the pure decoder; (c) `feat/ble-cscs` in sidecar — third BLE profile for older trainers / power meters that expose cadence outside FTMS."
