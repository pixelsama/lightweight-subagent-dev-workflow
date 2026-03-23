---
name: lightweight-subagent-dev-workflow
description: >-
  Use for implementation-oriented coding requests by default. Trigger when the
  user asks Codex to build a feature, fix a bug, refactor code, add or update
  tests, wire a workflow, or otherwise modify code to deliver working behavior.
  This skill provides a lightweight end-to-end development workflow for coding
  tasks: the main agent keeps ownership of requirements, plan, integration,
  verification, and final acceptance, and may use explorer, worker, and
  reviewer subagents only when complexity or risk justifies them. Do not use
  for pure brainstorming, explanation-only requests, reviews with no
  implementation, or tiny edits where extra workflow would add more
  coordination cost than value.
---

# Lightweight Subagent Development Workflow

Use this skill for real coding tasks without forcing every task through a heavy pipeline. Subagents are part of the toolbox, not the point of the workflow.

## Fast Path

If the task is small and low risk, stop here and use the short path:

1. Locate the relevant code
2. Implement the change in the main agent
3. Run the relevant checks
4. Self-review the final diff
5. Report any risks or gaps clearly

Default to no subagents on this path.

## Core Rules

1. Keep the main agent in charge of requirements, tradeoffs, plan, integration, and final output.
2. Delegate only bounded work with clear inputs, clear file ownership, and a clear definition of done.
3. Do not create a subagent for every phase by default.
4. Always perform a review step before finishing.
5. Treat a dedicated review subagent as conditional, not mandatory.
6. If delegation creates more coordination cost than execution benefit, stay in the main thread.

## Cost Awareness

Subagents have a real cost:

- More coordination turns
- More context packing and unpacking
- More token usage
- More chances for duplicated or conflicting work

Do not create a subagent unless it is likely to reduce total work, improve quality, or lower risk enough to justify that cost.

## Default Intensity

- Small task: `0` subagents by default
- Medium task: `1-2` subagents
- Large task: `3-5` total subagents is usually enough
- Parallel coding workers: usually at most `2`

If unsure, choose the smaller number.

## Task Sizing

### Small

Usually:

- Single bug fix or small feature
- One obvious write location
- Few files
- Low regression risk

Default approach:

- Main agent does discovery, implementation, verification, and self-review
- No dedicated reviewer unless the change is high risk

### Medium

Usually:

- Code location is not obvious
- A few modules are involved
- Tests need thought
- The change benefits from a second set of eyes

Default approach:

- Optional `explorer` for codebase discovery
- Main agent or one `worker` for implementation
- Optional `reviewer` for final review

### Large

Usually:

- Multiple modules or layers
- Work can be split into disjoint write scopes
- There is meaningful regression risk
- Integration needs coordination

Default approach:

- `1-2` explorers if discovery can be parallelized cleanly
- `1-2` workers for disjoint implementation tracks
- `1` reviewer near the end
- Main agent integrates and resolves conflicts

## Workflow

### 1. Understand and Scope

The main agent should:

1. Restate the request internally in concrete engineering terms.
2. Read enough code to understand the likely surface area.
3. Classify the task as small, medium, or large.
4. Decide whether subagents are needed at all.
5. Produce a short plan, usually no more than 5 steps.

Do not delegate immediately just because subagents are available.

### 2. Discovery

Create an `explorer` only when one of these is true:

- The code location is unclear
- The impact surface is unclear
- There are multiple candidate implementations
- A risky dependency chain must be mapped before coding

Expected explorer output:

- Relevant files and modules
- Key call paths or dependencies
- Constraints or hidden risks
- Suggested implementation options
- Suggested tests or checks

The explorer should mostly read, not write.

### 3. Implementation

Keep implementation in the main agent unless parallel work is clearly worth it.

Create `worker` subagents only when:

- The work can be split by module, directory, API boundary, or feature slice
- Each worker can own a mostly disjoint file set
- The interface between slices is already known or easy to define

Do not parallelize implementation when:

- Workers would touch the same files
- One slice depends tightly on another slice's unfinished design
- The task is short enough that coordination dominates execution

When creating a worker, always specify:

- The exact goal
- Owned files or modules
- Constraints and non-goals
- Required tests or checks
- Expected handoff summary

Tell the worker it is not alone in the codebase and must not revert unrelated changes.

### 4. Verification

Verification is mandatory even when no subagents are used.

Verification answers: "Does the change work and did we check it?"

The main agent should ensure the task includes the relevant subset of:

- Unit or integration tests
- Type checks
- Lint or formatting checks
- Manual behavior verification when needed

Create a dedicated verification pass only when:

- The testing strategy is non-obvious
- Parallel workers were merged and need independent inspection
- Relevant checks are expensive enough that separating execution and integration helps

### 5. Review

Always do a review step.

Review answers: "Is this the right change to ship, given the requirement, risks, and code quality?"

The review step can be:

- Main-agent self-review for small and low-risk changes
- Dedicated `reviewer` subagent for medium or high-risk changes

Use a dedicated reviewer when any of these apply:

- Authentication, authorization, permissions, or secrets
- Database schema, migration, or destructive data logic
- Public API or shared library behavior changes
- Concurrency, caching, queues, retries, or distributed coordination
- Large cross-file diffs
- Weak test coverage
- Parallel worker outputs were integrated
- The main implementation involved substantial uncertainty

Reviewer focus:

- Behavioral regressions
- Missing edge cases
- Incomplete tests
- Inconsistencies with the stated plan
- Maintainability or readability issues that materially affect the change

Do not use the reviewer as a ceremonial rubber stamp.

## Failure Handling

Use these defaults when the happy path breaks:

- Explorer results are incomplete or conflicting: the main agent decides, asks for one targeted follow-up only if needed, and does not keep spawning explorers to avoid a decision.
- Worker output is partially useful but flawed: prefer main-agent repair or a narrowly scoped follow-up over replacing the whole worker run.
- Worker output misses the requested scope: the main agent either finishes the missing piece directly or sends one corrective follow-up with tighter boundaries.
- Parallel workers touch the same files unexpectedly: stop parallel implementation and let the main agent integrate or repartition the work.
- Reviewer finds a serious bug or regression risk: return to implementation, fix the issue, then re-run the relevant checks before closing.
- Verification fails for unclear reasons: the main agent should own diagnosis first, then delegate only the bounded part that is actually blocking progress.

## Handoff Format For Subagents

Keep subagent prompts compact and structured. Include:

1. Goal
2. Why this subtask exists
3. Owned files or search area
4. Constraints
5. Non-goals
6. Expected output
7. Required checks

Prefer a short, task-local handoff over dumping the whole parent context.

Example handoff:

```text
Goal: Add keyboard controls to the settings modal and preserve existing mouse behavior.
Why this subtask exists: The feature is scoped to one UI area and can be implemented without touching backend code.
Owned files or search area: src/components/settings-modal.tsx, src/hooks/use-hotkeys.ts
Constraints: Preserve current props, do not change analytics events, follow existing accessibility patterns.
Non-goals: Do not redesign the modal or refactor unrelated state management.
Expected output: Code changes in the owned files, a short summary of behavior changes, and any follow-up risks.
Required checks: Run the modal test file and the relevant frontend lint command if available.
```

## Review Policy

Use this policy by default:

- `Review step`: always
- `Review subagent`: conditional

This means every implementation must be checked before handoff, but not every task deserves a separate review agent.

## Decision Heuristics

Use these defaults:

- If the task is small and low risk, stay local.
- If the task is unclear, add one explorer before coding.
- If the task is large but separable, add at most two workers.
- If testing is the confusing part, add a bounded verification pass instead of a general reviewer.
- If the task is risky or broadly scoped, add one reviewer near the end.
- If in doubt between more delegation and less delegation, choose less.

## Anti-Patterns

Avoid these:

- One subagent per phase for every task
- Mandatory reviewer on trivial diffs
- Parallel workers with overlapping write scopes
- Sending huge undifferentiated context dumps to subagents
- Using subagents to hide indecision in the main agent
- Treating review as complete just because another agent looked at it

## Definition Of Done

Before finishing, the main agent should confirm:

1. The requested behavior is implemented
2. The chosen approach still matches the requirement
3. Relevant checks were run, or any gaps were reported
4. The final diff was reviewed
5. Risks, assumptions, and follow-up items are summarized clearly
