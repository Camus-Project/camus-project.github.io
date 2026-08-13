---
id: TASK-4
title: Define Camus SL concrete syntax
status: Done
assignee: []
created_date: '2026-08-13 14:38'
updated_date: '2026-08-13 14:45'
labels: []
dependencies: []
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Fossil: d3d3ec292c

Design and document a concrete, embeddable syntax for Camus SL, building on the conceptual overview in Camus-SL/index.md.

Scope:
- @camus comment-tag convention (host-language-agnostic, works across //, #, /* */, -- comment styles)
- LEXICON file format: term { definition, realizedBy, subterm }
- Function claim directives: claim, requires, returns
- Constraint expression grammar (minimal: comparators, field access, and/or/not)
- Structured exceptions: exception, exception.reason, exception.scope
- Certification tags: certified-by, certified-at (linking to kiss.sh Ed25519 signatures)
- EBNF grammar sketch

Out of scope (for this task):
- Multi-module/namespace import mechanism across LEXICON files
- Internal binary/JSON intermediate format for kiss tooling
- Automatic documentation generation from annotations
- Closed vs. open list of exception-id values
<!-- SECTION:DESCRIPTION:END -->

## Implementation Plan

<!-- SECTION:PLAN:BEGIN -->
1. Write syntax.md under Camus-SL/ with the full syntax specification (comment-tag convention, LEXICON format, function claim directives, constraint grammar, structured exceptions, certification tags, EBNF sketch).
2. Use 'realizedBy' (not 'realizes') for the term-to-component mapping keyword, per human decision.
3. Cross-link from Camus-SL/index.md to the new syntax page.
4. Keep examples multi-language (Rust + Bash) to demonstrate host-language independence.
5. Mark the page as tentative/early-stage, consistent with the Status section of index.md.
<!-- SECTION:PLAN:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Created Camus-SL/syntax.md: a generic, host-language-agnostic syntax specification for Camus SL.

Key decisions:
- Generalized the block-based syntax already validated in production by Camus.sh (## CAMUS-LEXICON / CAMUS-SL / CAMUS-SIGNATURE ... CAMUS-END) instead of introducing a competing @camus line-tag convention, to avoid two contradictory canonical syntaxes for Camus SL.
- Introduced 'realizedBy' (per explicit human decision, replacing the initial 'realizes' proposal and Camus.rs's prose '- **Realized by**:' label) as the term-to-component mapping key.
- Introduced an explicit, minimal constraint expression grammar (comparators, field access, and/or/not) — not previously formalized in either existing adaptation.
- Documented known divergences from Camus.sh and Camus.rs (signature field names, lexicon location, constraint handling) in a 'Relation to Existing Adaptations' section, explicitly marking reconciliation with Camus.rs as tentative and out of scope for this task.
- Cross-linked from Camus-SL/index.md (Lexicon, Constraints, Structured Exceptions sections) to the new syntax page.

Verification:
- diagnostics: no errors/warnings on the new file.
- Local Jekyll build could not be run to completion (pre-existing Ruby gem environment issue, unrelated to this change: missing compiled native extensions for commonmarker/racc/etc.). Front matter and internal structure were manually checked for consistency with existing page_with_toc pages (Camus.sh/specification.md, Camus.rs/specification.md).

Out of scope (unchanged from task description): multi-module LEXICON import mechanism, kiss internal intermediate format, doc generation, closed exception-id list. Also out of scope: retrofitting Camus.sh/Camus.rs specifications to adopt 'realizedBy' and the new constraint grammar — flagged as a follow-up.
<!-- SECTION:NOTES:END -->
