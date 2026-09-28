---
id: TASK-21
title: 'codegen: minimal synchronous runtime for later/when/wait/ignore'
status: To Do
assignee: []
created_date: '2026-09-25 11:54'
labels:
  - camus-pl
dependencies:
  - TASK-20
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Map the Camus PL async API (later / when / wait / ignore) to a concrete execution model. For this milestone, implement a MINIMAL SYNCHRONOUS runtime (decision 3): later/when execute eagerly, wait is a no-op join, ignore is fire-and-forget-then-drop. This is explicitly a prototype: a tokio-based concurrent implementation is deferred and MUST be tracked separately. Part of the 'Camus PL to executable' milestone (Fossil 6a5506850d).
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 later/when/wait/ignore are lowered to a concrete, compilable synchronous execution model preserving the checker's async-safety semantics (read-after-wait ordering)
- [ ] #2 The async.cam / read_after_wait.cam style patterns produce correct runnable behavior under the synchronous model
- [ ] #3 A clear note in code and docs states this runtime is a synchronous prototype and that a tokio-based implementation is the required follow-up
- [ ] #4 A follow-up backlog task for the tokio-based concurrent runtime is created (deferred, not implemented here)
- [ ] #5 Tests cover the lowering of each async construct
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Fossil: 6a5506850d
IMPORTANT: this is a synchronous PROTOTYPE runtime only. A real concurrent implementation based on tokio is required later and must be tracked as a dedicated follow-up task before Camus PL is considered production-ready.
<!-- SECTION:NOTES:END -->
