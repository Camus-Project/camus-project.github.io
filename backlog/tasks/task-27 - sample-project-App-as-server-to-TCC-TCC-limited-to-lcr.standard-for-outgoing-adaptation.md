---
id: task-27
title: >-
  sample-project: App as server to TCC, TCC limited to lcr.standard for outgoing
  adaptation
status: To Do
assignee: []
created_date: '2026-09-29 11:05'
updated_date: '2026-09-29 15:37'
labels: []
dependencies:
  - task-30
  - task-32
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Restructure the sample-project TCC/App/LCR boundaries after architecture review 2026-09-28, re-scoped 2026-09-29 with architecture decisions D1-D9.

Violations today (tcc/Cli.cam): import + ctx.get of lcr.TaskStore and store.find(id) — direct use of a shared LCR method, already forbidden by ARCHITECTURE.md s1; and App used as a library (Task type exchanged, task.update on received task, id read from task) instead of a server receiving a request and answering.

SETTLED (D1): the App is a dedicated blueprint — new `app App` declaration in app/App.cam (import app.App, driven via ctx.get(App)); it owns the request-style operations (create_task/update_task/list_tasks/complete_task -> Result) plus their laws, and orchestrates the domain structs and LCRs via ctx. The domain model (Task) stays a plain struct owning its invariants; consumers do not repeat them.

Target: tcc/Cli.cam keeps only std.cli + lcr.standard, calls the App request entry points via ctx.get(App) + app.operation(...) (D9) and translates the returned Result into stdout + exit code. notify.success is removed from the App; the single feedback channel is the returned Result, printed by the TCC (D6). hello-cli stays as-is (pure lcr.standard debug client).

Fossil: b3b1b1c7399ae1ad496aae3a9102c3411b98e373
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 tcc/Cli.cam no longer imports or ctx.get any LCR (only lcr.standard) and never receives or mutates a Task; it resolves the App via ctx.get(App) and translates the returned Result into stdout + exit code
- [ ] #2 App exposes request-style operations ( blueprint) that perform find-by-id, mutation and persistence internally and keep their laws (requires/ensures); create returns the id via the Result value
- [ ] #3 notify.success removed from App; feedback flows only through the returned Result (TCC prints)
- [ ] #4 ARCHITECTURE.md hardened: TCC limited to lcr.standard for outgoing adaptation, App-as-server request/response model, application LCRs reserved to App via ctx, boundary rules enforced by the checker (D3)
- [ ] #5 sample-project still builds and runs as a CLI; existing tests + E2E updated and green
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Etat a jour au 2026-09-29 (instance precedente terminee).

Realise depuis:
- TASK-28: Role::App, parse_app, regles de frontiere TCC/App/LCR dans le checker, grammaire v0.5
- TASK-29: App blueprint genere comme struct concret dans Ctx, ctx.get(App), receiver app.op(...)
- TASK-30: Result<T> generique, when accepte une valeur Result, return construit le payload Ok, pass-through d un Result deja present. checker 26 puis 30, codegen 13 puis 14
- TASK-32: statement if (then/else obligatoires), discipline d initialisation minimale. Les deux etapes bloquantes pour cette tache sont levees.

Deux bugs du parseur corriges en TASK-32 (un ? renvoyant None sans diagnostic, appele par un appelant qui ignorait ce None): un let sans valeur etait supprime silencieusement, et un if sans else etait accepte silencieusement.

La proposition de refonte (5 items: App.cam, Task.cam reduit, Cli.cam reecrit, tests E2E, ARCHITECTURE.md) a ete presentee mais PAS executee. Aucune modification du sample-project.

Prealable de conception non resolu: la decision humaine du 2026-09-29 (ticket bdd9d017) impose que match et when partagent un mecanisme de pattern matching unique, differant par la temporalite. Cela vaut pour la reecriture de Cli.cam, qui doit traduire le Result en stdout + code de sortie. Voir TASK-33. TASK-31 traite l constructeur Err et les codes de sortie.
<!-- SECTION:NOTES:END -->
