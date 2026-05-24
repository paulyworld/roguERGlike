# ERGlike (formerly roguERGlike)

A platform for game-shaped workouts and workout-shaped games, driven by real-time data from connected fitness equipment. The name is a play on "-like" genre suffixes and "erg" — the unit of energy, and the common shorthand for ergometer (the family of indoor fitness machines this targets).

**Renamed 2026-05-23** from `roguERGlike` → `ERGlike`. The "rogu-" prefix locked the project into a single genre; the umbrella has since grown to include concert-driven rides (codename **gizzERG**), terrain modes, and other non-roguelike riding experiences. GitHub repo names remain `roguERGlike-*` for now; a coordinated rename is planned for a future clean break point.

This is the **umbrella repo** for the project. It contains conventions, cross-repo state, session history, and helper scripts. The actual application code lives in sibling repos under `repos/` (which is `.gitignore`'d here — each is its own independent git checkout).

## New here? Start with these

1. **`INSTRUCTIONS.md`** — canonical conventions for sessions, branches, commits, versioning, etc. Read this first.
2. **`HANDOFF.md`** — current state of the project across all four repos. What's happening right now.
3. **`SETUP.md`** — how to bootstrap all five repos from scratch.

## Scope and naming

`ERGlike` is the umbrella name for the whole stack. The first riding experiences are bike-trainer focused (see `gizzERG` below), but the underlying tech (sidecar, engine) is intentionally device-agnostic to support rowing ergs, treadmills, and other ergometers in future modes.

| Device | Protocol | Status |
|---|---|---|
| Smart bike trainers | FTMS + HR | Phase 1 target |
| Rowing ergs (Concept2 PM5, etc.) | BLE Rowing / FTMS Rower | Future |
| Treadmills | FTMS Treadmill | Future |
| Power meter pedals standalone | Cycling Power Service | Future |

The same card mechanic ("sprint surge generates bonus damage") works whether the surge is on a bike, a rower, or a treadmill — the engine subscribes to device-neutral signals.

## The code repos

| Path | GitHub repo | Visibility | Purpose | License |
|---|---|---|---|---|
| `repos/sidecar/` | roguERGlike-sidecar | **Public** | BLE bridge, telemetry, FIT/TCX/GPX export | MIT |
| `repos/engine/`  | roguERGlike-engine  | **Public** | Godot 4 deckbuilder framework | MIT |
| `repos/concert-mvp/` | **gizzERG** | **Public** | Browser ERG controller driven by YouTube concert profiles. Currently the primary near-term riding experience. | MIT |
| `repos/game/`    | roguERGlike-game    | **Private** | First game: cards, balance, art, theme | All rights reserved |
| `repos/server/`  | roguERGlike-server  | **Private** | Online matchmaking, leaderboards — deferred | All rights reserved |

### gizzERG codename

`concert-mvp` lives on GitHub as **paulyworld/gizzERG** — a play on King Gizzard & The Lizard Wizard (whose Greek tour video drives the first concert profile) and the ERG ergometer family. The codename is intentionally evocative for the current concert focus; the implementation generalizes to any YouTube-driven ride map.

## Training modalities as level archetypes

A core design pillar: real-world training protocols (HIIT, Zone 2, Sweet Spot, Tabata, Norwegian 4×4, ramp tests, etc.) have signature effort shapes that map naturally to different roguelike run archetypes. The player picks a modality at run start and gets both a real exportable workout and a roguelike run whose mechanical pacing matches the protocol.

- **Modality framework** (interval timing, effort-band enforcement, recovery scenes, FIT-segment export) lives in the public engine
- **Specific modality roster** for each game (cycling protocols vs. rowing protocols) lives in that game's private repo

See `repos/engine/docs/modality-framework.md` and `repos/game/docs/design/training-modalities.md`.

## Build-in-public posture

Devlogs, streams, screenshots, and PRs are welcome on the public repos. Specific card designs, exact balance numbers, and art reveals stay in the private game repo until release windows. See `INSTRUCTIONS.md` for the IDEAS.md / experiments / design progression that keeps speculative ideas separate from committed direction.

## License

This umbrella repo is MIT-licensed (see `LICENSE`). Each code repo has its own license — see those repos' READMEs.
