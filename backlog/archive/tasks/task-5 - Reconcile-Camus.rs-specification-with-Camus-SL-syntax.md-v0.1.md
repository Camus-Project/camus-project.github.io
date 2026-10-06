---
id: task-5
title: Reconcile Camus.rs specification with Camus SL syntax.md v0.1
status: Done
assignee:
  - '@ai-agent'
created_date: '2026-08-14 16:18'
updated_date: '2026-08-14 16:27'
labels: []
dependencies: []
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Fossil: b6bdffbb4f

Camus-SL/syntax.md v0.1 generalizes the block-based Camus SL syntax and introduces realizedBy, an explicit constraint grammar, and signatory/date/fingerprint/signature as the core CAMUS-SIGNATURE fields. It documents (Relation to Existing Adaptations) that reconciling Camus.rs's prose '- **Realized by**:' label and its author/date/version/hash/key_id signature fields with these generalized keys is tentative and explicitly out of scope, deferred to a dedicated approved task. Camus.rs/specification.md also mandates a camus/LEXICON.md-only lexicon location and a required actions: key, both diverging from syntax.md's more permissive rules. This divergence must be resolved (by aligning Camus.rs, or by explicitly documenting and justifying each remaining difference as a Rust-specific specialization) before cargo-kiss can implement a stable CAMUS-SL/CAMUS-LEXICON/CAMUS-SIGNATURE parser and checker against a single unambiguous spec.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [x] #1 Camus.rs/specification.md's term-to-component key is aligned with syntax.md's realizedBy, or the divergence is explicitly documented and justified
- [x] #2 Camus.rs/specification.md's CAMUS-SIGNATURE core fields are aligned with syntax.md's signatory/date/fingerprint/signature, with any Rust-specific fields (version, key_id) kept only as documented additional fields per syntax.md's extension allowance
- [x] #3 A decision on whether Camus.rs adopts syntax.md's explicit constraint grammar is documented in Camus.rs/specification.md, either by adopting it or by explicitly opting out with rationale
- [x] #4 Camus.rs/specification.md's camus/LEXICON.md-only rule is reconciled with syntax.md's lexicon location rule, or explicitly justified as a stricter Rust-specific constraint
- [x] #5 Camus.rs/specification.md's required actions: key is reconciled with syntax.md's optional actions, either by keeping it required with documented rationale or by aligning it as optional
- [x] #6 Camus-SL/syntax.md's Relation to Existing Adaptations table and tentative-divergence note for Camus.rs are updated to reflect the reconciled state
<!-- AC:END -->

## Implementation Plan

<!-- SECTION:PLAN:BEGIN -->
1. Re-read Camus-SL/syntax.md (generic v0.1) and Camus.rs/specification.md side by side, list every concrete divergence (realizedBy vs '- **Realized by**:' prose, signature fields, constraint grammar absence, LEXICON location, required actions key).\n2. For each divergence, decide: align Camus.rs to syntax.md, or keep a documented Rust-specific specialization with rationale. Present this decision matrix to the human for approval before editing normative spec text.\n3. Once approved, edit Camus.rs/specification.md: adopt realizedBy in the CAMUS-LEXICON section; adopt signatory/date/fingerprint/signature as core CAMUS-SIGNATURE fields, keeping version/key_id as documented additional fields; add or explicitly decline the constraint grammar with rationale; reconcile the camus/LEXICON.md-only rule; reconcile the required actions key.\n4. Update Camus-SL/syntax.md's 'Relation to Existing Adaptations' table and remove/update the 'tentative... out of scope' note for Camus.rs now that it is reconciled.\n5. Update the worked example in Camus.rs/index.md and specification.md if field names changed, so examples stay consistent.\n6. Run diagnostics on edited files; manually check Jekyll front matter consistency (local Jekyll build has a pre-existing, unrelated gem environment issue per task-4 notes).\n7. Record implementation notes and check off acceptance criteria one by one.
<!-- SECTION:PLAN:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Reconciled Camus.rs/specification.md (bumped to v0.1.1) with Camus-SL/syntax.md v0.1:
- Adopted realizedBy (was prose '- **Realized by**:') in CAMUS-LEXICON, keeping Camus.rs's Markdown bullet-list LEXICON.md layout (layout unification is a separate, larger question, not part of this task).
- Adopted signatory/date/fingerprint/signature as the core CAMUS-SIGNATURE fields (was author/date/version/hash/key_id). Fixed a real gap: Camus.rs previously had no field carrying the actual Ed25519 signature value (hash: was a content digest, not a signature) -- verification was not actually possible under the old field set. version/key_id kept as documented optional additional fields.
- Adopted the explicit constraint: key and grammar from syntax.md into CAMUS-SL (Camus.rs previously had no constraint mechanism at all; the Grammar-rules table wrongly mapped rule 5 to terms:/actions:).
- Added a new 'Relation to Camus SL Syntax Specification' section documenting the two remaining intentional, stricter Rust-specific specializations (camus/LEXICON.md-only location, required actions:) with rationale, per the approved decision matrix.
- Fixed two pre-existing block-marker bugs found while editing: the CAMUS-SL example was missing its closing ## camus-end, and the CAMUS-SIGNATURE syntax block closed with a repeated ## camus-signature instead of ## camus-end.
- Updated Camus.rs/SKILL.md (agent skill) to match: LEXICON format, CAMUS-SL keys/example, CAMUS-SIGNATURE keys/templates -- otherwise an agent following the skill would immediately reintroduce the old, now-retired syntax.
- Updated Camus-SL/syntax.md's Relation to Existing Adaptations table and reconciliation note to reflect the resolved state instead of 'tentative/out of scope'.

Incident during task setup: the backlog CLI reused id task-4 for this task's initial creation (remote git fetch failure confused its id counter), overwriting the pre-existing, untracked task-4 (Define Camus SL concrete syntax) on disk. Restored task-4 verbatim from content read earlier in session, moved the new task to task-5, and fossil-added both task-4, task-5 and Camus-SL/syntax.md (all three were previously untracked in Fossil, which is how the collision went undetected by 'fossil status'). Fossil ticket: b6bdffbb4f.
<!-- SECTION:NOTES:END -->
