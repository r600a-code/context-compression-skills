---
name: context-compression
description: Preserve task continuity across long sessions by using structured summaries, explicit planning artifacts, timeline checkpoints, and post-compression recovery rules. Use when sessions exceed context limits, summaries drift, or the agent starts following stale tasks after compression.
version: 1.1.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [context, compression, summarization, continuity, memory, planning, timeline, todo]
---

# Context Compression

Compression is not about making context smaller.
Compression is about making work resumable.

If a summary is short but the agent resumes the wrong task, loses touched files, or ignores the user's latest correction, the compression failed.

## What problem this skill solves

Long sessions usually fail in the same pattern:
- the early task survives, but the latest task change disappears
- modified files, URLs, IDs, and blockers get dropped
- the summary explains what happened, but not what should happen next
- old TODO state becomes louder than the user's latest turn
- after compression, the agent continues a dead branch and reports fake progress

This skill fixes that by making compression carry planning structure, not just prose.

## What this skill adds

This skill adds five things to ordinary summarization:

1. a current-objective-first summary format
2. a direction-change register so later corrections do not vanish
3. a timeline layer so phase order survives compression
4. a TODO layer so pending work stays explicit and ranked
5. a post-compression recovery protocol that always prioritizes the latest user turn

## When to use

Activate this skill when:
- a session is approaching context limits
- repeated summarization causes drift
- the agent starts forgetting modified files, decisions, or blockers
- a compression boundary happened and the surviving summary may be stale
- the user says the agent is stuck on an old task after compression
- a task has multiple phases and needs continuity across long execution

## Core idea

Optimize for tokens-per-task, not tokens-per-request.

A good compression strategy preserves the pieces that let work continue immediately:
- current user objective
- most recent direction change or correction
- timeline position
- files or artifacts already changed
- decisions already made
- active TODO state
- current blockers
- immediate next executable step

## Preferred strategy

Use anchored iterative summarization.

Instead of regenerating one giant summary every time:
1. keep a structured summary with fixed sections
2. summarize only the newly truncated span
3. merge it into the existing sections
4. preserve a compact continuity block for recovery
5. preserve timeline + TODO state as first-class objects

This is usually safer than opaque compression and more stable than repeated full-summary regeneration.

## Required summary structure

Every compression summary should preserve these sections:

```markdown
## Current Objective
[What the user currently wants, stated from the latest turn]

## Recent Direction Changes
- User corrected X
- Goal changed from A to B

## Timeline
- T1: What phase is already complete
- T2: What phase is in progress
- T3: What phase is next

## Active TODOs
- [in_progress] Current highest-priority action
- [pending] Next action
- [blocked] Waiting on auth / approval / external system

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
5. phase flattening
   - the work loses sequence, so the agent cannot tell what is done, current, and next

## Timeline design

A timeline makes compression resumable across multi-step work.

Do not preserve only facts.
Preserve order.

Use a minimal three-state timeline:
- done
- now
- next

Example:

```markdown
## Timeline
- done: public skill bundle drafted
- now: refine SKILL.md and README structure
- next: push to GitHub and backfill real URLs
```

For larger projects, expand to milestone form:

```markdown
## Timeline
- Phase 1 / Discovery: complete
- Phase 2 / Drafting: complete
- Phase 3 / Verification: in progress
- Phase 4 / Publish: pending
```

The timeline prevents compression from turning a process into a shapeless blob.

## TODO design

Compression should preserve active work as ranked TODO state, not as scattered notes.

Rules:
1. only one item should be `in_progress`
2. TODO priority should follow the latest user objective, not historical inertia
3. cancelled or stale tasks must remain visible long enough to avoid accidental resurrection
4. if the user redirects the work, rebuild TODO state immediately

Recommended format:

```markdown
## Active TODOs
- [in_progress] Rewrite public SKILL.md to explain the failure mode clearly
- [pending] Add bilingual README files
- [pending] Push repo to GitHub
- [cancelled] Continue old publishing path without fixing compression logic
```

## Post-compression recovery rules

When compression has already happened and the surviving context may be wrong, recover in this order:

1. Treat the latest explicit user message as the source of truth.
2. Treat old TODO state and compressed summaries as background only.
3. If the latest user message changed direction, rebuild the task list around that new goal.
4. Restate the active goal in one sentence before continuing work.
5. If old summary state conflicts with the latest user turn, mark the old state as stale and proceed with the latest turn.
6. Re-anchor the timeline so the agent knows what is done, now, and next.

Short version:
- latest user turn beats compressed summary
- latest user turn beats old todo state
- current goal must be restated before resuming execution
- timeline must be re-anchored before multi-step work continues

## Continuity block

At every compression boundary, preserve this minimum block:
- current objective
- most recent correction
- current timeline state
- touched files/artifacts
- active TODOs
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
- What phase are we in?
- Which files were modified?
- What TODO is actively in progress?
- What blocker remains?
- What is the next action?

If the agent cannot answer those precisely after compression, the compression failed.

## Practical checklist

- preserve structure, not just prose
- keep the latest user correction visible
- keep file paths and IDs literal
- preserve blockers explicitly
- preserve the next step explicitly
- preserve timeline state explicitly
- preserve ranked TODO state explicitly
- never let a stale todo list outrank the latest user message

## Example recovery handoff

```markdown
## Current Objective
Publish the updated compression skill to GitHub.

## Recent Direction Changes
- User said the real problem is bad compression recovery, not the original publishing task.
- User asked to turn that fix into a skill and publish it.
- User then asked for bilingual README files and a stronger planning structure.

## Timeline
- done: public bundle initialized
- now: rewrite skill and readmes with timeline + TODO support
- next: create GitHub repo and push

## Active TODOs
- [in_progress] Upgrade public SKILL.md
- [pending] Add README.md and README_EN.md
- [pending] Push to GitHub

## Files / Artifacts Touched
- ~/.hermes/skills/openclaw-imports/context-compression/SKILL.md: added recovery rules
- /Users/aiad/Desktop/claw/public-skills/context-compression-skill/SKILL.md: public bundle
- /Users/aiad/Desktop/claw/public-skills/context-compression-skill/README.md: Chinese README
- /Users/aiad/Desktop/claw/public-skills/context-compression-skill/README_EN.md: English README

## Decisions Made
- Compression recovery must prioritize the latest user turn over old TODO state.
- Timeline and TODO state must survive compression.

## Current Blockers
- Need GitHub repo creation/push access.

## Next Executable Step
1. Create or connect a GitHub repo.
2. Push the public skill bundle.
3. Update skill.json with the final raw URL.
```

## One-line rule

Compression is only good if the agent resumes the right task, with the right artifacts, on the right timeline, and takes the right next step.
