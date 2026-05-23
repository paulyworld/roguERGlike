# Instructions — Working on ERGlike

This document is the **canonical source of conventions** for the entire ERGlike project across all repos. Every Claude session (chat, Cowork, Claude Code) should treat this as authoritative. If a convention here conflicts with something elsewhere, this file wins until updated.

> **Naming note (2026-05-23):** The project was renamed `roguERGlike` → `ERGlike`. The old prefix locked us into a single genre; the umbrella now covers concert-driven rides (codename **gizzERG**), terrain modes, and other non-roguelike experiences. GitHub repo names still read `roguERGlike-*` for now — coordinated rename deferred to a clean break point.

## Project structure (on disk)

```
~/dev/roguERGlike/                  ← umbrella (this repo — directory keeps its old name for now)
├── INSTRUCTIONS.md                 ← you are here
├── HANDOFF.md                      ← cross-repo current state
├── CLAUDE.md                       ← Claude-specific entry point
├── docs/sessions/                  ← historical session logs
├── scripts/                        ← cross-repo helper scripts
└── repos/                          ← gitignored; each subdir is its own repo
    ├── sidecar/                    ← roguERGlike-sidecar (public, MIT)
    ├── engine/                     ← roguERGlike-engine  (public, MIT)
    ├── concert-mvp/                ← codename gizzERG (local-only, planned public)
    ├── engine-mvp/                 ← worktree of engine's feat/mvp-playable-loop
    ├── game/                       ← roguERGlike-game    (private)
    └── server/                     ← roguERGlike-server  (private, deferred)
```

The umbrella tracks conventions and project-wide state. The repos under `repos/` are independent git checkouts (except `engine-mvp` which is a worktree of `engine`, and `concert-mvp` which doesn't yet have a remote). Each has its own `CLAUDE.md` and `HANDOFF.md`.

## Session conventions

### Starting a session

Whether using Claude chat, Cowork, or Claude Code, the ritual is:

1. **Read the umbrella `HANDOFF.md`** — what's the current overall state, what was the last thing worked on, where are we headed
2. **Read the relevant repo's `HANDOFF.md`** if the session will touch a specific repo
3. **Skim the most recent session log** in `docs/sessions/` only if HANDOFF references it or context feels missing
4. **Confirm intent for this session** with one sentence before starting work

If using Claude Code, it reads `CLAUDE.md` automatically — that file points at HANDOFF.

### During a session

- Work on a branch (see Branching below) — never directly on `main` or `develop`
- Commit aggressively with `WIP:` prefix for in-progress work; clean up before merging
- When making non-trivial decisions, note them in a scratch section at the bottom of the current session log file
- If a decision overrides something in the design docs, flag it explicitly — don't silently contradict committed direction

### Ending a session

Before closing the session:

1. **Update HANDOFF.md** — see `HOW-TO-UPDATE-HANDOFF.md` for the 5-minute drill
2. **Create or finalize the session log** in `docs/sessions/YYYY-MM-DD-<slug>.md`
3. **Commit work** with clean conventional commits (squashing WIP commits)
4. **Push the branch** but don't auto-merge — review PRs deliberately

The session is not over until HANDOFF reflects reality.

### Session log format

Files live at `docs/sessions/YYYY-MM-DD-<short-slug>.md`. Slugs are kebab-case, descriptive, short: `2026-05-12-repo-setup`, `2026-05-15-sidecar-mock-mode`, etc. If multiple sessions in one day, append `-2`, `-3`, etc.

Each session log has this structure:

```markdown
# Session: <date> — <one-line title>

**Repo(s):** umbrella, sidecar, etc.
**Interface:** Claude chat / Claude Code / Cowork / mixed
**Branch(es):** branch-name(s) worked on

## Context loaded
- What was read at session start
- What state things were in

## Decisions made
- Explicit decisions, with rationale if non-obvious
- Anything that changes design direction goes here AND in the design doc

## Work done
- Files touched / created
- Features completed
- Tests added

## Open threads
- Things deferred to next session
- Questions raised but not answered
- Known issues introduced

## Next session entry point
- One sentence: what to do first next time
```

Session logs are append-only history. Don't edit old ones except to fix factual errors.

## Branching conventions

Same model across all repos:

- `main` — always shippable. Tagged releases only. Protected.
- `develop` — integration branch. PRs merge here from feature branches.
- `feat/<short-slug>` — new features
- `fix/<short-slug>` — bug fixes
- `chore/<short-slug>` — tooling, refactors, non-functional changes
- `docs/<short-slug>` — documentation-only changes
- `experiment/<short-slug>` — exploratory work that may never merge

Branches off `develop` (or `main` for hotfixes only). PRs target `develop`. `develop` → `main` only at release points, tagged with semver.

**Never push directly to `main` or `develop`.** Use branches even for solo work — it forces clean commit history and makes parallel work safer.

## Commit conventions

[Conventional Commits](https://www.conventionalcommits.org/):

- `feat:` new feature
- `fix:` bug fix
- `chore:` tooling, deps, non-code changes
- `docs:` documentation only
- `refactor:` code change that neither fixes nor adds
- `test:` adding tests
- `perf:` performance improvement
- `style:` formatting only
- `build:`, `ci:` build system or CI changes
- `WIP:` work-in-progress (must be squashed before merge to `develop`)

Format: `<type>(<scope>): <short description>`. Scope is optional but useful for monorepo-ish work — `feat(cards): add exhaust mechanic`. Keep the first line under 72 chars; use the body for detail.

Sign commits when possible (`git commit -S`). Required on the public repos.

## Versioning

[Semver](https://semver.org/) for the public repos:

- Pre-release: `0.x.y` — anything can change between minor versions
- Stable: `1.0.0` onwards — breaking changes only on major bumps

The private game repo doesn't strictly need semver but uses tags for release builds (`game-v0.3.0-alpha`, etc.).

When the game repo bumps its engine submodule pin, that's a `chore(deps):` commit referencing the engine version.

## File organization principles

- Design decisions live in `docs/design/`
- Experiments and playtests live in `docs/experiments/`
- Ideas not yet committed to live in `IDEAS.md` (game repo only — see below)
- Session history lives in `docs/sessions/`
- Per-repo conventions live in that repo's `CLAUDE.md`
- The umbrella `INSTRUCTIONS.md` (this file) is the meta-convention

## The IDEAS.md discipline

The game repo has an `IDEAS.md` at its root for unsorted brainstorms, half-formed mechanics, and "what if we tried..." thoughts.

**Rules for IDEAS.md:**
- It does NOT cross-reference design docs and design docs do NOT reference it
- Entries are dated and tagged with status: `[raw]`, `[exploring]`, `[promoted]`, `[parked]`, `[rejected]`
- When an idea graduates to "we're going to try this," it moves to `docs/experiments/<date>-<slug>.md`
- When an experiment succeeds and becomes direction, it gets written up in `docs/design/`
- Nothing in IDEAS.md is binding on anything else

This separation prevents two failure modes: (1) design docs getting polluted with speculative ideas that contradict committed direction, (2) ideas being lost because there was no place for them that wasn't a commitment.

## Interface-specific notes

### Claude chat (web/desktop)
- Best for: design conversation, planning, exploring tradeoffs, learning new concepts
- Has the broadest context window for documentation
- Paste in `HANDOFF.md` at session start to load context
- Update HANDOFF at session end

### Claude Code (CLI)
- Best for: implementation of decided work
- Reads `CLAUDE.md` automatically from the current directory and parents
- Run from inside the relevant repo, not the umbrella, when working on code
- Run from umbrella when working on cross-repo concerns or conventions

### Cowork (desktop)
- Best for: light edits, file-level refactors, working alongside the IDE
- Treat as Claude Code's lighter sibling — same conventions apply

**Avoid mixing interfaces mid-session.** Finish a session in the interface you started in, write the HANDOFF, then switch. This prevents lost context.

## Infosec basics

- Never commit secrets, API keys, or personal data (FIT files, HR logs)
- Use `gitleaks` (configured in pre-commit on the public repos)
- Sign commits on public repos
- Public repos: branch protection on `main`, require PR + CI to merge
- Private repos: less strict but still PR-based to maintain habits
- Personal ride data is PII — `.gitignore` already strips `*.fit`, `*.tcx`, `*.gpx`, `sessions/`, `rides/`

## When this document is wrong

Conventions evolve. If something here doesn't match how work is actually happening, fix this document — don't work around it silently. PR to the umbrella with a `docs:` commit explaining the change.
