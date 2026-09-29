---
id: task-14
title: >-
  Camus PL: laws (invariant/requires/ensures), spec functions and assert —
  grammar v0.4 + checker
status: Done
assignee: []
created_date: '2026-09-24 19:29'
updated_date: '2026-09-25 11:33'
labels:
  - camus-pl
dependencies: []
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Fossil: 32b10dfb1b

Follow-up on Backlog task-12 (ticket 01233cdb41). Camus PL grammar v0.4 introduces native laws: requires/ensures on function laws, invariant on struct laws (replacing the removed constraints blocks), spec functions (pure, single return), pure_expr (arithmetic, comparisons, result, old(...)) and assert statements.

Strictness (2026-09-23): an unproven law is a hard failure; no bypass construct exists. Verus is the first verification backend target (design only): Camus PL -> Rust+Verus -> verification -> compilable Rust -> executable.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [x] #1 grammar.ebnf documents laws (invariant/requires/ensures), spec functions, pure_expr, assert; constraints blocks removed
- [x] #2 checker parser accepts law blocks, spec functions and assert with canonical ordering (named laws sorted, requires before ensures, invariant never mixed)
- [x] #3 checker validates: struct laws invariant-only, function laws requires/ensures-only, result/old restricted to ensures, Boolean law clauses, pure spec functions, assert Boolean
- [x] #4 sample-project migrated from constraints to laws and passes camuspl check
- [x] #5 test suite green (14 integration tests incl. laws/spec/assert fixtures)
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Implemented and committed (fossil session update 2026-09-23 23:30).
- grammar.ebnf v0.4: law blocks (invariant/requires/ensures), spec functions, pure_expr (result/old restricted to ensures), assert; constraints blocks removed; canonical ordering (named laws sorted, requires before ensures, invariant never mixed); strictness: unproven law = hard failure.
- checker: parser + validator for laws, spec functions, assert; struct-field type resolution; fixed tokenizer 2-char operator bug and a parse infinite-loop.
- 14 integration tests green; sample-project migrated (Task.cam) and passes camuspl check.
- ARCHITECTURE.md updated incl. Verus backend mapping.
Ticket: 32b10dfb1b (closed).

Fossil: 14e0628c0a
<!-- SECTION:NOTES:END -->
