---
id: task-16
title: 'Camus PL: ctx configuration format and dependency resolution'
status: Done
assignee: []
created_date: '2026-09-24 20:26'
updated_date: '2026-09-24 20:55'
labels:
  - camus-pl
dependencies: []
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Fossil: 51ada3b1cd

Define the key/value configuration tree and lazy dependency resolution behind ctx.get(Type[, name]) (ARCHITECTURE.md section 6). Left out of scope by task-12; settles the open point.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [x] #1 ctx configuration format settled and documented (key/value tree, lazy resolution)
- [x] #2 Grammar/checker support or explicit runtime-only decision made and applied
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Settled the runtime-only configuration model in ARCHITECTURE.md section 6 (binding rule type(/name) -> tree key, lazy + cached resolution, runtime-only failure, single ctx.get API). Applied: checker rule ctx.get requires a declared/imported/primitive type; 5 fixtures + 4 tests; sample TaskStore.cam migrated from ctx.database to ctx.get(SqliteConnection, "database"). Ticket: 51ada3b1cd (Open).
<!-- SECTION:NOTES:END -->
