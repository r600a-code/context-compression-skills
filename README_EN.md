# context-compression

A public Hermes skill for continuity-safe context compression.

This skill is not only about making context shorter.
It is about preventing the agent from resuming the wrong work after compression.

## What problem it solves

Long sessions usually break in a few predictable ways after compression:
- the earliest task survives, but later redirections disappear
- files, URLs, IDs, and error messages get lost
- the summary explains what happened, but not what to do next
- stale TODO state gets mistaken for the live objective
- multi-phase work loses sequence, so the agent no longer knows what is done, current, and next

This skill is designed to keep compressed context executable, not merely readable.

## What it adds

Compared with ordinary summarization, it forces the summary to preserve:
- current objective
- latest direction change
- timeline state (`done / now / next`)
- active TODOs
- touched files / artifacts
- blockers
- next executable step

In other words: it compresses the planning spine together with the narrative.

## Key design ideas

### 1. Current objective wins
After a compression boundary, old summaries and stale TODO state must never outrank the user's latest explicit message.

Core rules:
- latest user turn beats compressed summary
- latest user turn beats old todo state

### 2. Timeline element
This skill requires a timeline, not just a fact dump.

Minimal timeline:
- done
- now
- next

That preserves execution order across compression.

### 3. TODO element
This skill requires active work to survive as ranked TODO state rather than scattered prose.

Rules:
- only one item should be `in_progress`
- when the user redirects the task, TODO state must be rebuilt
- stale or cancelled work should not silently resurrect

## Good use cases

- very long sessions
- multi-phase build / publishing / writing workflows
- tasks that frequently cross compression boundaries
- agents that need to preserve the main line across many turns
- situations where the user already complained that the agent resumed an outdated task

## Files
- `SKILL.md` — main skill
- `skill.json` — install metadata
- `README.md` — Chinese readme
- `README_EN.md` — English readme

## Install

Planned install style:

```bash
npx skills add <owner>/<repo>
```

Or copy the folder into your local skills directory.

## After publishing

Once the repository is on GitHub, replace the placeholder URLs in `skill.json` with the final links:
- `https://github.com/<owner>/context-compression-skill`
- `https://raw.githubusercontent.com/<owner>/context-compression-skill/main/SKILL.md`

## One line

Good compression is not shorter context.
It is context that still lets the agent continue the right work.
