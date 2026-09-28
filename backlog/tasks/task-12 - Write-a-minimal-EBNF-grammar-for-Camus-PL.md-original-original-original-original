---
id: task-12
title: Write a minimal EBNF grammar for Camus PL
status: To Do
assignee:
  - '@ai-agent'
created_date: '2026-09-10 21:31'
labels: []
dependencies: []
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Fossil: 01233cdb41

Agreed next step for Camus PL (see Camus-PL/ARCHITECTURE.md and Camus-PL/sample-project/). Everything designed so far exists only as prose and worked examples -- no grammar or parser exists to check a .cam file against, unlike Camus SL/cargo-kiss where a real parser already caught two real bugs during dogfooding (task-9).

Scope: write a minimal EBNF (or equivalent formal grammar notation, e.g. in a new Camus-PL/grammar.ebnf or similar) covering the Camus PL surface designed so far:
- struct declarations, including `role tcc` / `role lcr`
- the two-level declaration pattern: `<kind> <name>` / declarative properties / `<kind>` / concrete definition (ARCHITECTURE.md section 12)
- function declarations: visibility (public/private/shared), `intention`, `constraints` with `input:` entries, the nested `function(Type: name, ...) -> ReturnType` signature, `mut` on `self` and on parameters
- struct fields, including `mut`
- imports
- the constraint-predicate syntax used in `constraints` blocks (e.g. `!empty(name)`)

Out of scope for this task (still experimental, not settled -- see ARCHITECTURE.md): async/`later` semantics, `ctx`/dependency-resolution configuration format, combined `tcc+lcr` roles.

Suggested validation: run the grammar by hand against the three existing sample-project files (app/Task.cam, lcr/TaskStore.cam, tcc/Cli.cam) and confirm they parse under it; fix or flag any mismatch found between the grammar and the existing examples.

Not a request to build a compiler or interpreter -- a written, checkable grammar is the deliverable.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 A written EBNF (or equivalent) grammar file exists for Camus PL, covering struct/role, the two-level declaration pattern, function declarations (visibility, intention, constraints, mut), struct fields (incl. mut), imports, and the constraint-predicate syntax
- [ ] #2 The grammar is checked by hand against all three sample-project files (Task.cam, TaskStore.cam, Cli.cam); any mismatch found is fixed or explicitly flagged, not silently ignored
- [ ] #3 Async/later, ctx configuration, and tcc+lcr are explicitly left out of scope, not guessed at
<!-- AC:END -->
