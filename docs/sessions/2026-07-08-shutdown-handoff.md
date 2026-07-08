# Session: 2026-07-08 — shutdown handoff, no code changes

**Repo(s):** umbrella + read-through of concert-mvp, sidecar, engine
**Interface:** Claude Code
**Branch(es):** umbrella `docs/2026-06-06-full-concert-tuning-controls`
**Purpose:** Rider is shutting down for a break. Produce a clean handoff that captures both the merged annotation work (2026-06-28) and the uncommitted importer/background-training WIP (2026-07-01) so a future session can resume cold.

## Repo snapshot at shutdown

| Repo | Branch | HEAD | Working tree |
|---|---|---|---|
| umbrella | `docs/2026-06-06-full-concert-tuning-controls` | (this branch, session log-only) | HANDOFF.md updated this session |
| `repos/concert-mvp` (gizzERG) | `develop` | `8a07119` fix(annotations): shift-drag chart to mark range, then F2 (#13) | **WIP uncommitted** — importer overlay + background-training + auto-load trained curve. See details below. |
| `repos/sidecar` | `develop` | `12648d7` docs: document annotation range context (#32) | **WIP uncommitted** — `profile_store.py`, `training_runner.py`, background-run wiring in `ws_server`, events, CLI `--profile-builder`, tests. See details below. |
| `repos/engine` | `develop` | `f064798` docs: update music intensity feature model | Clean |
| `repos/game`, `repos/server` | — | — | Clean / no source |

## Merged since the 2026-06-27 reconcile

**gizzERG `develop`** (`9e57784` → `8a07119`):

- `#8` docs: update handoff after develop reconcile
- `#11` feat: add range annotations (Codex)
- `#9` fix: prevent F2 preset auto-submit — closes issue `#4`
- `#10` feat: show annotations in chart tooltip
- `#13` fix(annotations): shift-drag chart to mark range, then F2 — closes issue `#12`

Tests: **78/78** on develop.

**sidecar `develop`** (`3c49c39` → `12648d7`):

- `#32` docs: document annotation range context (opaque pass-through fields; no schema change)

Open PRs: **none** on either repo. Open issues: **none** on gizzERG.

## Uncommitted WIP (both repos) — do not lose

The 2026-07-01 session did not commit; both working trees still hold the changes. Session log: `docs/sessions/2026-07-01-importer-overlay-background-training.md`. This is the resume point.

### concert-mvp working-tree WIP

```
 M HANDOFF.md
 M index.html
 M src/app.js
 M styles.css
?? src/link-importer.js
?? tests/link-importer.test.mjs
```

Diff size ~+863 lines across 4 tracked files. Composition:

- `index.html` — `Import…` trigger by the Video select; `#videoImporterOverlay` modal (grouped In-browser vs Sidecar actions, per-button `title`, step hint, `#importerStatus` banner, labeled result area). Cache-bust `?v=profile-store-3` on both `styles.css` and `src/app.js`.
- `styles.css` — overlay/card/trigger + `.importer-status` color-coded states (idle / running-pulse / ok / error).
- `src/app.js` — overlay open/close (Esc/backdrop/✕); `log(text, level)` gains `ok`/`warn`/`bad` class; `handleProfileStoreEvent` rewritten with a `PROFILE_STORE_ACTIONS` map + `renderImporterStatus`; `sendSidecarCommand` outbound ack; `registerTrainedCurve()` + `safeProfileId()` auto-load the completed curve into the Curve dropdown and select it when the profile is on screen.
- `src/link-importer.js` + `tests/link-importer.test.mjs` — untracked new module. Tests count **84/84** with these included.

The PR #13 shift-drag/range-attach code (`chartAnnotationDrag`, `overlayAttachedRange`, live drag box, Shift+drag) is **still present** in the WIP — verified 24 hits in both HEAD and working tree.

### sidecar working-tree WIP

```
 M .gitignore
 M HANDOFF.md
 M docs/event-schema.md
 M src/roguerglike_sidecar/cli.py
 M src/roguerglike_sidecar/events.py
 M src/roguerglike_sidecar/session.py
 M src/roguerglike_sidecar/ws_server.py
 M tests/test_events.py
 M tests/test_ws_server.py
?? src/roguerglike_sidecar/profile_store.py
?? src/roguerglike_sidecar/training_runner.py
?? tests/test_profile_store.py
?? tests/test_training_runner.py
```

Diff size ~+603 lines across 9 tracked files, plus 4 new files. Composition (from the 2026-07-01 session log):

- New `training_runner.py` — `execute_training_run()` spawns the builder child process, splits output on `\r`/`\n` for `tqdm` progress, mirrors to CLI log + typed events, updates `input.json` status queued→running→completed/failed. Injectable spawner for tests.
- New `profile_store.py` — `ProfileStore.default(profile_builder_path=...)` precedence flag → env (`GIZZERG_PROFILE_BUILDER`) → relative default.
- `events.py` — added `training_run_started` / `training_run_progress` / `training_run_completed` / `training_run_failed` to `ProfileStoreData.action`.
- `ws_server.py` — `_spawn_training_run` launches a tracked background task after acking `training_run_created`; all profile-store commands `log.info` on success.
- `cli.py` — new `--profile-builder <path>` option threaded into mock + live modes. Precedence over env.
- Tests: `test_training_runner.py` (5 cases via fake spawner) + `test_profile_store.py` + ws-level background-run test + 2 `ProfileStore.default` precedence tests. **186 pass, `ruff check .` clean.**

## Not live-validated

The 2026-07-01 WIP has never been end-to-end validated with a real YouTube download. First thing to watch on a real run: the `training_run_completed` event's `profile_id` matching the loaded profile's `video_id` (raw + `safeProfileId` compare handles safe-id rewrites, but eyeball it once).

## Cold-start resume checklist

If a future session wants to ship the WIP:

1. `git -C repos/concert-mvp status` and `git -C repos/sidecar status` — confirm the WIP files listed above still exist. If they don't, check `git stash list` and `git reflog`.
2. Run tests as a baseline: sidecar `pytest && ruff check .`; concert-mvp `node --test tests/*.test.mjs`. Should hit 186 and 84 respectively.
3. Live-validate:
   ```
   cd repos/sidecar
   $env:PYTHONPATH="src"
   python -m roguerglike_sidecar.cli --mode mock --allow-trainer-control `
     --profile-builder C:\dev\roguERGlike\repos\concert-mvp\tools\profile-builder\build_profile.py
   ```
   In a second terminal: `python -m http.server 8430 --bind 127.0.0.1` from `repos/concert-mvp`. Open `http://127.0.0.1:8430`. Click `Import…`, paste a short YouTube link, click `Queue training`. Watch the sidecar CLI stream `[train <id>]` lines and the web `#importerStatus` banner pulse. On completion the trained curve should appear in the Curve dropdown and select automatically.
4. If validation passes, branch each repo's WIP off `develop` and open PRs per project convention (small, focused, `git commit -S`).

If a future session wants to skip the WIP and start fresh from `develop`, the 2026-07-01 changes are recoverable via `git stash push -u` before switching branches. Nothing is at risk of loss as long as the working trees are not blown away with `git clean -fd`.

## Open threads (cross-cutting)

Carried from prior HANDOFF; nothing new to add today:

- **Live-validation debt** — richer FIT export with distance + elevation still needs a real KICKR ERG ride.
- **Codex's `docs/concert-sidecar-planning`** engine branch — unmerged terrain docs.
- **Codex's `codex-terrain-ui-wip-2026-05-23`** stash on gizzERG — recoverable via `git stash pop`.
- **Three legacy repo renames** (`roguERGlike-*` → `*`) — deferred.

## Next session entry point

> "gizzERG `develop` is at `8a07119` and stable — all annotation work landed 2026-06-28. Sidecar `develop` at `12648d7`. **Uncommitted WIP on both concert-mvp and sidecar** from the 2026-07-01 importer + background-training session — read `docs/sessions/2026-07-01-importer-overlay-background-training.md` and the 2026-07-08 shutdown log for the exact file inventory and the live-validation resume steps. Once validated, branch each repo's WIP off `develop` and open PRs. Everything else is stable; no unresolved issues on gizzERG."
