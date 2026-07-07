---
layout: page
title: Camus.sh — Shell Scripting Adaptation
---

> **TL;DR**
>
> Camus.sh adapts the Camus Method to shell scripting. It provides a disciplined workflow for producing Bash scripts that can be specified, reviewed, audited, and ultimately certified by a human.
>
> **AI may write. Only a human may certify.**

[The Camus Method](/The-Camus-Method) was born from a simple observation: AI has dramatically increased our ability to produce software, but not our ability to acquire assumable confidence in its behavior.

Camus.sh is the first experimental realization of these ideas. It combines behavioral specifications, human review, and cryptographic attestation into a workflow that makes AI-assisted Bash development reviewable, auditable, and certifiable.

## How Camus.sh Emerged

The first implementation envisioned for the Camus Method was **Camus.rs**.

Its development began with the writing of documentation, specifications, and AI skills describing both the methodology and its future implementation. Since many of these artifacts were themselves produced with AI assistance, a natural question emerged:

> If AI-generated software should not be trusted without review and certification, why should AI-generated specifications?

To answer that question, a small Bash script was written to cryptographically sign those documents while waiting for Camus.rs.

Reviewing that script led to an unexpected realization: it had itself become AI-assisted software and therefore ought to satisfy the Camus Method before it could be trusted.

The roadmap changed.

The signing script gradually evolved into **[kiss.sh](https://github.com/Camus-Project/kiss.sh)**. Developing kiss.sh according to the Camus Method required adapting the methodology to shell scripting. Camus.sh emerged from that necessity.

In this sense, Camus.sh is more than an adaptation of the Camus Method to Bash. It is the result of the Camus Method being applied to its own artifacts.

This was the first demonstration that the Camus Method could be applied to its own development process.

Along the way, Camus.sh introduced the first experimental syntax for **[Camus SL (Specification Language)](/Camus-SL)** and provided the experience needed for Camus SL to evolve into an independent specification language.

As of today, the [Camus.sh specification](/adaptations/Camus.sh/specification.html), [its AI skill](/adaptations/Camus.sh/SKILL.html), and kiss.sh together provide the first complete implementation of the Camus.sh workflow. They also provide the tooling needed to develop Camus.rs according to the very methodology it is intended to embody.

## Core Principles

- **Behavioral specifications** expressed in `## CAMUS-SL` blocks
- **Function-level attestation** through cryptographic signatures
- **Automated compliance checking** with **kiss.sh**
- **Human accountability. AI may write. Only a human may certify.**

## Key Concepts

Every Camus.sh script is organized around three complementary concepts.

### Lexicon

A shared project vocabulary declared in `## CAMUS-LEXICON` blocks (optional). A project may also provide a standalone lexicon shared across the entire codebase.

### Grammar

Behavioral specifications written in `## CAMUS-SL`, describing a function's intent, inputs, outputs, side effects, assumptions, and guarantees.

### Certification

Human review followed by cryptographic attestation using `## CAMUS-SIGNATURE`.

## Deliverables

Camus.sh consists of three complementary deliverables.

### Specification

The normative specification of the Camus.sh methodology.

It also defines the first experimental syntax of **Camus SL (Specification Language)**, embedded directly in Bash comments. The experience gained through Camus.sh will inform the future standalone specification of Camus SL.

### SKILL.md

Instructions enabling AI agents to understand and apply the Camus.sh workflow.

### kiss.sh

The reference implementation of the Camus.sh workflow.

It automates:

- compliance checking against the Camus.sh specification;
- cryptographic signing of reviewed functions;
- signature verification;
- key management.

Certification itself, using kiss.sh, always remains a human decision.

Developed using Camus.sh itself, kiss.sh serves as both the reference implementation and the reference codebase of the methodology.

## Examples

### Excerpt from kiss.sh

The following function illustrates the three building blocks of Camus.sh: a **Camus SL specification**, the **implementation**, and the **cryptographic certification** performed after human review.

```bash
## CAMUS-SL
# intent: check that a file exists and exit with E_FILE_NOT_FOUND if not
# input[2]{param,desc}:
#   $1,file path
#   $2,label for error message (defaults to file path)
# output:
#   return[1]{code,desc}:
#     I_OK,file exists
#     E_FILE_NOT_FOUND,file not found
## CAMUS-END
assert_file_exists() {
    local file="$1" label="${2:-$1}"
    [ -f "$file" ] && return $I_OK
    echo "Error: file not found: ${label}" >&2
    return $E_FILE_NOT_FOUND
}
## CAMUS-SIGNATURE
# signatory: Lan Jing
# date: 2026-06-28T16:52:02Z
# fingerprint: sha256:52:69:05:07:66:BD:DE:55:C7:D2:B1:52:9C:8F:ED:7B:05:E3:8E:57:7E:10:98:1E:7C:BD:13:96:85:4E:83:89
# signature: 1WhT1xPw/a3+nRx80ed6dVxXVcmUXHYRwNIeW7pIXjBVf0Qbjg6UuDN7uOFEpXFRq3Ub7BT2JAoEID3gORtSAw==
## CAMUS-END
```

### Two `kiss sign` sessions


#### Copied/pasted from terminal
```bash
$ ./kiss.sh sign kiss.sh
Signatory name: Lan Jing
--- Running check on kiss.sh ---
Checking: kiss.sh

  [OK] Shebang present
  [OK] CAMUS-LEXICON block present
  [OK] No 'function' keyword
  [OK] main() defined
  [OK] Script ends with main "$@"
  [OK] No top-level executable code
  [OK] All functions preceded by CAMUS-SL blocks
  [OK] All CAMUS-SL blocks declare intent:
  [OK] All Camus blocks properly closed
  [WARN] Function 'usage' is 22 lines (SHOULD <= 20)
  [WARN] Function 'check_top_level_code' is 29 lines (SHOULD <= 20)

...

Result: 34 warning(s)
Skipping (already signed by Lan Jing): usage in kiss.sh
Skipping (already signed by Lan Jing): assert_file_exists in kiss.sh
...
Signed function(s) in kiss.sh
No signable files found.
```

#### A animated Gif

![Camus sign session](/img/term1.gif)

## Status

Camus.sh is an experimental implementation of the Camus Method.

Its purpose is to validate the methodology in practice, refine the first experimental form of Camus SL, and establish the foundations for Camus.rs and future adaptations of the Camus Method.

Discussion and feedback are welcome via the project's issue tracker.
