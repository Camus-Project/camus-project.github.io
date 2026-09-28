---
id: TASK-22
title: 'codegen: debug_assert laws and full camus build to a working binary'
status: Done
assignee: []
created_date: '2026-09-25 11:55'
updated_date: '2026-09-28 20:57'
labels:
  - camus-pl
dependencies:
  - TASK-21
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Emit runtime law checks and complete the end-to-end build. Law clauses (requires/ensures/invariant) and assert statements are emitted as debug_assert! (decision 4), so they are enforced in debug builds and free in release. Provide a 'camus build <dir>' that generates a complete cargo project from a Camus PL project and runs cargo build, producing a working binary. The sample-project must run as a real CLI (decision 5). Part of the 'Camus PL to executable' milestone (Fossil 6a5506850d).
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [x] #1 requires/ensures/invariant and assert are emitted as debug_assert! at the appropriate points (entry, exit, after mutation/construction)
- [x] #2 old(...) and result in ensures are correctly captured for the debug_assert at function exit
- [x] #3 camus build <dir> generates a complete, buildable cargo project from a Camus PL project
- [x] #4 camus build runs cargo build and produces a binary; non-zero exit on build failure
- [x] #5 The sample-project builds and runs as a working CLI (create/list/update tasks per its intent)
- [x] #6 An end-to-end test builds and runs the sample-project binary
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Fossil: 6a5506850d

Fossil: 1b5daf21964d35b0242030ea6ad9e32f3daefac5

Closed 2026-09-28. Laws: requires (entry), ensures + old() and result capture, invariant after new and mutation, assert stmt -> debug_assert!(...). LCR find adapter inlines law_ensures_checks around __result. camus build: new [[bin]] camus (src/camus_build.rs) generates Cargo.toml + src/main.rs, runs cargo build, non-zero exit on failure, prints binary path. Persistence: CamusRow trait, camus_store_dir (CAMUS_STORE_DIR env, else cwd), camus_load/camus_dump, <Lcr>.tsv store files, camus.counter uuid sequence persisted (seed at startup, write when n%8==1) so ids survive across processes. Tests: laws.rs (debug_assert panic verified via catch_unwind), build_cmd.rs (E2E binary + failing build), entrance.rs sample_persists_across_process_invocations; 11 tests green, clippy clean.
<!-- SECTION:NOTES:END -->
