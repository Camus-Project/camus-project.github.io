---
id: task-21
title: 'codegen: minimal synchronous runtime for later/when/wait/ignore'
status: Done
assignee: []
created_date: '2026-09-25 11:54'
updated_date: '2026-09-28 16:06'
labels:
  - camus-pl
dependencies:
  - task-20
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Map the Camus PL async API (later / when / wait / ignore) to a concrete execution model. For this milestone, implement a MINIMAL SYNCHRONOUS runtime (decision 3): later/when execute eagerly, wait is a no-op join, ignore is fire-and-forget-then-drop. This is explicitly a prototype: a tokio-based concurrent implementation is deferred and MUST be tracked separately. Part of the 'Camus PL to executable' milestone (Fossil 6a5506850d).
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [x] #1 later/when/wait/ignore are lowered to a concrete, compilable synchronous execution model preserving the checker's async-safety semantics (read-after-wait ordering)
- [x] #2 The async.cam / read_after_wait.cam style patterns produce correct runnable behavior under the synchronous model
- [x] #3 A clear note in code and docs states this runtime is a synchronous prototype and that a tokio-based implementation is the required follow-up
- [x] #4 A follow-up backlog task for the tokio-based concurrent runtime is created (deferred, not implemented here)
- [x] #5 Tests cover the lowering of each async construct
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Implementation: translate_stmt When->match over the eager handle (CamusResult::Ok/Err arms, Branch::Ignore->_), Wait->let _ = &h, Ignore->let _ = &e. translate_expr: Later(inner) runs eagerly, WaitHandle(h)->&h. map_ty Result->CamusResult<()>; notify.success/error (2-seg); order_args lowers 'self' to self.clone() (TASK-24 hones by-ref). PRELUDE documents SYNCHRONOUS PROTOTYPE + tokio follow-up (TASK-25, Fossil a32d5424). Sample E2E: 'create' -> 'Task stored successfully' + 'Task created: 1'. Tests: synchronous_runtime_lowers_all_async_constructs + sample_create_runs_end_to_end (run_generated). Fossil: aad6c7e9de8565f80a70eff564df2cda5b1faef9
<!-- SECTION:NOTES:END -->
