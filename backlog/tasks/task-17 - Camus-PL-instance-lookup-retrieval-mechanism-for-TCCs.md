---
id: task-17
title: 'Camus PL: instance lookup/retrieval mechanism for TCCs'
status: In Progress
assignee: []
created_date: '2026-09-24 20:26'
updated_date: '2026-09-24 20:55'
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
- [ ] #1 Lookup/retrieval mechanism designed for TCCs to obtain existing instances
- [ ] #2 Integrated into grammar/checker, or explicitly deferred with rationale
<!-- AC:END -->
