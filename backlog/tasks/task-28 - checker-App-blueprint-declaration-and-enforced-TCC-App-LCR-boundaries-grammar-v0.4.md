---
id: task-28
title: >-
  checker: App blueprint declaration and enforced TCC/App/LCR boundaries
  (grammar v0.4+)
status: To Do
assignee: []
created_date: '2026-09-29 11:09'
labels:
  - camus-pl
dependencies:
  - task-14
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Add a dedicated App blueprint to Camus PL and enforce the layer boundaries in the checker, per architecture review 2026-09-28 (decisions D1/D3/D4/D9).

Grammar: new production "app Name" (e.g. "app App" in app/App.cam), alongside the existing "struct X role tcc|lcr". An app is a blueprint, not a role: it owns the domain model + invariants, orchestrates LCRs via ctx, and exposes request-style operations (public functions -> Result) plus their laws. Imported as "import app.App".

Checker enforcement (D3): a TCC may import only lcr.standard + the app declarations; it can call app operations via ctx.get(App).method(...) but MUST NOT import or ctx.get any application LCR directly. The App may import and use any LCR via ctx. Reject: tcc importing lcr.* / tcc ctx.get of an LCR / app-side call lowering outside ctx.get.

Resolver: ctx.get(App|Lcr[, name]) single-segment canonical form (D4).

Fossil: efe4d64fc505bfa6a6ade922576b305c9eb456fc
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 grammar.ebnf declares the app production and distinguishes app blueprint vs struct roles (tcc|lcr)
- [ ] #2 checker parses App declarations and rejects a TCC that imports or ctx.get an application LCR (boundary rule enforced)
- [ ] #3 checker accepts App usage via ctx.get(App).op(...) from a TCC and LCR orchestration via ctx from the App
- [ ] #4 fixtures + tests cover: valid App, valid TCC->App, rejected TCC->LCR, rejected app lacking blueprint declaration
<!-- AC:END -->
