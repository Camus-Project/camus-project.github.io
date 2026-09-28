---
id: TASK-12
title: Write a minimal EBNF grammar for Camus PL
status: Done
assignee:
  - '@ai-agent'
created_date: '2026-09-10 21:31'
updated_date: '2026-09-25 11:33'
labels: []
dependencies: []
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Fossil: 01233cdb41

Agreed next step for Camus PL (see Camus-PL/ARCHITECTURE.md and Camus-PL/sample-project/). Everything designed so far exists only as prose and worked examples -- no grammar or parser exists to check a .cam file against, unlike Camus SL/cargo-kiss where a real parser already caught two real bugs during dogfooding (task-9).

Scope: write a minimal EBNF (or equivalent formal grammar notation) in Camus-PL/grammar.ebnf covering the Camus PL surface designed so far:
- canonical form: exactly one textual representation per program (single-space token separation, 2-space indentation levels, no trailing whitespace, alphabetical import order, fixed section order, full-line comments only). Motivation: formatting changes must never invalidate the hash of a signed unit.
- struct declarations: exactly one role per struct (tcc or lcr, role optional); combined tcc+lcr roles are dropped (2026-09-11)
- the two-level declaration pattern (ARCHITECTURE.md section 12)
- function declarations: visibility (public/private/shared), intention, constraints with input: entries, the nested function(Type: name, ...) -> ReturnType signature, mut on self and on parameters
- struct fields, including mut
- imports
- the constraint-predicate syntax used in constraints blocks (e.g. !empty(name))
- local bindings: let name: Type = value with explicit types (no inference), let mut ... for reassignable bindings
- copy for working copies (expression and parameter forms)
- async: later (bound only, never anonymous), when (success/failure branches, always both, at most one branch may be ignore -- both ignore is forbidden, use `ignore call` instead),
  wait (on a named handle), ignore (fire-and-forget replacing later)
- classical operator hierarchy: or / and / comparisons / + - / * / % / unary not, left-associative (ARCHITECTURE.md/grammar section 12)
- # full-line comments

Out of scope for this task (still experimental, not settled -- see ARCHITECTURE.md): ctx/dependency-resolution configuration format.

Suggested validation: run the grammar by hand against the three existing sample-project files (app/Task.cam, lcr/TaskStore.cam, tcc/Cli.cam) and confirm they parse under it; fix or flag any mismatch found. The sample project has been rewritten to match the settled syntax (explicit types, let, when/ignore, sorted imports, wait in store).

Not a request to build a compiler or interpreter -- a written, checkable grammar is the deliverable.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [x] #1 A written EBNF (or equivalent) grammar file exists for Camus PL, covering struct/role, the two-level declaration pattern, function declarations (visibility, intention, constraints, mut), struct fields (incl. mut), imports, and the constraint-predicate syntax
- [x] #2 The grammar encodes canonical form: exactly one textual representation per program (single-space tokens, 2-space indent, no trailing whitespace, alphabetical imports, fixed section order), so formatting changes cannot invalidate a signature hash
- [x] #3 The grammar covers let + explicit types (no inference), copy (expression and parameter), and the async API: later (bound only), when (both branches, ignore allowed), wait (named handle), ignore (fire-and-forget)
- [x] #4 A comment syntax (# full-line) is defined and covered by the grammar
- [x] #5 The grammar is checked by hand against all three sample-project files (Task.cam, TaskStore.cam, Cli.cam); any mismatch found is fixed or explicitly flagged, not silently ignored
- [x] #6 ctx configuration is explicitly left out of scope, not guessed at
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
grammar.ebnf reached v0.4 (2026-09-23), covering struct/role, two-level declarations, functions (visibility, intention, laws), fields (incl. mut), imports, constraint-predicate syntax (now law clauses), let/copy/async/operators/full-line comments.
Validated by the checker (camuspl check) against all three sample-project files (Task.cam, TaskStore.cam, Cli.cam) — passes.
ctx configuration remained out of scope (own task created).
Ticket: 01233cdb41 (closed).

Fossil: 14e0628c0a
<!-- SECTION:NOTES:END -->
