# Session: 2026-05-23 — Rebrand to ERGlike, gizzERG bootstrapped, annotations end-to-end

**Repo(s):** mixed (sidecar, engine, gizzERG/concert-mvp, umbrella)
**Interface:** Claude Code
**Branch(es):** sidecar `feat/record-flag` + `feat/rider-annotations`; engine `docs/sidecar-review-response` + `docs/music-intensity-proposal`; gizzERG `feat/rider-annotations`; umbrella `docs/erglike-rebrand-and-codex-sync`, `docs/2026-05-23-session-log` (this branch)
**Duration:** Long. One contiguous push from `--record` flag through schema reconciliation to merging everything.

## The headline

The annotation primitive shipped end-to-end across three repos in one session. By the time we wrapped, the sidecar accepts typed `annotate` commands and republishes `rider_annotation` envelopes; gizzERG sends them on F2 keypress with auto-populated ride context; and the recording captures everything into JSONL with `client_time_s` so analyzers can place markers on the ride timeline rather than the WS receipt timeline. Plus a project rebrand and a substantial direction shift from Codex toward the concert/terrain experience.

## Context loaded

- Umbrella `HANDOFF.md` (left in good shape by yesterday's session)
- Sidecar `HANDOFF.md` + yesterday's session log on first-live-ride and pause architecture
- Memory: `bailout-is-safety-not-pause`, `concert-mvp-repo`, `branch-off-develop-not-feature-branches`, `opt-in-flags-need-feedback`, `setup-branch-protection-bug`
- Mid-session: Codex's direction-shift docs in `repos/engine/docs/`: `claude-sidecar-review-brief.md`, `concert-mode-exploration.md` (with two new terrain-themes commits from Codex this morning), `concert-product-roadmap.md`, `riding-infrastructure-direction.md`, `training-platform-export-research.md`
- Rider's request on F2 vs. Excel for music feedback authoring — directly shaped the music-intensity proposal at the end of session

## Decisions made

- **Rename `roguERGlike` → `ERGlike`** (docs-only). The `rogu-` prefix locked us into one genre; the umbrella now covers concert rides, terrain modes, and other non-roguelike experiences. GitHub repo names (`roguERGlike-{sidecar,engine,game}`) stay unchanged for now — coordinated rename deferred to a clean break point.
- **Concert-mvp codenamed gizzERG** (King Gizzard + ERG, the band's Greek tour drives the first profile). Bootstrapped as `paulyworld/gizzERG` directly — no later rename required.
- **Annotation schema decided after Codex review**: hybrid `{tag, note?, client_id?, client_time_s?, context?}`. Kept open-string `tag` (rejected enum-leaning `reason`). Dropped Codex's proposed `label: "F2"` field (violates the principle that wire format shouldn't speak about input mechanism). Adopted Codex's `client_time_s` and `context` blob additions.
- **`Envelope.to_wire()` serializes with `exclude_none=True`**: optional unset fields stay off the wire rather than serializing as explicit `null`. Currently only matters for the annotation fields; documented as narrow behavior.
- **Engine PR #4 (Godot HIIT MVP) stays parked** as a side experiment per Codex's direction. The product center moves to gizzERG.
- **Pattern B pause** moves to sidecar core protocol — not a reason to promote the Godot MVP.
- **Ownership split formalized**: Claude → sidecar/protocol/runtime; Codex → gizzERG + engine bridge.
- **Crowdsourced authoring** captured as long-term direction in PR #13 — explicitly not near-term, but near-term piece designs preserve hooks (cross-profile training, `model_version` on curves, rider-context fields in F2) so the door stays open.
- **Terrain distance/elevation ownership**: gizzERG computes client-side first; sidecar becomes durable export authority once protocol exists; exports must distinguish synthetic / trainer-reported / GPS distance. Captured as memory `[[terrain-distance-ownership]]`.

## Work done

### Sidecar

- **PR #21 `feat/record-flag`** — first-class `--record PATH` CLI flag replacing the external `scripts/record_session.py`. In-process `JsonlRecorder` attaches as `EventBus` subscriber, inherits session-state replay-on-subscribe, flushes per write. Deleted `scripts/record_session.py` + `record-session.ps1`. `run-live-test.ps1` gained `-Record` (auto-timestamped) and `-RecordPath <path>`. **Live-validated against KICKR + Whoop + concert-mvp** — 3391-event ride captured cleanly. 116/116 tests. Merged → develop.
- **PR #22 `feat/rider-annotations`** — typed `AnnotateCommand` + `RiderAnnotationData`; new `"client"` device kind for non-device-sourced events; new `_publish_annotation` helper at the WS layer (annotations bypass `on_command` and the `--allow-trainer-control` gate since they're inert wrt the trainer); hybrid schema with `client_time_s` + opaque `context` blob; `exclude_none=True` serialization; expanded recommended tag vocabulary (`too-hard`, `too-easy`, `bad-sync`, `false-intensity`, `missed-intensity`, `cadence-mismatch` + the existing `ui-pause`, `walk-away`, `bug`, `marker`); event-schema doc updated with recommended `context` shape. Rebased onto post-#21 develop, force-pushed, merged → develop. 128/128 tests.

### gizzERG (`paulyworld/gizzERG`)

- **Bootstrapped the GitHub repo from scratch** — `git init`, MIT LICENSE copied from sidecar template, `.gitignore`, `gh repo create paulyworld/gizzERG --public`, pushed `main` + `develop`, branch protection on `main` matching sidecar pattern.
- **PR #1 `feat/rider-annotations`** — three commits: F2 overlay + preset hotkeys + chart markers (mine); terrain route model + tests (Codex's `77c1b48`); hybrid schema usage + `buildContextSnapshot` + preset hotkey rebalance toward tuning-feedback vocabulary + `CLIENT_ID` = `"gizzERG"` (mine). 36/36 tests (14 erg-controller + 17 annotations + 5 terrain). Merged → develop.
- Stashed Codex's uncommitted 134-line terrain-UI WIP (`app.js` / `index.html` / `styles.css`) as `codex-terrain-ui-wip-2026-05-23` after merge so the develop checkout would land cleanly.

### Engine

- **PR #12 `docs/sidecar-review-response`** — response to Codex's review brief. Three-difference analysis on annotation schema; counter-proposal of the hybrid shape; answers to the six open questions. Merged → develop.
- **PR #13 `docs/music-intensity-proposal`** — proposal for combined-feature intensity model (RMS + spectral centroid + spectral flux + HPSS + boundary detection) replacing BPM-only; scrub-mode profile editor as separate UX from mid-ride F2; Python preprocessing tool architecture (`tools/profile-builder/`); F2 schema additions (no protocol change). Added a long-term section on crowdsourced authoring, model versioning, and audio reconciliation (chromaprint/AcoustID + Archive.org etree + Bandcamp + setlist.fm) with explicit "not near-term" framing. Open for Codex review.
- Codex's `docs/concert-sidecar-planning` branch still has unmerged terrain docs — their call when to ship.

### Umbrella

- **PR #10 `docs/erglike-rebrand-and-codex-sync`** — README + INSTRUCTIONS + HANDOFF updated to ERGlike + gizzERG, cross-track sync after Codex's direction shift. Two commits — initial rebrand + a follow-up to reflect that gizzERG had moved from local-only to a real GitHub repo. Merged → develop.

### Live-test smoke runs

- Mock-mode `--record` smoke: 67 events in 25s, single session_id, session-state header + 1Hz telemetry triplets, JSONL valid.
- Live `--record` smoke (real KICKR + Whoop + concert-mvp): 3391 events, single session_id, monotonic seq, included unexpected `target_power_set` writes from concert-mvp's pause cycles (validated that ERG writes flow through the recorder correctly). One `cadence_bailout_engaged` at the rider's last stop-pedalling experiment.
- Diagnostic: 11 of the rider's "pause" experiences during the ride were concert-mvp's Pattern A soft-target writes, not actual cadence bailouts. The conflation reinforces the value of the annotation primitive — F2 would have made the two distinct events legible in the recording.

### Memories written this session

- `client-input-sidecar-contract` (feedback) — sidecar owns typed contract; client owns input mechanism. Wire speaks tags, not key names.
- `terrain-distance-ownership` (project) — gizzERG client-side first, sidecar later; exports must distinguish synthetic / trainer-reported / GPS.
- `concert-mvp-repo` (project) — updated to reflect gizzERG repo identity + Codex's direction shift.

### PRs touched this session

| PR | Repo | Action |
|---|---|---|
| #21 | sidecar | Created → merged (`--record` flag) |
| #22 | sidecar | Created → rebased → merged (annotations primitive) |
| #1 | gizzERG | Bootstrap repo → created → merged (F2 + terrain + schema) |
| #12 | engine | Created → reviewed by Codex → merged (sidecar response) |
| #13 | engine | Created (music intensity proposal — open) |
| #10 | umbrella | Created → merged (rebrand + sync) |

## Open threads

- **Engine PR #13** (music intensity proposal) awaits Codex's review of the five open questions (audio source policy, boundary detector + manual marker interplay, browser-side feature compute cost/benefit, crescendo event modeling, profile-version bump policy).
- **Codex's `docs/concert-sidecar-planning` branch** in engine still has unmerged terrain docs (concert-mode-exploration, concert-product-roadmap, training-platform-export-research). Codex's call when to ship.
- **Codex's uncommitted gizzERG terrain-UI work** stashed as `codex-terrain-ui-wip-2026-05-23` — recoverable via `git stash pop` when they return.
- **Engine PR #4 (HIIT MVP)** stays open as parked side experiment. Not a near-term decision.
- **Live-ride validation of the annotation pair** — pending. Best to run a real ride and confirm `rider_annotation` envelopes land in the JSONL with `client_time_s` and `context` populated.
- **Sidecar `hello` envelope** is the next protocol piece (Codex's sequence item #1). Recommended next.

## Next session entry point

> "Annotation primitive end-to-end (sidecar PR #21 + #22 merged, gizzERG PR #1 merged with F2 + terrain model + hybrid schema). Codex's sequence next: item #1 sidecar `hello` envelope (protocol-version + feature-list advertisement). Small piece — new `HelloData` model + add to `SESSION_STATE_TYPES` for subscribe-replay + tests. Once that lands, item #3 Pattern B structured pause (`pause` / `resume` commands + `paused` / `resumed` envelopes + `CadenceBailout.set_paused` plumbing). Engine PR #13 (music intensity proposal) is awaiting Codex review; nothing for Claude to do there. Live-ride validation of annotations is the other useful thing to do — pending rider time on the bike."

## Loose notes

- **PowerShell + Bash quirks tripped me up twice.** `pwsh` isn't on the agent's bash PATH on Windows; had to switch to the dedicated PowerShell tool for the live-mode smoke test. And `gh pr merge --delete-branch` returned "Aborting" on engine PR #12 but the merge itself succeeded — apparently the "Aborting" was about the branch-deletion step (worktree concern or similar), not the merge.
- **The engine repo's git boundary tripped me almost-fatally.** When I ran `git status` from inside `repos/concert-mvp/` initially, git resolved upward to the umbrella's `.git` and reported `repos/` was gitignored. That's how I learned concert-mvp had no remote yet. Worth remembering: when in `repos/<name>/`, sanity-check `git rev-parse --show-toplevel` before assuming you're in the sub-repo.
- **Bootstrapping a new repo with branch protection isn't conceptually hard but easy to get the gh-api flags wrong.** Memory `setup-branch-protection-bug` (use `-F` typed, include `[contexts][]`) saved real time. Pinning the same memory for the rename-sweep later when the three legacy repos eventually rename.
- **The schema reconciliation with Codex went well.** Asking the user to paste the prompt I drafted (rather than me trying to coordinate cross-agent state) worked — Codex returned a clear LGTM-with-nuance and the next move was unambiguous. Pattern worth repeating: when both Codex and I need to land on a shared contract, write a focused prompt the user can hand off, let them mediate, get back a clean decision.
- **The user's "F2 vs Excel" question at the end of session was a surprisingly rich design moment.** The right answer is *neither and both*: F2 stays for ride-time feel, profile editor for out-of-ride authoring, audio preprocessing tool for the model itself. Captured in PR #13 along with the long-term crowdsourcing direction.
- **`exclude_none=True` is a small change with downstream cleanliness.** Recordings stay compact, analyzers don't have to special-case explicit-null vs absent. Worth keeping in mind for future event-schema additions: prefer `None`-default optional fields over `""`/sentinel defaults; the serialization will handle the difference correctly.
