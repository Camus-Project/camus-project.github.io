---
id: task-6
title: 'cargo-kiss milestone 1: scaffold + keygen/list-keys/sign-verify (whole-file)'
status: Done
assignee:
  - '@ai-agent'
created_date: '2026-08-14 16:33'
updated_date: '2026-08-14 16:50'
labels: []
dependencies: []
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Fossil: b79684b535

First implementation milestone for cargo-kiss (currently only LICENSE + README.md in cargo-kiss/), now that Camus.rs/Camus-SL syntax has been reconciled (task-5). Ports kiss.sh's key management and whole-file signing (the camus-sig-1 marker format already used across this repo's README/blog files) to a Cargo binary crate, matching kiss.sh's on-disk key layout and signature format exactly so keys and signed files remain interoperable between the two tools. Per-function signing, cargo-kiss check (compliance checking against Camus.rs, which needs CAMUS-SL parsing), and interactive multi-file review are explicitly out of scope for this milestone.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [x] #1 cargo-kiss keygen generates a password-protected Ed25519 self-signed certificate and key pair in the same on-disk layout as kiss.sh (private-<fpr>.pem, public-<fpr>.pem, private.pem/public.pem symlinks), honoring --key-dir and CAMUS_KEY_PATH
- [x] #2 cargo-kiss list-keys lists public keys in a key directory with their SHA256 fingerprint and expiry date, matching kiss.sh's output format
- [x] #3 cargo-kiss sign appends a camus-sig-1 whole-file signature block (matching kiss.sh's exact block format) to .txt/.md/unknown-extension files, refusing to double-sign an already-signed file
- [x] #4 cargo-kiss verify can verify a camus-sig-1 whole-file signature block, including a signature produced by kiss.sh itself (cross-tool interoperability), reporting signatory, date, and fingerprint
- [x] #5 cargo-kiss verify rejects a signature whose signing key had already expired at the signature date, and warns when verifying with a key expiring soon
- [x] #6 A round-trip test (keygen, sign, verify) passes via cargo test
<!-- AC:END -->

## Implementation Plan

<!-- SECTION:PLAN:BEGIN -->
1. Scaffold Cargo binary crate in cargo-kiss/ (edition 2024): Cargo.toml with clap (derive), rpassword, base64, chrono, thiserror, tempfile.\n2. main.rs: clap CLI (keygen, list-keys, sign, verify, version; check stubbed as not-yet-implemented), argv normalization for both 'cargo-kiss <cmd>' and 'cargo kiss <cmd>' invocation.\n3. openssl_util.rs: thin wrapper to shell out to the openssl CLI (req -x509 -newkey ed25519 for keygen; x509 -fingerprint/-enddate/-pubkey; pkey -pubin -outform DER; pkeyutl -sign/-verify -rawin), for exact interoperability with kiss.sh-generated keys/certs and signatures.\n4. keys.rs: keygen (self-signed cert + password-protected key, named/symlinked like kiss.sh), fingerprint_of, fingerprint_filepath, find_key_by_fingerprint, list_keys, key_expiry_info/cert_valid_at (chrono-based parsing of openssl date output).\n5. password.rs: hidden password prompt (rpassword) + confirmation, honoring CAMUS_TEST_SIGN_PASSWORD like kiss.sh for testability.\n6. sign.rs: detect_file_type, is_signed (camus-sig-1 marker), compute_signature, format_whole_signature (byte-identical block format to kiss.sh, including the <pre>/</pre> wrapping for .md), sign_whole_file.\n7. verify.rs: locate last camus-sig-1 marker, reconstruct signed content by line count (mirroring kiss.sh's txt vs md offset), resolve pubkey by fingerprint or --pubkey/key-dir, prepare_pubkey (extract raw key from cert + expiry-at-signature-date check), verify signature.\n8. Round-trip integration test: keygen -> sign -> verify on a temp dir/key, plus a fixed-fixture test verifying an existing kiss.sh-signed file in this repo (e.g. kiss.sh/README.md) with the project's real public key, proving cross-tool interoperability.\n9. Update cargo-kiss/README.md installation/status section to reflect what is implemented vs. deferred (check, per-function signing).\n10. cargo build + cargo test; fix diagnostics; check off ACs; write implementation notes.
<!-- SECTION:PLAN:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Implemented in cargo-kiss/ (Cargo binary crate, edition 2024, Rust 1.97):
- keys.rs: keygen (openssl req -x509 -newkey ed25519 self-signed cert, password-protected key via -passin/-passout stdin rather than kiss.sh's pass:<value> CLI arg -- avoids leaking the password via process argument lists), fingerprint_of, find_key_by_fingerprint, list_keys, key_expiry_days/cert_valid_at (chrono-based parsing of openssl's date output). Same on-disk layout as kiss.sh: private-<fpr>.pem/public-<fpr>.pem + private.pem/public.pem symlinks.
- sign.rs / verify.rs: whole-file camus-sig-1 sign and verify (txt/md/unknown), byte-identical block format to kiss.sh including the <pre>/</pre> wrapping for Markdown. .sh files are explicitly skipped with a clear message (per-function signing deferred).
- openssl_util.rs: shells out to the openssl CLI for all X.509/Ed25519 operations, for exact interoperability with kiss.sh-generated keys and signatures, rather than reimplementing PKI handling in Rust.
- error.rs: named KissError enum (thiserror) whose exit codes mirror kiss.sh's E_* constants (e.g. 110 = invalid signature, 104 = key expired) so scripts checking exit status behave identically regardless of which tool ran.
- cargo-kiss check is a deferred, clearly-erroring stub (compliance checking needs a CAMUS-SL parser, out of scope here).

Verification:
- cargo build and cargo test pass (7 unit tests + 3 integration tests: full round-trip, tamper detection with correct exit code 110, double-sign refusal).
- Cross-tool interoperability verified manually and bidirectionally: a .txt file signed by kiss.sh verifies successfully with cargo-kiss, and a .md file (exercising the <pre> wrapping) signed by cargo-kiss verifies successfully with kiss.sh ('Signature Verified Successfully' / 'OK -- valid signature', matching fingerprints and dates both ways).
- cargo-kiss/README.md updated to reflect implemented vs. deferred commands.

Housekeeping: cargo-kiss/ turned out to already be its own git repository (github.com/Camus-Project/cargo-kiss, previously containing only LICENSE+README.md). Added cargo-kiss/.gitignore (/target) and committed the new source there (2 local commits, not yet pushed to origin -- pending explicit confirmation before pushing to the public GitHub repo). Also added cargo-kiss/target to the workspace's Fossil ignore-glob (tools/update-fossil-ignore.sh flagged it) so Rust build artifacts are never picked up by 'fossil addremove'.

Not implemented in this milestone (tracked as future follow-up, consistent with the task description's stated scope): cargo-kiss check, per-function signing, interactive multi-file review.
<!-- SECTION:NOTES:END -->
