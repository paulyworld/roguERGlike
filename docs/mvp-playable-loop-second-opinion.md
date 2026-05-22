# MVP Playable Loop Second Opinion

This note captures an outside review of the current architecture and development plan. The short version: the technical direction is defensible, but the next milestone should aggressively prove a playable loop before expanding the platform.

## Current Shape

The project is split into four code repos under the umbrella:

- `repos/sidecar/`: Python service for device/mock telemetry, normalized JSON events, WebSocket output.
- `repos/engine/`: Godot 4 reusable framework, effort bridge, card/run/combat primitives.
- `repos/game/`: private game layer for cards, balance, theme, modality roster, and design.
- `repos/server/`: deferred backend/multiplayer.

The intended runtime flow is:

```text
fitness device or mock UI -> sidecar -> WebSocket event stream -> Godot engine -> game rules/content
```

The most important boundary is the engine's `EffortBridge`: game code should consume typed effort signals, not raw BLE/device implementation details.

## Steel Man

The core architecture is sound.

Separating the sidecar from the engine keeps BLE, device quirks, export formats, and replay tooling out of the Godot project. Python is a pragmatic fit for this layer because it has good async networking, Pydantic validation, BLE libraries, and export-library options.

The local WebSocket JSON contract is also a good early choice. It is inspectable, testable, replayable, and easy to mock. It lets the game loop advance without real bike hardware attached.

Godot is a credible engine choice for a 2D deckbuilder with custom UI and lightweight animation. The project does not appear to need Unity/Unreal-scale machinery.

The public/private repo split is reasonable: sidecar and engine can be reusable public infrastructure, while game-specific cards, balance, art, theme, and progression stay private.

The strongest design idea is training modalities as run archetypes. Mapping real workout protocols like Zone 2, Tabata, Sweet Spot, VO2 Max 4x4, and ramp tests onto roguelike structures is more compelling than a generic "pedal to generate mana" model.

## Straw Man

The main risk is building a polished telemetry platform before proving that one minute of the actual game is fun.

There is already a lot of conceptual surface area: sidecar, engine, game repo, modality framework, device-neutral schema, card framework, FIT export, replay mode, future rower/treadmill support, and deferred multiplayer. That structure may be correct eventually, but it can hide the biggest unknown: whether card decisions while exercising feel good, readable, and motivating.

Specific risks to watch:

- The event schema documentation is ahead of implementation. Derived events such as effort surges, HR zone changes, effort pulses, and tempo steadiness are designed but not yet proven.
- The sidecar event model should eventually make `type` and `data` impossible to mismatch.
- The engine bridge currently works as a test bridge, but it will need reconnect/backoff and configurable endpoint behavior.
- The card framework is still skeletal; `Effect` and `CombatContext` are conceptually referenced but not yet real enough to prove play.
- Some handoff docs can drift behind branch reality, which is risky when using multiple agents.
- Multi-repo discipline is useful later, but it adds overhead while the core loop is still unproven.

## Recommended MVP

The next milestone should be a tiny playable vertical slice, not more framework.

Build:

- One mock ride source.
- One Godot play scene.
- One player.
- One enemy.
- One card.
- One effort mechanic.
- One complete two-minute encounter.

Success criteria:

- The player can start the sidecar in mock mode.
- The Godot scene connects to the sidecar.
- Moving mock power/cadence/HR changes game state immediately.
- The player can play at least one card.
- Effort affects that card or encounter outcome in a visible way.
- The encounter can be won or lost.
- The loop is readable without developer explanation.

This should be tested before implementing the full modality framework, live BLE support, FIT export, replay tooling, broad card abstractions, or multiplayer.

## Suggested Vertical Slice

A simple first slice:

```text
Mock Power -> Strike Damage
```

Example:

- Enemy has 30 HP.
- Player has one reusable `Strike` card.
- `Strike` has base damage.
- Current/mock power modifies damage:
  - low power: normal damage
  - moderate power: bonus damage
  - high power/surge: large damage
- Enemy attacks on a simple timer or turn cadence.
- Player wins by defeating the enemy before HP runs out.

This intentionally avoids proving the full roguelike. It proves the critical question: does physical effort meaningfully and pleasantly change card play?

## What To Defer

Defer until the playable slice works:

- Full modality implementation.
- FIT/TCX/GPX export.
- Real BLE pairing and device compatibility.
- Replay mode beyond simple developer testing.
- Multiple card archetypes.
- Full run map.
- Deckbuilding economy.
- Multiplayer/server work.
- Future rower/treadmill abstractions beyond preserving names in the event schema.

The architecture can keep these future paths open, but the implementation should not chase them until the core loop earns it.

## Engineering Priorities

Near-term order:

1. Keep mock sidecar stable and easy to run.
2. Harden the sidecar-to-engine event contract.
3. Add engine reconnect/configuration basics only as needed for the slice.
4. Build the smallest playable encounter.
5. Playtest readability and feel while actually riding or simulating ride intensity.
6. Only then decide which mechanic model deserves deeper implementation.

The project is not blocked by stack choice. It is blocked by unanswered product/gameplay evidence. The fastest way to reduce risk is a playable loop.
