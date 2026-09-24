---
id: task-15
title: 'Camus PL: Verus verification backend for laws'
status: Done
assignee: []
created_date: '2026-09-24 20:26'
updated_date: '2026-09-24 21:33'
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
- [x] #1 Law constructs mapped to Verus (requires/ensures/invariant/spec fn/assert) with no Camus PL syntax lost
- [x] #2 Pipeline exercised end-to-end on the sample project (laws proven by Verus)
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Implemented. Crate Camus-PL/verus-gen (lib gen + bin CLI). Classification Verified vs External/assumed per proposal; mutation re-establishes the struct invariant (ensures Self::Inv(*final(self))), constructors inject Self::Inv(result) on fresh instances; external bodies become #[verifier::external_body] ASSUMED boundary contracts. Type mappings String->Seq<char>, Int->i64, uuidv7->u64 (uuidv7_fresh), Bool->bool, Float->f64; old(x)->(*old(self)).field in ensures; result->named output via tail expression (Verus does not bind named returns in the body). Verified E2E on sample-project (3 verified, 0 errors) and on tests/fixtures (account: all law constructs; lookup: retrieval->mutation). Report lists each law clause with classification (7/4/6+ mapped). --verify exits non-zero on any Verus error (strictness). Executable stage deferred to kiss-rust/runtime. Closes ticket fc45fd12f3.
<!-- SECTION:NOTES:END -->
