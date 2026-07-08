# Session: 2026-07-01 — gizzERG importer overlay + background training with CLI/web readout

**Repo(s):** mixed (concert-mvp + sidecar)
**Interface:** Claude Code
**Branch(es):** working-tree WIP on both repos (uncommitted; concert-mvp `develop`, sidecar profile-store WIP)
**Duration:** ~1 session

## Context loaded

- Umbrella + concert-mvp + sidecar `HANDOFF.md`
- The uncommitted profile-store feature already in both working trees (a parallel session's work): sidecar `profile_store.py` + `save_profile`/`list_profiles`/`create_training_run`/`promote_curve` commands and `profile_store` events; concert-mvp importer buttons wired to them.
- Human report: importer button unusable (couldn't expand in the crowded sidebar); later, that sidecar actions produced "no readout" in either the CLI or the web app when training in the background.

## Decisions made

- **Importer becomes a top-row overlay**, not an inline `<details>` panel. Root cause of "won't expand": `.control-panel` is a fixed-height grid with `overflow: hidden`, so the squeezed `<details>` couldn't grow. Reused the existing annotation-overlay pattern.
- **Background training model chosen (user decision):** the sidecar should *actually run* the queued profile-builder in the background and stream progress — not stay a queue-only record. Root cause of "no readout": `create_training_run` only wrote a queued `input.json`; `run_training_command` was never called, so nothing ran and nothing logged.
- Progress surfacing split by frequency: milestones → always-visible event log; high-frequency `tqdm`/stdout lines → status banner + preview only, to avoid flooding.
- `--profile-builder` added as a discoverable CLI flag (precedence over the `GIZZERG_PROFILE_BUILDER` env var) rather than env-only.

## Work done

**Sidecar (`repos/sidecar`)**
- New `src/roguerglike_sidecar/training_runner.py` — `execute_training_run()` spawns the builder as a child process, splits output on `\r`/`\n` for live `tqdm` progress, mirrors lines to the CLI log and to typed events, updates `input.json` status queued→running→completed/failed. Injectable spawner for tests.
- `events.py` — added `training_run_started` / `training_run_progress` / `training_run_completed` / `training_run_failed` to `ProfileStoreData.action`.
- `ws_server.py` — `_spawn_training_run` launches a tracked background task after acking `training_run_created`; added `log.info` success logging to all profile-store commands.
- `profile_store.py` — `ProfileStore.default(profile_builder_path=...)` precedence (flag → env → relative default).
- `cli.py` — `--profile-builder <path>` option threaded into mock + live modes.
- Tests: `test_training_runner.py` (5 cases via fake spawner) + ws-level background-run test + 2 `ProfileStore.default` precedence tests. **186 pass, `ruff check .` clean.**

**Web app (`repos/concert-mvp`)**
- `index.html` — `Import…` trigger by the Video select; importer content moved into `#videoImporterOverlay` modal; grouped actions (In browser / Sidecar) with per-button `title` tooltips, step hint, labeled result area, `#importerStatus` banner.
- `styles.css` — overlay/card/trigger styles + color-coded `.importer-status` (idle/running-pulse/ok/error).
- `src/app.js` — overlay open/close (Esc/backdrop/✕); `log(text, level)` gains ok/warn/bad class; `handleProfileStoreEvent` rewritten with `PROFILE_STORE_ACTIONS` map + `renderImporterStatus`; `sendSidecarCommand` shows an outbound ack; `registerTrainedCurve()` + `safeProfileId()` auto-load the completed curve into the Curve dropdown and select it when the profile is on screen.
- Cache-bust `styles.css`/`app.js` → `?v=profile-store-3`. **84/84 tests pass**, `node --check src/app.js` clean, no duplicate IDs.

- No commits made — all work is uncommitted working-tree changes in both repos.

## Open threads

- **Not live-validated end-to-end.** Needs a running sidecar (`--profile-builder <abs path>`, deps yt-dlp + librosa + ffmpeg) plus a real YouTube download. First thing to watch: the completed event's `profile_id` matching the loaded profile's `video_id` (raw + `safeProfileId` compare handles safe-id rewrites, but eyeball it once).
- Sidecar spawns the builder with the builder's own `python`; ensure the interpreter that resolves has the builder deps.
- Everything is uncommitted on both repos — needs branching + PRs per project convention (branch off `develop`).
- Pre-existing lint nits in the parallel session's `profile_store.py` were fixed as part of getting `ruff check .` green.

## Next session entry point

> "Live-validate the full loop: start the sidecar with `--profile-builder` pointed at `repos/concert-mvp/tools/profile-builder/build_profile.py`, open gizzERG, Import… a short video, Queue training, and confirm CLI `[train <id>]` lines + the web status banner stream and the trained curve auto-selects. Then branch the uncommitted work off `develop` in each repo and open PRs."

## Loose notes

- The importer "buttons do nothing" symptom was two things: (1) sidecar training genuinely did nothing (queue-only), and (2) all feedback was buried in a small unlabeled `<pre>`. The overlay + banner + real execution fix both.
- The `training_run_progress` throttle is 0.5s on the sidecar side (`_TRAINING_PROGRESS_INTERVAL_S`); the runner still logs every line to the CLI.
