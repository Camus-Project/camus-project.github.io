---
id: TASK-19
title: 'codegen: TCC entry points and LCR in-memory adapters'
status: To Do
assignee: []
created_date: '2026-09-25 11:54'
labels:
  - camus-pl
dependencies:
  - TASK-18
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Generate the boundary layers as executable Rust. TCC-role structs become program entry points; LCR-role structs become a Rust trait plus a generated DEFAULT IN-MEMORY adapter implementation, so the program runs end-to-end without any hand-written adapter code (decision 2b). Hand-written adapters injected via ctx are deferred to a later iteration. Part of the 'Camus PL to executable' milestone (Fossil 6a5506850d).
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 TCC-role structs (e.g. Cli) generate a program entry point that parses arguments and dispatches to the corresponding methods
- [ ] #2 Each LCR-role struct generates a Rust trait describing its surface
- [ ] #3 A default in-memory adapter implementing each LCR trait is generated so the program is runnable with no hand-written code
- [ ] #4 Generated code for the sample-project boundaries (Cli + TaskStore) compiles under cargo
- [ ] #5 Tests cover TCC entry-point generation and LCR trait + in-memory adapter generation
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Fossil: 6a5506850d
<!-- SECTION:NOTES:END -->
