---
id: task-16
title: >-
  sample-project: App as server to TCC, TCC limited to lcr.standard for outgoing
  adaptation
status: To Do
assignee: []
created_date: '2026-09-28 21:13'
labels: []
dependencies: []
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Restructure the sample-project TCC/App/LCR boundaries after architecture review 2026-09-28 (NOT to be implemented that session — explicit user decision; revisit later, after open question is settled).

Violations today (tcc/Cli.cam): import + ctx.get of lcr.TaskStore and store.find(id) — direct use of a shared LCR method, already forbidden by ARCHITECTURE.md s1; and App used as a library (Task type exchanged, task.update on received task, id read from task) instead of a server receiving a request and answering.

Target: tcc/Cli.cam keeps only std.cli + lcr.standard, calls a request-style App entry point returning Result and translates success/failure into stdout + exit code. app/Task.cam owns ctx.get(TaskStore), find-by-id, mutation and persistence; entry points Task.create(...) -> Result and Task.update(id, ...) -> Result (id is an input: no id needed back for update; create returns the id in the Result value). hello-cli stays as-is (pure lcr.standard debug client, no App, no LCR).

Decided: (2) remove notify.success from App — single feedback channel is the returned Result, printed by the TCC. (3) harden ARCHITECTURE.md: TCC may access LCRs only through the well-known standard channel (lcr.standard) for outgoing adaptation; clarify "may use the interfaces exposed by LCRs where appropriate" to exclude application LCRs; state the App-as-server request/response model (TCC <-request/response-> App, App -> LCR via ctx).

OPEN (question 1): server surface shape — either the Task struct itself exposes the request entry points (tests: no extra layer, domain ops already public; cons: request shape coupled to domain, TCC sees domain type names) or a dedicated app facade/service struct (tests: explicit contract, domain private to App; cons: extra layer). Explain to the user and settle before implementing.

Fossil: b3b1b1c7399ae1ad496aae3a9102c3411b98e373
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 tcc/Cli.cam no longer imports or ctx.get any LCR (only lcr.standard) and never receives or mutates a Task; it calls an App request and translates the Result into stdout + exit code
- [ ] #2 App exposes request-style entry points that perform find-by-id, mutation and persistence internally; create returns the id via the Result value
- [ ] #3 notify.success removed from App; feedback flows only through the returned Result (TCC prints)
- [ ] #4 ARCHITECTURE.md hardened: TCC limited to lcr.standard for outgoing adaptation, App-as-server request/response model, application LCRs reserved to App via ctx
- [ ] #5 sample-project still builds and runs as a CLI; existing tests + E2E updated and green
<!-- AC:END -->
