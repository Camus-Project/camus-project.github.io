---
id: TASK-20
title: 'codegen: ctx composition root and instance lookup at runtime'
status: To Do
assignee: []
created_date: '2026-09-25 11:54'
labels:
  - camus-pl
dependencies:
  - TASK-19
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Turn the ctx configuration format (TASK-16) and the TCC instance lookup mechanism (TASK-17) into a real runtime composition root: instantiate LCR adapters, wire dependencies, and resolve ctx.get(X) / instance lookups to concrete generated instances at program startup. Part of the 'Camus PL to executable' milestone (Fossil 6a5506850d).
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 ctx configuration is compiled into a composition root that instantiates and wires the generated LCR adapters at startup
- [ ] #2 ctx.get(X) resolves to the concrete generated instance in the running program
- [ ] #3 Instance lookup/retrieval for TCCs (per TASK-17 semantics) is realized at runtime
- [ ] #4 The sample-project wires Cli -> Task -> TaskStore end-to-end and compiles
- [ ] #5 Tests cover ctx resolution and instance wiring
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Fossil: 6a5506850d
<!-- SECTION:NOTES:END -->
