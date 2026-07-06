---
layout: page_with_toc
title: Camus.sh Specification
---

# Camus.sh Specification v1 (Draft)

## Purpose

Camus.sh defines a restricted shell scripting profile intended for human-reviewed and human-assumed software artifacts
— typically produced by AI and, eventually, by humans. The profile applies the Camus Method to shell scripts.

A SKILL.md is provided for AI agents that wish to follow the Camus Method and the Camus.sh profile in the shell context. When loaded, the skill guides the agent through the Lexicon, Grammar, and Certification phases defined by the Method.

The profile provides a uniform structure that enables:

- human review;
- function-level documentation;
- function-level attestation;
- automated processing by tools;
- cryptographic signing of reviewed artifacts.

Camus.sh is intentionally more restrictive than the underlying shell languages it supports.

**kiss.sh** is the shell adaptation of the **kiss** companion tool and its first prototype.
kiss.sh will automate compliance checking, block validation, and cryptographic
signing of Camus.sh artifacts.

---

## Shell Compatibility

Camus.sh is developed and tested with **Bash**.

Scripts written under this profile SHOULD also be compatible with:

- Zsh
- Ksh
- Dash
- BusyBox ash

Compatibility with these shells is expected but not continuously tested.
Reports and feedback can be filed at the [issue tracker](https://github.com/Camus-Project/Camus.sh/issues).

The following shells are NOT compatible:

- Fish
- Nushell

---

## Source Encoding

A script MUST:

- use UTF-8 encoding;
- use LF line endings.

---

## Function Declaration

Functions MUST be declared using the following syntax:

```sh
function_name() {
    ...
}
```

The following syntax is prohibited:

```sh
function function_name {
    ...
}
```

This restriction ensures uniform parsing across supported shells.

---

## Executable Code

Executable statements MUST NOT appear at top level.

The following are allowed as exceptions:

- invocation of the entry point (`main "$@"`);
- `readonly` variable declarations (considered declarations, not
  executable code).

Valid example:

```sh
#!/usr/bin/env bash

readonly MY_CONFIG_PATH="$HOME/.config/myapp"
readonly MY_VERSION=42

main() {
    echo "Hello"
}

main "$@"
```

Invalid example:

```sh
#!/usr/bin/env bash

echo "Hello"
```

---

## Entry Point

Every Camus.sh script MUST define a function named:

```sh
main()
```

The script MUST terminate with:

```sh
main "$@"
```

No executable statements may appear after this invocation.

---

## Camus Blocks

Camus.sh uses delimiter-marked blocks for structured metadata.

A Camus block opens with a line starting with `## CAMUS-` and
closes with `## CAMUS-END`. These lines are reserved markers —
they are not ordinary comments and MUST NOT be used outside a
Camus block definition.

Content inside a block uses shell comments (`# key: value`).

| Block | Placement | Purpose |
|---|---|---|
| `## CAMUS-LEXICON` | File header (after shebang) | Declares project terms |
| `## CAMUS-SL` | Before a function definition | Declares intent, inputs, outputs |
| `## CAMUS-SIGNATURE` | After a function body | Cryptographic attestation |

### CAMUS-LEXICON

Placed at the top of a script (after shebang) to declare project terms.
If the script belongs to a larger project with a LEXICON.md, this block
MAY be omitted.

```sh
## CAMUS-LEXICON
# <term>: <definition>
## CAMUS-END
```

### CAMUS-SL

Placed immediately before a function definition.
Keys MUST be in lowercase. Sub-entries are indented (tooling MUST preserve
indentation).

The following keys are defined:

- `intent:` — REQUIRED. Describes what the function does.
- `input[N]{param,desc[,default]}:` — OPTIONAL. Present only if the function takes inputs. Uses TOON table format.
- `output:` — OPTIONAL. Present only if the function produces output.
- `return[N]{code,desc}:` — OPTIONAL. Present only if the function returns non-zero
  exit codes. Uses TOON table format. The `code` field MUST be the name of a
  named constant (see [Error Code Constants](#error-code-constants)). The `code` field MAY be `*` to indicate that
  the return code originates from a called function and is propagated forward.

Example with inputs and outputs:

```sh
## CAMUS-SL
# intent: compute the sum of two numbers
# input[2]{param,desc}:
#   $1,augend (integer)
#   $2,addend (integer)
# output:
#   stdout: sum of augend and addend
## CAMUS-END
add() {
    echo $(($1 + $2))
}
```

Example with no inputs or outputs:

```sh
## CAMUS-SL
# intent: print the application banner to stdout
## CAMUS-END
print_banner() {
    echo "=== My App v1.0 ==="
}
```

Example with return codes (wildcard for propagated errors):

```sh
## CAMUS-SL
# intent: prompt the user for a password twice to confirm
# output:
#   stdout: the confirmed password
#   return[2]{code,desc}:
#     *,propagated from prompt_password
#     E_PASSWORDS_MISMATCH,passwords do not match
## CAMUS-END
prompt_password_twice() {
    local p1 p2
    p1=$(prompt_password "Enter password: ") || return $?
    ...
}
```

### CAMUS-SIGNATURE

Placed immediately after the closing `}` of a function.

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

```sh
## CAMUS-SIGNATURE
# signatory: <identifier>
# date: <iso 8601>
# fingerprint: sha256:<hex>
# signature: <base64>
## CAMUS-END
```

---

## Size Limits

### Functions

Functions MUST NOT exceed 50 lines.

Functions SHOULD NOT exceed 20 lines.

### Line Length

Lines MUST NOT exceed 120 characters.

Lines SHOULD NOT exceed 80 characters.

---

## Signature Scope

Camus.sh v1 defines functions as the primary review and attestation unit.

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
delegated to kiss.sh (see [Relation to the Camus Method](#relation-to-the-camus-method)).

---

## Relation to the Camus Method

Camus.sh applies the Camus Method to shell scripting. The table below
maps the 15 Grammar rules defined by the Method to this specification.

| # | Rule | Status | Notes |
|---|---|---|---|
| 1 | Term Definition | Applicable | Via `## CAMUS-LEXICON` block ([CAMUS-LEXICON](#camus-lexicon)) |
| 2 | Component Realization | Applicable | Each function realizes a term, declared via `## CAMUS-SL` ([CAMUS-SL](#camus-sl)) |
| 3 | Function Terms | Applicable | Input and output terms declared in `## CAMUS-SL` ([CAMUS-SL](#camus-sl)) |
| 4 | Function Claim | Applicable | Declared via `intent:` in `## CAMUS-SL` ([CAMUS-SL](#camus-sl)) |
| 5 | Function Constraints | Applicable | Declared via `input:` in `## CAMUS-SL` ([CAMUS-SL](#camus-sl)) |
| 6 | Block Depth | Applicable | Shell structure prevents nested function definitions |
| 7 | Line Length | Applicable | See [Size Limits](#size-limits) |
| 8 | Function Length | Applicable | See [Size Limits](#size-limits) |
| 9 | Parameter Passing | Not applicable | Shell uses positional parameters; pass-by-reference is a language-level concept |
| 10 | Variable Mutability | Partial | `local` and `readonly` recommended but not enforced by spec v1 |
| 11 | No Public Primitives | Applicable | [Executable Code](#executable-code) prohibits top-level executable statements |
| 12 | No Anonymous Functions | Not applicable | Shell has no anonymous function syntax |
| 13 | No Inheritance | Not applicable | Shell has no inheritance mechanism |
| 14 | No Interfaces | Not applicable | Shell has no interface mechanism |
| 15 | Explicit Exceptions | Applicable | Deviations from this spec must be documented |

The companion tool **kiss.sh** — the shell adaptation of the kiss tool and
its first prototype — will automate compliance checking against these rules.

---

## Error Code Constants

All return and exit codes in Camus.sh scripts MUST use named `readonly`
constants instead of bare numeric values. This convention makes error
semantics self-documenting and enables systematic auditing.

### Range Convention

| Range | Prefix | Category | Description |
|---|---|---|---|
| 0–49 | `I_` | Information / boolean | Success indicators, boolean results, UI state codes |
| 50–99 | `W_` | Warning (SHOULD) | Non-fatal warnings, compliance hints |
| 100–255 | `E_` | Error (MUST) | Fatal errors, constraint violations |

Code 0 is reserved for success and is bound to the constant `I_OK`.

### Rules

1. Every `return N` or `exit N` with `N > 0` MUST reference a named constant.
2. Constants MUST be declared at the top of the script, after the
   `## CAMUS-LEXICON` block, using `readonly`.
3. Constants within a range MUST be assigned sequentially without gaps.
4. The `readonly` declaration is considered a declaration (not executable
   code) and is therefore exempt from the top-level code prohibition ([Executable Code](#executable-code)).
5. In CAMUS-SL `return[N]{code,desc}:` blocks, the `code` field MUST use
   the constant name (without `$` prefix). Example:
   `E_FILE_NOT_FOUND,file not found`.
6. Propagated return codes (those returned by calling `|| return $?`) use
   the wildcard code `*` in the CAMUS-SL block:
   `*,propagated from <function>`.

### Implementation Pattern

```sh
# --- Error code constants ---
# I_ (0-49): boolean / information
readonly I_OK=0
readonly I_FALSE=1
readonly I_USER_REFUSED=2
readonly I_USER_INTERRUPTED=3
# W_ (50-99): SHOULD warnings
readonly W_LEXICON_MISSING=50
readonly W_FUNC_SHOULD=51
readonly W_LINE_SHOULD=52
readonly W_COMPLIANCE_ONLY=53
# E_ (100-255): MUST errors
readonly E_EMPTY_PASSWORD=100
readonly E_FILE_NOT_FOUND=105
readonly E_UNREACHABLE=127
```

---

## Design Rationale

Camus.sh treats functions as units of human responsibility.

Each function SHOULD implement a single identifiable responsibility.

A reviewer is expected to understand, verify, and assume individual functions rather than arbitrary fragments of source code.

The mandatory `main()` entry point ensures a predictable program structure and simplifies both human review and automated tooling.
