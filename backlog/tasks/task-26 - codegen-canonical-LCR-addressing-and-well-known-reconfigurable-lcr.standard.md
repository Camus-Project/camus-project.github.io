---
id: TASK-26
title: 'codegen: canonical LCR addressing and well-known reconfigurable lcr.standard'
status: To Do
assignee: []
created_date: '2026-09-29 11:05'
labels: []
dependencies: []
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Replace the compiler-baked lcr.standard.* / lcr.notify.* free-function sugar with a first-class, composition-root-resolved LCR surface. RESCOPED 2026-09-29 (architecture decision D4): all resource access goes through ctx.get only; the dotted lcr.<X> path (including ctx.get(lcr.<X>)) is dropped and the lcr.<X>.method(...) sugar is NOT kept (Camus tone: lean against sugar). LCRs are addressed by their generated type name (ctx.get(TaskStore), ctx.get(SqliteConnection, "database")); lcr.standard remains the well-known runtime-bound standard channel (stdin/stdout/stderr by default, reconfigurable at the composition root / config tree without any Camus source change), addressed through ctx.get like any LCR.

Original scope note: admitting a dotted path in ctx.get is a parser + lowering change; today ctx_get lowering requires a single-segment type name (gen.rs) and would reject a dotted path.

Fossil: ffc96e0ff9903c4ff35e8e1e8cbdd333292968cc
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 All resource access uses the single ctx.get(Type[, name]) form; no lcr.<X> dotted path and no lcr.<X>.method(...) sugar are admitted
- [ ] #2 lcr.standard is bound to stdin/stdout/stderr by default and can be reconfigured at the composition root / config tree without any Camus source change
- [ ] #3 ctx.get is extended to resolve the well-known lcr.standard channel and every generated LCR/adapter
- [ ] #4 sample-project updated to the chosen form; tests cover LCR addressing and reconfiguration
<!-- AC:END -->
