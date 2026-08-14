---
id: task-7
title: 'cargo-kiss milestone 2: CAMUS-SL/LEXICON parser + cargo kiss check'
status: Done
assignee:
  - '@ai-agent'
created_date: '2026-08-14 16:52'
updated_date: '2026-08-14 17:02'
labels: []
dependencies: []
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Fossil: dc1ab6e48a

Builds on milestone 1 (task-6). Implements a parser for CAMUS-SL and CAMUS-SIGNATURE doc-comment blocks in Rust source, per the reconciled Camus.rs/specification.md v0.1.1 and Camus-SL/syntax.md v0.1, plus a parser for camus/LEXICON.md and a validator for the constraint: expression grammar. Wires all of this into a real 'cargo kiss check' command, replacing the milestone-1 not-implemented stub. Deep-semantic Grammar rules (component realization, parameter passing, mutability, public primitives, block depth) that cannot be reliably checked from source text alone without a full Rust AST/type-checker are explicitly out of scope and documented as deferred.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [x] #1 cargo-kiss parses ## camus-sl and ## camus-signature doc-comment blocks from a Rust file into a structured representation, associated with the function/item they precede, for both single-line and multi-line (input:/output:/actions: nested) key values
- [x] #2 cargo-kiss parses camus/LEXICON.md (Markdown bullet-list format: definition/realizedBy/actions) into a Lexicon of terms
- [x] #3 cargo-kiss validates a constraint: value against the Camus SL constraint grammar (comparators, field access, and/or/not, literals) and reports a syntax error for anything outside that grammar
- [x] #4 cargo kiss check <file> reports, per function/item: missing or empty intent/terms/actions, terms not found in the project Lexicon (when camus/LEXICON.md is found), malformed constraint expressions, unclosed camus-sl/camus-signature blocks, lines over 120 characters (error) and 80 characters (warning), and functions over 50 lines (error) and 20 lines (warning)
- [x] #5 cargo kiss check <file> reports traits without a camus-sl block declaring actions, and structured exceptions blocks missing rule/why/how/risks
- [x] #6 cargo kiss check exits 0 when only SHOULD warnings are present, and non-zero when a MUST rule is violated, matching kiss.sh's OK/WARN/ERROR reporting convention
- [x] #7 Unit tests cover the parser (single- and multi-line blocks, malformed/unclosed blocks) and the constraint grammar validator (valid and invalid expressions), plus an integration test running cargo kiss check against a small fixture Rust file
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Implemented in cargo-kiss/:
- camus_sl.rs: line-based scanner for ## camus-sl / ## camus-signature blocks (supports both inline 'key: value' and nested 'key:' + indented sub-keys, e.g. actions:/requires:/provides: and exceptions:/rule:/why:/how:/risks:), plus a syn-based item collector (functions incl. impl/trait fns, structs, traits, enums, and closures) using span-locations for precise, attribute-independent line boundaries. Deliberately does NOT rely on syn's doc-comment-to-item attachment for signature blocks, since rustc/syn would (mis)attach a trailing block to whatever item follows it rather than the one it certifies.
- lexicon.rs: parses camus/LEXICON.md's Markdown bullet-list format (definition/realizedBy/actions) and searches upward from a file's directory for camus/LEXICON.md, per Camus.rs's fixed lexicon-location rule.
- constraint.rs: hand-written recursive-descent validator for the Camus SL constraint grammar (comparators, field access via '.', and/or/not, number/string/boolean literals); syntax-only, since constraints are declarative and never executed.
- check.rs + cargo kiss check: reports per kiss.sh's [OK]/[WARN]/[ERROR] convention, grouped by category with synthesized [OK] lines for clean categories. Checks: camus-sl presence + intent/terms/actions non-empty, lexicon term references (when camus/LEXICON.md found), constraint syntax, trait semantic contracts (rule 14), exceptions block completeness (rule 15), no closures (rule 12), line length (120 MUST/80 SHOULD), function length (50 MUST/20 SHOULD). Exit codes: 125 (E_CHECK_FAILED) on any MUST violation, 53 (W_COMPLIANCE_ONLY) on SHOULD-only warnings, 0 when clean -- matching kiss.sh's do_check exactly.

Explicitly deferred (documented in cargo-kiss/README.md Status section): component realization, parameter-passing mode, variable mutability, public primitives, and block depth (rules 2, 9, 10, 11, 6) -- these need real semantic/type analysis, not just source text + syn's syntax tree. Per-function signing and verifying a camus-signature block's adjacency to its signed item are also deferred to a future milestone.

Verification: 20 unit tests (parser: inline/nested fields, unclosed blocks, function span excluding doc comments, closure detection; constraint grammar: valid/invalid expressions; lexicon parsing; check: missing block, undefined lexicon term, long function warning vs error, closures) + 3 integration tests running the compiled cargo kiss check binary against fixture files (compliant fixture, missing block, undefined lexicon term) -- all pass. Also manually verified against a small multi-function fixture combining a compliant function, an undefined-lexicon-term reference, and a syntactically invalid constraint: report correctly isolated both real issues while leaving the compliant function and its camus-signature block untouched.

Housekeeping: cargo-kiss/README.md Status section and command table updated to reflect 'check' now being implemented (with its documented scope limits). Committed to the cargo-kiss git repo (not yet pushed to origin, same as milestone 1).
<!-- SECTION:NOTES:END -->
