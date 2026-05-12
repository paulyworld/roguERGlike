# Session: 2026-05-12 — Repo setup and conventions

**Repo(s):** umbrella (new), affects all four code repos
**Interface:** Claude chat (web)
**Branch(es):** n/a — pre-bootstrap
**Duration:** extended planning session

## Context loaded

- Prior planning work on the bathhouse booking app and yoga-scraper extension (general dev context)
- High-level project pitch: a roguelike deckbuilder driven by smart trainer data
- Decision history: engine choice (Godot 4), sidecar architecture (Python + Bleak + WS), four-repo split (sidecar + engine public, game + server private)
- Renamed project from `roguERGbike` to `roguERGlike` to reflect device-agnostic ambition (bikes, rowers, treadmills, etc.)

## Decisions made

1. **Project structure: hybrid layout.** One umbrella repo for conventions and cross-repo state, four sibling code repos on disk under `repos/` (gitignored from umbrella). Rationale: keeps each code repo independent (independent visibility, license, CI) while centralizing project-wide conventions. Better than monorepo (can't easily go private→public/public→private later) and better than no-umbrella (conventions duplicate and drift).

2. **Start-session ritual: HANDOFF primary, session log secondary.** HANDOFF.md is short, current-state-focused, and updated every session. Session logs in `docs/sessions/YYYY-MM-DD-<slug>.md` are append-only history.

3. **IDEAS.md discipline.** Game repo gets an `IDEAS.md` for speculative ideas. It does not cross-reference design docs and design docs do not reference it. Ideas graduate to `docs/experiments/` and then to `docs/design/` only when validated. Default Claude context loading does NOT pull from IDEAS.md.

4. **Branch model.** `main` (protected) ← `develop` ← feature branches (`feat/`, `fix/`, `chore/`, `docs/`, `experiment/`). Conventional commits. `WIP:` prefix for in-session commits, squashed before merge. Signed commits on public repos.

5. **Interface separation.** Design conversation → Claude chat. Implementation → Claude Code. Light edits → Cowork. Don't mix interfaces mid-session; finish the HANDOFF before switching.

6. **Per-repo `CLAUDE.md` and `HANDOFF.md`** — each repo has its own. Claude Code reads them automatically via cwd walk-up. Umbrella `INSTRUCTIONS.md` is canonical and overrides anything inconsistent.

## Work done

Files created in the umbrella repo skeleton:

- `INSTRUCTIONS.md` — canonical conventions for all repos
- `CLAUDE.md` — umbrella entry point
- `HANDOFF.md` — initial cross-repo state
- `templates/CLAUDE.md.template` — template for per-repo CLAUDE
- `templates/HANDOFF.md.template` — template for per-repo HANDOFF
- `templates/SESSION-LOG.md.template` — session log template
- `docs/sessions/2026-05-12-repo-setup-and-conventions.md` — this file

Files added to per-repo skeletons:

- `repos/sidecar/CLAUDE.md`, `repos/sidecar/HANDOFF.md`
- `repos/engine/CLAUDE.md`, `repos/engine/HANDOFF.md`
- `repos/game/CLAUDE.md`, `repos/game/HANDOFF.md`, `repos/game/IDEAS.md`
- `repos/server/CLAUDE.md` (placeholder)

IDEAS.md was pre-populated with four `[raw]` ideas from this session's brainstorm:
- Inverted cognitive/physical load (battles = Zone 2 + complex; transitions = hard + simple)
- Open mode: effort modifies cards in real time
- Three orthogonal axes of mechanic design
- Limited info during the charge/transition phase

## Open threads

- None of the five repos have been `git init`'d yet — that's the next session's work
- Speculative ideas captured in `IDEAS.md` are not yet committed to and should not influence design until promoted
- Sub-title for the bike-themed game still TBD
- Server work deferred to Phase 5+

## Next session entry point

> "Bootstrap the umbrella and four code repos following `SETUP.md`: `git init` each, push to GitHub with correct visibility (umbrella + sidecar + engine public, game private, server placeholder private), set up branch protection on the three with active code."

## Loose notes

- The hybrid layout was the right call for the human's stated goals (novice-friendly, resilient, future-proof, infosec-conscious). Monorepo would have created a hard-to-undo coupling; pure four-repos would have duplicated conventions.
- The IDEAS.md / experiments / design progression is the most important convention here — it's what keeps "build in public" speculation from polluting committed direction. Without this separation, the chat-driven design process tends to leak unfinished thoughts into the repo.
- Worth revisiting in 4–6 weeks: are the conventions actually being followed? If HANDOFF is going stale or session logs aren't being written, simplify rather than enforce harder.
