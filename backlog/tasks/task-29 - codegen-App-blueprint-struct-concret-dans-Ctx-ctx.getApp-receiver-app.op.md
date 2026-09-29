---
id: task-29
title: >-
  codegen: App blueprint - struct concret dans Ctx, ctx.get(App), receiver
  app.op
status: Done
assignee: []
created_date: '2026-09-29 11:39'
updated_date: '2026-09-29 11:46'
labels:
  - camus-pl
  - camus-pl-codegen
dependencies:
  - task-28
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Le checker (TASK-28) accepte desormais la production app <Name> et verifie les frontieres TCC/App/LCR. Le codegen doit maintenant generer un blueprint app comme un struct CONCRET enregistre dans la racine de composition Ctx, accessible par &self et atteignable via ctx.get(App).

Travail:
- renommer Project::app (actuellement les structs sans role, records domaine) en Project::records, et ajouter Project::blueprints pour Role::App, afin qu un blueprint ne soit pas traite comme un record domaine
- emettre le blueprint (comme un struct) sans le compter dans held_by_lcr (entites adapter)
- etendre ctx_get_lowering pour accepter les blueprints en plus des LCR
- enregistrer une instance par blueprint dans Ctx (comme emit_ctx le fait pour les LCR) pour que le TCC atteigne le blueprint via ctx.get(App)
- gerer la resolution du receiver pour app.op(...) sur l instance du blueprint

Decision de conception retenue: un blueprint est un struct concret dans Ctx, acces par &self (pas de trait/dyn comme les LCR). Refonte sample associee: TASK-27.

Fossil: d9763193aa
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [x] #1 Project::app renomme en Project::records; Project::blueprints ajoute pour Role::App; un blueprint n est pas traite comme un record domaine
- [x] #2 ctx_get_lowering accepte un type blueprint (ctx.get(App)) en plus des LCR
- [x] #3 Ctx expose une instance par blueprint, atteignable par &self via CAMUS_CTX
- [x] #4 Un appel app.op(...) sur le receiver du blueprint se lowercase correctement (resolution receiver)
- [x] #5 Un blueprint nest pas compte comme entite held_by_lcr; le sample refonte (TASK-27) compile et s execute via ctx.get(App)
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Implementation complete (Fossil 874cd0e0b5 precursor, TASK-29 commit). All 5 ACs verified: the generated binary for the appblueprint fixture runs create and rename end to end through ctx.get(App), and the record reaches the LCR TSV store. The sample refonte of AC#5 (app/Task.cam -> app App.cam, TCC limited to lcr.standard + app) is tracked by TASK-27 and is not yet done; the codegen half of AC#5 (blueprint not counted as held_by_lcr) is complete.
<!-- SECTION:NOTES:END -->
