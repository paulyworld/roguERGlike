# Session: 2026-05-21 — cadence-bailout shipped, live validation paused

**Repo(s):** sidecar + engine + umbrella
**Interface:** Claude Code
**Branch(es):** sidecar `feat/cadence-bailout` (merged); engine `feat/bridge-bailout-signals` (merged); umbrella `docs/post-cadence-bailout-paused`
**Duration:** ~2 hours (fourth session of 2026-05-21; follows the ftms-erg-end-to-end session and the open MVP work in engine-mvp)

## Context loaded

- Umbrella + per-repo HANDOFFs from the morning's refresh
- Memories `intensity-aware-safety-curves`, `opt-in-flags-need-feedback`, `trainer-control-aggressive-defaults`, `bleak-windows-scan-filter-bug`, `hr-source-breadth`
- Open engine PR #4 (MVP HIIT loop, Ready for Review)

## Decisions made

- **Cadence bailout fully intensity-aware.** Linear curve over `pct_ftp ∈ [0.5, 1.5]` with anchors `bailout_s ∈ {90, 15}` and `ramp_s ∈ {2, 12}` per project memory `intensity-aware-safety-curves`. Patient + fast-ramp at recovery wattage; fast + slow-ramp at peak. Curve constants in code, not flags (will promote to flags only if real use shows tuning is needed).
- **Bundle silent-rejection fix in same PR as bailout.** When the WS recv loop receives a `set_target_power` without a configured handler (sidecar wasn't started with `--allow-trainer-control`), publish a typed `target_power_set accepted=false reason="trainer-control disabled (no --allow-trainer-control flag)"` envelope instead of dropping silently. Closes the silent-failure footgun captured in memory `opt-in-flags-need-feedback`.
- **Bailout intercepts the command path while paused.** `handle_set_target_power` records the new target as the *intended* restore value but doesn't write to the trainer until cadence resumes. Engine sees the queued ack with `reason="bailout-pending"`.
- **`target_power_set` is intentionally NOT replayed to late subscribers.** It's an ack of a specific command, not ambient state.
- **Branch hygiene during failed merges**: when GitHub fails to auto-retarget a stacked PR (parent merged but not deleted), rebase the child onto develop with `git rebase origin/develop` — git auto-skips the duplicate squashed commit — then re-open the child PR. Avoids the messy "merge again into wrong branch" trap from the prior session.

## Work done

### Sidecar — `feat/cadence-bailout` → PR #14, merged (`fb6543f`)

- `events.py` adds `CadenceBailoutEngagedData` + `CadenceBailoutDisengagedData`, the two new `EventType` literals, and the `_DATA_BY_TYPE` mapping.
- `ble/cadence_bailout.py` (new ~290 lines incl. docstrings):
  - `CadenceBailout` class. Watches cadence events from the bus + runs an idle watcher loop.
  - On low cadence for the dynamic bailout window: publishes `cadence_bailout_engaged`, writes `min_target_power` to the trainer, marks paused.
  - On cadence ≥ 30 rpm while paused: cancels any in-flight ramp, starts a new resume-ramp task that lerps from current target up to `pre_pause_target` over the dynamic ramp duration.
  - `handle_set_target_power` is the front of the command path; queues during pause, passes through otherwise.
  - Pure curve helper `_interp_for_pct_ftp(low, high, pct_ftp)` for both bailout and ramp scaling.
- `ble/ftms_control.py` gains public `min_target_watts` / `max_target_watts` properties.
- `ws_server.py` `_recv_loop` now publishes a typed rejection for `set_target_power` when no handler is configured.
- `cli.py` adds `--rider-ftp` + `--cadence-bailout-s` + `--target-power-ramp-s` flags. `_run_live` wires the bailout into the live runner: `_spawn_cadence_bailout` waits for the bike's `FtmsControl` to come online, constructs the bailout, and adds its `run()` task to the gather.
- `on_command` routes `set_target_power` through `bailout.handle_set_target_power` when the bailout is active, falls back to direct `control.set_target_power` otherwise.
- 11 new tests in `tests/test_cadence_bailout.py`. 105/105 total tests pass. ruff + format + mypy clean.

### Engine — `feat/bridge-bailout-signals` → PR #9, merged (`d9e50e7`)

- `src/effort/effort_bridge.gd` gains two signals (`cadence_bailout_engaged`, `cadence_bailout_disengaged`) and an `is_cadence_paused: bool` state property. Set on engaged, cleared on disengaged + `device_disconnected`.
- `tests/test_main.gd` (handshake scene) connects both signals and prints them to stdout; reflects pause state in the "Derived" label.
- gdlint clean. Headless boot smoke-tested against a live sidecar; bridge correctly reflects every lifecycle event including the new bailout pair.

### Memories captured this session

- `intensity-aware-safety-curves` — safety thresholds (cadence bailout, resume ramp, etc.) scale with current % FTP. Patient at low intensity, fast at high; ramp inverts.
- `opt-in-flags-need-feedback` — `--allow-*` flags must surface refusal as a typed event, not silent log drops.

## Live validation attempt — **failed; paused**

After both PRs merged we tried to validate end-to-end against the KICKR + Whoop, but **the live test did not succeed**. Three things stacked:

1. **First attempted command was missing `--allow-trainer-control`.** PowerShell paste swallowed the flag (the single-line semicolon-separated command we'd built up is hard to copy reliably). Sidecar ran read-only; KICKR's LED stayed blinking; engine's `set_target_power` calls would have hit the new silent-rejection path but the MVP scene currently doesn't *display* those rejections to the rider. Rider correctly perceived "trainer control did not work".

2. **Subsequent retry hit the Whoop ecosystem quirk.** Whoop's "Broadcast Heart Rate" mode (the secondary BLE advertisement that lets non-Whoop hosts read HR) is **per-activity**, not persistent. It resets when the previous activity ends, when the phone backgrounds the Whoop app, or on various firmware timeouts. By the time we retried, the Whoop was reading HR locally for the user but NOT broadcasting BLE-side. Three full BLE scans of the room confirmed: KICKR present at strong RSSI, Whoop genuinely absent — not just filtered. To re-enable broadcast the user has to actively **Start an Activity** in the Whoop app, not just toggle the broadcast setting.

3. **Test paused before retry**, to document and recover later.

Two real bugs surfaced during the failed attempts that need follow-up:

### Bug A: premature bailout — idle timer starts at sidecar startup

`CadenceBailout.__init__` initializes `_last_active_ts = time.monotonic()` at construction time, which happens when the bike source first connects (so within seconds of sidecar startup). The watcher's `idle_s = now - _last_active_ts` therefore reflects "time since sidecar startup", not "time since rider stopped pedalling" or "time since first ERG target was set".

Failure mode: rider waits >60s between sidecar startup and pressing Start Workout in the engine (entirely normal — launch sidecar, then engine, then configure rider settings, then press Start). When the engine sends the first `set_target_power(100)` for warmup, the watcher's next tick sees `last_target=100, idle_s=90+, pct_ftp=0.4` → engages bailout immediately, drops target right back to 0.

Rider perceives ERG as completely broken. The trainer briefly takes 100W then immediately drops to free spin. Easy to misread as "ERG never worked".

Fix (small, ~5 lines): reset `_last_active_ts` whenever `handle_set_target_power` routes a non-floor target through. The window then becomes "time since rider last pedaled OR last meaningful target was set" — which is the correct semantic for "rider has been idle at a meaningful target for X seconds".

### Bug B: sidecar dies on first scan if any device isn't broadcasting in the 8-second window

`_run_live` calls `scan_for_profiles(...)` once with `timeout_s=8.0`. If either device fails to advertise in that window (KICKR asleep, Whoop broadcast off, RSSI dropouts), the sidecar prints `no FTMS bike matched 'KICKR'` (or the HR equivalent) and exits. The user has to: (a) wake the missing device, (b) re-launch the entire sidecar command. Painful.

Fix: replace fail-once with "scan repeatedly for up to N seconds total, print every 5s what's still missing, succeed when all expected devices are found, fail only if N elapses." Maybe N=30 by default with a `--scan-timeout-s` flag for customization.

### Bug C (smaller, related): no clear log line for "control claimed"

The `BLE source FTMS Bike connected to KICKR CORE 1003` log line is emitted at BLE-GATT-connect time, which is BEFORE Request Control + Start. The actual "control is now claimed and `set_target_power` will work" moment isn't explicitly logged — you have to infer from absence of "Request Control rejected" warnings, or from the bailout watcher arming.

Fix: emit an explicit `INFO ... claimed control of KICKR CORE 1003 — set_target_power now active` after `request_control_and_start` succeeds.

## Open threads

- **Three small sidecar fixes queued** for the next session (bugs A/B/C above). Could fit in one `fix/bailout-startup-and-scan-ux` PR, ~45 min total.
- **Engine PR #4 (MVP HIIT loop) still Ready for Review** — engine-driven ERG live validation never actually happened against KICKR. The trainer-control writes themselves were live-validated 2026-05-21 via Python WS probe (PR #11's session); engine-as-driver validation will happen alongside the bailout retest after bugs A/B are fixed.
- **Whoop broadcast UX is environmental, not project-side**. The user has to Start an Activity in the Whoop app for broadcast to actually engage; "Broadcast Heart Rate toggle on" alone isn't enough on the user's firmware version. Worth a small note in `docs/ble-profiles.md` next time the HRS section is touched.
- **`docs/post-trainer-control-end-to-end-sidecar` (PR #13)** is still open and unmerged — contains the `scripts/record_session.py` diagnostic tool. The recorder was used successfully earlier today (the `20260521_203415.jsonl` file). Worth merging to keep develop self-contained.
- **Other small PRs queued from previous sessions:** `--record <path>` first-class flag (sidecar), FTMS SIM mode (sidecar), distance deriver (sidecar), CSCS profile (sidecar), card-system foundation (engine), GUT or hand-rolled test runner (engine).

## Next session entry point

> "Open `fix/bailout-startup-and-scan-ux` on the sidecar. Three small fixes in one PR: (1) reset `_last_active_ts` in `CadenceBailout.handle_set_target_power` whenever a non-floor target passes through, so the bailout's idle window starts on first meaningful ERG target rather than sidecar startup. (2) Replace `_run_live`'s fail-once scan with a forgiving retry loop (default total 30s, print per-5s what's still missing, succeed when complete). (3) Emit an INFO log line in `FtmsControl.request_control_and_start` after success: `claimed control of <device> — set_target_power now active`. After merging, retry engine-driven ERG live validation against KICKR (Whoop optional — bike-only is enough for bailout validation). MVP HIIT loop on engine PR #4 still awaits this validation pass."

## Loose notes

- Multi-line PowerShell commands with semicolons + flags are fragile to copy-paste. For the next live test, write a small `.ps1` script the user can double-click or `& .\start_test.ps1`, with all flags hard-coded. Avoids the "flag dropped on paste" failure mode that bit us twice this session.
- The KICKR LED state — "solid blue" — was tracked as a proxy for "control claimed" in this session and earlier. It's actually "BLE GATT connected to any host"; control claim happens later. Don't rely on LED for control-state verification; use the log line (once bug C is fixed) or the bridge's `control_acquired` signal.
- Whoop broadcast UX is a small but real moat that prevents reliable cross-device HR pairing. Worth noting in any future "device pairing UX" design (memory `device-pairing-ux-model`): even with the rider-friendly Zwift-style pairing screen, getting Whoop into broadcast state will remain a Whoop-app-side dance.
