---
id: task-9
title: 'Dogfood cargo-kiss: annotate its own source with Camus SL and a Lexicon'
status: Done
assignee:
  - '@ai-agent'
created_date: '2026-08-14 17:19'
updated_date: '2026-09-10 20:40'
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
- [x] #1 camus/LEXICON.md exists and defines every term referenced by a terms: field in the codebase
- [x] #2 Every non-test function in cargo-kiss/src/*.rs has a preceding ## camus-sl block declaring intent, terms, and actions
- [x] #3 cargo kiss check is run against every source file and its output (including any remaining errors/warnings) is recorded in the task notes rather than silently fixed or hidden
- [x] #4 Any systemic rule tension discovered (e.g. closures vs. rule 12, primitive exposure vs. rule 11) is documented as a proposed exception with why/how/risks and explicitly flagged for human approval, not self-granted
- [x] #5 cargo build and cargo test still pass after annotation (doc comments must not break compilation)
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Annotated all 12 cargo-kiss/src/*.rs files with ## camus-sl blocks on every non-test function (test-module functions under #[cfg(test)] were intentionally left unannotated, per the task's own scope). Extended camus/LEXICON.md with 5 new terms needed to cover the remaining source (Lexicon, SignableFile, OpenSslProcess, ProcessExitCode, Cli), and fixed error.rs's exit_code block, which had used the non-compliant 'terms: none' (Camus.rs spec makes terms: REQUIRED with no none escape; only errors: supports an explicit none) -- it now declares 'terms: ProcessExitCode'.

cargo build and cargo test pass (3/3 tests, unchanged behavior; only doc comments and one minor rewrite were added).

cargo kiss check was run against all 12 files. Every remaining error/warning falls into one of five known categories, none of which were silently fixed or hidden:
1. Missing-block errors are ALL on #[cfg(test)] functions (out of scope by design): check.rs (6), constraint.rs (2), sign.rs (4), verify.rs (3), lexicon.rs (1), func_sign.rs (3, incl. a test-only helper). Zero missing-block errors remain on production functions.
2. Rule-12 closure errors (35 total across check.rs, constraint.rs, keys.rs, main.rs, openssl_util.rs, sign.rs, verify.rs, func_sign.rs, lexicon.rs) -- pre-existing, covered by camus/EXCEPTIONS.md's already-proposed Exception Candidate 1, awaiting human decision.
3. Function-length MUST violations (>50 lines), pre-existing, not refactored in this pass: check_source (116), check_sl_block (52) in check.rs; tokenize (64) in constraint.rs; generate_keypair (51) in keys.rs; cmd_sign (99) in main.rs; sign_functions (62) in func_sign.rs. Plus several SHOULD (>20 lines) warnings left as-is.
4. Line-length MUST violations (>120 chars), pre-existing: 2 in verify.rs (lines 230, 246 pre-edit), 2 in func_sign.rs. Plus many SHOULD (>80 chars) warnings left as-is.
5. One NEW discovery from dogfooding, documented in camus/EXCEPTIONS.md: (a) cargo kiss check src/camus_sl.rs reports 1 permanent false-positive 'malformed CAMUS block' error, because parse_blocks' line-based text scan cannot distinguish a real doc comment from lookalike text inside camus_sl.rs's own test fixtures (specifically rejects_unclosed_block's deliberately-unclosed fixture) -- disclosed, not fixed. (b) cargo kiss check src/func_sign.rs used to crash outright ('cannot parse string into token stream') because format_signature_block's backslash-continued string literal had a continuation line that looked like a doc comment to blank_doc_comment_lines and got blanked, corrupting the string -- this one WAS fixed (single-line format!, no behavior change), since it blocked check/sign from ever running on that file.

No LEXICON term-reference errors appear anywhere, confirming AC #1's Lexicon completeness. Certification (signing) remains untouched, per the task's explicit scope.
<!-- SECTION:NOTES:END -->
