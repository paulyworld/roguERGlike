# First-Time Setup

Bootstrap order: **umbrella first**, then sidecar, engine, game, (server later).

## Prerequisites

```bash
# GitHub CLI
gh auth login

# Git LFS (needed for the game repo eventually)
git lfs install

# Signed commits (one-time)
gh auth setup-git
git config --global commit.gpgsign true
# Or for SSH signing:
# git config --global gpg.format ssh
# git config --global user.signingkey ~/.ssh/id_ed25519.pub
```

Pick a working directory: e.g. `~/dev/`. The umbrella and the four code repos will be siblings inside it.

## Step 1: Umbrella

```bash
cd ~/dev
mkdir roguERGlike && cd roguERGlike
# Copy in the umbrella skeleton files (INSTRUCTIONS.md, CLAUDE.md, HANDOFF.md, templates/, docs/sessions/)

# Create the gitignored repos/ directory
mkdir repos
echo "repos/" > .gitignore
echo "*.log" >> .gitignore
echo ".DS_Store" >> .gitignore

git init -b main
git add .
git commit -S -m "chore: initial umbrella skeleton"
git checkout -b develop
gh repo create roguERGlike --public --source=. --remote=origin --description "Project umbrella — conventions, sessions, scripts"
git push -u origin main develop
```

## Step 2: Sidecar (public)

```bash
cd repos
# Copy the sidecar skeleton into repos/sidecar/
cd sidecar

git init -b main
git add .
git commit -S -m "chore: initial sidecar skeleton"
git checkout -b develop
gh repo create roguERGlike-sidecar --public --source=. --remote=origin --description "BLE bridge and telemetry pipeline for smart trainers and HR sensors"
git push -u origin main develop

# Branch protection. Use -F (typed) not -f (string) — GitHub's protection
# endpoint rejects string "true"/"false"/"null"/numbers. The empty contexts
# array is also required, hence the `[contexts][]` line.
gh api -X PUT repos/:owner/roguERGlike-sidecar/branches/main/protection \
  -F 'required_status_checks[strict]=true' \
  -F 'required_status_checks[contexts][]' \
  -F enforce_admins=false \
  -F 'required_pull_request_reviews[required_approving_review_count]=0' \
  -F restrictions=null

# Local hooks
pip install pre-commit
pre-commit install
cd ..
```

## Step 3: Engine (public)

```bash
# Copy the engine skeleton into repos/engine/
cd engine

git init -b main
git add .
git commit -S -m "chore: initial engine skeleton"
git checkout -b develop
gh repo create roguERGlike-engine --public --source=. --remote=origin --description "Godot 4 deckbuilder framework"
git push -u origin main develop
cd ..
```

## Step 4: Game (private)

```bash
# Copy the game skeleton into repos/game/
cd game

git init -b main
git add .
git commit -S -m "chore: initial game skeleton"
git checkout -b develop
gh repo create roguERGlike-game --private --source=. --remote=origin
git push -u origin main develop

# Submodule will be added later, once engine has a v0.1.0 tag:
# git submodule add https://github.com/<you>/roguERGlike-engine addons/roguerglike_engine
# cd addons/roguerglike_engine && git checkout v0.1.0 && cd ../..
# git add .gitmodules addons/roguerglike_engine
# git commit -S -m "chore(deps): pin engine submodule to v0.1.0"

# Git LFS
git lfs install
cd ..
```

## Step 5: Server (private, deferred)

Wait until Phase 5. Create the repo when work actually begins.

---

## After bootstrap

Update `HANDOFF.md` in the umbrella to reflect the new state. Add a session log entry capturing the bootstrap. Commit to umbrella's `develop` branch, push.

## Ongoing workflow

- Work happens on feature branches off `develop`
- PRs target `develop`, CI must pass
- `develop` → `main` only at release points, tagged with semver
- Public repos: every release tagged, CHANGELOG.md updated, GitHub Release created
- Private game repo: bumps engine submodule pin at tagged engine releases

## Build-in-public discipline

Before pushing to public repos, ask:
- Does this commit reveal specific card numbers, names, or designs?
- Does this commit reveal theme, narrative, or art direction?
- If yes → it belongs in the private game repo, not public.

When in doubt, commit privately first and selectively port generic pieces back to public.
