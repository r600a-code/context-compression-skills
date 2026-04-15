# context-compression

A cross-agent skill for long-session context compression.

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

## Agent compatibility matrix

| Agent | Compatibility | Notes |
|---|---|---|
| Claude Code | High | Native skills / SKILL.md support; closest match to the current structure |
| Cline | High | Supports Skills and SKILL.md; very low migration cost |
| Cursor | High | Supports Skills / Rules / Subagents; core content transfers cleanly |
| Codex | Medium-high | Supports project-local skills, but packaging and directory layout need adaptation |
| Gemini CLI | Medium-high | Core method transfers well, usually via GEMINI.md / .gemini-style packaging |
| OpenHands | Medium-high | Supports skills and AGENTS.md, but works best with an OpenHands-style wrapper |
| Hermes | High | Also a strong fit; listed last because the repository aims to serve the broader open-source agent ecosystem first |

## How to think about this repository

This is not a format owned by one agent.

It is better understood as an open-source method:
- serve the broader agent ecosystem first
- avoid locking the method into a single product
- encourage adaptation and evolution across different agent runtimes

What actually transfers across agents:
- preserving the current objective after compression
- latest user turn beats stale summary
- timeline (`done / now / next`)
- active TODOs
- blocker / next action

What usually needs an agent-specific wrapper:
- install command
- directory layout
- frontmatter / metadata
- auto-trigger mechanism

So the more accurate framing is:
this is a cross-agent context engineering skill built in an open-source spirit, meant to be reused, adapted, and redistributed across agent ecosystems rather than kept as a single-platform package.

## One line

Good compression is not shorter context.
It is context that still lets the agent continue the right work.
