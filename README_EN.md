# context-compression

A Hermes skill for long-session context compression.

It is not only about making context shorter. It is about keeping the agent on the right task after compression.

## What problem it solves

After compression, long sessions often fail in the same ways:
- the earliest task survives, but later redirections disappear
- files, URLs, IDs, and error messages get lost
- the summary explains what happened, but not what to do next
- stale TODO state gets mistaken for the live objective
- multi-phase work loses sequence

This skill is designed to keep compressed context executable.

## What it preserves

Compared with ordinary summarization, it forces the summary to preserve:
- current objective
- latest direction change
- timeline state (`done / now / next`)
- active TODOs
- touched files / artifacts
- blockers
- next executable step

## Key design ideas

### Current objective wins
After a compression boundary, the user's latest explicit message has highest priority.

Core rules:
- latest user turn beats compressed summary
- latest user turn beats old todo state

### Timeline
This skill requires a minimal timeline:
- done
- now
- next

### TODO
Active work should survive as ranked TODO state.

Rules:
- only one item should be `in_progress`
- when the user redirects the task, TODO state must be rebuilt
- stale or cancelled work should not silently resurrect

## Good use cases

- very long sessions
- multi-phase build / publishing / writing workflows
- tasks that frequently cross compression boundaries
- agents that need to preserve the main line across many turns

## Files
- `SKILL.md` — main skill
- `skill.json` — install metadata
- `README.md` — Chinese readme
- `README_EN.md` — English readme

## Install

```bash
npx skills add <owner>/<repo>
```

Or copy the folder into your local skills directory.

## One line

Good compression is not shorter context.
It is context that still lets the agent continue the right work.
