# Session: 2026-05-21 — Hardware validation against KICKR + Phase 2 merges

**Repo(s):** sidecar + engine + umbrella
**Interface:** Claude Code
**Branch(es):** sidecar `feat/ble-ftms` (merged) → develop; engine `feat/handshake-test-scene` + `docs/handoff-refresh` (merged); umbrella `docs/handoff-bike-integration-handshake` (merged)
**Duration:** ~1 hour (follows 2026-05-20-phase-2-ble-ftms.md the previous evening)

## Context loaded

- Umbrella HANDOFF (post-Phase-2-merge state) — entry point was "pair the KICKR and live-validate"
- Sidecar `feat/ble-ftms` branch — Phase 2 code from previous session
- Memory `device-pairing-ux-model` — Zwift-style separate pairing model
- KICKR CORE 1003 — physically present, untouched by Windows pairing

## Decisions made

- **Skip Windows OS pairing for the KICKR.** BLE GATT (FTMS) doesn't require Windows-level bonding; Bleak connects to unpaired BLE devices directly. Windows pairing for BLE-only fitness devices is actually known to be flaky and was the symptom that made the user think pairing was needed. Saved as part of [[bleak-windows-scan-filter-bug]] context.
- **Scan unfiltered, post-filter in Python.** Bleak's `BleakScanner.discover(service_uuids=...)` filter on Windows silently drops devices that advertise the target service UUID in their **scan response** rather than the initial advertisement packet. KICKRs do exactly this. Saved as memory `bleak-windows-scan-filter-bug` so we don't rebuild this when adding HRS / CSCS.
- **Close engine PR #3** — opened against `main` by mistake (likely from the GitHub UI). Project conventions require all PRs target `develop`; closing is the right call. Branch and commits are untouched; draft PR #4 against develop still tracks the same work.
- **Squash-merge with explicit subjects** including the PR number suffix — matches the existing develop history format (`feat(sidecar): … (#4)`).
- **Don't auto-delete branches on merge.** Conservative default; user can prune later if they want a tidier branch list.

## Work done

### Sidecar — `feat/ble-ftms` extended
- Diagnosed empty `--scan` output: unfiltered Bleak scan found the KICKR (`F9:18:EE:27:40:02`, RSSI -73) advertising both `0x1818` (Cycling Power) and `0x1826` (FTMS), but our filtered scan returned nothing.
- Fix in `src/roguerglike_sidecar/ble/scan.py`: scan with `BleakScanner.discover(timeout, return_adv=True)` (no `service_uuids=`), then post-filter on `adv.service_uuids` in Python. One extra dict iteration; reliable on Windows.
- Verified: `roguerglike-sidecar --scan` now lists "KICKR CORE 1003".
- 30/30 tests still pass; ruff + mypy clean.
- Commit: `fix(sidecar): scan unfiltered then post-filter by service UUID` (signed `c73d949`), pushed to PR #4 before merge.

### End-to-end live validation
With sidecar in live mode + engine `test_main` running, real KICKR data flowed:
```
[test_main] connection_state_changed connected=true
[test_main] device_connected kind=bike_trainer name=KICKR CORE 1003
[test_main] cadence_changed rpm=47, 47, 47, 48, 48, 48, 48
[test_main] power_changed   watts=35, 34, 39, 44, 37, 37, 37, 18
```
KICKR correctly omits speed (no roller-speed sensor) and HR (no strap paired) — matches FTMS Indoor Bike Data packet flags. Pipeline proven end-to-end: Bleak → FTMS decoder → EventBus → WS → effort_bridge.gd → typed signals.

### Merges
- **sidecar #4** (Phase 2 BLE) → develop, squashed `42ba9f9`
- **sidecar #5** (HANDOFF refresh) → develop, squashed `1924af4`
- **engine #1** (handshake test scene) → develop, squashed `83eea53`
- **engine #2** (engine HANDOFF refresh) → develop, squashed `c3755d3`
- **umbrella #4** (cross-repo HANDOFF + both 2026-05-20 session logs) → develop, squashed `fd1d848`
- **engine #3** (wrong base) → CLOSED with explanation; branch and commits untouched

### Memory captured
- `bleak-windows-scan-filter-bug` — `BleakScanner.discover(service_uuids=)` is unreliable on Windows; always scan unfiltered and post-filter in Python.

### Umbrella — `docs/post-phase-2-merge`
- This session log.
- HANDOFF refresh.

## Open threads

- **Engine PR #4** (draft, MVP HIIT playable loop on `feat/mvp-playable-loop`) — leave as draft until the user is ready to land it.
- **Stashed engine WIP** — `git stash list` in `repos/engine/` shows "mvp-playable-loop WIP — saved before connection-test branch switch 2026-05-21". Recoverable via `git stash pop` when returning to that branch.
- **HRS profile** (`feat/ble-hrs`) — next BLE work. Same shape as FTMS: one profile module + decoder + tests + a `--device-hr` flag. Apply the [[bleak-windows-scan-filter-bug]] pattern from the start.
- **Distance deriver** — small follow-up so the FTMS decoder can emit `DistanceData`. State-tracking (last_total → delta) belongs outside the pure decoder.
- **CI status across PRs** — `statusCheckRollup` was empty on all merged PRs; matches the [[github-actions-first-push-quirk]] memory. Next push should fire CI properly.
- **Slider UI / pairing UI** — long-term, the eventual Zwift-style pairing screen ([[device-pairing-ux-model]]). Bootstrap UX is the current CLI flags.

## Next session entry point

> "Pick a direction: (a) HRS profile on `feat/ble-hrs` for chest-strap HR + HR dedup policy, (b) distance deriver on `feat/distance-deriver` so FTMS emits DistanceData, or (c) restore the stashed MVP WIP (`git stash list` in repos/engine/, `git stash pop`) and continue the HIIT playable loop with real bike data now wired in."

## Loose notes

- KICKR CORE LED reference: **blinking blue** = advertising / no connection; **solid blue** = one active BLE connection. The user's "blue light still blinking" while Godot showed no data was the diagnostic that surfaced "sidecar not running" as the cause.
- Godot 4 keys: **F5** = run main scene (project's configured `run/main_scene`); **F6** = run currently-open scene. F5 occasionally silently no-ops; F6 or the ▶ Play button is more reliable.
- `gh pr merge <num> --squash --subject "title (#N)"` matches the existing history format that GitHub auto-generates. Don't pass `--delete-branch` unless you really mean it.
- The user opened PR #3 against `main` via the GitHub UI by accident; lesson is the GitHub web UI defaults base to `main` unless you explicitly switch. Prefer `gh pr create --base develop --head <branch>` from the CLI to avoid this.
- Background processes holding port 8421/8422 are still a recurring footgun. `Get-NetTCPConnection -LocalPort 8421 -State Listen` + `Stop-Process -Id <pid> -Force` is the pattern.
