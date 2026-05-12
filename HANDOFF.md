# HANDOFF — roguERGlike (umbrella)

> The current state of the project across all four repos. Updated at the end of every session. Read this first.

**Last updated:** 2026-05-12
**Last session log:** `docs/sessions/2026-05-12-bootstrap-code-repos.md`
**Current branch:** none — on develop (this wrap-up commits on `chore/bootstrap-code-repos` → PR → develop)
**Current focus:** All three active code repos bootstrapped on GitHub. Sidecar Phase 1 (mock mode) is the next implementation work.

## Where we are

Bootstrap complete. The three active code repos (sidecar, engine, game) are now independent git repos pushed to GitHub with the correct visibility, signed initial commits, `main` + `develop` branches, and branch protection on `main` for the two public ones. Engine ships with a working `effort_bridge.gd`; sidecar ships with the event-schema doc and an empty Python package; game ships with design docs only (no source — by design). Server stays deferred until Phase 5.

## What's next (immediate)

1. **Sidecar Phase 1 — mock mode.** In `repos/sidecar/` on a `feat/mock-mode` branch: Pydantic v2 event models matching `docs/event-schema.md`, WebSocket server on `localhost:8421`, slider UI for power/cadence/HR. Validate by pointing the engine's `effort_bridge.gd` at it.
2. **Engine test scene.** Once mock mode emits, build a minimal Godot test scene in `repos/engine/` that visualizes the live event stream — proves the WS contract end-to-end.
3. **Per-repo HANDOFF refresh.** Each repo's HANDOFF.md still says "not yet `git init`'d"; update each at the start of its first real work session.

## Open threads

- **Mechanic exploration ideas** captured in `repos/game/IDEAS.md` (the Zone 2 + cognitive load inversion, the modifying-vs-charging axis). Not yet promoted to experiments.
- **Sub-title for the bike-themed first game** still TBD (working title: `roguERGlike-game`).
- **Server architecture** (Nakama) deferred to Phase 5 — don't touch yet.
- **Mobile / tablet build path** acknowledged as long-term goal but not designed.

## Repo state

| Repo | State | Branch | Notes |
|---|---|---|---|
| umbrella | bootstrapped, pushed | develop | live at github.com/paulyworld/roguERGlike |
| sidecar  | bootstrapped, pushed | develop | live at github.com/paulyworld/roguERGlike-sidecar (public, MIT); branch protection on main; pre-commit installed locally |
| engine   | bootstrapped, pushed | develop | live at github.com/paulyworld/roguERGlike-engine (public, MIT); branch protection on main; effort_bridge.gd already working |
| game     | bootstrapped, pushed | develop | live at github.com/paulyworld/roguERGlike-game (private); git lfs installed; no source yet, by design |
| server   | not yet `git init` | n/a | placeholder only, defer to Phase 5 |

## Entry point for next session

> "Begin sidecar Phase 1: in repos/sidecar/ on a feat/mock-mode branch, implement Pydantic v2 event models matching docs/event-schema.md, a WebSocket server on localhost:8421, and a slider UI for power/cadence/HR. Validate by pointing the engine's effort_bridge.gd at it."
