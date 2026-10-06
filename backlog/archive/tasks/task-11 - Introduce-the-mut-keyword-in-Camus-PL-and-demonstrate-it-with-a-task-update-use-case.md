---
id: task-11
title: >-
  Introduce the mut keyword in Camus PL and demonstrate it with a task-update
  use case
status: Done
assignee:
  - '@ai-agent'
created_date: '2026-09-10 21:14'
updated_date: '2026-09-10 21:14'
labels: []
dependencies: []
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Fossil: fc28cf30bd

Camus PL is immutable by default everywhere (struct fields, function parameters, local variables). `mut` is the explicit exception, written immediately in front of the name it modifies. A method mutating its own structure's fields must declare `mut self` explicitly as its first parameter. Documents this as a new "Mutability" section in Camus-PL/ARCHITECTURE.md (renumbering the sections that followed it), and demonstrates it in the sample project via a concrete task-update use case: Task.cam's name/description fields become mut, and a new Task.update function mutates them through mut self; TaskStore.cam gains a shared update() LCR method issuing a SQL UPDATE.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [x] #1 ARCHITECTURE.md documents mut for struct fields, function parameters, mut self, and local variables, with immutable-by-default as the stated rule
- [x] #2 Task.cam's name/description fields are mut, and a new update() function demonstrates mut self plus persistence of the change
- [x] #3 TaskStore.cam gains a shared update() LCR method distinct from store()
- [x] #4 The mut design decision is recorded in project memory (Hippo)
- [x] #5 Open questions (constraint re-validation on mutation, TCC-side lookup of an existing instance) are explicitly flagged rather than silently resolved
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Added a new numbered section 11 'Mutability' to Camus-PL/ARCHITECTURE.md (renumbering the former 11-13 to 12-14: Declaration structure, Signatures, Current example), covering the four mut positions (struct field, function parameter, mut self, local variable) and the immutable-by-default rule, plus a worked Task.update example. Updated Task.cam: name/description are now mut fields; added a private store_update() helper mirroring the existing store() async-persistence pattern but calling TaskStore.update() and a different notification message; added public update(mut self, String: name, String: description), which mutates self's mut fields (hence needs mut self) while its own name/description parameters stay immutable (only read, never reassigned). Updated TaskStore.cam: added shared update(Task: task) -> Result issuing 'UPDATE tasks SET name = ?, description = ? WHERE id = ?', distinct from store()'s INSERT. Left Cli.cam untouched -- deliberately did not invent a Task lookup/find mechanism to expose 'update' through the CLI, since none is designed yet; flagged as an open question in both ARCHITECTURE.md and the Fossil ticket. Also flagged, not resolved: whether mutating a mut field should re-trigger that field's structural constraints the way construction does. Recorded the mut decision in Hippo as sem_503d33968943. Fossil ticket fc28cf30bd intentionally left Open (not Closed) because of these two unresolved questions, pending human decision.
<!-- SECTION:NOTES:END -->
