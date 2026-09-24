---
id: task-15
title: 'Camus PL: Verus verification backend for laws'
status: To Do
assignee: []
created_date: '2026-09-24 20:26'
labels:
  - camus-pl
dependencies: []
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Fossil: fc45fd12f3

Implement the pipeline Camus PL -> Rust + Verus specifications -> Verus verification -> compilable Rust -> executable. Mapping: law requires -> Verus requires, law ensures -> Verus ensures, struct invariant -> type invariant, spec function -> spec fn, assert -> assert(...). See ARCHITECTURE.md section 17. Makes the strictness rule (unproven law = hard failure) enforceable.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 Law constructs mapped to Verus (requires/ensures/invariant/spec fn/assert) with no Camus PL syntax lost
- [ ] #2 Pipeline exercised end-to-end on the sample project (laws proven by Verus)
<!-- AC:END -->
