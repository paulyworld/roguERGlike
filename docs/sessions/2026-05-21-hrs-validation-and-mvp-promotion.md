# Session: 2026-05-21 — HRS profile + Whoop validation + MVP promotion

**Repo(s):** sidecar + engine + umbrella
**Interface:** Claude Code
**Branch(es):** sidecar `feat/ble-hrs` (merged); engine `feat/mvp-playable-loop` (PR #4 promoted ready); umbrella `docs/post-hrs-mvp-promotion` (this PR)
**Duration:** ~1.5 hours (follows `2026-05-21-hardware-validation-and-merges.md` earlier the same day)

## Context loaded

- Umbrella HANDOFF (post-Phase-2-merge state) — entry point listed HRS as one of three options
- Sidecar develop (Phase 2 BLE merged + live-validated)
- Memories `device-pairing-ux-model`, `bleak-windows-scan-filter-bug`
- BLE HRS spec (service `0x180D`, characteristic `0x2A37`)
- Whoop community docs on BLE broadcast support per generation

## Decisions made

- **HRS as the broad-ecosystem profile.** Rather than vendor-specific protocols, build the one standard BLE Heart Rate Service profile and document explicitly which devices speak it natively, which need a broadcast mode toggled, and which don't speak it at all. Saved as memory [[hr-source-breadth]] so future work doesn't narrow back to "chest strap only".
- **Drop, don't clamp, out-of-range HR.** HRS uses `bpm=0` for "no contact"; bpm<20 or >240 is almost certainly sensor garbage. A wrong HR actively misleads the game in a way wrong power doesn't, so the decoder drops the event rather than emitting a clamped value.
- **Default HR policy: prefer standalone sensor over bike-embedded.** When both `--device-bike` (FTMS that embeds HR) and `--device-hr` are paired, the bike source gets `drop_event_types={"heart_rate"}` so the strap is the single source of truth. `--prefer-bike-hr` flag inverts. Matches Zwift's default.
- **Policy resolution as a tiny pure function** (`_resolve_drop_types` in `cli.py`). Easy to unit-test in all four (has_bike, has_hr, prefer_bike_hr) combinations; easy to extend.
- **Multi-profile scan in one BLE pass.** `scan_for_profiles(profiles)` does one unfiltered scan and categorizes results per profile. Forward-compatible with future profiles (CSCS, RSC) without changing the CLI's scan UX.
- **Don't trust device names to contain vendor strings.** Custom name from the user's Whoop app ("MUDRAT-DETECTOR") was the actual broadcast name. Devices are discovered by service UUID, not name pattern; the name only matters for the user's `--device-*` substring matcher.

## Work done

### Sidecar — `feat/ble-hrs` → PR #7, merged
- `src/roguerglike_sidecar/ble/hrs.py` — `HrsProfile` (service 0x180D, char 0x2A37, device_kind `hr_sensor`). Pure decoder; parses flags + u8/u16 HR; consumes energy / RR-intervals without emitting; drops 0-bpm and out-of-range.
- `src/roguerglike_sidecar/ble/scan.py` — replaced `scan_for_service` with `scan_for_profiles(profiles)` for one-pass multi-profile discovery. Kept the Windows-post-filter pattern from the previous session.
- `src/roguerglike_sidecar/ble/source.py` — `BleSource` gained optional `drop_event_types: frozenset[EventType]` parameter for HR de-dup.
- `src/roguerglike_sidecar/cli.py` — `--device-hr` flag, `--prefer-bike-hr` flag, `_resolve_drop_types` policy function, `_run_live` orchestrates concurrent sources via `asyncio.gather`, `--scan` extended to multi-profile grouped output. Live mode now requires at least one of `--device-bike` / `--device-hr`.
- `tests/test_hrs.py` — 13 decoder fixtures; `tests/test_ble_source.py` +2 (drop_event_types); `tests/test_cli.py` +5 (dedup policy resolution).
- `docs/ble-profiles.md` — HRS row in profiles table; HR ecosystem compatibility section; HR de-dup policy section.
- `CLAUDE.md` — Common tasks updated with `--device-hr` + dedup examples.
- 51/51 unit tests pass; ruff + mypy clean; pre-commit healthy.
- **Live-validated against Whoop MG5** (broadcast HR enabled in Whoop app; broadcast name was the user's chosen "MUDRAT-DETECTOR"). End-to-end test: `--mode live --device-hr "mudrat"` → engine logged real `heart_rate_changed bpm=88..90`. Realistic resting HR.
- **Concurrent bike + HR live-validated** with both `--device-bike "KICKR" --device-hr "mudrat"` against the engine's MVP HIIT loop scene. Both BLE sources connected simultaneously; KICKR fed power+cadence; Whoop fed HR; the HIIT loop rendered and responded to live telemetry.
- Follow-up commit `d304d71`: noted Whoop MG5 confirmed-working in `docs/ble-profiles.md`, with the lesson that the BLE device name is whatever the user has set in the Whoop app — substring matching should not assume "Whoop" appears in the device name.
- Merged via PR #7 (squashed `6e7c983`).

### Engine — PR #4 promoted to ready
- Branch `feat/mvp-playable-loop` (commits authored by the user across earlier sessions): `prototype HIIT MVP playable loop` → `add HIIT session charting controls` → `add FTP-based phase targets`.
- Live-validated against bike + Whoop simultaneously this session.
- PR #4 body rewritten from the placeholder "draft for review-when-ready" to a real summary + test plan, noting the body is Claude's best-inference of intent and the commits remain the source of truth. PR promoted from **draft → ready for review** via `gh pr ready 4`.

### Memory captured
- `hr-source-breadth` — HR support must remain open to the wide ecosystem (chest straps, smartwatches with broadcast apps, Whoop 4+); build via standard BLE HRS, document what does and doesn't speak it.

### Umbrella — this docs branch
- This session log.
- HANDOFF refresh.

### Sidecar — `docs/post-hrs-mvp-promotion-sidecar` (separate PR)
- HANDOFF refresh.

### Engine — `docs/post-hrs-mvp-promotion-engine` (separate PR)
- HANDOFF refresh.

## Open threads

- **Engine PR #4** (now Ready for Review) — MVP HIIT playable loop. Awaiting your review pass and merge.
- **Stale stash on `feat/mvp-playable-loop`** (`git stash list` in engine): `mvp-playable-loop WIP — saved before connection-test branch switch 2026-05-21`. The branch has progressed past it (two new commits since the stash). Likely obsolete; `git stash drop` once you've confirmed.
- **Card-system foundation** (`Effect: Resource` with `apply(context)`, `CombatContext: RefCounted`) — still needed before the engine's `card.gd` parses cleanly and real card variety can land. The MVP loop currently uses a single hard-coded strike.
- **Distance deriver** (sidecar `feat/distance-deriver`) — FTMS decoder consumes `meters_total` but no `DistanceData` events yet.
- **CSCS profile** — older trainers / power meters that expose cadence outside FTMS.
- **Pairing UI** (~Phase 3 per [[device-pairing-ux-model]]) — Zwift-style web pairing + persistent config.
- **Mechanic exploration ideas** in `repos/game/IDEAS.md` — not yet promoted to experiments.
- **`fit-tool`** runtime dep is carried but unused — for eventual FIT export.
- **Reconnect-on-drop chaos test** — `BleSource.run`'s reconnect loop hasn't been stress-tested (yank trainer power mid-stream, etc.).
- **First CI runs** still pending on the next push; per [[github-actions-first-push-quirk]] memory the initial workflows occasionally don't auto-fire.

## Next session entry point

> "Pick one: (a) review and merge engine PR #4 (MVP HIIT playable loop) then either continue iterating on the loop or branch `feat/card-system-foundation` in engine for minimal `Effect` + `CombatContext` so card variety can land; (b) distance deriver on sidecar `feat/distance-deriver`; (c) CSCS profile on sidecar `feat/ble-cscs`. The cycling BLE pipeline is fully shipped end-to-end (mock + FTMS + HRS) and live-validated against KICKR + Whoop MG5; the engine's MVP loop has been driven by real bike + HR concurrently and works."

## Loose notes

- The Whoop MG5 broadcasts as standard BLE HRS the moment "Broadcast Heart Rate" is enabled in the Whoop app — no per-workout toggle required (at least on this model + firmware as of 2026-05). The advertised name is whatever the user has set in the Whoop app, NOT necessarily containing "Whoop". This bit us in the diagnostic phase — I assumed "MUDRAT-DETECTOR" was a neighbour's device.
- The HRS spec has bit 0 of the flags byte indicating u16 HR vs u8 — opposite of "more data" in FTMS. Field order: flags → HR (u8 or u16) → optional energy expended (u16) → optional repeating RR-intervals (u16). The decoder correctly steps past energy + RR even when not emitting them; an unknown future spec extension wouldn't break.
- Bleak's `BleakScanner.discover(service_uuids=)` is still unreliable on Windows per [[bleak-windows-scan-filter-bug]]; the multi-profile scanner inherits the unfiltered-then-post-filter pattern.
- Ruff's `tool.ruff.lint.isort.known-first-party = ["roguerglike_sidecar"]` config from PR #4 keeps local CLI ruff and the pre-commit ruff in agreement; no more flap on commits.
- Concurrent multi-source live mode connected the slower of the two devices last (KICKR took ~9s while Whoop was up in ~3s); this is fine because session_start fires before either device source runs, and the WS broadcaster's replay-to-late-subscribers logic (sidecar PR #2) means even if Godot connects between source-start-times it still gets the full session-state. No engine-side coordination needed.
- F5 in Godot still occasionally silently no-ops; the ▶ Play button in the top-right of the editor is the reliable way.
