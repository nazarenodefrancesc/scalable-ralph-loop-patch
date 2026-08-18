---
name: prd-context-router
version: 1.4.2
description: "Default skill for agentic software development. Use whenever you or a sub-agent need to perform development work on a repository, including planning, coding, debugging, testing, reviewing, refactoring, or continuing previous work. The skill governs PRD structure, task lifecycle, progress tracking, and progressive context loading."
---

# PRD Context Router Workflow

## Purpose

Use this skill when managing a long-running software or research project with agentic workflows.

The documentation system is designed to preserve three properties as the project grows:

- `PRD.md` provides a compact global project overview and macro-task routing index;
- each macro-task has a self-contained specification and bounded execution state;
- agents load detailed context progressively, only for the work they are performing.

The project uses this documentation structure:

```text
PRD.md

prd/
├── T001.md
├── T002.md
└── T003.md

progress/
├── PROGRESS-T001.md
├── PROGRESS-T002.md
└── PROGRESS-T003.md

reports/
├── REPORT-T001-SUB001.md
├── REPORT-T001-SUB002.md
└── REPORT-T002-SUB001.md
```

The structure is intentionally shallow. There are no dedicated Markdown files for sub-tasks. Sub-task specifications live inside the corresponding macro-task file.

The primary design constraint is **bounded working context**: the amount of documentation an agent must load for one unit of work should remain approximately stable as the overall project grows.

---

## Core model

```text
PRD.md
    global project description, invariants, and macro-task index

prd/T001.md
    normative specification of macro-task T001
    including its T001-SUBxxx sub-tasks and acceptance criteria

progress/PROGRESS-T001.md
    bounded current execution state of macro-task T001

reports/REPORT-T001-SUB001.md
    optional detailed evidence/report for work performed on T001-SUB001

current implementation
    actual working tree, code, tests, generated artifacts, and uncommitted changes

Git history
    historical provenance: commits, diffs, reversals, previous decisions, and evolution
```

Keep the distinction between specification, state, implementation, evidence, and provenance:

- **PRD** defines what should be built and how completion is judged.
- **Progress** records the current execution state, blockers, recent meaningful transitions, and next action.
- **Current implementation** is the actual present state of the repository and working tree.
- **Report** documents investigation, implementation details, verification, findings, and decisions when deeper evidence is useful.
- **Git history** records how the project evolved and provides historical provenance.

### Terminology

Use these terms consistently:

- **root PRD**: `PRD.md`;
- **macro-task PRD**: `prd/Txxx.md`;
- **macro-task progress**: `progress/PROGRESS-Txxx.md`;
- **sub-task report**: `reports/REPORT-Txxx-SUByyy*.md`;
- **current implementation**: the present working tree, code, tests, and generated artifacts.

Do not introduce alternative documentation roles unless the project owner explicitly changes the workflow.

---

## Context temperature model

Treat project memory as layers with different loading costs:

```text
PRD.md
    ROUTING CONTEXT

prd/Txxx.md
    WORKING SPECIFICATION

progress/PROGRESS-Txxx.md
    HOT EXECUTION MEMORY

current implementation
    PRESENT REALITY

reports/
    COLD DETAILED MEMORY

Git history
    HISTORICAL PROVENANCE
```

Do not load cold memory unless it is relevant.

An agent working on one sub-task should read only the global routing context, the active macro-task context, and any deeper evidence required for that work.

---

## Project state and memory contract

The current operational state of a project is:

```text
current operational state =
    current specification
    + current progress
    + current implementation
```

Historical provenance is:

```text
historical provenance =
    relevant reports
    + relevant Git history
```

These layers have different authority.

### Current authority

```text
prd/Txxx.md
    normative specification and acceptance criteria

progress/PROGRESS-Txxx.md
    canonical current execution state

PRD.md
    global project map; canonical macro-task priority/dependencies; synchronized macro-task status projection

current working tree / implementation
    actual present implementation
```

### Historical provenance

```text
reports/
    detailed evidence, findings, investigation, and handoff material

Git history
    historical evolution, previous implementations, reversals, and context
```

Git history is not automatically authoritative for the current desired state. Old commits may describe experiments, reverted decisions, obsolete implementations, or incomplete work.

Use Git primarily to answer:

> How and why did the project arrive at the current state?

Do not use an old commit to override an explicit current specification.

If current Markdown, current implementation, and Git history appear inconsistent, preserve the discrepancy, inspect the relevant evidence, determine the latest verified current state, and document the correction rather than silently guessing.

---

## Mandatory identifiers

Use stable, never-reused identifiers:

- macro-task: `T001`
- sub-task: `T001-SUB001`
- report: `REPORT-T001-SUB001`
- progress file: `PROGRESS-T001`

Identifiers must not depend on titles. If a task is renamed, its identifier remains unchanged.

Use zero-padded numeric identifiers consistently:

```text
T001
T002
T010
T001-SUB001
T001-SUB002
```

Do not introduce alternative forms such as `TASK-1`, `task-001`, or `T1` in the same project.

---

## Commit identifier convention

When a commit primarily implements, verifies, fixes, or documents a specific sub-task, include its stable identifier in the commit subject.

Preferred forms:

```text
T001-SUB003: implement tokenizer validation
```

or:

```text
[T001-SUB003] Implement tokenizer validation
```

For macro-task-wide work:

```text
T001: finalize data pipeline integration
```

This makes Git history directly searchable:

```bash
git log --grep="T001-SUB003"
git log --oneline --grep="T001"
```

Do not force a misleading task ID onto unrelated repository maintenance.

---

## Startup procedure

When starting work on a project:

1. Read repository-level instructions such as `AGENTS.md`, `CLAUDE.md`, or equivalent, if present.
2. Read the root `PRD.md`.
3. Inspect the current working tree with:

   ```bash
   git status --short
   ```

4. Identify the active, blocked, or next macro-task primarily from the root `PRD.md` status index and dependencies.
5. Inspect existing uncommitted changes relevant to the selected macro-task. Never overwrite or revert unrelated pre-existing working-tree changes.
6. Read the relevant `prd/Txxx.md`.
7. Read `progress/PROGRESS-Txxx.md`.
8. Locate the relevant `Txxx-SUByyy` specification inside the macro-task PRD.
9. Inspect the current implementation, tests, and files relevant to that sub-task.
10. Read linked reports only when they are relevant to the current work.
11. Inspect Git history only when historical context is needed, preferring task-scoped queries such as:

    ```bash
    git log --oneline --grep="Txxx-SUByyy"
    ```

    or scoped diffs for the relevant files.

`git status --short` is mandatory because the working tree is part of present project reality.

General Git history is not mandatory startup context. Load it progressively when needed to resolve provenance, ambiguity, regressions, prior decisions, or unexplained implementation state.

Do not read every task specification or every report by default. Load context progressively.

Before editing, verify that required paths exist and that IDs and links agree with the index.

Do not guess a task file from a title or silently create a missing specification.

If `PRD.md` points to a missing `prd/Txxx.md` or `progress/PROGRESS-Txxx.md`, treat it as a documentation defect and repair or report it before implementing work whose specification cannot be reliably established.

---

## Task selection rule

When choosing what to work on, use this deterministic order.

### Macro-task selection

1. Explicit user-selected macro-task.
2. Otherwise, consider `IN_PROGRESS` macro-tasks whose dependencies are satisfied and whose work can proceed.
3. Among multiple eligible `IN_PROGRESS` macro-tasks, choose the highest-priority one.
4. If multiple eligible `IN_PROGRESS` macro-tasks have the same priority, choose the lowest stable macro-task ID.
5. If no eligible `IN_PROGRESS` macro-task exists, choose the highest-priority `PLANNED` macro-task whose dependencies are satisfied.
6. If multiple eligible `PLANNED` macro-tasks have the same priority, choose the lowest stable macro-task ID.
7. Do not automatically select `BLOCKED`, `IN_REVIEW`, `DEFERRED`, `COMPLETE`, or `CANCELLED` macro-tasks as new implementation work.
8. Ask the user only when automatic selection could materially change product intent, scope, architecture, or an explicitly stated priority.

### Sub-task concurrency rule

By default, a macro-task may have at most one sub-task in `IN_PROGRESS`.

If the project owner explicitly enables parallel sub-task execution in the root `PRD.md`, the workflow must also define:

- how multiple active sub-tasks are represented in Progress;
- how conflicts and shared files are coordinated;
- how task selection behaves when several sub-tasks are active.

Without that explicit project-wide extension, do not mark a second sub-task `IN_PROGRESS` while another sub-task in the same macro-task is already `IN_PROGRESS`.

### Sub-task selection inside the selected macro-task

1. Explicit user-selected sub-task.
2. Existing `IN_PROGRESS` sub-task.
3. Otherwise, the lowest-ID `PLANNED` sub-task whose dependencies are satisfied.
4. A `BLOCKED`, `DEFERRED`, `CANCELLED`, `COMPLETE`, or `IN_REVIEW` sub-task is not automatically selected as new implementation work.
5. An `IN_REVIEW` sub-task may be resumed only for the review, approval, or verification action explicitly recorded in Progress, not as ordinary implementation work.
6. Ask the user only when the specification itself is ambiguous or automatic selection could materially change intent.

Do not arbitrarily switch away from already active work unless it is blocked, completed, cancelled, deferred, superseded, or explicitly reprioritized.

---

## Root PRD contract

`PRD.md` is the global project map. It should remain short enough to scan quickly.

It normally contains:

- project description;
- objective;
- global scope and non-goals;
- global constraints and invariants;
- macro-task index;
- high-level dependencies;
- global open questions;
- links to task specifications and progress files.

The root PRD must not duplicate the complete content of each macro-task.

Recommended macro-task index:

```markdown
## Macro-task index

| ID | Macro-task | Status | Priority | Dependencies | Specification | Progress |
|---|---|---|---|---|---|---|
| T001 | Data pipeline | IN_PROGRESS | P0 | — | [T001](prd/T001.md) | [PROGRESS-T001](progress/PROGRESS-T001.md) |
| T002 | Training loop | PLANNED | P0 | T001 | [T002](prd/T002.md) | [PROGRESS-T002](progress/PROGRESS-T002.md) |
```

The canonical status vocabulary is:

```text
PLANNED
IN_PROGRESS
BLOCKED
IN_REVIEW
COMPLETE
DEFERRED
CANCELLED
```

These statuses are a closed enum unless the project owner explicitly changes the project-wide workflow in `PRD.md`.

Do not invent, alias, abbreviate, or locally redefine statuses.

### Status semantics

Use each status with the following meaning:

- `PLANNED`: specified and in scope, but execution has not started.
- `IN_PROGRESS`: execution has started and can currently proceed.
- `BLOCKED`: execution cannot currently proceed because of a concrete blocker or unsatisfied dependency.
- `IN_REVIEW`: implementation or investigation work is substantially complete, but final acceptance depends on review, approval, or an external verification step.
- `COMPLETE`: all applicable acceptance criteria are satisfied and required verification has passed.
- `DEFERRED`: intentionally postponed while remaining in scope; it is not complete.
- `CANCELLED`: intentionally removed from active scope.

### Allowed status transitions

Use these transitions unless the project owner explicitly changes the workflow:

```text
PLANNED
  -> IN_PROGRESS
  -> DEFERRED
  -> CANCELLED

IN_PROGRESS
  -> BLOCKED
  -> IN_REVIEW
  -> COMPLETE
  -> DEFERRED
  -> CANCELLED

BLOCKED
  -> IN_PROGRESS
  -> DEFERRED
  -> CANCELLED

IN_REVIEW
  -> IN_PROGRESS
  -> BLOCKED
  -> COMPLETE

DEFERRED
  -> PLANNED
  -> CANCELLED

COMPLETE
  -> IN_PROGRESS      only when explicitly reopened

CANCELLED
  -> PLANNED          only when explicitly reactivated
```

A transition outside this state machine requires an explicit project-wide workflow decision recorded in the root `PRD.md`. Macro-task PRDs, Progress files, and reports must not introduce local status semantics or local transition rules.

The macro-task status in `PRD.md` is a synchronized navigation projection of the canonical current state in `progress/PROGRESS-Txxx.md`.

Macro-task **priority** and **dependencies** are canonical in the root `PRD.md` index because they are global routing data. Do not duplicate them as mutable metadata inside `prd/Txxx.md`.

**Root overview invariant:** reading `PRD.md` alone must always be sufficient to determine the current status, priority, and declared macro-level dependencies of every macro-task, including whether it is `COMPLETE` or not. An agent must not need to read every `progress/PROGRESS-Txxx.md` file merely to obtain the global project overview.

---

## Dependency semantics

Dependencies are readiness constraints, not informational hints.

By default, a dependency is satisfied only when the referenced task is `COMPLETE`.

The following statuses do not satisfy a dependency:

```text
PLANNED
IN_PROGRESS
BLOCKED
IN_REVIEW
DEFERRED
```

A `CANCELLED` dependency is not automatically considered satisfied. If a dependency is cancelled, the dependent specification must be reviewed and the dependency must be explicitly removed, replaced, or otherwise resolved before dependent work is considered ready.

For sub-task dependencies within the same macro-task, apply the same rule: the referenced sub-task must be `COMPLETE` unless the specification is explicitly updated.

Do not infer dependency satisfaction from partial implementation, elapsed time, or apparent irrelevance.

---

## Priority model

Macro-task priorities use this closed default vocabulary:

```text
P0
P1
P2
P3
```

with:

```text
P0 = highest priority
P1 = high priority
P2 = normal priority
P3 = low priority
```

A project may explicitly define a different project-wide priority vocabulary in `PRD.md`, but agents must not invent local priority values.

Sub-tasks do not have independent priority fields by default.

Inside a selected macro-task, ordering is determined by:

1. an explicitly user-selected sub-task;
2. an already active `IN_PROGRESS` sub-task;
3. dependency readiness;
4. stable sub-task identifier, lowest first.

If the project owner needs independent sub-task priorities, they must explicitly extend the workflow and define that field project-wide.

---

## Macro-task sizing rule

A macro-task must remain small enough that:

```text
prd/Txxx.md
+
progress/PROGRESS-Txxx.md
+
relevant implementation context
```

can be loaded together as a practical working context.

A macro-task should represent one coherent deliverable, subsystem, research objective, or implementation phase.

Practical smell test:

```text
5–20 sub-tasks
    normal range for many macro-tasks

20–30 sub-tasks
    review whether the macro-task should be split

30+ sub-tasks
    usually a sign that the macro-task should be reconsidered or split
```

These are heuristics, not hard limits.

Split a macro-task when it accumulates many largely independent sub-tasks, distinct implementation phases, unrelated contracts, or a specification/progress context that no longer fits comfortably within one local working context.

Do not create extra hierarchy merely for aesthetic organization. Keep the structure shallow unless scale actually requires a split.

---

## Macro-task PRD contract

Each file in `prd/` describes one macro-task, for example `prd/T001.md`.

The macro-task PRD is **normative specification only**. It must not carry mutable execution status for individual sub-tasks.

Recommended structure:

```markdown
# T001 — Macro-task title

## Metadata

- Last specification update: YYYY-MM-DD

## Objective

What this macro-task must achieve.

## Scope

### In scope

- ...

### Out of scope

- ...

## Context

Relevant technical or product context.

## Contracts and interfaces

Inputs, outputs, APIs, file formats, permissions, invariants, compatibility requirements, or architectural constraints.

## Sub-tasks

### T001-SUB001 — Sub-task title

- Goal: ...
- Dependencies: ...
- Acceptance criteria:
  - Observable criterion 1
  - Observable criterion 2
- Verification:
  - Command, test, probe, or inspection procedure
- Reports:
  - Add links only to reports that actually exist.

### T001-SUB002 — Sub-task title

- Goal: ...
- Dependencies: ...
- Acceptance criteria:
  - ...
- Verification:
  - ...

## Risks

- ...

## Open questions

- ...
```

Do not add mutable fields such as:

```text
Status: IN_PROGRESS
Status: COMPLETE
```

to individual sub-task specifications.

Do not use completion checkboxes inside the PRD as execution state. Acceptance criteria describe the definition of done; actual completion belongs in the progress file.

The macro-task file is the only specification file for its sub-tasks. Do not create `T001-SUB001.md` unless the project owner explicitly changes this convention.

---

## Sub-task rules

A sub-task must have:

- a stable identifier;
- a clear goal;
- explicit dependencies when applicable;
- acceptance criteria that are observable;
- a verification method;
- optional links to existing reports.

Acceptance criteria must answer:

> How will we know this sub-task is complete?

Prefer:

```markdown
- The loader rejects malformed records with the documented error type.
- A fixture containing valid records completes successfully.
- The verification command exits with status 0.
```

Avoid vague criteria:

```markdown
- Make the loader robust.
- Improve performance.
- Finish the implementation.
```

Acceptance criteria belong in `prd/Txxx.md`.

A report may discover that a criterion needs revision, but the specification must be updated explicitly rather than silently redefining completion inside the report.

---

## Macro-task completion rule

A macro-task may be marked `COMPLETE` only when all of the following are true:

1. the macro-task objective is satisfied;
2. required macro-task-level verification, if any, has passed;
3. every sub-task still in scope is `COMPLETE`;
4. any sub-task no longer required has been explicitly moved to `CANCELLED`, with the specification or scope updated when necessary;
5. there are no remaining `PLANNED`, `IN_PROGRESS`, `BLOCKED`, `IN_REVIEW`, or `DEFERRED` sub-tasks;
6. the macro-task progress file records the final outcome and verification evidence;
7. the root `PRD.md` is updated to `COMPLETE` in the same change.

A `DEFERRED` sub-task prevents macro-task completion because it remains in scope.

A `CANCELLED` sub-task does not prevent macro-task completion only when its removal from scope is intentional and consistent with the macro-task objective and specification.

Do not use `COMPLETE` as a synonym for "no more work is currently planned."

---

## Progress contract

Each macro-task has exactly one dedicated progress file:

```text
progress/PROGRESS-T001.md
```

The progress file records **bounded execution state**, not the full specification and not an unlimited project diary.

Recommended structure:

```markdown
# PROGRESS-T001 — Macro-task title

## Current state

- Status: IN_PROGRESS
- Active sub-task: T001-SUB001
- Blocked by: —
- Last update: YYYY-MM-DD

## Sub-task status

| ID | Status | Last update | Report |
|---|---|---|---|
| T001-SUB001 | IN_PROGRESS | YYYY-MM-DD | — |
| T001-SUB002 | PLANNED | — | — |

## Current blockers

- ...

## Decisions still relevant

- ...

## Recent events

### YYYY-MM-DD — T001-SUB001 started

- ...

### YYYY-MM-DD — Verification failed

- ...

## Next action

- ...
```

The progress file is the canonical current-state authority for the macro-task.

---

## Bounded progress rule

`progress/PROGRESS-Txxx.md` must remain compact enough to serve as hot execution memory.

Record:

- current macro-task status;
- active sub-task;
- sub-task status table;
- current blockers;
- decisions still needed for continuation;
- recent meaningful state transitions;
- latest verification result when relevant;
- next action.

Do **not** record:

- every shell command;
- every file opened;
- every trivial edit;
- long implementation narratives;
- obsolete investigation details;
- historical events that are no longer necessary to resume work.

When detailed evidence is useful, move or summarize it into a report.

When older history is no longer needed to continue execution, rely on Git history and reports rather than preserving it indefinitely in Progress.

A practical default is to keep only a small number of recent meaningful events, for example the last 5–10, unless additional events are still operationally relevant.

---

## Progress compaction on completion

When a macro-task becomes `COMPLETE`, compact its progress file.

Recommended completed form:

```markdown
# PROGRESS-T001 — Macro-task title

## Current state

- Status: COMPLETE
- Completed: YYYY-MM-DD
- Last update: YYYY-MM-DD

## Outcome

Short summary of what was delivered.

## Verification

- Final verification:
- Result:

## Sub-task status

| ID | Status | Evidence |
|---|---|---|
| T001-SUB001 | COMPLETE | ... |
| T001-SUB002 | COMPLETE | ... |

## Important final decisions

- ...

## Relevant reports

- ...

## Remaining follow-up

- None.
```

Remove stale operational chatter that is no longer needed for future work.

Git history and reports preserve historical detail.

---

## Report contract

Reports are optional.

Create a report when preserving deeper evidence would materially help future work, review, debugging, reproducibility, or handoff.

Strong report triggers include:

- non-trivial investigation;
- failure analysis;
- experiments or benchmarks;
- design alternatives or trade-off analysis;
- verification whose evidence is too detailed for Progress;
- implementation detail that future agents are likely to need;
- externally useful handoff material.

A small, routine implementation with obvious verification does not require a report merely for completeness.

Report creation is intentionally judgment-based within these rules.

Use:

```text
reports/REPORT-T001-SUB001.md
```

Recommended structure:

```markdown
# REPORT-T001-SUB001 — Report title

## Related specification

- [T001](../prd/T001.md)
- Sub-task: T001-SUB001

## Purpose

Why this work was performed.

## Context and inputs

...

## Work performed

...

## Findings

Separate verified facts from interpretations and hypotheses.

## Verification

- Command or probe:
- Result:
- Relevant artifacts:

## Decision

...

## Remaining issues

...

## Follow-up

...
```

A report must not silently become:

- a second specification;
- the only location of a requirement;
- the only location of a current blocker;
- a hidden task list.

If the work changes requirements, contracts, invariants, scope, dependencies, or acceptance criteria, update the relevant canonical specification explicitly.

If more than one report is needed for the same sub-task, use an explicit suffix:

```text
REPORT-T001-SUB001-01.md
REPORT-T001-SUB001-02.md
```

Do not overwrite a previous report merely to avoid creating a second evidence artifact.

Add report links to PRD/progress files only after the report exists.

---

## Decision promotion rule

Decisions must live at the narrowest authoritative level that future work needs.

### Local or historical decision

If a decision matters mainly as evidence for one investigation or implementation:

```text
report
```

### Macro-task decision

If a decision changes how future sub-tasks inside the same macro-task must behave:

```text
prd/Txxx.md
```

Update the relevant contract, invariant, dependency, scope, or context.

### Global project decision

If a decision changes project-wide behavior, architecture, compatibility, conventions, or invariants:

```text
PRD.md
```

Promote it to the global constraints/invariants section.

Do not require future agents to discover an active requirement by accidentally reading an old report.

---

## Working-tree safety rule

The current working tree is part of present project reality.

Before modifying files:

```bash
git status --short
```

Inspect overlapping uncommitted changes.

Rules:

- Never silently discard unrelated changes.
- Never overwrite another agent's uncommitted work merely because it is absent from Git history.
- If uncommitted changes overlap the active task, inspect and incorporate them into the current-state assessment.
- If the working tree contradicts Progress, document the discrepancy and resolve it from evidence rather than guessing.
- Use `git diff` or scoped diffs when needed to understand current changes.

---

## Update procedure

When starting and completing work on a sub-task:

1. Read `PRD.md`.
2. Inspect repository instructions.
3. Run `git status --short`.
4. Read recent Git history.
5. Read `prd/Txxx.md`.
6. Read `progress/PROGRESS-Txxx.md`.
7. Confirm the target `Txxx-SUByyy` specification exists.
8. Confirm dependencies are satisfied.
9. Inspect relevant current implementation and tests.
10. Inspect task-specific reports or Git history only when relevant.
11. Update `progress/PROGRESS-Txxx.md` to mark the sub-task active.
12. Perform the work.
13. Run the verification defined by the sub-task.
14. Create or update a report if the work warrants one.
15. If requirements or contracts changed, update `prd/Txxx.md`.
16. Update `progress/PROGRESS-Txxx.md` with the result and evidence.
17. If the macro-task status changed, update the root `PRD.md` in the same change. This is mandatory when a macro-task becomes `COMPLETE`.
18. Compact stale progress history when necessary.
19. Verify documentation links and repository consistency.
20. Commit using the relevant stable task ID when the commit maps to a task.

If a dependency is unsatisfied, acceptance criteria are not met, or verification fails, do not mark the sub-task `COMPLETE`.

Keep it `IN_PROGRESS` or mark it `BLOCKED`, record the reason in the progress file, and link relevant evidence when useful.

Do not mark a sub-task complete merely because code or documents were written. Completion requires verified acceptance criteria.

---

## State synchronization rule

The documents have different authorities:

```text
prd/Txxx.md
    normative specification and acceptance criteria

progress/PROGRESS-Txxx.md
    canonical current execution state

PRD.md
    global index, canonical macro-task priority/dependencies, and synchronized macro-task status projection

reports/REPORT-*.md
    evidence, findings, decisions, and handoff material

current implementation
    actual repository reality

Git history
    historical provenance
```

Only macro-task status is projected into `PRD.md`.

Do not duplicate sub-task execution status inside `prd/Txxx.md`.

When macro-task status changes:

1. update `progress/PROGRESS-Txxx.md`;
2. update the root `PRD.md` in the same change.

A macro-task transition to `COMPLETE` is not fully recorded until both files agree. Do not consider the documentation/state update complete while `PRD.md` still shows the previous macro-task status.

When acceptance criteria, scope, dependencies, contracts, or invariants change:

1. update `prd/Txxx.md`;
2. record relevant execution consequences in Progress;
3. promote project-wide decisions to `PRD.md` when necessary.

---

## Intentional degrees of freedom

The workflow is deterministic where inconsistent agent behavior would damage project state, and judgment-based where rigid rules would add unnecessary overhead.

### Closed or explicitly governed

The following are not free-form unless the project owner explicitly changes the project-wide workflow in the root `PRD.md`:

- identifier formats;
- status vocabulary;
- status semantics;
- allowed status transitions;
- dependency semantics;
- macro-task priority vocabulary;
- task-selection ordering;
- source-of-truth roles;
- macro-task completion conditions;
- file naming and documentation roles.

### Intentionally judgment-based

The following remain contextual within the rules of this skill:

- whether a report is useful for a specific routine task;
- how much detail belongs in `Context`;
- the exact moment to compact Progress, provided it remains bounded;
- whether a macro-task near the sizing heuristics should be split;
- how much historical Git/report context must be inspected once a concrete need for deeper context exists.

When exercising judgment, prefer the option that preserves bounded working context, minimizes duplicated state, and leaves future agents enough evidence to continue safely.

---

## Change discipline

When modifying the documentation system:

- preserve existing task IDs;
- do not rename IDs merely because titles change;
- do not move sub-tasks into separate files;
- do not duplicate full task specifications in `PRD.md`;
- maintain exactly one dedicated progress file per macro-task;
- do not put mutable sub-task status in macro-task PRDs;
- keep Progress bounded;
- keep links relative and verify them;
- keep status vocabulary consistent;
- keep macro-task priority and dependencies canonical in the root `PRD.md`;
- allow at most one `IN_PROGRESS` sub-task per macro-task unless project-wide parallelism is explicitly defined;
- separate specification, current state, implementation, evidence, and history;
- promote decisions to the correct authoritative level;
- preserve unrelated working-tree changes;
- use stable task IDs in relevant commit messages;
- split macro-tasks before their local working context becomes too broad.

---

## Verification checklist

Before considering a documentation or task-state update complete, verify:

- `PRD.md` still contains the complete macro-task index.
- Every status value uses the canonical project-wide status vocabulary.
- Every status transition is allowed by the state machine or is explicitly documented as a workflow change.
- Every macro-task priority uses the canonical project-wide priority vocabulary.
- Macro-task priorities and dependencies are defined canonically in the root `PRD.md`, not duplicated as mutable metadata in macro-task PRDs.
- Every declared dependency is satisfied according to the dependency semantics before dependent work starts.
- No macro-task has more than one `IN_PROGRESS` sub-task unless project-wide parallelism is explicitly defined.
- Every indexed macro-task has one `prd/Txxx.md` file.
- Every macro-task has one `progress/PROGRESS-Txxx.md` file.
- Every sub-task has a unique `Txxx-SUByyy` identifier.
- Sub-task acceptance criteria remain inside the macro-task PRD.
- No mutable sub-task status was added to `prd/Txxx.md`.
- No unintended `Txxx-SUByyy.md` files were created.
- Reports use the `REPORT-Txxx-SUByyy.md` convention.
- Report links point only to files that exist.
- Macro-task status changes are synchronized between task Progress and root `PRD.md`.
- `PRD.md` alone is sufficient to determine whether each macro-task is complete or not.
- Any macro-task marked `COMPLETE` in Progress is also marked `COMPLETE` in the root `PRD.md` before the change is considered finished.
- A `COMPLETE` macro-task has no remaining `PLANNED`, `IN_PROGRESS`, `BLOCKED`, `IN_REVIEW`, or `DEFERRED` sub-tasks.
- Any `CANCELLED` sub-task under a completed macro-task is intentionally out of scope and consistent with the final objective.
- Progress remains bounded and operationally useful.
- Internal links resolve.
- Acceptance criteria have a verification path.
- `git status --short` was inspected before editing.
- Existing relevant uncommitted changes were preserved and understood.
- Git history was loaded only when historical context was needed.
- Task-specific Git history or scoped diffs were used when historical context was needed.
- Current Markdown, current implementation, and relevant history are consistent, or discrepancies are documented.
- Relevant decisions were promoted to the appropriate authoritative level.
- The macro-task still fits within a practical local working context.
- Relevant commits use stable task identifiers when appropriate.
- `git diff --check` passes when the project uses Git.

---

## Common pitfalls

- Combining execution state from multiple macro-tasks into one shared progress file.
- Allowing a macro-task progress file to accumulate unbounded historical detail.
- Allowing a macro-task to grow beyond a practical local working context.
- Creating one Markdown file for every sub-task despite the project convention.
- Putting mutable sub-task status in both PRD and Progress.
- Putting acceptance criteria only in reports.
- Treating a report as a replacement for the task specification.
- Hiding an active requirement or global invariant inside an old report.
- Duplicating the full content of `T001.md` inside `PRD.md`.
- Updating `progress/PROGRESS-T001.md` but forgetting the macro-task status projection in `PRD.md`.
- Completing a macro-task without updating its root `PRD.md` status in the same change, which breaks the global overview contract.
- Reusing an old task ID for a new task.
- Inventing local status names or priority values.
- Treating `DEFERRED` as equivalent to `COMPLETE`.
- Marking a macro-task `COMPLETE` while in-scope sub-tasks remain unfinished.
- Selecting a "highest-priority sub-task" even though sub-task priority is not part of the default schema.
- Using vague, non-verifiable acceptance criteria.
- Loading every PRD and report when only one macro-task is active.
- Treating Git history as automatically authoritative over the current specification.
- Loading general Git history by default even when no historical context is needed.
- Treating a non-`COMPLETE` dependency as satisfied.
- Treating a `CANCELLED` dependency as automatically satisfied without updating the dependent specification.
- Duplicating macro-task priority or dependency state inside `prd/Txxx.md`.
- Marking multiple sub-tasks `IN_PROGRESS` in one macro-task without an explicit project-wide parallelism model.
- Ignoring uncommitted working-tree changes.
- Assuming that a completed checklist proves correctness without running verification.
- Preserving obsolete execution history in Progress even though Git already records it.
- Failing to include stable task IDs in task-specific commits, making history difficult to query.

---

## Design principle

The workflow should keep the context required for one unit of work approximately bounded as the project grows.

An agent working on `T017-SUB004` should normally need only:

```text
repository instructions
+ PRD.md
+ prd/T017.md
+ progress/PROGRESS-T017.md
+ relevant current implementation
+ relevant report(s), only if needed
+ relevant Git history, only if needed
```

The number of completed tasks, old reports, and historical commits in the overall project should not force the agent to load proportionally more context.

The goal is not to build a project-management framework.

The goal is to use Markdown + Git as a lightweight **context-routing system** for long-running agentic work: compact global orientation, bounded local execution context, deterministic task lifecycle, and deeper historical evidence only when needed.
