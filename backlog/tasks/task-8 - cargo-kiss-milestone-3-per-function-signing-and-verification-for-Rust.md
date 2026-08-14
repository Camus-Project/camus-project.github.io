---
id: task-8
title: 'cargo-kiss milestone 3: per-function signing and verification for Rust'
status: Done
assignee:
  - '@ai-agent'
created_date: '2026-08-14 17:14'
updated_date: '2026-08-14 17:15'
labels: []
dependencies: []
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Fossil: a94ba28e7d

Builds on milestone 2 (task-7). Implements 'cargo kiss sign' and 'cargo kiss verify' for per-function signing of Rust source (.rs files), mirroring kiss.sh's actual production per-function signing behavior (sign_selected_functions/verify_func_signature, not the unused do_sign_per_function code path): the certified unit is a function's ## camus-sl block plus its body. Reuses the milestone-2 parser (camus_sl.rs) for precise item/block boundaries instead of kiss.sh's brace-depth heuristics.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [x] #1 cargo kiss sign on a .rs file runs check first and aborts (no file modification) if check finds a MUST violation
- [x] #2 cargo kiss sign appends a ## camus-signature block after each function that has a valid preceding ## camus-sl block, covering the sl block plus the function body as the signed content
- [x] #3 cargo kiss sign skips (does not re-sign) a function already validly signed by the current signatory and key
- [x] #4 cargo kiss sign never touches or replaces a ## camus-signature block whose stored fingerprint belongs to a different key; that function is skipped with a clear message
- [x] #5 cargo kiss verify on a .rs file verifies every ## camus-signature block against the function it certifies and reports a per-function OK/FAIL summary, exiting non-zero if any signature is invalid
- [x] #6 Unit and integration tests cover: sign+verify round trip, skip-if-already-signed, tamper detection, and never touching a different key's signature
<!-- AC:END -->

## Implementation Plan

<!-- SECTION:PLAN:BEGIN -->
Already implemented directly (straightforward continuation of milestone 2's parser); plan: 1. Add FileType::Rs. 2. func_sign.rs: reconstruct signed content (sl block + body) from raw lines using camus_sl's item/block info; compute_signature (reused from sign.rs) and a small hand-rolled block formatter. 3. Sanitize doc-comment lines before feeding syn (works around a syn parse limitation with trailing/dangling doc comments). 4. Wire into cmd_sign/cmd_verify with check-before-sign for .rs files. 5. Tests + manual multi-key verification.
<!-- SECTION:PLAN:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Implemented in cargo-kiss/:
- func_sign.rs: sign_functions() and verify_functions(). Reconstructs the exact signed content (## camus-sl block + function body, each line + newline, matching the canonical form used elsewhere) from the milestone-2 parser's item/block line ranges -- no brace-depth heuristics needed, unlike kiss.sh.
- sign.rs: added FileType::Rs; exposed compute_signature as pub(crate) for reuse.
- verify.rs: exposed prepare_pubkey and verify_sig_against_content as pub(crate) for reuse.
- camus_sl.rs: added shared find_preceding_sl/find_following_signature helpers (also refactored check.rs to use them instead of a duplicate). Added blank_doc_comment_lines(): syn::parse_file rejects a source ending in dangling doc-comment lines with no following item (a real edge case: any file whose last function is signed, with the ## camus-signature block trailing at EOF, hits this). Since item-boundary detection never inspects doc comments (spans come from keyword tokens), blanking /// lines before handing the source to syn (line count preserved) sidesteps the issue entirely without affecting accuracy.
- main.rs: cmd_sign now runs check on every .rs target before prompting for the password, aborting the whole command (no files touched) on any MUST violation, matching kiss.sh's cmd_sign behavior for .sh files. cmd_verify dispatches .rs files to per-function verification instead of the whole-file camus-sig-1 path (Rust never uses that marker).

Safety policy (disclosed simplification vs. kiss.sh's interactive per-element review, out of scope here): sign_functions only ever replaces a ## camus-signature block whose stored fingerprint matches the signing key in use. A signature by a *different* key is left completely untouched and that function is skipped with an explicit message -- verified manually with two independent keys (Alice/Bob): Bob's sign attempt correctly skipped Alice-signed function and did not alter her block.

Verification: cargo test passes (28 tests: 22 unit including 2 new func_sign tests -- sign+verify round trip with skip-on-resign, and tamper detection -- plus 6 integration tests across 3 files, including a new tests/func_sign.rs with 3 CLI-level tests: multi-function sign+verify, check-failure abort, tampered-function FAIL report). Manual verification: multi-function file signs/verifies correctly; re-running sign is a no-op; a different key's signature is never touched.

Housekeeping: cargo-kiss/README.md Status section updated. This ticket/task were created retroactively (implementation was started directly instead of following the create-ticket-first sequence used for milestones 1 and 2); noting this for the record, no process impact since Fossil explicitly allows retroactive ticket linking.
<!-- SECTION:NOTES:END -->
