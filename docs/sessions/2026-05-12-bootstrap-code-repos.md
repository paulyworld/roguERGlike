# Session: 2026-05-12 — Bootstrap code repos

**Repo(s):** umbrella + sidecar + engine + game (first init for the three code repos)
**Interface:** Claude Code
**Branch(es):** umbrella `chore/bootstrap-code-repos`; sidecar/engine/game initial commit on `main`, working branch `develop`
**Duration:** ~30 min

## Context loaded

- Umbrella `HANDOFF.md` and `INSTRUCTIONS.md` — confirmed bootstrap is the immediate next step
- `SETUP.md` — followed verbatim for sidecar, engine, game (Steps 2–4); umbrella step already done last session
- `HOW-TO-UPDATE-HANDOFF.md` — for the wrap-up edits
- Each repo's per-skeleton `HANDOFF.md` — all three say "not yet `git init`'d", all three agree on what comes after bootstrap
- Last session log `2026-05-12-repo-setup-and-conventions.md` — confirmed convention decisions are settled

## Decisions made

- **Bootstrap-only scope.** Sidecar Phase 1 mock-mode work is a separate session — keeps this one tight and matches the HANDOFF's stated boundary.
- **Default branch stays `main`** on GitHub for all three new repos (per SETUP.md). Push both `main` and `develop`; PRs target `develop` locally, `develop` → `main` only at release points.
- **Branch protection on `main` for both public repos** (sidecar, engine): requires PR, dismisses stale state on push (`strict: true`), zero required approving reviews (solo dev), enforce_admins off. Game is private — no protection per `INSTRUCTIONS.md`.
- **Game repo gets `git init` + push now, submodule wiring deferred** until engine has a `v0.1.0` tag — matches SETUP.md Step 4.
- **`.claude/` added to umbrella `.gitignore`** alongside other editor/IDE dirs.

## Work done

Repos created on GitHub (under `paulyworld`):
- `roguERGlike-sidecar` — public, MIT, branch protection on `main`, signed initial commit `ca79a6d`
- `roguERGlike-engine` — public, MIT, branch protection on `main`, signed initial commit `60892f8`
- `roguERGlike-game` — private, git LFS active locally, signed initial commit `dc5fe11`

Per-repo state after bootstrap:
- Sidecar: 13 files (CI workflow, pyproject.toml, pre-commit config, event-schema doc, empty Python package, docs). `pre-commit install` ran successfully.
- Engine: 13 files including the working `src/effort/effort_bridge.gd` and the `card.gd` skeleton.
- Game: 8 files (design docs, IDEAS.md, .gitattributes for LFS, README, CLAUDE.md, HANDOFF.md).

Umbrella changes on `chore/bootstrap-code-repos`:
- `.gitignore`: appended `.claude/`
- `HANDOFF.md`: overwrote the five fields; flipped the repo-state table to "bootstrapped, pushed" for sidecar/engine/game; rewrote the entry-point quote
- `docs/sessions/2026-05-12-bootstrap-code-repos.md`: this file
- Commit: `docs: handoff after bootstrapping sidecar, engine, game` (signed)

Tooling installed:
- `pre-commit 4.6.0` (system-wide via pip)
- Sidecar pre-commit hook wired into `.git/hooks/pre-commit`

## Open threads

- **Per-repo HANDOFF refresh** — each of sidecar/engine/game still has `Current branch: none — not yet git init'd`. Left for the start of each repo's first real session (pragmatic; the bootstrap commit itself is the "initial skeleton" milestone).
- **Sidecar Phase 1 — mock mode** (Pydantic event models + WS server + slider UI) — next session's work, on a `feat/mock-mode` branch in `repos/sidecar/`.
- **Engine v0.1.0 tag** — needed before game can pin engine as a submodule. Likely after the engine's first usable card/encounter loop ships.
- **Sub-title for the bike-themed game** — still TBD.
- **Server (Phase 5)** — untouched.
- **Mobile/tablet build path** — long-term.

## Next session entry point

> "In repos/sidecar/ on a feat/mock-mode branch: implement Pydantic v2 event models matching docs/event-schema.md, a WebSocket server on localhost:8421, and a slider UI for power/cadence/HR. Validate end-to-end by pointing the engine's effort_bridge.gd at it."

## Loose notes

- `gh api -f` always serializes values as strings — GitHub's branch-protection endpoint rejected the SETUP.md invocation with HTTP 422 (`"true" is not a boolean`, etc.). Working solution: write the protection body as raw JSON to a temp file and pass via `gh api --input <file>`. SETUP.md should be updated to use this form before the next person hits it. (Filed mentally; not changed this session because scope was bootstrap-only.)
- Piping JSON to `gh api --input -` from PowerShell sent a BOM and HTTP 400'd. The temp-file workaround sidesteps both the BOM and the typed-value issue.
- Signed commits via SSH (`gpg.format=ssh`, `user.signingkey=~/.ssh/id_ed25519.pub`) worked silently on all three initial commits — `git log --show-signature` shows "Good signature" on each.
- `gh repo create … --source=. --remote=origin` configures the remote but does NOT push — you push manually after. SETUP.md got this right.
