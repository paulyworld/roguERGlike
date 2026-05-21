# Session: 2026-05-20 — Phase 2 sidecar FTMS bike (live mode)

**Repo(s):** sidecar (+ umbrella for docs)
**Interface:** Claude Code
**Branch(es):** sidecar `feat/ble-ftms`; umbrella `docs/handoff-bike-integration-handshake` (extended)
**Duration:** ~1 hour (immediately following the same-day handshake session)

## Context loaded

- Earlier this day's session log `2026-05-20-bike-integration-handshake.md` — Phase 1 mock mode + replay fix + engine handshake all shipped
- Sidecar PRs #1, #2, #3 all merged by the user before starting Phase 2
- `repos/sidecar/docs/event-schema.md` — wire contract
- Sidecar `CLAUDE.md` — confirmed scope (FTMS bike + HR sensor; rowers/treadmills later)
- Project memory `device-pairing-ux-model` (Zwift-style separate pairing)
- `bleak` 0.22+ API docs (Indoor Bike Data characteristic spec, BleakClient.start_notify, BleakScanner.discover with return_adv)

## Decisions made

- **Profile-registry architecture.** Each BLE device type is a `BleProfile` (pure data + a pure `decode` function); a `BleSource` wraps one profile + one device address and feeds the shared `EventBus`. This shapes the work explicitly around the Zwift-style separate-pairing model — adding the HR sensor next is one more profile module of the same shape, not a refactor.
- **Pure decoder, no I/O.** `decode(bytes) → Iterable[(EventType, EventData)]`. Lets us unit-test against handcrafted FTMS packets without a Bluetooth adapter, which was useful since the trainer wasn't paired this session.
- **Decoder is total.** Empty / truncated / malformed payloads yield nothing rather than raising. Unknown flag bits skip the corresponding bytes so alignment of later fields stays correct. A bad packet must never take the live stream down.
- **Schema enforcement lives in the decoder.** Negative FTMS power (regen / measurement noise) is clamped to 0 instead of dropping the event; `bpm=0` (FTMS "no reading") is dropped instead of emitting a 0-bpm `heart_rate` that would lie to the engine.
- **Distance not emitted yet.** FTMS gives total distance, but `DistanceData` in our schema requires `meters_delta` too — needs a stateful deriver (last_total → delta) that doesn't belong inside a pure decoder. Tracked as follow-up; the decoder *consumes* the distance bytes correctly so later fields decode.
- **`--mode live` does not start the slider UI.** A real trainer overrides any slider you'd move; the UI would be confusing alongside it. May expose a small status/diagnostic page in a later PR.
- **`--scan` is independent of `--mode`.** Diagnostic that scans and exits; intentionally not tied to a mode.
- **CLI device matching is `address || exact_name || substring`.** Lets `--device-bike "KICKR"` work without the user having to type "KICKR CORE 8B2A". Empty string returns None (don't auto-pair to a random device on accidental `--device-bike ""`).
- **Session lifecycle factored.** `session.announce_session_start(bus, device_kind)` is shared by mock and live; both fire it exactly once per process. `device_connected` stays per-source.
- **ruff config aligned.** Added `tool.ruff.lint.isort.known-first-party = ["roguerglike_sidecar"]` to pyproject so local ruff and the pre-commit ruff hook agree on import grouping (was flapping every commit and forcing manual re-stages).
- **No hardware validation this session.** Trainer is available but not paired to Windows. All pieces unit-tested; live smoke deferred to next session.

## Work done

### Sidecar — `feat/ble-ftms` → PR #4
- `src/roguerglike_sidecar/ble/__init__.py` — package boundary + one-line architecture doc
- `src/roguerglike_sidecar/ble/profile.py` — `BleProfile` Protocol (service_uuid, char_uuid, device_kind, decode)
- `src/roguerglike_sidecar/ble/ftms_bike.py` — `FtmsBikeProfile` decoder for FTMS Indoor Bike Data (`0x1826` / `0x2AD2`); emits power / cadence / speed / heart_rate
- `src/roguerglike_sidecar/ble/source.py` — `BleSource` owns one `BleakClient`; reconnect-with-backoff loop; publishes `device_connected` / `device_disconnected`
- `src/roguerglike_sidecar/ble/scan.py` — `scan_for_service(uuid)` → `list[DiscoveredDevice]`
- `src/roguerglike_sidecar/session.py` — `announce_session_start(bus, device_kind)` shared helper
- `src/roguerglike_sidecar/cli.py` — added `--scan`, `--device-bike`; refactored `_run_mock` / `_run_live` / `_run_scan`; `--mode live` requires `--device-bike`
- `src/roguerglike_sidecar/mock.py` — factored `session_start` out to `session.py`; `announce_device` now only handles `device_connected`. Wire output unchanged.
- `tests/test_ftms_bike.py` — 9 tests against handcrafted FTMS packet fixtures
- `tests/test_ble_source.py` — 3 tests for `on_packet` → bus routing and `_publish_connected`
- `tests/test_cli.py` — 5 tests for `_match_device` (address / name / substring / no-match / empty query)
- `pyproject.toml` — `ruff.lint.isort.known-first-party` config
- `CLAUDE.md` — "Common tasks" updated with `--scan` + `--mode live` examples
- `docs/ble-profiles.md` — new doc describing the profile model so the HRS PR has a doc to extend
- 30/30 tests pass (17 new on top of the previous 13). ruff + mypy clean.
- Mock mode regression-checked: WS probe confirms `session_start` → `device_connected` → live ticks still emit correctly with no behaviour change.
- Commit: `feat(sidecar): Phase 2 — FTMS bike trainer (live mode)` (signed `8b28ae1`).

### Umbrella — `docs/handoff-bike-integration-handshake` (extending PR #4)
- This session log.
- HANDOFF refresh to reflect Phase 2 in flight.

### Sidecar — `docs/handoff-phase-2-ble` (new branch)
- Per-repo HANDOFF refresh — Phase 2 in flight, hardware validation pending.

## Open threads

- **Hardware validation pending.** Trainer is in the room but not paired. Next session: pair to Windows Bluetooth, run `roguerglike-sidecar --scan` to confirm it's discoverable, then `roguerglike-sidecar --mode live --device-bike "<substring>"`, run engine `test_main`, ride briefly, confirm realistic `power_changed` / `cadence_changed` values in the engine's stdout.
- **`--mode live` doesn't replay distance.** Decoder consumes FTMS total distance but doesn't emit `DistanceData` because `meters_delta` needs a stateful deriver. Small follow-up PR (`feat/distance-deriver`?).
- **HR de-dup policy.** Some FTMS bikes embed HR in the bike packet AND a chest strap may also be paired. Default plan: prefer standalone HR sensor over bike-embedded HR, with a `--prefer-bike-hr` override. Decision deferred to when HRS lands.
- **`fit-tool` runtime dep is unused for now.** Carrying it for the eventual FIT export work. Trim if YAGNI bites.
- **Bleak version pin.** Currently `bleak>=0.22.0`. The new `websockets.asyncio.server` API needed `websockets>=12`; Bleak similarly evolved. Watch for adapter / OS quirks on Linux/macOS CI runners — Windows is where Bleak is most mature.
- **Slider UI in live mode.** Intentionally off; consider a small status/diagnostic page later (connection state + most recent packet timestamps + RSSI), reusing the aiohttp server.
- **Three other PRs still open across project**: engine #1 (handshake scene), engine #2 (engine HANDOFF), umbrella #4 (this — soon to include Phase 2 docs).

## Next session entry point

> "Pair the FTMS trainer to Windows Bluetooth, then run `roguerglike-sidecar --scan` to confirm discovery, then `roguerglike-sidecar --mode live --device-bike '<name-substring>'` alongside the engine's `test_main` headless and confirm real `power_changed` / `cadence_changed` events flow with realistic values. If anything misbehaves, fix in sidecar PR #4 before merge. After validation + merge, start HRS profile on `feat/ble-hrs` (chest strap support) — one more profile module + HR de-dup policy + one `--device-hr` flag."

## Loose notes

- The `bleak` 0.22+ API split into `bleak.BleakScanner.discover(return_adv=True)` returning a dict `{address: (BLEDevice, AdvertisementData)}` — the older list-of-devices return shape is gone. Watch for this when reading older tutorials.
- `BleakClient.set_disconnected_callback` is deprecated in newer Bleak; the simple poll-on-`is_connected` approach used in `BleSource.run` is more portable across Bleak minor versions.
- FTMS spec §4.9: the flags field's bit 0 is the inverted *More Data* bit — when **clear**, the instantaneous speed field IS present. Easy to invert by accident; the test fixtures include both the bit-0=0 (full bike) and bit-0=1 (power-only) cases.
- The ruff CLI / pre-commit-ruff disagreement on import grouping was the same flap that hit during the morning session. Now fixed in `pyproject.toml` once and for all — both honour the same `known-first-party` config because pre-commit ruff reads the repo's pyproject when it runs.
- Mock mode's wire output is byte-identical before and after the `session.py` refactor — confirmed via a quick `websockets.connect` probe that printed the first 4 envelopes from a freshly-started sidecar.
