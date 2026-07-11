---
layout: page
title: Camus.rs — Rust Adaptation
---

> **TL;DR**
>
> Camus.rs adapts the Camus Method to Rust. It provides a disciplined workflow for producing Rust code that can be specified, reviewed, audited, and ultimately certified by a human.
>
> **AI may write. Only a human may certify.**

[The Camus Method](/The-Camus-Method) was born from a simple observation: AI has dramatically increased our ability to produce software, but not our ability to acquire assumable confidence in its behavior.

Camus.rs is the Rust adaptation of these ideas. It combines behavioral specifications, human review, and cryptographic attestation into a workflow that makes AI-assisted Rust development reviewable, auditable, and certifiable.

## How Camus.rs Emerged

The first implementation envisioned for the Camus Method was **Camus.rs**.

Its development began with the writing of documentation, specifications, and AI skills describing both the methodology and its future implementation. Since many of these artifacts were themselves produced with AI assistance, a natural question emerged:

> If AI-generated software should not be trusted without review and certification, why should AI-generated specifications?

To answer that question, a small Bash script was written to cryptographically sign those documents while waiting for Camus.rs.

Reviewing that script led to an unexpected realization: it had itself become AI-assisted software and therefore ought to satisfy the Camus Method before it could be trusted.

The roadmap changed.

The signing script gradually evolved into **[kiss.sh](https://github.com/Camus-Project/kiss.sh)**. Developing kiss.sh according to the Camus Method required adapting the methodology to shell scripting. **[Camus.sh](/adaptations/Camus.sh/)** emerged from that necessity.

In this sense, Camus.sh is more than an adaptation of the Camus Method to Bash. It is the result of the Camus Method being applied to its own artifacts.

This was the first demonstration that the Camus Method could be applied to its own development process.

Along the way, Camus.sh introduced the first experimental syntax for **[Camus SL (Specification Language)](/Camus-SL)** and provided the experience needed for Camus SL to evolve into an independent specification language.

Camus.rs now benefits from that experience. The Camus.sh workflow has been validated in practice. The Camus SL syntax has stabilized through real use. The **[cargo-kiss](https://github.com/Camus-Project/cargo-kiss)** tool provides the same signing and verification capabilities as kiss.sh, extended to Rust source code and Cargo projects.

## Core Principles

- **Behavioral specifications** expressed in doc-comment blocks
- **Function-level attestation** through cryptographic signatures
- **Automated compliance checking** with **cargo-kiss**
- **Human accountability. AI may write. Only a human may certify.**

## Key Concepts

Every Camus.rs project is organized around three complementary concepts.

### Lexicon

A shared project vocabulary declared in module-level doc comments or in a standalone `LEXICON.md`. Rust's type system provides a natural substrate for lexicon terms: each term is realized by a type, and type signatures enforce term boundaries at compile time.

### Grammar

Behavioral specifications written in structured doc comments, describing a function's intent, inputs, outputs, side effects, assumptions, and guarantees. The Rust type system enforces several Grammar rules at compile time — mutability defaults, ownership constraints, and type boundaries — reducing the burden on runtime verification.

### Certification

Human review followed by cryptographic attestation. The human reviewer inspects each function, verifies that its implementation matches its specification, and signs it using `cargo kiss sign`. The signature is appended to the source file as a structured block.

## Deliverables

Camus.rs consists of three complementary deliverables.

### Specification

The normative specification of the Camus.rs methodology.

It defines how Camus SL is embedded in Rust source code through structured doc comments, how compliance is checked against the 15 Grammar rules of the Camus Method, and how cryptographic attestation is structured in the Rust context.

### SKILL.md

Instructions enabling AI agents to understand and apply the Camus.rs workflow.

### cargo-kiss

The reference implementation of the Camus.rs workflow.

It automates:

- compliance checking against the Camus.rs specification;
- cryptographic signing of reviewed functions;
- signature verification;
- key management.

Certification itself, using cargo-kiss, always remains a human decision.

cargo-kiss is the Rust rewrite of the Camus tooling. It provides the same guarantees as kiss.sh while leveraging Rust's type system and build tooling for tighter integration with Cargo projects.

## Examples

### Excerpt from a Rust module

The following function illustrates the three building blocks of Camus.rs: a **Camus SL specification**, the **implementation**, and the **cryptographic certification** performed after human review.

```rust
/// ## camus-sl
/// intent: check that a file path exists on disk
/// input:
///   path: file path to verify
/// output:
///   Result<PathBuf, CamusError>: the validated path, or an error
/// errors:
///   CamusError::FileNotFound: file does not exist at the given path
/// ## camus-end
pub fn assert_file_exists(path: impl AsRef<Path>) -> Result<PathBuf, CamusError> {
    let path = path.as_ref();
    if path.exists() {
        Ok(path.to_path_buf())
    } else {
        Err(CamusError::FileNotFound {
            path: path.display().to_string(),
        })
    }
}
/// ## camus-signature
/// signatory: Lan Jing
/// date: 2026-07-10T12:00:00Z
/// fingerprint: sha256:52:69:05:07:66:BD:DE:55:C7:D2:B1:52:9C:8F:ED:7B:05:E3:8E:57:7E:10:98:1E:7C:BD:13:96:85:4E:83:89
/// signature: <base64-encoded Ed25519 signature>
/// ## camus-end
```

### `cargo kiss check` session

```
$ cargo kiss check
Checking: src/lib.rs

  [OK] All public functions preceded by camus-sl blocks
  [OK] All camus-sl blocks declare intent
  [OK] All camus blocks properly closed
  [OK] No function exceeds 50 lines
  [OK] All lines under 120 characters
  [WARN] Function 'parse_config' is 32 lines (SHOULD <= 20)

Result: 1 warning(s)
```

## Status

Camus.rs is an early-stage adaptation of the Camus Method to Rust.

Its purpose is to transfer the validated Camus.sh workflow to Rust, leverage Rust's type system for stronger compile-time guarantees, and establish the foundations for `cargo-kiss` as the reference Rust tool.

The specification, AI skill, and tool are under active development.

Discussion and feedback are welcome via the project's issue tracker.
