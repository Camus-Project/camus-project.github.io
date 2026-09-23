---
id: task-9
title: 'Dogfood cargo-kiss: annotate its own source with Camus SL and a Lexicon'
status: In Progress
assignee:
  - '@ai-agent'
created_date: '2026-08-14 17:19'
updated_date: '2026-08-14 17:19'
labels: []
dependencies: []
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Fossil: 9281d5dcf0

Applies the Camus.rs profile cargo-kiss itself implements to cargo-kiss's own source (camus_sl.rs, check.rs, constraint.rs, error.rs, func_sign.rs, keys.rs, lexicon.rs, main.rs, openssl_util.rs, password.rs, sign.rs, verify.rs). Creates camus/LEXICON.md and adds ## camus-sl blocks to every non-test function, then runs cargo kiss check against the codebase and reports results honestly. Test functions (#[cfg(test)] modules) are explicitly out of scope for annotation in this pass, to keep the effort focused on the certified production surface. Certification (signing) remains exclusively human and is out of scope for this task.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 camus/LEXICON.md exists and defines every term referenced by a terms: field in the codebase
- [ ] #2 Every non-test function in cargo-kiss/src/*.rs has a preceding ## camus-sl block declaring intent, terms, and actions
- [ ] #3 cargo kiss check is run against every source file and its output (including any remaining errors/warnings) is recorded in the task notes rather than silently fixed or hidden
- [ ] #4 Any systemic rule tension discovered (e.g. closures vs. rule 12, primitive exposure vs. rule 11) is documented as a proposed exception with why/how/risks and explicitly flagged for human approval, not self-granted
- [ ] #5 cargo build and cargo test still pass after annotation (doc comments must not break compilation)
<!-- AC:END -->
