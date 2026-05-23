# HANDOFF — ERGlike (umbrella)

> The current state of the project across all repos. Updated at the end of every session. Read this first.

**Last updated:** 2026-05-23
**Last session log:** `docs/sessions/2026-05-22-first-live-ride-and-pause-architecture.md`
**Current branch:** umbrella `docs/erglike-rebrand-and-codex-sync`
**Current focus:** Cross-track sync after Codex shipped a major direction shift in `repos/engine`. Project rebranded `roguERGlike` → `ERGlike` (docs only; GitHub repo renames deferred). concert-mvp codenamed **gizzERG**. Three sidecar PRs are in flight or proposed; one of them (#22 annotations) is held pending schema reconciliation with Codex.

## Project rename (docs-only for now)

The project is now **ERGlike**, not `roguERGlike`. `concert-mvp` is codenamed **gizzERG** (King Gizzard + ERG — the band's Greek tour video drives the first concert profile). The "rogu-" prefix locked us into a single genre; the umbrella now covers concert rides, terrain modes, and other non-roguelike experiences.

This rebrand is **docs-only as of today.** GitHub repo names (`roguERGlike-sidecar`, `roguERGlike-engine`, `roguERGlike-game`) remain unchanged to avoid breaking in-flight PRs. A coordinated rename will happen at a clean break point — likely when concert-mvp moves to GitHub (as `paulyworld/gizzERG` or similar) and the protocol work settles.

## Where we are

The 2026-05-22 live ride validated the sidecar trainer-control loop end-to-end against a real KICKR + Whoop. Today (2026-05-23) brought two big shifts driven by what that ride taught us:

**1. Codex's direction shift in engine HANDOFF.** `repos/engine/HANDOFF.md` and the new docs under `repos/engine/docs/` reorient the project around `repos/concert-mvp` as the primary near-term riding experience. Engine PR #4 (`feat/mvp-playable-loop`) is parked as a side experiment. Pattern B pause moves to sidecar as core protocol, not as a reason to promote the Godot MVP. Ownership split: Claude → sidecar/protocol/runtime; Codex → concert-mvp UI + engine bridge.

**2. Three sidecar work-streams started today.** All on the alt-tracks queue from yesterday:
- **PR #21 `feat/record-flag`** — first-class `--record` CLI flag replacing the external `scripts/record_session.py`. Live-validated against the KICKR with concert-mvp; 3391-event ride captured cleanly. **Ready to merge; not blocked.**
- **PR #22 `feat/rider-annotations`** — typed `annotate` command + `rider_annotation` envelope so clients can mark moments mid-ride (closes the 2026-05-22 "this bailout was a ui-pause, not a walk-away" debugging gap). 120/120 tests, schema docs updated. **Held pending schema reconciliation with Codex.**
- **concert-mvp F2 handler** — overlay + preset hotkeys + chart markers, paired with PR #22's schema. **Held as a local-only working-tree diff** (concert-mvp has no git remote yet).
- **engine PR #12** — sidecar's response doc to Codex's review brief, with hybrid annotation schema counter-proposal and answers to the six open questions.

## Annotation schema reconciliation

Concrete disagreement between PR #22 (already shipped) and Codex's brief in `repos/engine/docs/claude-sidecar-review-brief.md`:

| Field | PR #22 shipped | Codex brief proposed |
|---|---|---|
| Category | `tag` (open string) | `reason` (enum-leaning) |
| Free text | `note` | (none) |
| Input mechanism | (none) | `label: "F2"` ← violates `[[client-input-sidecar-contract]]` memory |
| Client ride time | (missing) | `client_time_s` ← good addition |
| Rich context | (missing) | `context` blob ← good addition |

**Hybrid counter-proposal** (in engine PR #12): `{tag, note?, client_id?, client_time_s?, context?}`. Keep `tag` open-string, drop `label`, adopt `client_time_s` + `context`.

Once Codex agrees: I update PR #22 (<1 hour) and the held concert-mvp diff, then both can land.

## What's next (immediate)

1. **Wait for Codex's review of engine PR #12.** Schema agreement unblocks PR #22 + concert-mvp F2 work.
2. **Sidecar PR #21 (`--record`) can merge anytime** — independent of the schema discussion.
3. **Then resume Codex's sequence:**
   - Item #1 hello/feature-negotiation envelope (sidecar)
   - Item #3 Pattern B structured pause (sidecar)
   - Item #4 distance + FIT export groundwork (sidecar)
   - Item #5 Terrain/SIM protocol (sidecar)
4. **When concert-mvp moves to GitHub** (likely as `paulyworld/gizzERG`), do the umbrella rename in the same sweep.

## Open threads

- **Annotation schema reconciliation** in flight — engine PR #12, blocking sidecar PR #22 and concert-mvp F2 work.
- **Engine PR #4** stays open as side experiment per Codex's direction; not the near-term focus.
- **Concert-mvp has no GitHub remote yet.** Local-only. F2 handler diff sits in working tree until the rename + remote setup.
- **Codex's open questions** (in `repos/engine/docs/claude-sidecar-review-brief.md`) answered in `repos/engine/docs/sidecar-review-response.md`.
- **Whoop broadcast UX is fragile** — broadcast HR mode resets per-activity in the Whoop app.
- **PowerShell long-line paste fragility** — use `run-live-test.ps1` rather than pasting backtick continuations.

## Repo state

| Path | Friendly name | GitHub repo | Branch / state |
|---|---|---|---|
| umbrella | ERGlike umbrella | roguERGlike | `docs/erglike-rebrand-and-codex-sync` (this branch) |
| `repos/sidecar/` | sidecar | roguERGlike-sidecar | `develop` clean; PR #21 (`--record`, ready), PR #22 (annotations, **hold**) |
| `repos/engine/` | engine framework | roguERGlike-engine | `develop` clean; PR #12 (sidecar response doc) open; PR #4 (MVP loop) parked |
| `repos/concert-mvp/` | **gizzERG** | *(none — local only)* | local `feat/rider-annotations` working-tree diff |
| `repos/engine-mvp/` | engine MVP worktree | (worktree of engine) | tied to engine PR #4 |
| `repos/game/` | game | roguERGlike-game | `develop`, no source yet |
| `repos/server/` | server | *(none — deferred)* | placeholder |

## Entry point for next session

> "Two PRs ready and one held. Sidecar PR #21 (`--record`) can merge anytime. PR #22 (annotations) is held pending engine PR #12's schema-reconciliation review — Codex's brief proposed a different annotation shape; counter-proposal is a hybrid `{tag, note?, client_id?, client_time_s?, context?}`. Concert-mvp (codename gizzERG) F2 handler is a working-tree diff awaiting that schema decision. Project rebrand to ERGlike is docs-only; GitHub repo renames deferred to a clean break point."
