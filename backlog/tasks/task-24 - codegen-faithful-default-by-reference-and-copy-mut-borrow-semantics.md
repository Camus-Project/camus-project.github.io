---
id: TASK-24
title: 'codegen: faithful default-by-reference and copy/mut borrow semantics'
status: To Do
assignee: []
created_date: '2026-09-25 13:17'
labels:
  - camus-pl
dependencies:
  - TASK-18
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Follow-up to TASK-18. The walking skeleton lowers all non-self parameters by value. Implement Camus PL's real binding model (ARCHITECTURE.md section 11): plain parameters are references (&T), copy parameters are owned working copies, mut parameters are mutable references (&mut T), inserting clones where a borrowed value must be stored into an owned field or passed to an owned position. Part of the 'Camus PL to executable' milestone (Fossil 6a5506850d).
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 Plain (non-mut, non-copy) parameters are lowered as shared references, preserving the caller's value
- [ ] #2 copy parameters are lowered as owned values (private working copy)
- [ ] #3 mut parameters are lowered as mutable references to the caller's value
- [ ] #4 Clones are inserted where a borrowed value is stored into an owned field or passed to an owned position, and generated code still compiles
- [ ] #5 Copy primitive types may be passed by value where semantically equivalent, documented as such
- [ ] #6 Tests cover each parameter mode and the sample-project still compiles
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Fossil: 6a5506850d
<!-- SECTION:NOTES:END -->
