# Session: 2026-05-24 — Sidecar sequence complete (all 5 items in Codex's brief merged)

**Repo(s):** mixed (sidecar mostly; umbrella for HANDOFF/log)
**Interface:** Claude Code
**Branch(es):** sidecar `feat/hello-envelope` → `feat/structured-pause` → `feat/distance-and-export` → `feat/sim-mode`; umbrella `docs/2026-05-24-sidecar-sequence-complete`
**Duration:** Long. Five sidecar PRs end-to-end through Codex's full sequence.

## The headline

All five items in `repos/engine/docs/claude-sidecar-review-brief.md` are now merged to sidecar develop. The sidecar's protocol surface for gizzERG (and any future client) is feature-complete relative to the brief. 169 tests passing, ruff + mypy clean, real-ride-validated for the recording + distance paths.

| # | Item | Sidecar PR |
|---|---|---|
| 1 | hello envelope (protocol versioning + features list) | #23 |
| 2 | recording (#21, prior session) + annotations (#22, prior session) | (merged earlier) |
| 3 | Pattern B structured pause | #24 |
| 4 | distance deriver + FIT export | #25 |
| 5 | SIM mode (FTMS Set Indoor Bike Simulation Parameters) | #26 |

Yesterday's session set up the alt-tracks and the schema-reconciliation pattern with Codex. Today's was the implementation push — one PR per Codex sequence item, each merged before the next began so the chain stayed clean.

## Context loaded

- Yesterday's session log (`2026-05-23-rebrand-and-annotations-end-to-end.md`)
- `repos/engine/docs/claude-sidecar-review-brief.md` (Codex's original brief — re-read for each PR)
- Memories: `terrain-distance-ownership` (for #4), `client-input-sidecar-contract` (for the `sidecar` device kind in #1), `opt-in-flags-need-feedback` (for the rejection-reason pattern reused across PRs), `setup-branch-protection-bug` (not needed today but always relevant), `bailout-is-safety-not-pause` (for #3)

## Decisions made

- **`protocol_version = "1.0.0"`** as the starting point. Semver from here forward. Minor bumps for backward-compat features; major bumps for breaking changes.
- **New `"sidecar"` device kind** distinct from `"client"`. `client` = client-originated events (rider_annotation); `sidecar` = sidecar-self-described events (hello, future server_status / heartbeat).
- **`set_target_power` advertises in features regardless of `--allow-trainer-control`.** The command exists in the protocol whether or not the sidecar will execute it; runtime gating surfaces via the typed `target_power_set` rejection. Otherwise a client connecting before the operator flips the flag would gate UI off forever.
- **Pattern B coexists with cadence bailout in ONE class** (`CadenceBailout`), with two state-machine flags. `_paused` means "either kind of pause"; `_structured_paused` distinguishes for the things that differ (rejection reason, auto-resume-on-cadence behavior, completion envelope type).
- **Structured pause's `set_target_power` ack uses `reason="deferred-paused"`** (distinct from cadence's `"bailout-pending"`). Same shape, different category — clients can route differently.
- **Distance delta tracking lives in `BleSource`, not the decoder.** Decoder stays stateless/shared; per-connection state belongs at the per-connection lifecycle. First event emits `delta=0` (avoids spurious huge delta from a counter that may carry kilometers from a prior un-cleared session); backward jumps (counter reset) also emit `delta=0`, not negative.
- **`DistanceData.source: Literal["trainer", "synthetic", "gps"]`** enforces the distinction memory `[[terrain-distance-ownership]]` required. Same field on the new `ElevationData`. Exports must preserve the distinction.
- **`Envelope.to_wire()` uses `exclude_none=True`** (landed in #22). Optional unset fields stay off the wire rather than serializing as `null`. Narrow blast radius — only the annotation fields have None defaults.
- **FIT export as a separate CLI entry-point** (`roguerglike-export fit ...`) rather than a Click subcommand on the main CLI. Lower coupling; the export is offline and doesn't need to share runtime with the sidecar process.
- **Hello envelope occupies seq=0 in every recording.** Useful invariant for post-ride analyzers — first line of the JSONL always tells you what protocol the recording was made under.
- **SIM mode's `indoor_bike_simulation` feature advertises unconditionally** (the sidecar always supports the command). Clients combine with `device_capabilities.indoor_bike_simulation` for the per-trainer gate. Three-way gating: --allow-trainer-control + sidecar feature + device capability.
- **Session log first, then start coding `hello`.** Established session-ritual discipline mid-session when the user asked "what's next" before bedtime; the log went first.

## Work done

### Sidecar (five PRs, all merged today)

| PR | Branch | What |
|---|---|---|
| #23 | `feat/hello-envelope` | `HelloData` model; `"hello"` event type; new `"sidecar"` device kind; added to `SESSION_STATE_TYPES` so late subscribers receive it via replay; `announce_hello()` + `build_features()` registry in `session.py`; both runners call it first so hello = seq=0; schema doc gets recommended features vocabulary. 134/134 tests. |
| #24 | `feat/structured-pause` | `PauseCommand` / `ResumeCommand` (inbound) + `PausedData` / `ResumedData` (envelopes); `CadenceBailout` refactored to host two coexisting state machines; `structured_pause()` / `structured_resume()` methods; idempotent; cadence return doesn't auto-resume a structured pause; `set_target_power` during pause queues + emits `deferred-paused`; mock parity via `mock_structured_pause/resume`; CLI flags `--pause-easy-spin-pct-ftp` / `--pause-easy-spin-w`. 152/152 tests, all brief acceptance criteria covered. |
| #25 | `feat/distance-and-export` | FTMS `meters_total` extraction (bit 4 of Indoor Bike Data flags); `DistanceData.source: Literal["trainer", "synthetic", "gps"]`; new `ElevationData` event type (no producer yet — clients can push); per-connection delta tracking in `BleSource` (handles first-event + counter-reset); new `roguerglike_sidecar.export.fit` module + `roguerglike-export fit` CLI entry-point; `sport=CYCLING, sub_sport=INDOOR_CYCLING` so platforms classify as trainer rides; fit-tool dependency activated; mypy override for fit-tool's missing stubs. **Live-validated against today's 14-min KICKR ride: 845-record FIT.** 159/159 tests. |
| #26 | `feat/sim-mode` | `SetSimulationCommand` (inbound) + `SimulationSetData` (ack); FTMS opcode `0x11` write path in `FtmsControl.set_simulation()`; full FTMS wire format (`<hhBB` packing of wind_speed/grade/crr/cw); three-way gating (`--allow-trainer-control` + sidecar feature + device capability); typed rejection reasons; mock parity via `mock_set_simulation`; CLI dispatch in both runners; `indoor_bike_simulation` added to `_BASE_FEATURES`. 169/169 tests. |

### Engine

- **No new engine PRs from Claude.** Codex updated their planning docs (`docs/concert-sidecar-planning` branch) with terrain themes + feedback loop; not merged.
- **Codex extended engine PR #13** (music intensity proposal) with a "Piece 3b — Blended Terrain Model" section. Uncommitted at end-of-day pending Claude commentary.

### gizzERG

- **No new gizzERG PRs from Claude.** Codex updated `repos/concert-mvp/HANDOFF.md` with the same Blended Terrain Model proposal. Uncommitted at end-of-day pending Claude commentary.

### Umbrella

- Session log for yesterday (`2026-05-23`) merged via PR #11.
- Today's umbrella PR is this session log + HANDOFF update.

### Codex's uncommitted notes (left for Claude review)

`repos/concert-mvp/HANDOFF.md` + `repos/engine/docs/music-intensity-proposal.md` both got a "Blended Terrain Model" section added. Codex explicitly chose not to commit, asking for Claude commentary first. Commentary captured in umbrella HANDOFF + below.

## Commentary on the Blended Terrain Model proposal

For Codex when they pick this back up:

- **Architecture is right.** Blend = derived signal *modified by* authored overrides (not arithmetic averaging) is the correct framing. Averaging would smear authored intent.
- **`model_version` on the derived curve picks up the long-term hook** flagged in PR #13's crowdsourcing section. Good — keeps profile_version meaningful when the audio-feature algorithm bumps.
- **Override events overlap F2 annotation vocabulary** (`crescendo`, `song-boundary`, etc.). These should converge to ONE vocabulary so post-ride analyzers can join F2 annotations against authored overrides. Currently the F2 tags include `false-intensity`, `missed-intensity`, `bad-sync`, `cadence-mismatch`; the override events are `cap`, `floor`, `anchor`, `crescendo`, `drop`, `song-boundary`, `manual-override`. Some are distinct (overrides modify the profile; annotations annotate a ride), but where they overlap they should share names.
- **`event: drop intensity: 1.05`** (>1.0 on a normalized 0..1 scale): clarify whether this is "over-FTP burst" (intentional — the rider gets a >100% FTP sprint window) or "over-normalized" (engine should clamp). Both make sense; pick one and document.
- **Step 4 tests** — "blended soft-anchor, cap, and crescendo-not-sustained behavior" — the crescendo-not-sustained one is the canonical validator for the rolling-window-confirmation fix from the original proposal. Worth a sharper assertion than feel.
- **HANDOFF section and engine doc section are near-mirrors.** Slight risk of drift. Recommend either consolidating (engine doc canonical; HANDOFF summarizes + links) or actively keeping in sync on every edit.
- **Step 5 ("Keep sidecar out until route profiles are sent") respects the ownership split.** Claude will pick up `set_terrain_profile` when Codex asks.

## Open threads

- **Strava / TrainingPeaks FIT upload verification** — pinned to top of both umbrella and sidecar HANDOFFs. Code-complete; needs rider's 5-minute upload smoke.
- **Live-ride validation of #3 (pause), #5 (SIM)** against KICKR — code is FTMS-spec-correct; real-trainer firmware quirks best caught in a real ride.
- **gizzERG terrain UI prototype** is now in Codex's lane. Sidecar will get its `set_terrain_profile` request when ready.
- **Blended Terrain Model proposal** awaits Codex's review of Claude's commentary (above), then they can commit and iterate.
- **Codex's `docs/concert-sidecar-planning` branch in engine** still has unmerged terrain docs. Codex's call when to ship.
- **Engine PR #4 (Godot MVP)** stays parked per the established direction.

## Next session entry point

> "All five items in Codex's sequence are merged to sidecar develop (169 tests, ruff + mypy clean). Two acceptance items remain rider-side: Strava/TrainingPeaks FIT upload, and live-ride validation of pause + SIM against the KICKR. Codex's gizzERG side picks up next: Terrain Mode UI prototype, scrub-mode profile editor, audio preprocessing tool, plus their in-flight Blended Terrain Model proposal (uncommitted as of end-of-night — Claude commentary in HANDOFF, awaits Codex review). Vocabulary convergence (F2 annotation tags ↔ terrain override events) is the one cross-cutting design point worth raising in the next sync."

## Loose notes

- **Five PRs in one session is a lot.** The rhythm that worked: merge each PR before starting the next, so the develop branch was always the integration point and no rebase chains piled up. Did get a HANDOFF.md conflict on #22's rebase (because PR #21 had updated it first) — resolved with `git checkout --theirs HANDOFF.md` then `git rebase --continue`. The lesson: when stacking PRs that all touch HANDOFF.md, the rebase will conflict; just resolve to the incoming side and let the next commit's HANDOFF update take over.
- **Ruff's "use `X | Y` in isinstance instead of `(X, Y)`"** (UP038) caught me on `isinstance(ts, (int, float))` — corrected to `isinstance(ts, int | float)`. Python 3.10+ syntax that ruff prefers. Worth remembering.
- **fit-tool ships without type stubs.** Added a per-module mypy override in pyproject. The wrapper in `roguerglike_sidecar.export.fit` is the typed seam; mypy still validates everything else strictly.
- **The "Strava verification later" pattern** (code-complete now, validate against real platform later) is fine for items where I literally don't have account access. But it needs to be VISIBLE in HANDOFF so it doesn't get forgotten. Pinning at the top of HANDOFF rather than just mentioning it in passing makes the next session's first read remind the rider.
- **Sequence ordering matters more than I gave it credit for at the start.** When Codex's brief said "hello first, then everything else," I thought it was preference. By PR #26 I see why: every subsequent item advertises itself via the hello features list. If hello had landed later, every other PR would have needed to come back and add itself. The brief's sequence was a dependency graph, not a preference order.
- **CadenceBailout absorbed structured pause well**, partly because Codex had hinted "`_paused` plumbing already there — wire to it" and partly because the cadence-pause and structured-pause state machines share the same trainer-write path and the same `pre_pause_target` capture. Two flags + a `source=` parameter on the ramp method = 80% of the work; the rest was test coverage of the coexistence cases.
- **The Codex-uncommitted-edits-await-Claude-review pattern is working surprisingly well.** Codex flagged "i did not commit — you may want Claude to read/comment first." That avoids merge conflicts in shared docs and creates an explicit handoff signal. Reusable pattern.
