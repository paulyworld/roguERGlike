# HANDOFF — roguERGlike (umbrella)

> The current state of the project across all four repos. Updated at the end of every session. Read this first.

**Last updated:** 2026-05-12
**Last session log:** `docs/sessions/2026-05-12-repo-setup-and-conventions.md`
**Current focus:** Initial setup — none of the four repos have been `git init`'d yet.

## Where we are

The project is in pre-implementation planning. Architecture is decided (four-repo split with umbrella, sidecar pattern for BLE), naming is settled (`roguERGlike`), and design docs are written:

- Project structure decided: hybrid (umbrella + four sibling repos)
- Sidecar architecture decided: Python + Bleak, WebSocket → Godot
- Engine: Godot 4 with device-agnostic effort bridge
- Game: bike-themed first title, private, consumes engine as submodule
- Training modalities researched and documented (8 protocols → archetypes)
- Conventions for sessions, branches, commits, versioning written (see `INSTRUCTIONS.md`)

## What's next (immediate)

The next session should:

1. Create the umbrella repo on GitHub (public) and `git init` locally
2. Create the four code repos on GitHub (sidecar + engine public; game + server private)
3. Bootstrap each from the skeleton files in `/templates/repo-bootstrap/` (or the previously generated zip)
4. Make first commits + push
5. Set up branch protection on public repos

After bootstrap, the realistic first work session is **Phase 1: sidecar mock mode** — Python service that emits the event schema over WebSocket with a slider UI, no real BLE yet.

## Open threads

- **Mechanic exploration ideas** captured in `repos/game/IDEAS.md` (the Zone 2 + cognitive load inversion, the modifying-vs-charging axis). Not yet promoted to experiments.
- **Sub-title for the bike-themed first game** still TBD (working title: `roguERGlike-game`).
- **Server architecture** (Nakama) deferred to Phase 5 — don't touch yet.
- **Mobile / tablet build path** acknowledged as long-term goal but not designed.

## Repo state

| Repo | State | Branch | Notes |
|---|---|---|---|
| umbrella | bootstrapped, pushed | develop | live at github.com/paulyworld/roguERGlike |
| sidecar  | skeleton only, not yet `git init` | n/a | files in repos/sidecar/, ready to bootstrap next session |
| engine   | skeleton only, not yet `git init` | n/a | files in repos/engine/, ready to bootstrap next session |
| game     | skeleton only, not yet `git init` | n/a | files in repos/game/, ready to bootstrap next session |
| server   | not yet `git init` | n/a | placeholder only, defer to Phase 5 |

## Entry point for next session

> "Bootstrap the three active code repos (sidecar, engine, game) following SETUP.md, in that order. Server stays deferred. After bootstrap, update HANDOFF.md and write a session log to docs/sessions/."
