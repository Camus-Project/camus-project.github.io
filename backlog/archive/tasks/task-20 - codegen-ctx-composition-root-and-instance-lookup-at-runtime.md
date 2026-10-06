---
id: TASK-20
title: 'codegen: ctx composition root and instance lookup at runtime'
status: Done
assignee: []
created_date: '2026-09-25 11:54'
updated_date: '2026-10-06 14:26'
labels:
  - camus-pl
dependencies: []
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Turn the ctx configuration format (TASK-16) and the TCC instance lookup mechanism (TASK-17) into a real runtime composition root: instantiate LCR adapters, wire dependencies, and resolve ctx.get(X) / instance lookups to concrete generated instances at program startup. Part of the 'Camus PL to executable' milestone (Fossil 6a5506850d).
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [x] #1 ctx configuration is compiled into a composition root that instantiates and wires the generated LCR adapters at startup
- [x] #2 ctx.get(X) resolves to the concrete generated instance in the running program
- [x] #3 Instance lookup/retrieval for TCCs (per TASK-17 semantics) is realized at runtime
- [x] #4 The sample-project wires Cli -> Task -> TaskStore end-to-end and compiles
- [x] #5 Tests cover ctx resolution and instance wiring
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Implementation: generated composition root (entrance::emit_ctx) instantiates InMemory<Lcr> adapters in Ctx::new(); static CAMUS_CTX: OnceLock<Ctx> initialized in main() at startup. ctx.get(Type[,name]) lowers to CAMUS_CTX.get().unwrap().<snake>(Type)() returning &dyn Type; LCR-typed let bindings annotate as &dyn; store.find(id) dispatches via trait object (Cli.update). Adapters switched RefCell->Mutex<HashMap> so Ctx is Sync. Sample project wires Cli->Task->TaskStore: generated code compiles and runs until the TASK-21 async path (Task::store unimplemented by design). Tests: entrance.rs ctx_composition root assertions (5 ACs). Fossil: 45ae927f606bf24baa2e42c06a00394a51dc0b98
<!-- SECTION:NOTES:END -->
