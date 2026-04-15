---
name: context-compression
description: Preserve task continuity across long sessions by using structured summaries, continuity blocks, timeline checkpoints, TODO state, and post-compression recovery rules. Use when sessions exceed context limits, summaries drift, or the agent starts following stale tasks after compression.
version: 1.2.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [context, compression, summarization, continuity, recovery, planning, timeline, todo, verification]
---

# Context Compression

Compression is not about making context smaller.
Compression is about making work resumable.

If a summary is short but the agent resumes the wrong task, loses touched files, ignores the user's latest correction, or acts before re-anchoring the timeline, the compression failed.

## What problem this skill actually solves

This skill exists to prevent a specific failure pattern:
- the session gets compressed
- the surviving summary keeps old narrative but loses the current objective
- file changes, runtime state, decisions, or pending actions disappear
- stale TODOs survive and hijack the session
- the agent resumes from the wrong branch and reports motion instead of verified progress

In other words, this is not just a token-saving skill.
It is a continuity-preservation skill for long-running agent work.

The practical job is to preserve enough execution state that the agent can continue correctly after compression without making the user restate the task.

If a compression strategy cannot reliably preserve:
- the latest user goal
- what problem is currently being solved
- current task status
- changed artifacts
- blockers
- what counts as verified completion
- the immediate next executable step

then it has failed, even if the token count looks efficient.

## What this skill adds

This skill adds six things to ordinary summarization:

1. a current-objective-first summary format
2. an active-problem field so the real issue does not disappear into narrative
3. a direction-change register so later corrections do not vanish
4. a timeline layer so phase order survives compression
5. a TODO layer so pending work stays explicit and ranked
6. a post-compression recovery protocol that always prioritizes the latest user turn

## When to use

Activate this skill when:
- a session is approaching context limits
- repeated summarization causes drift
- the agent starts forgetting modified files, decisions, blockers, or verification state
- a compression boundary happened and the surviving summary may be stale
- the user says the agent is stuck on an old task after compression
- a task has multiple phases and needs continuity across long execution
- the agent is at risk of resuming from old TODO scaffolding instead of the latest instruction

## Core idea

Optimize for tokens-per-task, not tokens-per-request.

A good compression strategy preserves the pieces that let work continue immediately:
- active problem
- current user objective
- most recent direction change or correction
- why the current work matters
- timeline position
- files or artifacts already changed
- decisions already made
- active TODO state
- current blockers
- verification state
- immediate next executable step

## Failure modes to guard against

Bad compression usually fails in one of these ways:

1. Goal drift
   - the old objective survives, but the latest user redirect is lost
2. Artifact loss
   - files, paths, URLs, commands, error messages, IDs, and identifiers disappear
3. Timeline collapse
   - done / now / next are no longer distinguishable
4. TODO resurrection
   - stale pending items reappear as if still active
5. Verification loss
   - the summary says something was handled without preserving what was actually verified
6. Recovery blindness
   - after a weak handoff, the agent continues confidently instead of re-anchoring on the latest user turn

A usable compression workflow must explicitly defend against all six.

## Preferred strategy

Use anchored iterative summarization.

Instead of regenerating one giant summary every time:
1. keep a structured summary with fixed sections
2. summarize only the newly truncated span
3. merge it into the existing sections
4. preserve a compact continuity block for recovery
5. preserve timeline + TODO state as first-class objects
6. preserve verification state explicitly instead of implying completion

This is usually safer than opaque compression and more stable than repeated full-summary regeneration.

## Required minimum handoff schema

Every compression handoff should preserve these sections:

```markdown
## Active Problem
[one sentence: what is broken / missing / being decided right now]

## Current Objective
[one sentence: what the user currently wants, not what they wanted 50 turns ago]

## Why This Matters
[one sentence: what failure, risk, or outcome this work affects]

## Recent Direction Changes
- User corrected X
- Goal changed from A to B

## Timeline
- done:
- now:
- next:

## Artifacts
- changed:
- referenced:
- pending to inspect:

## Decisions Made
- Decision 1
- Decision 2

## Active TODOs
- [in_progress]
- [pending]
- [blocked]
- [cancelled/stale]

## Verification State
- verified:
- not yet verified:

## Immediate Next Step
[the next concrete tool action to take]
```

If a section has nothing in it, write `None`.
Do not silently drop the section.
If any of these sections are missing, the handoff is incomplete.

## Continuity block

At every compression boundary, preserve this minimum block:
- active problem
- current objective
- most recent correction
- why this matters
- current timeline state
- touched files/artifacts
- active TODOs
- blockers
- verification state
- next step

If this block is missing, resumption quality will usually collapse.

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
- now: refine SKILL.md and recovery schema
- next: verify handoff examples and publish
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
5. blocked work must stay visible instead of being silently dropped

Recommended format:

```markdown
## Active TODOs
- [in_progress] Rewrite SKILL.md to define compression failure modes clearly
- [pending] Add required handoff schema and continuity block
- [pending] Verify public and installed copies match
- [cancelled] Continue old branch without fixing compression logic
```

## Verification state design

A readable summary is not enough.
The handoff must also distinguish what is actually verified from what is merely prepared, drafted, or assumed.

Recommended format:

```markdown
## Verification State
- verified: installed skill patched and saved
- verified: public skill patched and saved
- not yet verified: runtime compression prompt updated
- not yet verified: real long-session recovery tested end-to-end
```

Without this section, compression often turns preparation into fake completion.

## Execution protocol

Run compression and recovery as gates, not as loose advice.

### Gate 1: Pre-compression

Before compressing, the agent should be able to answer all of these explicitly:
- What problem is being solved right now?
- What is the current objective?
- What changed most recently?
- What is the current `in_progress` TODO?
- What has actually been verified?
- What is the next executable step?

If any answer is missing or vague:
1. rebuild the continuity block first
2. keep literal paths / IDs / blockers visible
3. keep verification state explicit
4. do not compress yet

### Gate 2: Post-compression validation

Immediately after generating a compressed summary, verify that the summary still preserves:
- active problem
- current objective
- most recent direction change
- why this work matters
- done / now / next timeline state
- active TODO state
- touched files / artifacts / IDs
- blockers
- verification state
- next executable step

If any required item is missing, contradictory, or over-abstracted:
1. mark the compression as failed
2. regenerate the structured summary
3. do not resume execution from the failed summary

### Gate 3: Resume-before-action

Before taking the next tool or execution step after a compression boundary, restate:
1. one-line active problem
2. one-line current objective
3. any stale-summary conflicts
4. rebuilt active TODOs
5. what is verified vs not verified
6. next executable step

If the latest user turn conflicts with the compressed summary, the conflict must be resolved before execution continues.

### Gate 4: Recovery fallback

If the agent cannot confidently recover the live task state:
1. stop execution on the old branch
2. mark the surviving summary as incomplete or stale
3. reconstruct continuity from the latest user turn + literal artifacts + remaining TODO state
4. if recovery is still impossible, ask for clarification instead of guessing

## Post-compression recovery rules

When a compression boundary has already happened and the surviving context is imperfect, the agent must not blindly obey stale summaries, old TODOs, or earlier task scaffolding.

Apply this recovery order:

1. Treat the user's latest explicit message as the current source of truth.
2. Reinterpret any surviving TODO list as background only, not as the active objective.
3. If the latest user message changes direction, immediately rebuild the task list around the new goal.
4. In the next response, explicitly state the active goal in one sentence before continuing execution.
5. If old summary state conflicts with the latest message, prefer the latest message and mark the older state as stale.
6. Re-anchor the timeline so the agent knows what is done, what is in progress now, and what is next.
7. Distinguish verified completion from preparation before reporting progress.

Short version:
- latest user turn beats compressed summary
- latest user turn beats old todo state
- current objective must be restated after compression before resuming work
- timeline must be re-anchored before multi-step execution resumes
- verified completion beats optimistic narrative

## Edge cases

Handle these explicitly instead of relying on generic summarization:

1. Repeated user redirects
   - preserve both the latest change and the latest stable objective
   - if they differ, follow the latest explicit instruction

2. Multi-track work
   - distinguish primary work, parked work, and cancelled work
   - do not flatten parallel threads into one ambiguous TODO list

3. No-file but high-decision sessions
   - if no files changed, preserve key decisions, external URLs, IDs, errors, and selected options as artifacts
   - do not let an empty file section imply that nothing important happened

4. Summary-vs-latest-turn conflict
   - state the conflict explicitly
   - mark stale summary items as stale
   - continue with the latest user turn, not the older summary

5. Incomplete continuity
   - if the current objective or next step cannot be recovered reliably, do not guess
   - pause, rebuild, or ask

6. Verification ambiguity
   - if something was drafted but not checked, mark it as not yet verified
   - do not compress “prepared” and “verified” into the same status

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
- What problem is being solved right now?
- What is the current user objective?
- What changed most recently?
- What phase are we in?
- Which files were modified?
- What TODO is actively in progress?
- What blocker remains?
- What has actually been verified?
- What is the next action?

If the agent cannot answer those precisely after compression, the compression failed.

## Anti-patterns

Avoid these failure modes:
- letting old TODO state outrank the latest user turn
- continuing execution before restating the current objective
- collapsing done / now / next into one prose blob
- replacing literal paths, IDs, or errors with vague abstractions
- treating a structurally complete summary as trustworthy when its contents conflict with the latest turn
- guessing through incomplete continuity instead of rebuilding or asking
- reporting something as done when only the draft or setup exists

## Practical checklist

- preserve structure, not just prose
- keep the latest user correction visible
- keep the active problem visible
- keep file paths and IDs literal
- preserve blockers explicitly
- preserve the next step explicitly
- preserve timeline state explicitly
- preserve ranked TODO state explicitly
- preserve verification state explicitly
- never let a stale todo list outrank the latest user message

## Example recovery handoff

```markdown
## Active Problem
The session was compressed and the surviving state may still over-weight old task scaffolding.

## Current Objective
Upgrade the context-compression skill so it fully defines the real failure modes and recovery contract.

## Why This Matters
If the skill only explains summarization theory, future sessions will keep drifting after compression.

## Recent Direction Changes
- User said the skill does not explain what problem it solves.
- User then said the skill still had not actually been updated.
- User clarified the target is the context-compression skill itself, and it is still not complete enough.

## Timeline
- done: inspected installed and public copies
- now: merge and strengthen both copies
- next: verify the rewritten content matches in both places

## Artifacts
- changed: ~/.hermes/skills/openclaw-imports/context-compression/SKILL.md
- changed: /Users/aiad/Desktop/claw/public-skills/context-compression-skill/SKILL.md
- referenced: prior compressed-session failure pattern
- pending to inspect: runtime compression prompt outside the skill file

## Decisions Made
- This skill must define continuity preservation, not just token reduction.
- Verification state must survive compression explicitly.

## Active TODOs
- [in_progress] Rewrite the skill with active problem, failure modes, handoff schema, and verification rules
- [pending] Verify installed and public copies match
- [pending] Evaluate whether runtime prompt also needs alignment
- [cancelled/stale] Continue discussing without patching the actual skill files

## Verification State
- verified: installed and public files were inspected
- not yet verified: rewritten files saved and re-read
- not yet verified: runtime compaction prompt changed

## Immediate Next Step
Write the merged skill content into both installed and public SKILL.md files, then re-read them to confirm.
```

## One-line rule

Compression is only good if the agent resumes the right task, with the right artifacts, on the right timeline, with honest verification state, and takes the right next step.
