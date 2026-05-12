# CLAUDE.md — roguERGlike umbrella

You're working in the umbrella repo for the roguERGlike project. This repo holds project-wide conventions, cross-repo session logs, and helper scripts. **Code lives in sibling directories under `repos/`, not here.**

## Start every session by reading

1. `HANDOFF.md` (in this directory) — current cross-repo state
2. `INSTRUCTIONS.md` (in this directory) — full conventions, authoritative
3. If working on a specific repo: that repo's `HANDOFF.md` and `CLAUDE.md` in `repos/<name>/`

## What this repo contains

- `INSTRUCTIONS.md` — canonical conventions for the whole project (session ritual, branching, commits, versioning, infosec, idea management)
- `HOW-TO-UPDATE-HANDOFF.md` — quick reminder for the end-of-session HANDOFF update
- `HANDOFF.md` — current state across all four repos
- `docs/sessions/` — historical session logs (`YYYY-MM-DD-<slug>.md`)
- `scripts/` — helper scripts
- `templates/` — templates for new session logs, new repo bootstrap, etc.
- `README.md` — project overview

## What this repo does NOT contain

- Application code (lives in `repos/sidecar`, `repos/engine`, `repos/game`, `repos/server`)
- Game-specific design (lives in `repos/game/docs/design/`)
- Sidecar event schemas (lives in `repos/sidecar/docs/`)

## When to update this repo

- After every session: update `HANDOFF.md`, add/finalize the session log
- When conventions change: PR to `INSTRUCTIONS.md`
- When adding helper scripts that cross repos: commit to `scripts/`

## When NOT to update this repo

- Don't put repo-specific design or implementation here
- Don't put speculative ideas here (those go in `repos/game/IDEAS.md`)
- Don't push to `main` directly — use `develop` and PRs even for solo work

## The four repos at a glance

| Path | Repo | Visibility | Primary tech |
|---|---|---|---|
| `repos/sidecar/` | roguERGlike-sidecar | public, MIT | Python + Bleak |
| `repos/engine/`  | roguERGlike-engine  | public, MIT | Godot 4 / GDScript |
| `repos/game/`    | roguERGlike-game    | private | Godot 4 + content |
| `repos/server/`  | roguERGlike-server  | private | Nakama (Go), deferred |

When working on application code, `cd repos/<name>/` first, then run Claude Code from there. It'll find that repo's `CLAUDE.md` and walk up to find this one too.

## House style

- See `INSTRUCTIONS.md` for conventions
- Prefer small, focused commits with `WIP:` prefix during a session, squashed before merge
- Sign commits (`git commit -S`)
- Never commit secrets or personal ride data (`.gitignore` covers this)
