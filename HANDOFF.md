# HANDOFF — ERGlike (umbrella)

> The current state of the project across all repos. Updated at the end of every session. Read this first.

**Last updated:** 2026-07-08 (shutdown handoff — no code, docs reconcile only)
**Last session log:** `docs/sessions/2026-07-08-shutdown-handoff.md`
**Current branch:** umbrella `docs/2026-06-06-full-concert-tuning-controls`
**Current focus:** Project is paused. Two active states to be aware of when resuming:
1. **Merged (2026-06-28):** all annotation-side gizzERG PRs — `#8`, `#9`, `#10`, `#11`, `#13` — plus sidecar `#32`. gizzERG develop `8a07119` / sidecar develop `12648d7`. Issues `#4` and `#12` closed.
2. **Uncommitted WIP (2026-07-01):** importer overlay + background training + trained-curve auto-load, spanning both concert-mvp and sidecar working trees. **Not live-validated end-to-end.**

The 2026-07-08 session log is the cold-start resume guide. Read it first, then the 2026-07-01 log for the WIP details, then the 2026-06-28 log for the annotation-merge context.

## 2026-06-28 — gizzERG annotation PR triage + shift-drag range flow (merged)

**One-line:** Four annotation PRs shipped to gizzERG `develop` (`#8` docs, `#11` range annotations, `#9` F2 preset auto-submit fix, `#10` annotation tooltip). Sidecar `#32` shipped alongside for the annotation context docs. Then fixed a rider-reported bug on `#11`'s range checkbox by replacing it with a Shift+drag flow (`#13`, closes issue `#12`). gizzERG issue `#4` also closed as resolved by `#9`.

**Wire-format touch:** none. `#32` is docs-only for existing pass-through context fields.

**Validation on merge:** gizzERG `78/78` tests; sidecar unchanged (docs-only PR).

**Details:** `docs/sessions/2026-06-28-annotation-merges-and-range-drag.md`.

## 2026-07-01 — gizzERG importer overlay + background training (WIP, uncommitted, both repos)

**One-line:** The video importer is now a top-row overlay, sidecar actions have a real CLI + web readout, `create_training_run` actually runs the builder in the background and streams progress, and a completed run's curve auto-loads into the Curve dropdown.

**Why:** The importer button couldn't expand in the crowded sidebar (`.control-panel` grid is `overflow: hidden`), and "Queue training" produced no readout anywhere because `create_training_run` was queue-only — it wrote `input.json` and never executed the builder (`run_training_command` was dead code).

| Area | What landed (uncommitted) |
|---|---|
| **Importer overlay** (concert-mvp) | `Import…` trigger by the Video select opens `#videoImporterOverlay` modal (Esc/backdrop/✕). Actions grouped **In browser** (`Parse setlist`, primary `▶ Load into player`, `Copy train command`) vs **Sidecar · needs connection**; per-button `title` tooltips; step hint; labeled result box. |
| **Sidecar-action readout** (concert-mvp) | Color-coded `#importerStatus` banner (idle/running-pulse/ok/error); milestones mirror to the event log (`log(text, level)`); `training_run_progress` updates banner+preview only; outbound `sendSidecarCommand` ack. |
| **Background training** (sidecar) | New `training_runner.py` `execute_training_run()` spawns the builder child process, streams `tqdm`/stdout (splits on `\r`) to the CLI log + new `training_run_started`/`progress`/`completed`/`failed` events; `input.json` status queued→running→completed/failed; `ws_server` runs it as a tracked background task; all profile-store commands now `log.info` on success. |
| **`--profile-builder` flag** (sidecar) | New CLI option (precedence over `GIZZERG_PROFILE_BUILDER` env). Required for training to run; builder needs yt-dlp + librosa + ffmpeg. Without it, runs fail *visibly* (red banner + CLI error). |
| **Auto-load trained curve** (concert-mvp) | `registerTrainedCurve()` on `training_run_completed` matches `profile_id`→loaded profile (raw + `safeProfileId` compare), appends the curve to `available_intensity_curves` as "Trained · &lt;model_version&gt;", and selects+activates it when that profile is on screen. |

**Validation:** sidecar **186 tests pass, `ruff check .` clean**; concert-mvp **84/84 tests**, `node --check src/app.js` clean, cache-bust `?v=profile-store-3`. **Not yet live-validated end-to-end** (needs a running sidecar with `--profile-builder` + a real YouTube download).

**Next:** live-validate the loop, then branch each repo's uncommitted work off `develop` and open PRs. First thing to watch on a real run: the completed event's `profile_id` matching the loaded profile's `video_id`.

## gizzERG 2026-06-06 wrap

| Area | What landed on `feat/full-concert-curves-tuning-controls` |
|---|---|
| Full-concert v0.3/v0.4 | 3974 audio-derived points covering t=833→8779 at 2s sample step. Manual-seed splice removed for t≥833 (was only used outside the prior 20-min sample). Library ids renamed: `audio-v0.3-20m`/`audio-v0.4-subjective-20m` → `audio-v0.3`/`audio-v0.4-subjective`. |
| Per-window BPM | `librosa.feature.tempo(..., aggregate=None)` in `build_profile.py`. Lands in `audio_features.bpm` unnormalized. Chart line + tooltip + guidance prefer per-window; fall back to per-section `cue.bpm` when curve has no audio data. Tooltip labels source explicitly. Range 68-172 BPM, median 112. |
| Manual seed v0.2 | New `bnnIdWzGSYIManualSeedV02Curve` — 26 dense anchor points across t=1607-2036 derived from F2 annotations in `repos/sidecar/docs/recordings/semantic-test-02.jsonl`. Outside that window inherits v0.1. |
| Default style segments | v0.4 `styleSegments` extended to The Balrog (0.40), Iron Lung (0.35, label `heavy`), Evil Death Roll (0.50, label `thrash`), Hog Calling Contest (0.40), alongside the existing Gila/Motor Spirit. Other 10 Night-2 songs still unsegmented (`style_prior = 0`). |
| Tuning UI | Intensity smoothing slider (0-60s symmetric centered MA; drives chart derived overlay + controller target series + BPM line consistently). Authored Cues chart overlay (`cues` toggle, rose-magenta step line) so you can compare authored/derived/blended at a glance. Curve dropdown selection persists per-video via localStorage. Boot-order fix: restored choice now actually loads (was a dropdown-only restore before). |
| F2 fix validation | Codex's note-attach fix on concert-mvp `develop` validated 2026-05-27 via 10-keypress smoke (`repos/sidecar/docs/recordings/f2-smoke-test.jsonl`). 27-event semantic session followed (`semantic-test-02.jsonl`) — produced the data for v0.2. |
| gizzERG issue #4 | F2 overlay digit hotkey auto-submits before note can be typed. Filed at https://github.com/paulyworld/gizzERG/issues/4 with three suggested fixes. Documented in concert-mvp HANDOFF + commit referenced in repos/sidecar/HANDOFF earlier. |

## Open gizzERG tuning threads

1. **Per-song music-end vs authored cue boundary** (`task #7`). Bandcamp track durations don't precisely match where music ends within each song — banter/applause sits in the section tail and the per-section BPM bleeds into it. Three approaches captured: auto-detect from existing audio features (sustained loudness drop / onset_density floor), F2-driven manual song-end markers, or hybrid.
2. **styleSegments coverage** for the other 10 Night-2 songs as F2 tuning reveals which sections want a genre prior.
3. **Browser ES-module cache** keeps catching us after JS regenerations. Hard-refresh works; consider adding `?v=...` to `audio-derived-curves.js` import if regens become frequent.

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

**2026-07-08 snapshot:**

| Path | Friendly name | GitHub repo | Branch / state |
|---|---|---|---|
| umbrella | ERGlike umbrella | roguERGlike | `docs/2026-06-06-full-concert-tuning-controls` in flight (session-log branch) |
| `repos/sidecar/` | sidecar | roguERGlike-sidecar | `develop` at `12648d7` (PR #32 annotation-range docs); **uncommitted WIP** for profile-store + background training (see 2026-07-01 section) |
| `repos/engine/` | engine framework | roguERGlike-engine | `develop` at `f064798`; PR #14 (vocabulary) merged; PR #13 (music intensity proposal) open with Codex's uncommitted Blended section pending; PR #4 (MVP loop) parked; `docs/concert-sidecar-planning` has unmerged terrain docs |
| `repos/concert-mvp/` | **gizzERG** | paulyworld/gizzERG | `develop` at `8a07119` (PRs #8, #11, #9, #10, #13 all merged 2026-06-28); **uncommitted WIP** for importer overlay + trained-curve auto-load (see 2026-07-01 section); zero open PRs, zero open issues |
| `repos/engine-mvp/` | engine MVP worktree | (worktree of engine) | tied to engine PR #4 |
| `repos/game/` | game | roguERGlike-game | `develop`, no source yet |
| `repos/server/` | server | *(none — deferred)* | placeholder |

## Entry point for next session

> "Project paused 2026-07-08. gizzERG `develop` at `8a07119` and sidecar `develop` at `12648d7` are stable — all annotation-side work landed 2026-06-28 (`#8`/`#9`/`#10`/`#11`/`#13`, plus sidecar `#32`), issues `#4` and `#12` closed. **Uncommitted WIP on both concert-mvp and sidecar** from the 2026-07-01 importer + background-training session; the code still exists in the working trees but has never been live-validated end-to-end. Read `docs/sessions/2026-07-08-shutdown-handoff.md` for the cold-start resume checklist (baseline tests → live-validate importer/training loop → branch WIP off develop → PR). Then read `2026-07-01-importer-overlay-background-training.md` for the WIP context and `2026-06-28-annotation-merges-and-range-drag.md` for the merge context. No open PRs; no open gizzERG issues."
