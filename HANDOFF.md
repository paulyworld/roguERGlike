# HANDOFF — ERGlike (umbrella)

> The current state of the project across all repos. Updated at the end of every session. Read this first.

**Last updated:** 2026-05-23
**Last session log:** `docs/sessions/2026-05-22-first-live-ride-and-pause-architecture.md`
**Current branch:** umbrella `docs/erglike-rebrand-and-codex-sync`
**Current focus:** Cross-track sync after Codex shipped a major direction shift in `repos/engine`. Project rebranded `roguERGlike` → `ERGlike` (docs only; existing GitHub repo renames deferred). concert-mvp bootstrapped on GitHub as **paulyworld/gizzERG** (public, MIT, branch protection on main). Three sidecar PRs are in flight or proposed; one of them (#22 annotations) is held pending schema reconciliation with Codex.

## Project rename (docs-only for now)

The project is now **ERGlike**, not `roguERGlike`. `concert-mvp` is codenamed **gizzERG** (King Gizzard + ERG — the band's Greek tour video drives the first concert profile). The "rogu-" prefix locked us into a single genre; the umbrella now covers concert rides, terrain modes, and other non-roguelike experiences.

This rebrand is **docs-only for existing repos as of today.** GitHub repo names (`roguERGlike-sidecar`, `roguERGlike-engine`, `roguERGlike-game`) remain unchanged to avoid breaking in-flight PRs. The newly bootstrapped concert-mvp is named **paulyworld/gizzERG** directly (no rename later required). A coordinated rename of the other three repos will happen at a clean break point.

## Where we are

The 2026-05-22 live ride validated the sidecar trainer-control loop end-to-end against a real KICKR + Whoop. Today (2026-05-23) brought two big shifts driven by what that ride taught us:

**1. Codex's direction shift in engine HANDOFF.** `repos/engine/HANDOFF.md` and the new docs under `repos/engine/docs/` reorient the project around `repos/concert-mvp` as the primary near-term riding experience. Engine PR #4 (`feat/mvp-playable-loop`) is parked as a side experiment. Pattern B pause moves to sidecar as core protocol, not as a reason to promote the Godot MVP. Ownership split: Claude → sidecar/protocol/runtime; Codex → concert-mvp UI + engine bridge.

**2. Three sidecar work-streams started today.** All on the alt-tracks queue from yesterday:
- **sidecar PR #21 `feat/record-flag`** — first-class `--record` CLI flag replacing the external `scripts/record_session.py`. Live-validated against the KICKR with gizzERG; 3391-event ride captured cleanly. **Ready to merge.**
- **sidecar PR #22 `feat/rider-annotations`** — typed `annotate` command + `rider_annotation` envelope, updated to the hybrid schema (`tag`, `note?`, `client_id?`, `client_time_s?`, `context?`) per Codex review. Expanded recommended vocabulary covers the tuning-feedback loop. 125/125 tests, schema docs updated. **Ready to merge.**
- **gizzERG PR #1 `feat/rider-annotations`** — F2 overlay + preset hotkeys + chart markers + terrain route model (Codex commit `77c1b48`) + hybrid schema usage. Auto-populates `context` from current ride state at F2 press. 36/36 tests. **Ready for review.**
- **engine PR #12 `docs/sidecar-review-response`** — sidecar's response doc to Codex's review brief. Codex approved the hybrid schema; both client PRs updated. **Mergeable as historical record.**

## Annotation schema — resolved 2026-05-23

Codex approved the hybrid schema in engine PR #12. Final shape:

```json
{
  "type": "annotate",
  "tag": "too-hard",
  "note": "optional free text (≤280)",
  "client_id": "gizzERG",
  "client_time_s": 2412.5,
  "context": { "any": "client-side opaque blob — recommended shape in event-schema.md" }
}
```

Sidecar treats `context` opaque. Recommended fields by convention (analyzers join on these): `profile_id`, `profile_version`, `mode`, `video_id`, `section`, `target_watts`, `power`, `cadence`, `hr`, `wkg`, `grade`, `speed_kph`, `distance_m`, `elevation_gain_m`, `hardware_source`.

Expanded tag vocabulary covers the model-training feedback loop (Codex's `concert-mode-exploration.md`): `too-hard`, `too-easy`, `bad-sync`, `false-intensity`, `missed-intensity`, `cadence-mismatch` join the existing `ui-pause`, `walk-away`, `bug`, `marker`.

## What's next (immediate)

1. **Merge the three open PRs in any order** — sidecar #21, sidecar #22, gizzERG #1. Engine #12 mergeable as historical record.
2. **Resume Codex's sequence** once the annotation pair lands:
   - Item #1 hello/feature-negotiation envelope (sidecar) — protocol versioning + capability advertisement
   - Item #3 Pattern B structured pause (sidecar) — `pause` / `resume` commands + `paused` / `resumed` envelopes
   - Item #4 distance + FIT export groundwork (sidecar) — depends on `[[terrain-distance-ownership]]` rules: distinguish synthetic / trainer-reported / GPS
   - Item #5 Terrain/SIM protocol (sidecar) — FTMS Set Indoor Bike Simulation Parameters; gated on device capability
3. **Codex's gizzERG follow-ups** in parallel (per `repos/engine/docs/concert-mode-exploration.md`):
   - Terrain Mode UI prototype (ERG Terrain skin first; client-side distance/elevation per `[[terrain-distance-ownership]]`)
   - SvelteKit migration when the prototype settles
   - Dev/test terrain tuning popout
4. **Live-ride validation** of the annotation pair once both merge: F2 mid-ride, verify recording contains the context fields populated correctly.
5. **When the three legacy repos rename** (drop the `roguERGlike-` prefix), do it in one coordinated sweep — once protocol work settles and PRs are quiet.

## Open threads

- **Engine PR #4** stays open as side experiment per Codex's direction; not the near-term focus.
- **Codex's open questions** (in `repos/engine/docs/claude-sidecar-review-brief.md`) answered in `repos/engine/docs/sidecar-review-response.md`; all six confirmed by Codex with the terrain-distance nuance captured in memory `[[terrain-distance-ownership]]`.
- **gizzERG terrain math** landed on the annotation branch (Codex `77c1b48`). Next step is the Terrain Mode UI prototype on a fresh branch per the engine's `concert-mode-exploration.md`.
- **Whoop broadcast UX is fragile** — broadcast HR mode resets per-activity in the Whoop app.
- **PowerShell long-line paste fragility** — use `run-live-test.ps1` rather than pasting backtick continuations.

## Repo state

| Path | Friendly name | GitHub repo | Branch / state |
|---|---|---|---|
| umbrella | ERGlike umbrella | roguERGlike | `docs/erglike-rebrand-and-codex-sync` (this branch) |
| `repos/sidecar/` | sidecar | roguERGlike-sidecar | `develop` clean; PR #21 (`--record`, ready), PR #22 (annotations, **hold**) |
| `repos/engine/` | engine framework | roguERGlike-engine | `develop` clean; PR #12 (sidecar response doc) open; PR #4 (MVP loop) parked |
| `repos/concert-mvp/` | **gizzERG** | paulyworld/gizzERG (public, MIT) | `main` clean (branch-protected); PR #1 (F2 + terrain model + hybrid schema) open |
| `repos/engine-mvp/` | engine MVP worktree | (worktree of engine) | tied to engine PR #4 |
| `repos/game/` | game | roguERGlike-game | `develop`, no source yet |
| `repos/server/` | server | *(none — deferred)* | placeholder |

## Entry point for next session

> "Annotation schema settled (hybrid `{tag, note?, client_id?, client_time_s?, context?}`); three PRs ready to merge — sidecar #21 (`--record`), sidecar #22 (annotations), gizzERG #1 (F2 + terrain model + hybrid schema). Engine PR #12 is the historical record of the agreement. Next: live-ride validation of the annotation pair, then resume Codex's sequence — sidecar `hello` envelope (item #1), Pattern B pause (item #3), distance/FIT export (item #4), SIM mode (item #5). gizzERG side: Terrain Mode UI prototype per `concert-mode-exploration.md`."
