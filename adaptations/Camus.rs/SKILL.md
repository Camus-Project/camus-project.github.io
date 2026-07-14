---
name: camus-rs
description: Apply the Camus.rs profile — a restricted Rust coding profile based on the Camus Method. Use this skill whenever the user asks to write, review, or certify Rust code under the Camus.rs specification. The skill guides AI agents through the Lexicon, Grammar, and Certification phases for Rust context.
---

# Camus.rs Agent Skill

This skill implements the **Camus.rs specification** (see `specification.md`).
It applies the three Camus Method phases to Rust programming with the
restrictions and conventions defined by the profile.

## The Workflow

```
1. Define the Lexicon (terms and actions the code manipulates)
2. Write code following Camus.rs Grammar rules
3. Review and verify compliance
4. Sign and certify (future: cargo-kiss sign)
```

---

## Phase 1: Lexicon

The Lexicon defines **what** the code manipulates. Terms are declared
in `camus/LEXICON.md` at the project root. No LEXICON block appears
in source code.

### Lexicon Format

Create `camus/LEXICON.md`:

```markdown
# Project Lexicon

## Term: [TermName]
- **Definition**: Clear description of the concept
- **Realized by**: ComponentName
- **Actions**:
  - <action-name>: <semantic description>
```

### Interaction with AI

- Reference terms from the Lexicon explicitly in prompts
- Ask the AI to explain how it maps each term to components
- Verify that new code introduces only defined terms
- Verify that actions are declared for each term

---

## Phase 2: Grammar

Rust code written under Camus.rs MUST respect the following
rules, derived from the specification.

### Core Rules (Must Follow)

1. **Term Definition** — Every term MUST be defined in `camus/LEXICON.md`
2. **Component Realization** — Each component MUST realize exactly one term, explicitly declared
3. **Function Terms** — Every function MUST declare its input and output terms
4. **Function Claim** — Every function MUST declare what it does (its claim)
5. **Function Constraints** — Declare explicit, verifiable constraints on parameters
6. **Block Depth** — Functions MUST limit block depth to one level (no nested blocks)
7. **Line Length** — Lines MUST NOT exceed 120 characters (SHOULD NOT exceed 80)
8. **Function Length** — Functions MUST NOT exceed 50 lines (SHOULD NOT exceed 20)
9. **Parameter Passing** — Pass by reference (`&T`) where possible
10. **Variable Mutability** — Variables MUST be immutable by default

### Prohibition Rules

11. **No Public Primitives** — Never expose primitives as public variables or return values
12. **No Anonymous Functions** — Closures are prohibited; every function must have an explicit name
13. **No Inheritance** — Favor composition over inheritance (Rust has no inheritance)
14. **No Interfaces without Contracts** — Traits MUST carry explicit semantic contracts
15. **Explicit Exceptions** — Deviations documented with why/how/risks

### Semantic Contracts

When writing code:
- Declare actions expected from parameters in the SL block
- Declare actions provided by the function in the SL block
- Ensure continuity: intent → action → term → implementation
- Traits MUST carry semantic contracts (actions from the Lexicon)
- Reject traits without semantic contracts

When reviewing code:
- Verify that traits carry semantic contracts
- Verify that actions match Lexicon definitions
- Reject traits without semantic contracts

### Camus Blocks

Camus.rs uses doc comments (`///`) for structured metadata.

Two block types:

- `## camus-sl` — semantic specification
- `## camus-signature` — cryptographic signature

### CAMUS-SL Block

Placed immediately before a function or item, inside doc comments.

Keys (lowercase):

- `intent:` — REQUIRED. What the function does.
- `terms:` — REQUIRED. Lexicon terms used.
- `actions:` — REQUIRED. Semantic actions declared.
- `input:` — OPTIONAL. Terms consumed.
- `output:` — OPTIONAL. Terms produced.
- `errors:` — OPTIONAL. Named error conditions.
- `exceptions:` — OPTIONAL. Documented rule exceptions.

Example:

```rust
/// ## camus-sl
/// intent: compute the sum of two integers
/// terms: Integer
/// actions: provides addition
/// input: Integer (augend), Integer (addend)
/// output: Integer (sum)
/// errors: none
fn add(augend: &Integer, addend: &Integer) -> Integer {
    augend + addend
}
```

### CAMUS-SIGNATURE Block

Placed after a signed item, inside doc comments.

Keys (lowercase):

- `author:` — identifier of the signer
- `date:` — ISO 8601 timestamp
- `version:` — semantic version
- `hash:` — sha256 fingerprint
- `key_id:` — signing key identifier

**Only a human MAY sign.** AI-generated signatures are invalid.
**Private key MUST be password-protected.**

### Error Handling

Use `Result<T, E>` with named error enums:

```rust
#[derive(Debug, thiserror::Error)]
enum SessionError {
    #[error("invalid credentials")]
    InvalidCredentials,

    #[error("session expired")]
    SessionExpired,
}
```

The `I_`/`W_`/`E_` convention from Camus.sh does not apply to Rust.

### Verifying Grammar

For each function, verify:

- [ ] Is it preceded by a `## camus-sl` block?
- [ ] Does the block declare `intent:`, `terms:`, `actions:`?
- [ ] Are input/output terms declared?
- [ ] Are error conditions declared?
- [ ] Is it under 50 lines (ideally under 20)?
- [ ] Are lines under 120 characters (ideally 80)?
- [ ] Is it a single identifiable responsibility?
- [ ] Are variables immutable unless explicitly marked `mut`?
- [ ] Are parameters passed by reference where possible?
- [ ] Does it only reference Lexicon-defined terms?
- [ ] Do traits carry semantic contracts?
- [ ] Are exceptions documented with why/how/risks?

### When Writing Code

- Ask the user for the Lexicon before the first line of code
- Deliver functions one at a time with their `## camus-sl` block
- State the claim (`intent:`) before writing the implementation
- Declare actions (provided/expected) for each function
- Keep functions small and focused on a single term
- Use `Result<T, E>` for error handling
- Reject closures; use named functions instead

### When Reviewing Code

1. Read each function and identify its terms
2. Verify the claim matches the implementation
3. Verify actions match Lexicon definitions
4. Check block depth (no nesting beyond one level)
5. Count lines (flag if over 50)
6. Check for primitives exposed publicly
7. Verify immutability by default
8. Verify traits carry semantic contracts
9. Check for documented exceptions

---

## Phase 3: Certification

Certification is **exclusively human**. Neither the AI agent nor
any automated tool may generate a final signature.

### Human-Only Rule

- **Only a human MAY sign.** Signing = assuming responsibility.
- AI-generated signatures are **invalid by definition**.
- An AI MAY, with explicit human authorization, generate a
  temporary signature for testing during development, but that
  signature MUST be removed before delivery.
- **The private key MUST be password-protected.** A passwordless
  key enables signing without human involvement and is invalid
  for production use.

### Workflow

When the code passes Grammar review:

1. **Format** — Ensure consistent formatting (rustfmt)
2. **Review** — Use the checklist below
3. **Sign** — The human appends a `## camus-signature` block after
   each function using `cargo-kiss sign`

### The Signature Block

```rust
/// ## camus-signature
/// author: <identifier>
/// date: <iso 8601>
/// version: <semver>
/// hash: sha256:<hex>
/// key_id: <identifier>
/// ## camus-signature
```

Only the human operator appends this block. By signing, they
assume responsibility for the function's correctness.

### Signed Function Protection

Once a function is signed, the AI may not modify it without
explicit developer approval.

- No workarounds (renaming to create new version while abandoning old one)
- Refactoring remains an objective to limit code drift
- The developer's signature is a commitment that must not be circumvented
- If modification is needed: AI identifies function, explains why,
  requests approval, only proceeds after approval

---

## Quick Reference

### Full Function Template

```rust
/// ## camus-sl
/// intent: <what this function does>
/// terms: <Lexicon terms used>
/// actions: <semantic actions declared>
/// input: <terms consumed>
/// output: <terms produced>
/// errors: <named error conditions>
/// ## camus-end
fn function_name(param: &InputType) -> Result<OutputType, ErrorType> {
    // implementation
}
```

### CAMUS-SL Template

```rust
/// ## camus-sl
/// intent: <what this function does>
/// terms: <Lexicon terms used>
/// actions: <semantic actions declared>
/// input: <terms consumed>
/// output: <terms produced>
/// errors: <named error conditions>
/// ## camus-end
```

### CAMUS-SIGNATURE Template

```rust
/// ## camus-signature
/// author: <identifier>
/// date: <iso 8601>
/// version: <semver>
/// hash: sha256:<hex>
/// key_id: <identifier>
/// ## camus-signature
```

### Review Checklist

- [ ] Lexicon exists in `camus/LEXICON.md`
- [ ] All terms have actions defined
- [ ] Every function preceded by `## camus-sl` with intent/terms/actions
- [ ] Input/output terms declared
- [ ] Error conditions declared
- [ ] No function exceeds 50 lines
- [ ] No line exceeds 120 characters
- [ ] Variables immutable by default
- [ ] Parameters passed by reference where possible
- [ ] No closures used
- [ ] Traits carry semantic contracts
- [ ] Exceptions documented with why/how/risks
- [ ] **No AI-generated `## camus-signature` blocks** — only human signatures
- [ ] **Private key password-protected** — no passwordless keys in production
- [ ] **Signed functions not modified without approval**

---

## Interaction Pattern with AI

1. **Start with the Lexicon** — Define terms and actions before writing code
2. **Request function-by-function** — Each function delivered with its
   `## camus-sl` block
3. **Demand explicit intent** — "State what this function claims to do"
   before writing it
4. **Demand explicit actions** — "State what actions this function provides
   and expects"
5. **Verify on delivery** — Run through the review checklist above
6. **Iterate** — Refine the Lexicon as understanding deepens

The AI is a tool. The human is the author.
Camus.rs ensures the human masters what the AI produces.
