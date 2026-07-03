---
layout: page
title: Camus.sh — Shell Scripting Adaptation
---

Camus.sh applies the Camus Method to shell scripting. It defines a restricted profile for writing, auditing, and certifying Bash scripts produced by AI and reviewed by humans.

## What Camus.sh Provides

- **Function-level documentation** via `## CAMUS-SL` blocks
- **Function-level attestation** via cryptographic signatures
- **Automated compliance checking** via [kiss.sh](https://github.com/Camus-Project/kiss.sh)
- **Human responsibility** — only a human may sign

## Key Concepts

Every Camus.sh script follows a strict structure:

1. **Lexicon** — project terms declared in `## CAMUS-LEXICON`
2. **Grammar** — functions with `## CAMUS-SL` blocks declaring intent, inputs, outputs
3. **Certification** — human review and cryptographic signing via `## CAMUS-SIGNATURE`

## Documents

- [Specification](specification.md) — the normative rules for Camus.sh scripts
- [SKILL.md](SKILL.md) — agent skill for applying the Camus.sh workflow

## Tooling

The companion tool **[kiss.sh](https://github.com/Camus-Project/kiss.sh)** automates:

- Compliance checking against the Camus.sh specification
- Cryptographic signing of reviewed functions
- Signature verification and key management

## Status

Camus.sh is a **draft specification**. The profile and tooling are evolving.
Discussion and feedback are welcome via the [issue tracker](https://github.com/Camus-Project/kiss.sh/issues).
