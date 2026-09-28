---
id: task-15
title: >-
  codegen: canonical LCR addressing as ctx.get(lcr.<X>) with well-known
  reconfigurable lcr.standard
status: To Do
assignee: []
created_date: '2026-09-28 20:52'
labels: []
dependencies: []
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Replace the compiler-baked `lcr.standard.*` / `lcr.notify.*` free-function sugar with a first-class, composition-root-resolved LCR surface. The canonical identity for addressing an LCR resource becomes `lcr.<X>`: `ctx.get(lcr.standard)` yields the well-known client channel, bound to stdin/stdout/stderr by default and reconfigurable through the runtime config tree (no Camus source change). The form extends to every LCR (`ctx.get(lcr.TaskStore)`, `ctx.get(lcr.SqliteConnection, "database")`).

Optional sugar: keep `lcr.<X>.method(...)` as desugaring of `ctx.get(lcr.<X>).method(...)`, or drop it — is to be decided against the Camus Method tone (Camus leans against sugar; argue before keeping). Admitting `lcr.<X>` in ctx.get is a parser + lowering change: today ctx_get lowering requires a single-segment type name (gen.rs) and would reject the dotted path.

Fossil: ffc96e0ff9903c4ff35e8e1e8cbdd333292968cc
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 ctx.get accepts a dotted `lcr.<X>` identity for every LCR (standard, notify, and generated adapters), lowering to the composition root
- [ ] #2 lcr.standard is bound to stdin/stdout/stderr by default and can be reconfigured at the composition root / config tree without any Camus source change
- [ ] #3 The sugar question is resolved explicitly: keep `lcr.<X>.method(...)` as sugar, or drop it in favour of the canonical `ctx.get(lcr.<X>)` form (Camus tone: lean against sugar)
- [ ] #4 hello-cli and/or sample-project updated to the chosen form; tests cover LCR addressing and reconfiguration
<!-- AC:END -->
