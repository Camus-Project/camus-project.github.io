---
layout: page_with_toc
title: Camus SL Syntax Specification
---

# Camus SL Syntax Specification v0.1 (Draft)

## Purpose

This document defines the **generic, host-language-agnostic syntax** of Camus SL.

Camus SL is embedded inside existing programming languages through structured
comments. It does not execute and does not compile. It constrains meaning, not
runtime behavior.

This specification generalizes the conventions first validated by
[Camus.sh](/adaptations/Camus.sh/specification) and
[Camus.rs](/adaptations/Camus.rs/specification), the two existing host-language
adaptations of Camus SL. Host adaptations MAY restrict, extend, or specialize
this syntax to fit the idioms of their target language, but MUST preserve the
semantics defined here. Divergences MUST be documented (see
[Relation to Existing Adaptations](#relation-to-existing-adaptations)).

This document complements the conceptual overview in
[Camus SL](/Camus-SL/). Where the overview describes *why* Camus SL exists and
*what* it expresses, this document describes *how* it is written.

---

## Design Principles

1. **Comment-embedded** — Camus SL lives entirely inside the host language's
   comment syntax. A file with all Camus SL content stripped MUST remain valid
   source code in the host language.
2. **Marker-based, not parser-based** — tooling locates Camus SL content by
   recognizing reserved markers, not by parsing the host language's grammar.
   This allows a single tool (`kiss`) to support many host languages through a
   small per-language adapter (comment leader + block markers).
3. **Blocks, not single-line tags** — structured metadata is enclosed between
   an opening marker and a matching `CAMUS-END` marker, following the
   convention already validated by Camus.sh and Camus.rs. This keeps
   multi-line content (descriptions, tables, nested keys) unambiguous.
4. **Lowercase keys** — all keys inside a Camus block MUST be lowercase.
5. **Graceful degradation** — a reader unfamiliar with Camus SL, or without
   tooling, MUST still be able to read the block as an ordinary comment.
6. **Gradual adoption** — a project MAY apply Camus SL to a subset of files or
   modules. Coverage measurement is out of scope for this document.

---

## Comment Adaptation

Camus SL blocks are written using the host language's own comment syntax.
The **block marker** (`CAMUS-LEXICON`, `CAMUS-SL`, `CAMUS-SIGNATURE`,
`CAMUS-END`) is prefixed by the host's comment leader; content lines inside
the block are prefixed the same way.

| Host language | Comment leader | Example opening |
|---|---|---|
| Bash / shell | `#` | `## CAMUS-SL` |
| Rust (doc comment) | `///` | `/// ## camus-sl` |
| C-family (line) | `//` | `// ## CAMUS-SL` |
| C-family (block) | `/* ... */` | `/* ## CAMUS-SL` |
| SQL / Lua | `--` | `-- ## CAMUS-SL` |
| Python | `#` | `## CAMUS-SL` |

The exact casing and doubling of the comment leader (e.g. `##` vs `#`, or
lowercase `camus-sl` in Rust doc comments) is a per-adaptation convention. A
host adaptation specification MUST fix one convention and apply it
consistently. This document uses the uppercase `## CAMUS-*` form in examples
for readability; this is not normative for every host language.

---

## Camus Blocks

Three block kinds are defined generically. A host adaptation MAY introduce
additional block kinds for host-specific concerns (for example, error-code
tables), but MUST NOT repurpose these three names.

| Block | Placement | Purpose |
|---|---|---|
| `CAMUS-LEXICON` | File or project root | Declares project terms (see [Lexicon Format](#lexicon-format)) |
| `CAMUS-SL` | Immediately before a function, item, or type | Declares intent, terms, inputs, outputs, constraints, exceptions |
| `CAMUS-SIGNATURE` | Immediately after a signed unit | Cryptographic attestation by a human reviewer |

A block opens with a line containing its marker and closes with a line
containing `CAMUS-END`. Markers are reserved: they MUST NOT be used in
ordinary comments outside of a genuine Camus block.

---

## Lexicon Format

The Lexicon is the project's single source of truth for terms. It MAY live in
a dedicated file (recommended: `LEXICON.md`, or `camus/LEXICON.md` for
larger projects) or, for small standalone scripts, inline in a
`CAMUS-LEXICON` block at the top of a file.

### Term Declaration

```
term <Name> {
  definition: "<clear, natural-language description of the concept>"
  realizedBy: <qualified-component-name>
  subterm <name>: <Term>
  action <name>: "<semantic description>"
}
```

- `definition` — REQUIRED. Natural language. Describes *meaning*, not type or
  implementation.
- `realizedBy` — REQUIRED. Points to the host-language component that realizes
  the term, qualified by language when the project is polyglot
  (`<lang>::<ComponentName>`, e.g. `rust::Session`, `sh::session_token`).
- `subterm` — OPTIONAL, repeatable. Declares a named part of a composite term,
  itself typed by another term.
- `action` — OPTIONAL, repeatable. Declares a semantic action the term
  provides or expects (see [Actions](#actions)).

### Example

```
term Session {
  definition: "An authenticated connection between client and server."
  realizedBy: rust::SessionManager
  subterm token: SessionToken
  subterm owner: UserId
  action authenticate: "validate credentials and establish a session"
  action revoke: "terminate a session"
}
```

### Inline Lexicon (small/standalone scripts)

For a standalone script with a small vocabulary, terms MAY be declared inline
instead of in a separate file:

```
## CAMUS-LEXICON
# term SessionToken: "An opaque, time-bound credential identifying a session."
## CAMUS-END
```

The inline form only supports a `term: definition` pair per line. It is a
convenience shorthand and MAY be omitted entirely if a `LEXICON.md` already
exists at the project root.

---

## Function / Item Specification (`CAMUS-SL`)

Placed immediately before the function, item, or type it describes.

### Keys

| Key | Required | Meaning |
|---|---|---|
| `intent` | Yes | Natural-language claim: what this unit asserts it does |
| `terms` | Yes | Lexicon terms this unit manipulates |
| `input` | If applicable | Parameters, each mapped to a term |
| `output` | If applicable | Return value(s), mapped to a term |
| `constraint` | Optional, repeatable | Explicit, checkable condition (see [Constraint Grammar](#constraint-grammar)) |
| `exceptions` | Optional | Documented deviations from Camus Method grammar rules (see [Structured Exceptions](#structured-exceptions)) |

`intent` is the direct realization of the **claim** described in the
[Camus SL overview](/Camus-SL/): the answer to *"what does this function
assert it does?"*.

Host adaptations MAY add further keys for host-specific reporting needs (for
example, named error/return-code tables). Such additions MUST be documented in
the adaptation's own specification and MUST NOT redefine the meaning of the
keys above.

### Example

```
## CAMUS-SL
# intent: transfer funds from one account to another, atomically
# terms: Account, Money, TransferReceipt
# input:
#   from: Account
#   to: Account
#   amount: Money
# output:
#   result: TransferReceipt
# constraint: amount > 0
# constraint: from.balance >= amount
## CAMUS-END
```

---

## Actions

An **action** names a unit of behavior a term provides to callers or requires
from implementors. Actions connect the Lexicon to `CAMUS-SL` blocks:

- A term declares the actions it `provides` or `requires` (in the Lexicon).
- A `CAMUS-SL` block declares which actions the described unit fulfills.

```
## CAMUS-SL
# intent: define authentication behavior
# terms: Session, Credentials
# actions:
#   requires: validate credentials
#   provides: establish session, terminate session
## CAMUS-END
```

The chain of meaning is:

> intent → action → term → implementation

---

## Constraint Grammar

A constraint is an explicit, checkable condition on the terms a unit receives
or produces. Constraints use a deliberately minimal expression grammar — not
a general-purpose expression language.

Rationale: a constraint must be readable by a human in a single glance. A
condition that cannot be expressed in this grammar is a signal that it
belongs in tested code, not in a declared constraint.

### Grammar

```
constraint  = expr ;
expr        = comparison , { ("and" | "or") , comparison } ;
comparison  = ["not"] , operand , [ comp_op , operand ] ;
operand     = identifier , { "." , identifier } | literal ;
comp_op     = ">" | "<" | ">=" | "<=" | "==" | "!=" ;
literal     = number | string | boolean ;
```

- Field access uses `.` (e.g. `from.balance`).
- Logical connectors are `and`, `or`, `not`.
- No function calls, no arithmetic beyond literals, no arbitrary expressions.

### Examples

```
# constraint: amount > 0 and from.balance >= amount
# constraint: not input.is_empty()
```

The second example is illustrative of intent only; method-call-like syntax
(`input.is_empty()`) is host-specific sugar. Adaptations MAY permit it if it
maps directly to a boolean predicate already defined in the host language;
otherwise, constraints are limited to field access and literals as defined
above.

---

## Structured Exceptions

Camus SL is strict by design; real systems are not. Any deviation from a
Camus Method grammar rule MUST be declared, scoped, and justified. Silence is
forbidden.

### Format

```
## CAMUS-SL
# intent: <intent>
# terms: <terms>
# exceptions:
#   rule: <rule-id>
#   why: <reason this exception is necessary>
#   how: <implementation approach taken instead>
#   risks: <what this exception implies>
## CAMUS-END
```

- `rule` — REQUIRED. Identifies the grammar rule being deviated from. Rule
  identifiers correspond to the 15 Grammar rules of the Camus Method (e.g.
  `block-depth`, `function-length`, `no-primitives`, `no-anonymous-functions`).
- `why` — REQUIRED. Natural-language justification.
- `how` — REQUIRED. Describes the approach taken in place of the rule.
- `risks` — REQUIRED. States what the exception implies for reviewers.

### Workflow

1. The AI identifies the need for an exception.
2. The AI requests human approval, presenting the full `exceptions` block.
3. The human approves or rejects.
4. Implementation proceeds only after approval.

Presenting an exception as a fait accompli at review time is forbidden — it
must be proposed and approved before the deviating code is written.

### Example

```
## CAMUS-SL
# intent: validate a raw request and produce a typed command
# terms: RawRequest, Command
# exceptions:
#   rule: block-depth
#   why: early-return guard clauses; flattening would obscure error handling
#   how: sequential guard clauses, each returning immediately on failure
#   risks: readability depends on guard clauses remaining short and independent
## CAMUS-END
```

---

## Certification (`CAMUS-SIGNATURE`)

Placed immediately after the unit it certifies. A signature is the act of a
human reviewer assuming responsibility for a unit's correctness.

**Only a human MAY append or authorize a signature block.** Automated or
AI-generated signatures are invalid. An AI MAY, with explicit human
authorization, generate a temporary signature for testing during development,
but it MUST be removed before the artifact is delivered.

**The signing private key MUST be password-protected.** A passwordless key
enables unsupervised signing and defeats the accountability signatures are
meant to provide. Passwordless keys are valid for local testing only.

### Core Fields

```
## CAMUS-SIGNATURE
# signatory: <identifier>
# date: <ISO 8601>
# fingerprint: sha256:<hex>
# signature: <base64>
## CAMUS-END
```

- `signatory` — REQUIRED. Identifies the human certifier.
- `date` — REQUIRED. ISO 8601 timestamp of certification.
- `fingerprint` — REQUIRED. Fingerprint of the signing key.
- `signature` — REQUIRED. Base64-encoded cryptographic signature.

Host adaptations MAY add further identifying fields (for example, a
`version` or `key_id` field) where useful for their tooling. Such additions
are appended after the core fields and MUST NOT replace them.

Signature scope (which units MUST be signed — all functions, only public
ones, etc.) is defined by each host adaptation.

### Signed Unit Protection

Once a unit is signed, it MUST NOT be modified without explicit human
approval. Renaming a unit to create a new, unsigned version while abandoning
the signed original is prohibited. A signature is a commitment; it must not
be circumvented.

---

## Grammar Summary (EBNF Sketch)

```
camus_block     = open_marker , { block_line } , close_marker ;
open_marker     = comment_leader , "CAMUS-" , block_name ;
close_marker    = comment_leader , "CAMUS-END" ;
block_name      = "LEXICON" | "SL" | "SIGNATURE" ;
block_line      = comment_leader , key , ":" , value ;

key             = "intent" | "terms" | "input" | "output" | "constraint"
                | "exceptions" | "actions"
                | "signatory" | "date" | "fingerprint" | "signature" ;

lexicon_file    = { term_decl } ;
term_decl       = "term" , identifier , "{" ,
                    "definition" , ":" , string ,
                    "realizedBy" , ":" , qualified_name ,
                    { subterm_decl } ,
                    { action_decl } ,
                  "}" ;
subterm_decl    = "subterm" , identifier , ":" , identifier ;
action_decl     = "action" , identifier , ":" , string ;

constraint      = expr ;
expr            = comparison , { ("and" | "or") , comparison } ;
comparison      = ["not"] , operand , [ comp_op , operand ] ;
operand         = identifier , { "." , identifier } | literal ;
comp_op         = ">" | "<" | ">=" | "<=" | "==" | "!=" ;

exception_decl  = "rule" , ":" , identifier ,
                   "why" , ":" , string ,
                   "how" , ":" , string ,
                   "risks" , ":" , string ;
```

---

## Relation to Existing Adaptations

Camus.sh and Camus.rs predate this generic specification and independently
validated the block-based, comment-embedded approach generalized here. Their
existing specifications remain authoritative for their respective host
languages. This section records known points of alignment and divergence.

| Aspect | Camus.sh | Camus.rs | This specification |
|---|---|---|---|
| Block delimiters | `## CAMUS-*` / `## CAMUS-END` | `/// ## camus-*` / `/// ## camus-end` | `CAMUS-*` / `CAMUS-END`, leader adapted per host |
| Lexicon location | Inline `CAMUS-LEXICON` or project `LEXICON.md` | `camus/LEXICON.md` only | Dedicated file recommended; inline shorthand allowed |
| Lexicon term-to-component key | Not formalized (prose) | `- **Realized by**: ComponentName` | `realizedBy: <qualified-component-name>` |
| Claim key | `intent:` | `intent:` | `intent` |
| Constraint declaration | Folded into `input:` descriptions | Not formalized | Explicit `constraint:` key with defined grammar |
| Signature fields | `signatory`, `date`, `fingerprint`, `signature` | `author`, `date`, `version`, `hash`, `key_id` | Core: `signatory`, `date`, `fingerprint`, `signature`; extra fields allowed |

The `realizedBy` key and the explicit `constraint` grammar are new in this
specification. Reconciling `Camus.rs`'s `- **Realized by**:` prose label and
`author`/`hash`/`version` signature field names with the generalized keys
above is **tentative** and out of scope for this document; it requires a
dedicated, separately approved task against the Camus.rs specification.

---

## What This Specification Does Not Cover

- A namespace or import mechanism across multiple `LEXICON.md` files in a
  multi-module project.
- The internal binary or JSON intermediate representation used by `kiss`
  tooling to parse Camus blocks.
- Automatic documentation generation from Camus SL annotations.
- A closed list of `exceptions.rule` identifiers (currently open, informally
  aligned with the 15 Grammar rules of the Camus Method).

---

## Status

This specification is an early-stage draft. It generalizes syntax already
validated in production by Camus.sh and exercised by Camus.rs, and introduces
two new elements — `realizedBy` and the explicit `constraint` grammar — that
have not yet been field-tested.

Discussion and critique are welcome via the project's issue tracker.