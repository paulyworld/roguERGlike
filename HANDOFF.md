# HANDOFF — ERGlike (umbrella)

> The current state of the project across all repos. Updated at the end of every session. Read this first.

**Last updated:** 2026-05-25 (live-validation session)
**Last session log:** `docs/sessions/2026-05-24-sidecar-sequence-complete.md`
**Current branch:** umbrella `docs/2026-05-24-live-validations-closed`
**Current focus:** Three big acceptance items closed against real hardware today. Live-ride smoke validated Pattern B structured pause + SIM mode end-to-end on the KICKR (textbook-perfect timeline, no anomalies). Strava manual upload confirmed working from sidecar-exported FIT. FIT exporter pre-emptively gained `elevation` handling so when gizzERG starts pushing synthetic elevation samples, they flow to Strava/TP without further sidecar work. Plus a new hands-free smoke runner (`run-smoke-pause-sim.ps1`) so future protocol regressions are scriptable.

## Live-validation status

| Acceptance item | Status |
|---|---|
| FIT export — Strava manual upload | ✅ Confirmed. Test FIT (from pre-distance ride) uploaded cleanly. |
| FIT export — TrainingPeaks manual upload | ⏳ Pending (low priority — Strava is the primary platform; TP can wait until a richer test ride exists) |
| Pattern B structured pause — live KICKR | ✅ Smoke run validated all five acceptance criteria: deferred-paused rejection, ramp to deferred target (not pre-pause), no spurious cadence bailout during structured pause, clean envelope timing, idempotency |
| SIM mode — live KICKR | ✅ Smoke run validated 5%/9%/-3% grade transitions + switch-back-to-ERG. KICKR CORE 1003 firmware does advertise `indoor_bike_simulation` — confirmed. |
| FIT export — richer ride (distance + elevation populated) | ⏳ Awaiting either (a) a full ERG ride on the KICKR with FTMS distance flowing, or (b) gizzERG Terrain Mode ride with synthetic elevation. Sidecar already handles both; just needs a real recording. |

## Strava Relative Effort note

Strava computes Relative Effort (RE) server-side from HR-zone time-in-zone. **Not an input** — can't be set in the FIT. As long as your sidecar recording has continuous HR samples (Whoop or chest strap), RE will compute meaningfully when you upload. TrainingPeaks uses TSS instead; same FIT file, different lens. Documented in `repos/engine/docs/vocabulary.md` Part 2 if needed for future reference.

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

1. **Richer FIT export test** — do a real ERG ride on the KICKR with `--record`. Sidecar's distance deriver auto-populates from FTMS `meters_total`; FIT exporter already handles it. Upload to Strava and confirm distance + RE both look reasonable.
2. **Codex picks up `set_terrain_profile`** (assigned 2026-05-25). When the contract design lands, Claude implements the sidecar side per the established patterns.
3. **Codex's other gizzERG follow-ups** in parallel:
   - Terrain Mode UI prototype (ERG Terrain skin first; client-side distance/elevation)
   - Scrub-mode profile editor (out-of-ride authoring)
   - Python preprocessing tool (`tools/profile-builder/` — yt-dlp + librosa → derived intensity curve)
   - Blended Terrain Model implementation (per engine `music-intensity-proposal.md` Piece 3b + `vocabulary.md` Part 3)
4. **Strava auto-upload** (optional follow-up; rider-asked). ~3-4 hours of sidecar work — OAuth flow, refresh token storage, `POST /api/v3/uploads` + async polling, new `--auto-upload-strava` flag. Scope when convenient; manual upload works fine in the meantime.
5. **Sidecar return-to items** when the gizzERG side asks:
   - Pattern A vs Pattern B mid-ride switching (sidecar contract handles both)
   - `set_terrain_profile` command (per #2 above)
   - SIM-mode firmware quirks for trainers other than KICKR CORE 1003 (only matters when other riders join)

## Today's session — live validations + ergonomics

| Work | What |
|---|---|
| Live smoke runner | New `repos/sidecar/scripts/run-smoke-pause-sim.ps1` — hands-free ~100-second auto-paced sequence covering ERG baseline → pause/resume (with deferred-target proof) → SIM 5%/9%/-3% → back to ERG. Rider just pedals; no copy-paste. Future protocol regressions are now scriptable. |
| Live-ride validation | Smoke run executed on real KICKR; recording at `repos/sidecar/docs/recordings/20260524_194504.jsonl`. Textbook-perfect timeline (analyzed in chat); all PR #24 + PR #26 acceptance criteria validated. |
| Strava manual upload | Test FIT uploaded successfully. Sport=Indoor Cycling shown correctly. (Test FIT was pre-distance/elevation so a richer follow-up ride is queued.) |
| FIT exporter elevation handling | Pre-emptive: sidecar PR #31 closes the gap so when gizzERG starts pushing synthetic elevation samples, they flow into Strava/TP exports automatically. 170/170 tests. |
| send-cmd helper bug fixes | Two PowerShell native-command quoting bugs (heredoc + argv) — final fix uses stdin to bypass PS quoting entirely. |

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

> "Three big acceptance items closed against real hardware today: Pattern B structured pause + SIM mode live-validated on KICKR via the new `run-smoke-pause-sim.ps1` hands-free smoke runner (textbook-perfect), Strava manual FIT upload confirmed. Sidecar at 170/170 tests; FIT exporter pre-emptively handles `elevation` for when gizzERG starts pushing synthetic terrain. Next: richer FIT test on a real ride (distance auto-flows; just record + export + upload), Codex's `set_terrain_profile` design when ready, and optional Strava auto-upload (~3-4 hours of OAuth work when convenient). Canonical vocabulary lives at `repos/engine/docs/vocabulary.md`. gizzERG local develop divergence still pending Codex's rebase."
