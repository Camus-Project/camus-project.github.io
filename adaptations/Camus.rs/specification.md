---
layout: page_with_toc
title: Camus.rs Specification
---

# Camus.rs Specification v0.1.1 (Draft)

## Purpose

Camus.rs defines a restricted Rust profile intended for human-reviewed and human-assumed software artifacts — typically produced by AI and, eventually, by humans. The profile applies the Camus Method to Rust code.

A SKILL.md is provided for AI agents that wish to follow the Camus Method and the Camus.rs profile in the Rust context. When loaded, the skill guides the agent through the Lexicon, Grammar, and Certification phases defined by the Method.

The profile provides a uniform structure that enables:

- human review;
- function-level documentation;
- function-level attestation;
- automated processing by tools;
- cryptographic signing of reviewed artifacts.

Camus.rs is intentionally more restrictive than the underlying Rust language it supports.

**cargo-kiss** is the Rust adaptation of the **kiss** companion tool. It automates compliance checking, block validation, and cryptographic signing of Camus.rs artifacts.

---

## Relation to Camus SL Syntax Specification

Camus.rs implements the generic [Camus SL Syntax Specification](/Camus-SL/syntax). Block
delimiters, the `realizedBy` lexicon key, the `constraint:` grammar, and the core
`## camus-signature` fields (`signatory`, `date`, `fingerprint`, `signature`) follow that
specification exactly.

Two Rust-specific specializations remain, both intentional and stricter than the generic
specification:

| Aspect | Camus SL Syntax Specification (generic) | Camus.rs |
|---|---|---|
| Lexicon location | Dedicated file recommended; inline `CAMUS-LEXICON` shorthand allowed for small/standalone scripts | `camus/LEXICON.md` only, no inline form ([Camus Directory](#camus-directory)) |
| `actions:` | Optional | REQUIRED on every `## camus-sl` block ([CAMUS-SL](#camus-sl)) |

**Why stricter:**

- **Lexicon location** — the inline shorthand targets small, standalone scripts (e.g. Bash).
  A Cargo project always has a natural place for a dedicated file; allowing an inline form
  would fragment the vocabulary across source files without a compensating benefit.
- **`actions:` required** — Camus.rs's [Semantic Contracts](#semantic-contracts) model ties
  trait compatibility directly to declared actions; making the key optional would silently
  weaken that guarantee.

Camus.rs also defines one additional key not present in the generic specification:

- `errors:` — OPTIONAL, Rust-specific. Named error conditions using Rust error enums (see
  [CAMUS-SL](#camus-sl) and [Error Handling](#error-handling)). Permitted under the generic
  specification's allowance for host adaptations to add further keys for host-specific
  reporting needs.

---

## Rust Version

Camus.rs targets **Rust Edition 2024** (latest stable).

The profile is forward-looking: it adopts current stable features and does not provide backward compatibility guarantees for older editions.

---

## Source Encoding

Source files MUST:

- use UTF-8 encoding;
- use LF line endings.

---

## Camus Directory

Every Camus.rs project MUST contain a `camus/` directory at the project root.

This directory is lowercase, not hidden, and contains:

- `LEXICON.md` — the project's single source of truth for terms and actions

No LEXICON block appears in source code. The Lexicon lives exclusively in `camus/LEXICON.md`.

---

## Function Declaration

Functions MUST be declared using Rust's standard syntax:

```rust
fn function_name() {
    // ...
}
```

No alternative declaration syntax is permitted within Camus.rs scope.

---

## Camus Blocks

Camus.rs uses doc comment blocks for structured metadata.

Camus blocks are embedded inside Rust doc comments (`///`) and delimited by markers:

- `/// ## camus-sl` — opens a specification block
- `/// ## camus-signature` — opens a signature block

Content inside a block uses standard key-value pairs.

| Block | Placement | Purpose |
|---|---|---|
| `## camus-sl` | Before a function/item | Declares intent, terms, actions, inputs, outputs, constraints, errors |
| `## camus-signature` | After a signed item | Cryptographic attestation |

### CAMUS-SL

Placed immediately before a function or item definition, inside doc comments.

Keys MUST be in lowercase.

The following keys are defined:

- `intent:` — REQUIRED. Natural language description of what the function does.
- `terms:` — REQUIRED. Lexicon terms used by this function.
- `actions:` — REQUIRED. Semantic actions declared by this function (provided or expected).
- `input:` — OPTIONAL. Terms consumed as parameters.
- `output:` — OPTIONAL. Terms produced as return values.
- `constraint:` — OPTIONAL, repeatable. Explicit, checkable condition on the terms received or
  produced, using the constraint grammar defined in the
  [Camus SL Syntax Specification](/Camus-SL/syntax#constraint-grammar).
- `errors:` — OPTIONAL. Named error conditions (using Rust error enums).

Example:

```rust
/// ## camus-sl
/// intent: compute the sum of two integers
/// terms: Integer
/// actions: provides addition
/// input: Integer (augend), Integer (addend)
/// output: Integer (sum)
/// constraint: augend >= 0 and addend >= 0
/// errors: none
/// ## camus-end
fn add(augend: &Integer, addend: &Integer) -> Integer {
    augend + addend
}
/// ## camus-signature
/// signatory: reviewer-identifier
/// date: 2026-07-14T12:00:00Z
/// fingerprint: sha256:<hex>
/// signature: <base64-encoded Ed25519 signature>
/// version: 0.1.0
/// ## camus-end
```

### CAMUS-SIGNATURE

Placed after a signed item, inside doc comments.

**Only a human MAY append or authorize a signature block.**
Automated or AI-generated signatures are invalid. An AI may, with
explicit human authorization, generate a temporary signature for
testing purposes during development, but that signature MUST be
removed before the artifact is delivered.

**The signing private key MUST be protected by a password.**
A passwordless key enables automated signing without human
involvement and is therefore invalid for production use.
A passwordless key MAY be used temporarily for local testing.

Keys MUST be in lowercase.

Core fields, aligned with the generic [Camus SL Syntax Specification](/Camus-SL/syntax#certification-camus-signature):

- `signatory:` — REQUIRED. Identifies the human certifier.
- `date:` — REQUIRED. ISO 8601 timestamp of certification.
- `fingerprint:` — REQUIRED. Fingerprint of the signing key.
- `signature:` — REQUIRED. Base64-encoded Ed25519 signature.

Camus.rs additionally allows the following documented fields, appended after the core fields:

- `version:` — OPTIONAL. Crate/semver version at signing time.
- `key_id:` — OPTIONAL. Short local identifier for the signing key, when referencing the full
  fingerprint is impractical.

```rust
/// ## camus-signature
/// signatory: <identifier>
/// date: <iso 8601>
/// fingerprint: sha256:<hex>
/// signature: <base64>
/// version: <semver>
/// key_id: <identifier>
/// ## camus-end
```

---

## CAMUS-LEXICON

The Lexicon is defined in `camus/LEXICON.md` at the project root.

It defines terms AND actions. Each term declares:

- Its definition
- The component that realizes it
- The actions it provides or expects

Format:

```markdown
## Term: <TermName>

- **definition**: Clear description of the concept
- **realizedBy**: ComponentName
- **actions**:
  - <action-name>: <semantic description>
```

Example:

```markdown
## Term: Session

- **definition**: An authenticated connection between client and server
- **realizedBy**: SessionManager
- **actions**:
  - authenticate: validate credentials and establish session
  - refresh: extend session lifetime
  - revoke: terminate session
```

The `realizedBy` key name is shared with the generic
[Camus SL Syntax Specification](/Camus-SL/syntax#lexicon-format); Camus.rs keeps its
Markdown bullet-list layout for `camus/LEXICON.md` rather than the brace-delimited
`term { ... }` form, since that layout question is separate from key naming and is not
part of this reconciliation.

Without a Lexicon, there is no shared vocabulary between human and AI.
With a Lexicon, ambiguity shrinks.

---

## Size Limits

### Functions

Functions MUST NOT exceed 50 lines.

Functions SHOULD NOT exceed 20 lines.

### Line Length

Lines MUST NOT exceed 120 characters.

Lines SHOULD NOT exceed 80 characters.

---

## Error Handling

Camus.rs uses Rust's `Result<T, E>` type with named error enums.

Error enums MUST be explicitly defined for each module or crate:

```rust
#[derive(Debug, thiserror::Error)]
enum SessionError {
    #[error("invalid credentials")]
    InvalidCredentials,

    #[error("session expired")]
    SessionExpired,

    #[error("session not found: {0}")]
    NotFound(String),
}
```

The `I_`/`W_`/`E_` convention from Camus.sh does not apply to Rust.
Error semantics are expressed through enum variants and their names.

---

## Signature Scope

Camus.rs v0.1.1 defines functions as the primary review and attestation unit.

All functions MUST be signed, regardless of visibility:

- `pub` functions
- `pub(crate)` functions
- private functions

**Only a human may sign.** Signing is the act of assuming
responsibility for a function's correctness. An automated or
AI-generated signature carries no such responsibility and is
therefore invalid by definition.

**The signing private key MUST be password-protected.** A
passwordless private key enables unsupervised signing and
defeats the human accountability that signatures are meant to
provide. Passwordless keys are valid for local testing only.

This specification defines the structure of signature blocks
(see [CAMUS-SIGNATURE](#camus-signature)). Automated verification and key management are
delegated to cargo-kiss.

---

## Closures

Closures are **prohibited** in Camus.rs code.

Rationale: AI-generated code must prioritize readability. Closures are not truly named functions. Every code unit must have an explicit name to be auditable and certifiable.

This applies to:

- `|x| x + 1` syntax
- `Fn`, `FnMut`, `FnOnce` trait bounds in function signatures
- Closure parameters in function definitions

Exception via rule 15 with documented justification.

---

## Semantic Contracts

Traits (interfaces) are compatible with Camus.rs ONLY if they carry explicit semantic contracts.

A trait without semantic contract is **prohibited** (exception via rule 15).

A semantic contract declares:

- The **actions** the trait requires from implementors
- The **actions** the trait provides to callers
- The **terms** involved in each action

Example:

```rust
/// ## camus-sl
/// intent: define authentication behavior
/// terms: Session, Credentials
/// actions:
///   requires: validate credentials
///   provides: establish session, terminate session
/// ## camus-end
trait Authenticator {
    fn authenticate(&self, credentials: &Credentials) -> Result<Session, AuthError>;
    fn revoke(&self, session: &Session) -> Result<(), AuthError>;
}
```

A structure realizing a term MUST declare the actions it fulfills:

```rust
/// ## camus-sl
/// intent: manage session lifecycle
/// terms: Session
/// actions:
///   provides: authenticate, refresh, revoke
/// ## camus-end
struct SessionManager { /* ... */ }
```

Semantic continuity:

> intent → action → term → implementation

---

## Relation to the Camus Method

Camus.rs applies the Camus Method to Rust. The table below
maps the 15 Grammar rules defined by the Method to this specification.

| # | Rule | Status | Notes |
|---|---|---|---|
| 1 | Term Definition | Applicable | Via `camus/LEXICON.md` ([CAMUS-LEXICON](#camus-lexicon)) |
| 2 | Component Realization | Applicable | Each component realizes a term, declared in Lexicon |
| 3 | Function Terms | Applicable | Terms and actions declared in `## camus-sl` ([CAMUS-SL](#camus-sl)) |
| 4 | Function Claim | Applicable | Declared via `intent:` in `## camus-sl` ([CAMUS-SL](#camus-sl)) |
| 5 | Function Constraints | Applicable | Declared via the `constraint:` key in `## camus-sl`, using the Camus SL constraint grammar ([CAMUS-SL](#camus-sl)) |
| 6 | Block Depth | Applicable | Enforced by Rust's syntax (no nested function definitions) |
| 7 | Line Length | Applicable | See [Size Limits](#size-limits) |
| 8 | Function Length | Applicable | See [Size Limits](#size-limits) |
| 9 | Parameter Passing | Applicable | Pass by reference (`&T`) where possible; ownership rules apply |
| 10 | Variable Mutability | Applicable | Immutable by default; `mut` keyword required for mutation |
| 11 | No Public Primitives | Applicable | No public primitive fields or return types without wrapper types |
| 12 | No Anonymous Functions | Applicable | Closures prohibited ([Closures](#closures)); every function named |
| 13 | No Inheritance | Applicable | Rust has no inheritance; composition via traits and enums |
| 14 | No Interfaces without Contracts | Applicable | Traits require semantic contracts ([Semantic Contracts](#semantic-contracts)) |
| 15 | Explicit Exceptions | Applicable | Deviations documented in `## camus-sl` with why/how/risks |

---

## Exceptions

Any Camus rule may be excepted.

Documentation is mandatory in the `## camus-sl` block:

- **Why**: the reason for the exception
- **How**: the implementation approach
- **Risks**: what the exception implies

Workflow:

1. AI identifies need for exception
2. AI requests human approval with full justification
3. Human approves or rejects
4. Only after approval: implementation

Presenting a fait accompli at review time is **forbidden**.

Exception documentation in the SL block:

```rust
/// ## camus-sl
/// intent: [function intent]
/// terms: [terms]
/// actions: [actions]
/// exceptions:
///   rule: 12 (No Anonymous Functions)
///   why: [reason for exception]
///   how: [implementation approach]
///   risks: [what this implies]
/// ## camus-end
```

---

## Signed Function Protection

Once a function is signed, the AI may not modify it without explicit developer approval.

No workarounds are permitted:

- Renaming a function to create a new version while abandoning the old one is prohibited
- Refactoring remains an objective to limit code drift
- The developer's signature is a commitment that must not be circumvented

If a signed function needs modification, the AI must:

1. Identify the function and its current signature
2. Explain why the modification is needed
3. Request explicit approval from the developer
4. Only proceed after approval is received

---

## Design Rationale

Camus.rs treats functions as units of human responsibility.

Each function SHOULD implement a single identifiable responsibility.

A reviewer is expected to understand, verify, and assume individual functions rather than arbitrary fragments of source code.

The type system is a complement to Camus, not a substitute. Types express structural contracts; Camus expresses semantic contracts. Both are needed for human auditability.

Rust's ownership model provides strong guarantees that simplify review: memory safety, thread safety, and absence of data races are enforced by the compiler, allowing the reviewer to focus on semantic correctness.
