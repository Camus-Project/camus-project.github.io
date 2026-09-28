---
id: TASK-17
title: 'Camus PL: instance lookup/retrieval mechanism for TCCs'
status: Done
assignee: []
created_date: '2026-09-24 20:26'
updated_date: '2026-09-25 11:33'
labels:
  - camus-pl
dependencies: []
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Fossil: e303a3a002

Resolve the open design question (ARCHITECTURE.md section 11): how a TCC obtains an existing structure instance to mutate, e.g. a CLI looking up an existing Task by id before calling update.

Deliverable: design decision and grammar/checker integration, or explicit deferral with rationale.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [x] #1 Lookup/retrieval mechanism designed for TCCs to obtain existing instances
- [x] #2 Integrated into grammar/checker, or explicitly deferred with rationale
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Design decision (2026-09-24): lookup is a shared LCR function returning the record type, e.g. TaskStore.find(uuidv7: id) -> Task, composed by the TCC as ctx.get(TaskStore) + let mut task: Task = store.find(id) + mut self method call. Contract carried by a law (ensures: result.id == id). No new grammar: reuses call/return/let-mut forms. Checker integration: fixed check_call_args which counted the implicit self parameter of methods and made mut self methods uncallable; fixtures (good/retrieve_and_mutate.cam, bad/method_arity.cam) + 2 tests; sample project gained TaskStore.find and Cli.update. Documented in ARCHITECTURE.md section 11 and grammar.ebnf. Out of scope (runtime/backend): physical fetch, not-found policy, mut-receiver enforcement -> task-15.

Fossil: 14e0628c0a
<!-- SECTION:NOTES:END -->
