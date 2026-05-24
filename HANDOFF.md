# HANDOFF — ERGlike (umbrella)

> The current state of the project across all repos. Updated at the end of every session. Read this first.

**Last updated:** 2026-05-24 (late)
**Last session log:** `docs/sessions/2026-05-24-sidecar-sequence-complete.md`
**Current branch:** umbrella `docs/2026-05-24-sidecar-sequence-complete`
**Current focus:** **All five items in Codex's `claude-sidecar-review-brief.md` sequence are now sidecar-side code-complete and merged.** Sidecar at 169 tests, ruff + mypy clean. Next: live-ride validation + Strava upload verification + Codex's gizzERG work picks up the next phase (Terrain Mode UI, scrub-mode profile editor, audio preprocessing tool).

## ⚠️ Return-to: Strava / TrainingPeaks FIT upload verification

The FIT export (sidecar PR #25) is **code-complete and parser-validated**, but the acceptance criterion from the brief — "manual FIT upload works in Strava / manual FIT upload works in TrainingPeaks" — needs the rider to actually upload one. Quick rep:

```powershell
cd C:\dev\roguERGlike\repos\sidecar
roguerglike-export fit docs/recordings/20260523_121056.jsonl ride.fit
# → drag ride.fit into Strava (https://www.strava.com/upload/select)
#   and TrainingPeaks. Report back any rejection errors.
```

Until this is done, the note stays at the top of HANDOFF.

## Codex's sidecar sequence — DONE

All five items from `repos/engine/docs/claude-sidecar-review-brief.md` are merged to sidecar develop:

| # | Item | PR | Wire surface added |
|---|---|---|---|
| 1 | hello / feature-negotiation envelope | #23 | `hello` envelope on every WS subscribe (replayed); `protocol_version=1.0.0`, `sidecar_version`, `features[]`, `mode` |
| 2a | First-class recording | #21 | `--record <path>` CLI flag (in-process JSONL recorder) |
| 2b | Semantic annotations | #22 | `annotate` command + `rider_annotation` envelope; hybrid schema with `client_time_s` + `context` |
| 3 | Pattern B structured pause | #24 | `pause` / `resume` commands + `paused` / `resumed` envelopes; `CadenceBailout` gained a second coexisting state machine |
| 4a | Distance deriver | #25 | FTMS `meters_total` → `distance` events with `source="trainer"`; `BleSource` per-connection delta tracking |
| 4b | FIT export | #25 | `roguerglike-export fit <jsonl> <fit>` CLI; `sport=CYCLING, sub_sport=INDOOR_CYCLING` |
| 5 | SIM mode | #26 | `set_simulation` command + `simulation_set` envelope; FTMS opcode `0x11` write path |

**Hello envelope features list as of today:** `set_target_power`, `recording`, `annotations`, `structured_pause`, `distance`, `activity_export`, `indoor_bike_simulation`. Clients gate UI on this list combined with `device_capabilities` for hardware-specific gates.

## What's next

1. **Strava / TrainingPeaks FIT upload verification** — see top of HANDOFF.
2. **Live-ride validation of items #3–#5** against the KICKR — code is correct against the FTMS spec; real-trainer firmware quirks are best caught with a real ride.
3. **Codex's gizzERG work picks up the next phase** (per `repos/engine/docs/concert-mode-exploration.md` + `repos/engine/docs/music-intensity-proposal.md`):
   - Terrain Mode UI prototype (ERG Terrain skin first; client-side distance/elevation)
   - Scrub-mode profile editor (out-of-ride authoring)
   - Python preprocessing tool (`tools/profile-builder/` — yt-dlp + librosa → derived intensity curve)
   - Blended Terrain Model (Codex's in-flight proposal — see uncommitted notes in `repos/concert-mvp/HANDOFF.md` and `repos/engine/docs/music-intensity-proposal.md` Piece 3b)
4. **Sidecar return-to items** when the gizzERG side asks:
   - Pattern A vs Pattern B mid-ride switching (sidecar contract handles both; only client UX work needed)
   - SIM-mode KICKR-specific firmware quirks (if real-trainer write surfaces issues)
   - `set_terrain_profile` command (when gizzERG terrain UI lands and wants sidecar to own synthetic distance computation per `[[terrain-distance-ownership]]`)

## Codex's uncommitted notes (2026-05-24 night)

Codex left two documentation-only edits **uncommitted** at end-of-day, asking for Claude review first:

- **`repos/concert-mvp/HANDOFF.md`** — new "Blended Terrain Model Proposal" section. UI selector (`Authored cues` / `Derived intensity` / `Blended`), blend slider, sample-step control. Profile shape gains `derived_intensity_curve` (with `model_version` + `sample_step_s`) + `terrain_overrides` (typed events: `cap`, `floor`, `anchor`, `event:crescendo`, `event:drop`, `event:song-boundary`, `manual-override`).
- **`repos/engine/docs/music-intensity-proposal.md`** — new "Piece 3b — Blended Terrain Model" section. Same content as the gizzERG HANDOFF, calibrated to the engine planning doc. Closes with a 5-step implementation plan for Codex, including "Keep sidecar out of this until route profiles are being sent for recorded distance/elevation authority" — explicit handoff signal.

**Claude commentary, for tomorrow's session:**

- Architecture is right. Blend = derived signal modified by authored overrides (not arithmetic averaging) is the correct framing.
- `model_version` on the derived curve picks up the long-term hook flagged in PR #13's crowdsourcing section — good.
- Override events overlap with F2 annotation vocabulary (`crescendo`, `song-boundary`, etc.). **Should converge to one vocabulary** so post-ride analyzers can join F2 annotations against authored overrides. Worth a 5-minute sync next session.
- `event: drop intensity: 1.05` (>1.0 on normalized scale): clarify whether this is "over-FTP burst" (intentional) or "over-normalized" (engine should clamp).
- The HANDOFF section and engine doc section are near-mirrors. Risk of drift — recommend either consolidating (engine doc is canonical; gizzERG HANDOFF summarizes + links) or actively keeping in sync on every edit.
- Step 5 ("Keep sidecar out until route profiles are sent") respects ownership split. Claude can pick up the `set_terrain_profile` work when Codex says ready.

## Open threads (cross-cutting)

- **Engine PR #4** (Godot HIIT MVP) stays open as side experiment per Codex's direction. Not the near-term focus.
- **Strava / TP FIT upload** — see top.
- **Codex's `docs/concert-sidecar-planning` branch in engine** still has unmerged terrain docs (`concert-mode-exploration.md`, `concert-product-roadmap.md`, `training-platform-export-research.md`). Codex's call when to ship.
- **Codex's gizzERG terrain-UI WIP** stashed locally as `codex-terrain-ui-wip-2026-05-23` — recoverable via `git stash pop` when they return.
- **Three legacy repos rename** (`roguERGlike-*` → `*`) — deferred to a coordinated sweep at a future clean break.

## Repo state

| Path | Friendly name | GitHub repo | Branch / state |
|---|---|---|---|
| umbrella | ERGlike umbrella | roguERGlike | this session log branch in flight |
| `repos/sidecar/` | sidecar | roguERGlike-sidecar | `develop` clean; **all 5 sequence items merged** (#21, #22, #23, #24, #25, #26) |
| `repos/engine/` | engine framework | roguERGlike-engine | `develop` clean; PR #13 (music intensity proposal, **awaiting Codex review of Blended section**); PR #4 (MVP loop) parked; `docs/concert-sidecar-planning` has unmerged terrain docs |
| `repos/concert-mvp/` | **gizzERG** | paulyworld/gizzERG | `develop` clean; uncommitted Blended-terrain HANDOFF edit |
| `repos/engine-mvp/` | engine MVP worktree | (worktree of engine) | tied to engine PR #4 |
| `repos/game/` | game | roguERGlike-game | `develop`, no source yet |
| `repos/server/` | server | *(none — deferred)* | placeholder |

## Entry point for next session

> "Sidecar sequence complete (all 5 items in Codex's brief merged: hello, recording, annotations, structured pause, distance + FIT export, SIM mode). 169 sidecar tests, ruff + mypy clean. **First priority: Strava/TrainingPeaks FIT upload verification** (drag `roguerglike-export fit ...` output into both platforms; reportable acceptance criterion). Second: live-ride validation of pause/SIM against KICKR. Codex picks up gizzERG side — Terrain Mode UI, scrub editor, audio preprocessor, and a new Blended terrain model proposal (uncommitted notes in their HANDOFF + engine PR #13). Claude commentary on the Blended proposal is in the umbrella HANDOFF; the converge-on-one-vocabulary point is worth raising in the next sync."
