# Scalable Ralph Loop Patch

### Progressive Disclosure and Bounded Project Memory for Agentic Development

**Scalable Ralph Loop Patch** is a memory-layer patch for Ralph-style agentic development workflows.

It does **not** replace or redesign the Ralph Loop itself. The core execution cycle — planning work, selecting tasks, implementing changes, verifying results, delivering progress, and iterating with fresh agent context — remains unchanged.

This project patches one specific scalability problem in the persistent project memory used by that loop by combining **progressive disclosure, bounded project memory, deterministic task routing, and on-demand historical provenance**.

## Goal

In long-running agentic development projects, `PRD.md` and `PROGRESS.md`-style memory files tend to grow continuously as tasks, decisions, verification results, blockers, and implementation history accumulate.

After dozens or hundreds of tasks, every new agent iteration may need to read increasingly large files before useful work can begin.

The result is a simple but important scaling problem:

```text
project history ↑
completed tasks ↑
persistent memory files ↑
required startup context ↑
```

The goal of **Scalable Ralph Loop Patch** is to break that relationship.

Project history should be able to grow without forcing the working context of every new agent iteration to grow with it.

> **Keep the Ralph Loop. Patch the memory layer.**

The patch replaces a growing flat memory layer with a bounded, progressively disclosed structure in which:

```text
PRD.md
    compact global project overview and routing context

prd/Txxx.md
    local macro-task specification

progress/PROGRESS-Txxx.md
    bounded hot execution state

reports/
    deeper evidence loaded only when relevant

Git history
    historical provenance loaded only when needed
```

An agent first receives the minimum context required to orient itself, then progressively loads deeper information only for the macro-task and sub-task it is actually working on.

The intended scaling property is:

```text
total project history ↑

while

context required for the next unit of work ≈ bounded
```

## Scope

This patch changes **only the persistent memory and context-loading layer** around the Ralph Loop.

It does not attempt to replace or redefine:

- the Ralph execution loop;
- planning;
- task selection;
- implementation;
- delivery;
- verification and testing;
- iteration between fresh agent contexts;
- the underlying coding agent or agent framework.

Instead, it provides a scalable way to preserve, route, and retrieve the project state that those iterations depend on.

The implementation remains deliberately lightweight:

**Markdown + Git + progressive disclosure.**

No database.
No task-management server.
No orchestration framework.

Just a structured project memory model and a deterministic context-loading protocol.

---

## The problem

A simple PRD-driven Ralph-style development loop works well:

```text
read requirements
→ inspect progress
→ choose next task
→ implement
→ verify
→ update progress
→ repeat with fresh context
```

As a project grows, however, its persistent memory grows with it:

- completed requirements;
- implementation history;
- verification results;
- previous blockers;
- design decisions;
- failed approaches;
- reports;
- handoff information;
- task state.

If all of this remains part of the default context, every new agent iteration must consume an increasingly large amount of historical information before doing useful work.

The execution loop still works.

Its **context cost does not scale well**.

---

## The patch

Scalable Ralph Loop Patch organizes project memory into progressively disclosed layers:

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

Each layer has one responsibility:

```text
PRD.md
    global routing context

prd/Txxx.md
    local working specification

progress/PROGRESS-Txxx.md
    hot execution memory

current implementation
    present repository reality

reports/
    cold detailed evidence

Git history
    historical provenance
```

The agent starts with the smallest useful context and drills down only when necessary.

---

## Progressive disclosure

An agent entering the repository first reads:

```text
PRD.md
```

That should be enough to understand:

- the project objective;
- global constraints and invariants;
- all macro-tasks;
- macro-task status;
- priority;
- dependencies;
- what is complete;
- what is active;
- what can be worked on next.

If the agent needs to work on `T017`, it then loads:

```text
prd/T017.md
progress/PROGRESS-T017.md
```

If implementation context is required:

```text
relevant source files
relevant tests
git status --short
```

Only when deeper historical context is necessary does the agent inspect:

```text
relevant reports
task-scoped Git history
relevant historical diffs
```

An agent working on:

```text
T017-SUB004
```

should therefore normally need only:

```text
repository instructions
+ PRD.md
+ prd/T017.md
+ progress/PROGRESS-T017.md
+ relevant implementation
+ relevant reports, only if needed
+ relevant Git history, only if needed
```

It does not need to load the specifications and execution history of every completed task in the project.

---

## Target scaling property

The workflow aims to move from:

```text
working context ∝ total project history
```

toward:

```text
working context ≈ global routing context
                + active macro-task context
                + active implementation context
```

Conceptually:

```text
context cost ≈ O(active working set)
```

rather than:

```text
context cost ≈ O(total project history)
```

The total project memory may continue growing.

The context required for the next unit of work should remain approximately bounded.

---

## `PRD.md`: global routing context

The root `PRD.md` is the global entry point.

It remains compact and contains the macro-task index.

Example:

```markdown
## Macro-task index

| ID | Macro-task | Status | Priority | Dependencies | Specification | Progress |
|---|---|---|---|---|---|---|
| T001 | Data pipeline | COMPLETE | P0 | — | [T001](prd/T001.md) | [Progress](progress/PROGRESS-T001.md) |
| T002 | Training loop | IN_PROGRESS | P0 | T001 | [T002](prd/T002.md) | [Progress](progress/PROGRESS-T002.md) |
| T003 | Evaluation | PLANNED | P1 | T002 | [T003](prd/T003.md) | [Progress](progress/PROGRESS-T003.md) |
```

A core invariant is:

> **Reading `PRD.md` alone must always be sufficient to determine the status of every macro-task.**

An agent should not need to read every Progress file just to understand the global state of the project.

Macro-task priority and macro-level dependencies are also canonical in the root PRD because they are routing information.

---

## `prd/Txxx.md`: working specification

Each macro-task has one specification:

```text
prd/T001.md
```

It contains:

- objective;
- scope;
- contracts and interfaces;
- sub-tasks;
- sub-task dependencies;
- acceptance criteria;
- verification procedures;
- risks;
- open questions.

Example:

```markdown
### T001-SUB003 — Validate tokenizer input

- Goal: Reject malformed tokenizer records before training.
- Dependencies: T001-SUB002

- Acceptance criteria:
  - malformed records return the documented error;
  - valid fixtures complete successfully;
  - verification command exits with status 0.

- Verification:
  - pytest tests/test_tokenizer_validation.py
```

The specification defines **what must be true**.

Mutable execution state does not live here.

---

## `progress/PROGRESS-Txxx.md`: hot execution memory

Each macro-task has one bounded execution-state file:

```text
progress/PROGRESS-T001.md
```

It contains:

- current macro-task status;
- active sub-task;
- sub-task statuses;
- current blockers;
- decisions still relevant to execution;
- recent meaningful events;
- latest relevant verification;
- next action.

Example:

```markdown
## Current state

- Status: IN_PROGRESS
- Active sub-task: T001-SUB003
- Blocked by: —
- Last update: 2026-08-18

## Sub-task status

| ID | Status |
|---|---|
| T001-SUB001 | COMPLETE |
| T001-SUB002 | COMPLETE |
| T001-SUB003 | IN_PROGRESS |
| T001-SUB004 | PLANNED |

## Next action

Run tokenizer validation against malformed-record fixtures.
```

Progress is **hot memory**, not an unlimited historical diary.

---

## Bounded Progress

Progress files should remain compact enough to be loaded routinely.

Keep:

- active execution state;
- blockers;
- current decisions;
- latest relevant verification;
- next action;
- a small number of meaningful recent events.

Do not keep:

- every command executed;
- every file inspected;
- trivial edits;
- long implementation narratives;
- obsolete investigations;
- historical detail that is no longer needed to resume work.

Deeper evidence belongs in reports.

Historical evolution belongs in Git.

When a macro-task is completed, its Progress can be compacted into a concise final record containing:

```text
final status
outcome
verification
sub-task completion state
important final decisions
relevant reports
remaining follow-up
```

---

## Reports: cold project memory

Reports are optional.

They are useful for information worth preserving without loading it into every future iteration:

- investigations;
- failure analysis;
- experiments;
- benchmarks;
- design trade-offs;
- complex verification;
- implementation findings;
- handoffs.

Example:

```text
reports/REPORT-T012-SUB004.md
```

Reports are loaded only when relevant to the current task.

They therefore provide **cold project memory** without continuously increasing the working context.

---

## Git: historical provenance

Git has a different role from current project state.

Current authority comes from:

```text
PRD.md
prd/Txxx.md
progress/PROGRESS-Txxx.md
current implementation
```

Git primarily answers:

> **How and why did the project arrive at its current state?**

Historical Git context is therefore loaded on demand.

The working tree is different: it is part of present repository reality.

Agents should always inspect:

```bash
git status --short
```

before modifying the repository.

Stable task IDs make historical investigation directly searchable:

```bash
git log --oneline --grep="T012-SUB004"
```

Task-specific commits should include the corresponding identifier when appropriate:

```text
T012-SUB004: implement tokenizer validation
```

---

## Stable identifiers

Macro-tasks:

```text
T001
T002
T010
```

Sub-tasks:

```text
T001-SUB001
T001-SUB002
```

Supporting artifacts derive their identity from these stable IDs:

```text
PROGRESS-T001
REPORT-T001-SUB001
```

Identifiers do not change when titles change.

This makes linking, search, Git inspection, handoff, automation, and state tracking deterministic.

---

## Deterministic task lifecycle

The default status vocabulary is closed:

```text
PLANNED
IN_PROGRESS
BLOCKED
IN_REVIEW
COMPLETE
DEFERRED
CANCELLED
```

Completion is evidence-based.

A sub-task does not become `COMPLETE` merely because implementation work was performed.

It becomes complete when:

```text
acceptance criteria satisfied
+
required verification passed
```

A macro-task may become `COMPLETE` only when:

- its objective is satisfied;
- required verification has passed;
- every sub-task still in scope is `COMPLETE`;
- no `PLANNED`, `IN_PROGRESS`, `BLOCKED`, `IN_REVIEW`, or `DEFERRED` work remains;
- intentionally removed work is explicitly `CANCELLED`;
- Progress records the final outcome;
- the root `PRD.md` is updated to `COMPLETE` in the same change.

In particular:

```text
DEFERRED ≠ COMPLETE
```

---

## Dependencies

Dependencies are execution constraints.

By default:

```text
dependency satisfied ⇔ dependency COMPLETE
```

These statuses do not satisfy a dependency:

```text
PLANNED
IN_PROGRESS
BLOCKED
IN_REVIEW
DEFERRED
```

A `CANCELLED` dependency is not automatically considered satisfied.

The dependent specification must explicitly remove, replace, or otherwise resolve it.

---

## Decision promotion

Progressive disclosure works only if active requirements are not buried in deep historical documents.

Decisions are therefore promoted according to their scope:

```text
local implementation or investigation decision
    → report

decision affecting one macro-task
    → prd/Txxx.md

project-wide invariant or architectural decision
    → PRD.md
```

The wider the effect of a decision, the higher it moves in the context hierarchy.

This allows future agents to trust upper layers without reading old reports "just in case."

---

## Typical agent iteration

```text
1. Read repository instructions
        ↓
2. Read PRD.md
        ↓
3. git status --short
        ↓
4. Select macro-task deterministically
        ↓
5. Read prd/Txxx.md
        ↓
6. Read progress/PROGRESS-Txxx.md
        ↓
7. Select active or next sub-task
        ↓
8. Inspect relevant implementation
        ↓
9. Implement
        ↓
10. Verify acceptance criteria
        ↓
11. Update Progress
        ↓
12. Update specification if requirements changed
        ↓
13. Update PRD.md if macro-task status changed
        ↓
14. Load reports or Git history only when deeper context is needed
```

The project grows globally.

The active context stays local.

---

## Context hierarchy

| Layer | Role | Load frequency |
|---|---|---|
| `PRD.md` | Global routing context | Always |
| `prd/Txxx.md` | Working specification | Active macro-task |
| `progress/PROGRESS-Txxx.md` | Hot execution memory | Active macro-task |
| Current implementation | Present reality | Active sub-task |
| `reports/` | Cold detailed memory | When relevant |
| Git history | Historical provenance | When relevant |

This hierarchy is the core of the patch.

---

## What this is not

Scalable Ralph Loop Patch is deliberately **not**:

- a replacement for the Ralph Loop;
- a new planning methodology;
- a new delivery methodology;
- a new verification methodology;
- a project-management application;
- an issue tracker;
- an orchestration server;
- a database;
- an agent framework;
- a replacement for Git;
- a replacement for testing;
- a mechanism for putting unlimited project memory into context.

It is a small protocol for answering:

> **What does the agent need to know right now, where should that information live, and when should deeper context be disclosed?**

---

## Agent skill

The workflow is implemented by the **PRD Context Router** skill.

The current skill is provided in:

[`SKILL.md`](SKILL.md)

Its activation rule is intentionally broad:

> **Use whenever you or a sub-agent need to perform development work on a repository.**

The skill formalizes:

- PRD structure;
- task lifecycle;
- deterministic task selection;
- dependency semantics;
- progress tracking;
- completion rules;
- progressive context loading;
- working-tree safety;
- report usage;
- Git provenance;
- bounded project memory;

---

## Recommended pairing with TDD Development Skill

The **PRD Context Router** and **TDD Development Skill** govern two separate layers of agentic software development:

- **PRD Context Router** governs project memory, task state, context routing, progress, and handoff.
- **TDD Development Skill** governs how repository code is changed, tested, refactored, and verified.

Together, they separate two concerns that are often mixed:

```text
PRD Context Router
    WHAT to work on
    project state
    task routing
    bounded memory
    progress tracking
        ↓
selected task
        ↓
TDD Development Skill
    HOW to change the code
    RED → GREEN → REFACTOR
    verification
```

TDD Development Skill repository:

[https://github.com/nazarenodefrancesc/agentic-tdd-skill](https://github.com/nazarenodefrancesc/agentic-tdd-skill)

The TDD skill is standalone and can also be used without PRD Context Router when only the implementation discipline is needed.

---

## Adoption

To apply the patch to a Ralph-style agentic software project:

1. Make [`SKILL.md`](SKILL.md) available to the development agent.
2. Create or migrate the root `PRD.md`.
3. Define macro-tasks using stable IDs.
4. Create one specification and one Progress file per macro-task.
5. Let agents progressively load only the active context.
6. Use reports when deeper evidence is worth preserving.
7. Use Git for historical provenance.
8. Keep the underlying Ralph execution loop unchanged.

The exact Markdown formatting is secondary.

The important contracts are:

```text
global overview stays global
specification stays normative
execution state stays local
Progress stays bounded
deep evidence stays optional
history stays retrievable
important decisions are promoted
```

---

## Design goal

```text
project size ↑
project history ↑
completed work ↑

while

context required for the next unit of work ≈ bounded
```

The project should be allowed to become large.

The agent's immediate working memory should not have to become large with it.

---

## Author

**Nazareno De Francesco**

---

## License

This work is licensed under the **Creative Commons Attribution 4.0 International License (CC BY 4.0)**.

Reuse, adaptation, and redistribution are permitted, including commercially, subject to the attribution requirements of CC BY 4.0.

Please attribute the original work as:

**Nazareno De Francesco — Scalable Ralph Loop Patch**

See the repository [`LICENSE`](LICENSE) file for licensing information.
