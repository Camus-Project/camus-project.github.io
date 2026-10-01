---
id: task-25
title: 'codegen: concurrent tokio runtime for later/when/wait/ignore'
status: To Do
assignee: []
created_date: '2026-09-29 11:05'
updated_date: '2026-10-01 19:55'
labels:
  - camus-pl
dependencies:
  - task-21
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Replace the synchronous prototype execution model (TASK-21, decision 3) with a real CONCURRENT runtime based on tokio: later spawns tasks, when branches on completion, wait awaits the handle, ignore detaches. Required before Camus PL is considered production-ready. Deferred by design from TASK-21; do not implement a concurrency runtime in the generated prelude until this is done.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 A tokio-based runtime crate is used for generated code (later spawns, wait awaits, when branches, ignore detaches)
- [ ] #2 The sample-project CLI still works and behaves identically to the synchronous prototype
- [ ] #3 Tests cover concurrent later/when/wait/ignore behavior
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Fossil: a32d5424a42d940c91d3b7924ab74ca666764823

2026-10-01 — runtime re-integrated on trunk (Fossil b7123cf107), task left To Do.

The TASK-25 work was stranded when the two branches were reconciled: leaf
9f7a409757 was closed rather than merged, so trunk had neither the runtime crate
nor rtlink.rs. Cherry-picking 7ce2235c6c whole was rejected — it carried 24
files with 8 conflict hunks spanning grammar.ebnf and ARCHITECTURE.md, which
encode the v0.9 when-only design. Resolving those would pre-empt the decision
open in Fossil 78688008 / task-37. Only the design-neutral parts were taken.

Present and working:
- camus_runtime crate builds; its own suite passes 7/7, including
  concurrent_waiters_all_observe_the_outcome and
  later_runs_the_work_off_the_calling_thread
- codegen declares the module (gen.rs: `pub mod rtlink;` plus the `pub use`)
- asyncall fixtures and the codegen test helper are in the tree

Deliberately not taken: grammar.ebnf, ARCHITECTURE.md, entrance.rs, generate.rs,
laws.rs. So the asyncall fixtures exist but nothing drives them.

Why each AC stays unchecked:
- AC1 tokio runtime used by generated code: partially true. The crate exists and
  is wired into codegen, but no codegen test exercises it, so "used by generated
  code" is not demonstrated.
- AC2 sample-project CLI behaves identically to the sync prototype: NOT
  verifiable without the entrance.rs / generate.rs changes.
- AC3 tests cover concurrent later/when/wait/ignore: partially true. The runtime
  tests cover later/wait/ignore concurrency. `when` is not covered, because the
  when-side tests live in the files that were not taken.

Non-obvious adaptation to preserve: runtime/Cargo.toml carries an empty
[workspace] table. This is deliberate. rtlink.rs builds the crate with
--manifest-path and then looks for the rlib under runtime/target/debug, which is
where cargo places it only for a standalone package. Adding runtime to the
Camus-PL workspace members would relocate the artifact to the shared target
directory and find_rlib would fail. Do not "fix" this by joining the workspace.

Tracked, not worked: the sample-project CLI comparison (AC2) and the when-side
concurrency coverage (AC3) both depend on decisions still open in Fossil
78688008 and task-37. Finishing TASK-25 before those land would re-introduce the
grammar churn that task-37 is trying to settle.
<!-- SECTION:NOTES:END -->
