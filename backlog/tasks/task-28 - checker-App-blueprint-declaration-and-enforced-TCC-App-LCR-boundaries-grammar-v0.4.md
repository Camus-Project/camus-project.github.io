---
id: task-28
title: >-
  checker: App blueprint declaration and enforced TCC/App/LCR boundaries
  (grammar v0.4+)
status: Done
assignee: []
created_date: '2026-09-29 11:09'
updated_date: '2026-09-30 07:55'
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
- [x] #1 grammar.ebnf declares the app production and distinguishes app blueprint vs struct roles (tcc|lcr)
- [x] #2 checker parses App declarations and rejects a TCC that imports or ctx.get an application LCR (boundary rule enforced)
- [x] #3 checker accepts App usage via ctx.get(App).op(...) from a TCC and LCR orchestration via ctx from the App
- [x] #4 fixtures + tests cover: valid App, valid TCC->App, rejected TCC->LCR, rejected app lacking blueprint declaration
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Reconciliation 2026-09-30 (instance de suivi): les 4 AC ont ete reverifies UN PAR UN contre le code et les fixtures, pas seulement lus. AC1: grammar.ebnf:277 app_decl = "app" , identifier ; role_kind reste "tcc" | "lcr" (grammar.ebnf:269), donc struct X role app est bien invalide. AC2: check.rs:156 check_import_boundaries rejette l import d un lcr applicatif et l import d un app.X non blueprint; check.rs:937 rejette le ctx.get d un LCR applicatif depuis un TCC. AC3: good/result_payload.cam (app TicketApp + struct TicketCli role tcc qui fait ctx.get(TicketApp) puis app.create_ticket, l App orchestrant le LCR via ctx.get(TicketStore)) est accepte par le checker. AC4: fixtures good/app.cam, good/result_payload.cam, bad/tcc_imports_lcr.cam, bad/app_not_blueprint.cam, chacune avec un test dedie dans checker/tests/check.rs (tcc_imports_lcr_rejected, tcc_ctx_get_lcr_rejected, tcc_app_not_blueprint_rejected, result_payload_passes). La fixture TCC->App a ete formalisee en TASK-30 plutot qu en TASK-28, mais la couverture exigee par l AC est presente et testee. Conclusion: TASK-28 livree, la tacher etait restee To Do par defaut de mise a jour.
<!-- SECTION:NOTES:END -->

## Final Summary

<!-- SECTION:FINAL_SUMMARY:BEGIN -->
TASK-28 livree et reverifiee: production app dans la grammaire (role_kind reste tcc|lcr, donc role app invalide), enforcement des frontieres TCC/App/LCR dans le checker (imports, import d un app.X non blueprint, ctx.get d un LCR applicatif), et 4 fixtures Positive/n egative couvertes par des tests dedies. Le statut Backlog etait reste To Do et le ticket Fossil ouvert; corriges le 2026-09-30.
<!-- SECTION:FINAL_SUMMARY:END -->
