---
name: context-compression
description: Preserve task continuity across long sessions by using structured summaries, anchored iterative compression, and explicit post-compression recovery rules. Use when sessions exceed context limits, summaries start losing artifact trails, or an agent begins following stale tasks after compression.
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [context, compression, summarization, continuity, memory, agent]
---

# Context Compression

Compression is not about minimizing tokens per request.
It is about minimizing total tokens per finished task.

If compression saves a few tokens now but forces the agent to re-fetch files, re-ask questions, or resume the wrong task, the system got worse.

## When to use

Activate this skill when:
- a session is approaching context limits
- repeated summarization causes drift
- the agent starts forgetting modified files, decisions, or blockers
- a compression boundary happened and the surviving summary may be stale
- the user says the agent is stuck on an old task after compression

## Core idea

Optimize for tokens-per-task, not tokens-per-request.

A good compression strategy preserves the pieces that let work continue immediately:
- current user objective
- recent direction change or correction
- files or artifacts already changed
- decisions already made
- current blockers
- immediate next executable step

## Preferred compression strategy

Use anchored iterative summarization.

Instead of regenerating one giant summary every time:
1. keep a structured summary with fixed sections
2. summarize only the newly truncated span
3. merge it into the existing sections
4. preserve a compact continuity block for recovery

This is usually safer than opaque compression and more stable than repeated full-summary regeneration.

## Required summary structure

Every compression summary should preserve these sections:

```markdown
## Current Objective
[What the user currently wants, stated as of the latest turn]

## Recent Direction Changes
- User corrected X
- Goal changed from A to B

## Files / Artifacts Touched
- path/to/file: what changed
- asset.png: created / selected / rejected

## Decisions Made
- Decision 1
- Decision 2

## Current Blockers
- Missing auth
- Waiting on external system

## Next Executable Step
1. The next concrete action to take
```

If a section has nothing in it, write `None`.
Do not silently drop the section.

## Why compression fails

Compression usually breaks in one of these ways:

1. task drift
   - summary keeps the original task but loses the later redirect
2. artifact loss
   - modified files, URLs, IDs, or error messages vanish
3. continuity collapse
   - summary explains what happened, but not what to do next
4. stale todo capture
   - old task scaffolding survives and gets mistaken for the current goal

## Post-compression recovery rules

When compression has already happened and the surviving context may be wrong, recover in this order:

1. Treat the latest explicit user message as the source of truth.
2. Treat old TODO state and compressed summaries as background only.
3. If the latest user message changed direction, rebuild the task list around that new goal.
4. Restate the active goal in one sentence before continuing work.
5. If old summary state conflicts with the latest user turn, mark the old state as stale and proceed with the latest turn.

Short version:
- latest user turn beats compressed summary
- latest user turn beats old todo state
- restate the current goal before resuming execution

## Continuity block

At every compression boundary, preserve this minimum block:
- current objective
- most recent correction
- touched files/artifacts
- blockers
- next step

If this block is missing, resumption quality will usually collapse.

## Compression triggers

Useful trigger strategies:

| Strategy | Typical trigger | Trade-off |
|---|---|---|
| Fixed threshold | 70–80% context used | Simple, predictable |
| Sliding window | keep last N turns + summary | Stable context budget |
| Task boundary | compress after a completed phase | Cleaner summaries, less predictable |
| Importance-based | compress low-signal spans first | Better quality, more complex |

For most agent workflows, sliding window + structured summary is the safest default.

## Evaluation

Do not judge compression by lexical similarity alone.
Judge it by whether the agent can continue correctly.

Useful probes:
- What is the current user objective?
- What changed most recently?
- Which files were modified?
- What blocker remains?
- What is the next action?

If the agent cannot answer those precisely after compression, the compression failed.

## Practical checklist

- preserve structure, not just prose
- keep the latest user correction visible
- keep file paths and IDs literal
- preserve blockers explicitly
- preserve the next step explicitly
- never let a stale todo list outrank the latest user message

## Example recovery handoff

```markdown
## Current Objective
Publish the updated compression skill to GitHub.

## Recent Direction Changes
- User said the real problem is bad compression recovery, not the original publishing task.
- User asked to turn that fix into a skill and publish it.

## Files / Artifacts Touched
- ~/.hermes/skills/openclaw-imports/context-compression/SKILL.md: added recovery rules
- /Users/aiad/Desktop/claw/public-skills/context-compression-skill/SKILL.md: public bundle

## Decisions Made
- Compression recovery must prioritize the latest user turn over old TODO state.

## Current Blockers
- Need GitHub repo creation/push access.

## Next Executable Step
1. Create or connect a GitHub repo.
2. Push the public skill bundle.
3. Update skill.json with the final raw URL.
```

## One-line rule

Compression is only good if the agent resumes the right task, with the right artifacts, and takes the right next step.
