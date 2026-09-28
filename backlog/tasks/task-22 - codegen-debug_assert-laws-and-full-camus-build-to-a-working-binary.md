---
id: TASK-22
title: 'codegen: debug_assert laws and full camus build to a working binary'
status: To Do
assignee: []
created_date: '2026-09-25 11:55'
labels:
  - camus-pl
dependencies:
  - TASK-21
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Emit runtime law checks and complete the end-to-end build. Law clauses (requires/ensures/invariant) and assert statements are emitted as debug_assert! (decision 4), so they are enforced in debug builds and free in release. Provide a 'camus build <dir>' that generates a complete cargo project from a Camus PL project and runs cargo build, producing a working binary. The sample-project must run as a real CLI (decision 5). Part of the 'Camus PL to executable' milestone (Fossil 6a5506850d).
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 requires/ensures/invariant and assert are emitted as debug_assert! at the appropriate points (entry, exit, after mutation/construction)
- [ ] #2 old(...) and result in ensures are correctly captured for the debug_assert at function exit
- [ ] #3 camus build <dir> generates a complete, buildable cargo project from a Camus PL project
- [ ] #4 camus build runs cargo build and produces a binary; non-zero exit on build failure
- [ ] #5 The sample-project builds and runs as a working CLI (create/list/update tasks per its intent)
- [ ] #6 An end-to-end test builds and runs the sample-project binary
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Fossil: 6a5506850d
<!-- SECTION:NOTES:END -->
