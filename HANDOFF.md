# HANDOFF — ERGlike (umbrella)

> The current state of the project across all repos. Updated at the end of every session. Read this first.

**Last updated:** 2026-05-25
**Last session log:** `docs/sessions/2026-05-24-sidecar-sequence-complete.md`
**Current branch:** umbrella `docs/vocabulary-and-drift-fix-merged`
**Current focus:** Codex's sidecar sequence fully merged. Today's session resolved the three follow-up items from yesterday: canonical vocabulary doc landed (engine PR #14 → develop), gizzERG HANDOFF drift fix landed (gizzERG PR #2 → develop), and the intensity-scale question is pinned in the vocabulary doc with industry-standard Coggan zones. Strava verification still pending. Codex's `set_terrain_profile` work expected on the gizzERG side.

## ⚠️ Return-to: Strava / TrainingPeaks FIT upload verification

The FIT export (sidecar PR #25) is **code-complete and parser-validated**, but the acceptance criterion from the brief — "manual FIT upload works in Strava / manual FIT upload works in TrainingPeaks" — needs the rider to actually upload one. Quick rep:

```powershell
cd C:\dev\roguERGlike\repos\sidecar
roguerglike-export fit docs/recordings/20260523_121056.jsonl ride.fit
# → drag ride.fit into Strava (https://www.strava.com/upload/select)
#   and TrainingPeaks. Report back any rejection errors.
```

Until this is done, the note stays at the top of HANDOFF.

## Canonical vocabulary

**`repos/engine/docs/vocabulary.md`** is now the single source of truth for terms shared across sidecar schema, gizzERG HANDOFF, engine planning docs, and the F2 annotation overlay. Three parts:

1. **Rider-facing** — F2 tags with plain-language "I want to say X" lookup
2. **Backend** — intensity scale (Coggan zones / %FTP), annotation context fields, mode names, hardware sources, distance/elevation sources, hello features
3. **Profile authoring** — Codex's terrain vocab (override types, events, themes)

**Editorial rule:** if a term in another doc disagrees with the vocabulary doc, the vocabulary doc wins until updated. Future sessions: edit there once, reference from elsewhere.

**Intensity scale settled:** `intensity = target_watts / rider_ftp`, schema range `[0, 2.0]` (Z1 active recovery through Z7 neuromuscular sprints). Sidecar's `--max-target-power` flag is the separate hardware-safety clamp at the BLE write layer. Codex's `event:drop intensity:1.05` is intentional Z4 (top of lactate threshold).

## ⚠️ gizzERG / Codex local divergence (pre-existing)

Codex has **three unpushed local commits** on gizzERG `develop` from 2026-05-23 (terrain tooling: derived intensity, terrain timeline, terrain source tooling). My drift-fix PR #2 was deliberately branched from `origin/develop` to avoid bundling Codex's in-flight work — that drift-fix is now on origin/develop, but Codex's local commits will need a rebase against it when they next push gizzERG.

Their local HANDOFF was substantially rewritten in those 3 commits (terrain-dev UI content); my PR #2 just added a 20-line pointer section. The reconcile will likely keep Codex's HANDOFF rewrite + slot in the pointer paragraph somewhere reasonable. **No work lost. Codex's lane to resolve.**

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
3. **Codex picks up `set_terrain_profile`** (item from yesterday's commentary — user assigned to Codex). When that contract lands, Claude implements the sidecar side per the established patterns (typed command, ack envelope, hello feature, mock parity, tests).
4. **Codex's other gizzERG follow-ups** in parallel:
   - Terrain Mode UI prototype (ERG Terrain skin first; client-side distance/elevation)
   - Scrub-mode profile editor (out-of-ride authoring)
   - Python preprocessing tool (`tools/profile-builder/` — yt-dlp + librosa → derived intensity curve)
   - Blended Terrain Model implementation (per engine `music-intensity-proposal.md` Piece 3b + `vocabulary.md` Part 3)
5. **Sidecar return-to items** when the gizzERG side asks:
   - Pattern A vs Pattern B mid-ride switching (sidecar contract handles both; only client UX work needed)
   - SIM-mode KICKR-specific firmware quirks (if real-trainer write surfaces issues)
   - `set_terrain_profile` command (per #3 above)

## 2026-05-24 commentary items — all resolved 2026-05-25

| Flagged 2026-05-24 | Resolution 2026-05-25 |
|---|---|
| F2 annotation vocab + override-event vocab should converge | `repos/engine/docs/vocabulary.md` Parts 1 + 3 (engine PR #14, merged). Same noun names where they refer to the same musical event; different verb-form per layer (riders observe; authors decide). |
| `event: drop intensity: 1.05` — over-FTP burst or clamp? | Over-FTP burst, intentional. Schema range `[0, 2.0]` per Coggan zones. Pinned in vocabulary.md Part 2 with citations to British Cycling, TrainerRoad, Coggan canonical. |
| HANDOFF / engine-doc near-mirror drift risk | Engine doc is canonical (`music-intensity-proposal.md` + `vocabulary.md`). gizzERG HANDOFF reduced to a pointer (gizzERG PR #2, merged). Editorial rule baked into vocabulary.md: "if a term in another doc disagrees with this one, this file wins." |
| `set_terrain_profile` — sidecar pickup when Codex ready | Acknowledged. Awaiting Codex's design. Hooks left in place: `ElevationData` event type exists; `DistanceData.source="synthetic"` already supported. |

Codex's pending engine doc edit (Piece 3b Blended Terrain Model in `music-intensity-proposal.md`, branch `docs/music-intensity-proposal`) is still uncommitted — that's Codex's commit to make. The vocabulary doc anchors the terms they'll use when they commit.

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
| `repos/engine/` | engine framework | roguERGlike-engine | `develop` clean; PR #14 (vocabulary) merged today; PR #13 (music intensity proposal) open with Codex's uncommitted Blended section pending; PR #4 (MVP loop) parked; `docs/concert-sidecar-planning` has unmerged terrain docs |
| `repos/concert-mvp/` | **gizzERG** | paulyworld/gizzERG | origin `develop` got drift-fix PR #2 today; local `develop` is 3 commits ahead (Codex's unpushed terrain tooling) — rebase needed on next push |
| `repos/engine-mvp/` | engine MVP worktree | (worktree of engine) | tied to engine PR #4 |
| `repos/game/` | game | roguERGlike-game | `develop`, no source yet |
| `repos/server/` | server | *(none — deferred)* | placeholder |

## Entry point for next session

> "Sidecar sequence + vocabulary doc + drift-fix all landed. **First priority: Strava/TrainingPeaks FIT upload verification** (still pending; quick rep at top of HANDOFF). Second: live-ride validation of pause/SIM against KICKR. Canonical vocabulary now lives at `repos/engine/docs/vocabulary.md` — settles intensity scale (`[0, 2.0]` per Coggan zones), F2 tags, mode names, override events. Codex's lane: `set_terrain_profile` design, Blended Terrain Model implementation, scrub editor, audio preprocessing. gizzERG local develop is 3 commits ahead of origin (Codex's unpushed terrain tooling) — they'll need to rebase against PR #2's HANDOFF pointer addition when they next push."
