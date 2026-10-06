---
id: task-10
title: 'Revive Camus PL design: TCC/App/LCR architecture and sample project'
status: Done
assignee:
  - '@ai-agent'
created_date: '2026-09-10 21:03'
updated_date: '2026-09-10 21:03'
labels: []
dependencies: []
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Fossil: 068e3a11ac

Camus PL design work resumes as the primary target, superseding the earlier "PL dropped in favor of Method+SL+kiss" decision, now informed by lessons learned from the Camus SL effort. Publishes Camus-PL/ARCHITECTURE.md (co-designed with ChatGPT), documenting three application layers/roles -- TCC (Tightly Coupled Client, incoming interface), App (domain logic), LCR (Loosely Coupled Resource, outgoing interface) -- declared via 'struct X role tcc|lcr', ctx-based dependency resolution, default-async LCR invocation, declarative constraints, the intention-first two-level declaration pattern, and the reintroduction of anonymous functions in Camus PL. Updates the experimental sample project (Camus-PL/sample-project/: app/Task.cam, lcr/TaskStore.cam, tcc/Cli.cam) to reflect the intention/constraints/declaration-structure ideas from the note.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [x] #1 Camus-PL/ARCHITECTURE.md exists in the repository and matches the design note
- [x] #2 app/Task.cam declares its struct-level constraints and gives its create/store functions an intention, following the note's two-level declaration pattern
- [x] #3 lcr/TaskStore.cam's shared store function follows the note's intention + function(Type: name) -> ReturnType pattern
- [x] #4 tcc/Cli.cam's create function declares an intention
- [x] #5 The superseded 'Camus PL dropped' decision is recorded as such in project memory (Hippo)
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Published Camus-PL/ARCHITECTURE.md with the full TCC/App/LCR design note (co-designed with ChatGPT), verbatim plus a short status header noting it is experimental. Updated the sample project to follow the note's intention-first, two-level declaration pattern: Task.cam now declares struct-level constraints (!empty(name), !empty(description)) and gives both store() and create() an intention plus a nested function(...) block (create() also keeps its own input: constraints on the raw String parameters, which is not redundant with the struct-level constraints since those guard the Task type itself, not create()'s pre-construction parameters); TaskStore.cam's shared store() now matches the note's own worked example verbatim (intention + function(Task: task) -> Result); Cli.cam's create() gets an intention. Struct field blocks without any declarative properties yet (TaskStore's database field, Cli itself) were left in their existing flat form rather than inventing an untested 'empty declarative properties' nesting convention not shown in the note. Fossil ticket 9281d5dcf0 (task-9, now Done) was closed to keep ticket/task state in sync. The superseded 'Camus PL dropped' decision was recorded in Hippo as sem_85852a45f12d (supersedes sem_a6b228e17eb3) in the prior session turn.
<!-- SECTION:NOTES:END -->
