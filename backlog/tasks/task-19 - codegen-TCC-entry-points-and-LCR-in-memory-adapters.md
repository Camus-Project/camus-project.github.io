---
id: task-19
title: 'codegen: TCC entry points and LCR in-memory adapters'
status: Done
assignee: []
created_date: '2026-09-25 11:54'
updated_date: '2026-09-28 16:00'
labels:
  - camus-pl
dependencies:
  - task-18
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Generate the boundary layers as executable Rust. TCC-role structs become program entry points; LCR-role structs become a Rust trait plus a generated DEFAULT IN-MEMORY adapter implementation, so the program runs end-to-end without any hand-written adapter code (decision 2b). Hand-written adapters injected via ctx are deferred to a later iteration. Part of the 'Camus PL to executable' milestone (Fossil 6a5506850d).
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [x] #1 TCC-role structs (e.g. Cli) generate a program entry point that parses arguments and dispatches to the corresponding methods
- [x] #2 Each LCR-role struct generates a Rust trait describing its surface
- [x] #3 A default in-memory adapter implementing each LCR trait is generated so the program is runnable with no hand-written code
- [x] #4 Generated code for the sample-project boundaries (Cli + TaskStore) compiles under cargo
- [x] #5 Tests cover TCC entry-point generation and LCR trait + in-memory adapter generation
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Implementation: codegen src/entrance.rs (TCC main() dispatch + camus_parse_* helpers + usage table; LCR trait + InMemory adapter w/ CRUD bodies, fallback unimplemented!). translate_call lowers lcr.standard.print / lcr.notify.success/error and .toString(); PRELUDE adds CamusResult + console. Cli.cam TCC emitted as struct; TaskStore -> trait TaskStore + pub struct InMemoryTaskStore. hello-cli example runs: 'greet Alice' -> 'Hello, Alice!'. Tests: tests/entrance.rs (2: hello entry runs binary + LCR adapter shape), generate.rs sample updated. Fossil: cfdac15ddd0e539e460e6ffdb11d6a1dedc24dc1
<!-- SECTION:NOTES:END -->
