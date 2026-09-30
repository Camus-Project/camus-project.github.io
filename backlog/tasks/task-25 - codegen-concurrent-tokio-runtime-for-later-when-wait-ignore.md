---
id: task-25
title: 'codegen: concurrent tokio runtime for later/when/wait/ignore'
status: To Do
assignee: []
created_date: '2026-09-29 11:05'
labels:
  - camus-pl
dependencies:
  - task-21
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Replace the synchronous prototype execution model (TASK-21, decision 3) with a real CONCURRENT runtime based on tokio: later spawns tasks, when branches on completion, wait awaits the handle, ignore detaches. Required before Camus PL is considered production-ready. Deferred by design from TASK-21; do not implement a concurrency runtime in the generated prelude until this is done.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 A tokio-based runtime crate is used for generated code (later spawns, wait awaits, when branches, ignore detaches)
- [ ] #2 The sample-project CLI still works and behaves identically to the synchronous prototype
- [ ] #3 Tests cover concurrent later/when/wait/ignore behavior
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Fossil: a32d5424a42d940c91d3b7924ab74ca666764823
<!-- SECTION:NOTES:END -->
