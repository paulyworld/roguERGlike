# How to Update HANDOFF

Quick reference for end-of-session HANDOFF updates. Should take **5 minutes or less**. If it's taking longer, you're writing too much.

## When to do it

At the **end of every session** that changed anything — code, docs, or decisions. Before closing the chat, before `git push`, before switching interfaces.

If you skip this once, no harm done. If you skip it three sessions in a row, you've lost the thread.

## What to update

| You touched | Update |
|---|---|
| Multiple repos, or cross-repo concerns | Umbrella `HANDOFF.md` |
| A specific repo's code or docs | That repo's `HANDOFF.md` |
| Both | Both. Umbrella first. |

You don't always need to update the umbrella. If a session was 100% inside `repos/sidecar/`, only update sidecar's HANDOFF.

## The five fields

Open the HANDOFF and overwrite these five things. Everything else stays.

### 1. Last updated
Today's date in `YYYY-MM-DD`.

### 2. Last session log
Filename of the session log you're about to write. Convention: `YYYY-MM-DD-<short-slug>.md` in `docs/sessions/` (umbrella).

### 3. Current branch
The active feature branch — or `none — on develop` if you finished and merged.

### 4. Current focus
**One sentence.** What is the project working on *right now*? Not what was done, not what's next. Present tense, immediate.

Good: `Building sidecar mock mode with slider UI.`
Bad: `We did some work on the sidecar and also fixed a small bug and added a test for the event schema.`

### 5. Entry point for next session
**One sentence in quotes.** The first concrete action the next session should take. Specific enough that a new Claude session can start without rebuilding context.

Good: `"Add Pydantic validation to the mock-mode event emitter, then verify the engine's effort_bridge receives all event types."`
Bad: `"Continue working on the sidecar."`

## The three lists

Then update three short lists:

### "Where we are"
Two or three sentences. The current state of the codebase. What works. What was the last meaningful change. **Edit the existing text** — don't append.

### "What's next (immediate)"
Ordered list, max 3 items. The next 1–3 concrete actions. Each item small enough to start without further planning.

### "Open threads"
Bullet list. Things deferred, questions raised, known issues introduced. Reference session logs by filename if context matters. This list grows; prune items as they get resolved.

## Then write the session log

Copy `templates/SESSION-LOG.md.template` (in the umbrella) to `docs/sessions/YYYY-MM-DD-<slug>.md` and fill it in. The template has its own structure. This is the **historical record** — HANDOFF is the cover sheet, session log is the detail.

If the session was tiny (a typo fix, a one-line tweak), skip the session log. HANDOFF update is enough.

## Then commit

```bash
git add HANDOFF.md docs/sessions/YYYY-MM-DD-<slug>.md
git commit -S -m "docs: handoff after <short description>"
git push
```

## What NOT to do

- Don't append to HANDOFF — overwrite the five fields each time. HANDOFF is a snapshot, not a log.
- Don't write a novel. If a field needs more than 2 sentences, the detail belongs in the session log.
- Don't update HANDOFF mid-session. Do it at the end, after you know what actually happened.
- Don't leave it for "tomorrow." Tomorrow you won't remember.
- Don't fill in fields you can't fill in honestly. If you don't know what's next, write "TBD — needs planning session" and that's fine.

## If HANDOFF is already stale

Don't try to reconstruct what happened across the missing sessions. Just update it to reflect *now* and move on. Use the session log of the current session to capture what's going on. Lost history is lost; don't compound it by faking detail.

## If you're switching interfaces mid-feature

Update HANDOFF before switching. The point of the HANDOFF is to be the seam between Claude chat, Claude Code, and Cowork. If it's stale, the seam leaks.

---

**One-line summary:** overwrite the five fields, edit the three lists, write a session log if the session was non-trivial, commit, done.
