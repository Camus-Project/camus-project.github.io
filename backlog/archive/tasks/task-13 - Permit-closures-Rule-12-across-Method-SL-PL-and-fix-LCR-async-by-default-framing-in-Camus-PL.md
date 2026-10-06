---
id: task-13
title: >-
  Permit closures (Rule 12) across Method/SL/PL, and fix LCR async-by-default
  framing in Camus PL
status: Done
assignee:
  - '@ai-agent'
created_date: '2026-09-10 21:31'
updated_date: '2026-09-10 21:31'
labels: []
dependencies: []
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Fossil: 5608db129a

Human decision: closures/anonymous functions are no longer prohibited by the Camus Method's Rule 12 (existing complexity limits -- block depth, line length, function length -- are what keep a closure auditable, not a per-unit naming requirement). Applies uniformly across the Method, Camus SL, and Camus PL. Also fixes Camus-PL/ARCHITECTURE.md's LCR section, which incorrectly framed asynchronous `later` invocation as the default for LCR calls; corrected to synchronous-by-default with `later` as the explicit opt-in, mirroring `mut`.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [x] #1 Site/Camus-SL/index.md Grammar Rules #12 reflects closures permitted
- [x] #2 Site/adaptations/Camus.rs/specification.md's Closures section, Relation-to-Method table, and Exceptions example are updated
- [x] #3 Site/adaptations/Camus.rs/SKILL.md and the Camus Method SKILL.md (.agents/.opencode) no longer instruct rejecting closures
- [x] #4 cargo-kiss/camus/EXCEPTIONS.md's Rule 12 exception candidate is marked Resolved, and check.rs no longer reports closures as findings (cargo build/test still pass)
- [x] #5 Camus-PL/ARCHITECTURE.md section on LCR states synchronous-by-default with later as explicit opt-in, not the reverse
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
All 5 criteria verified directly: Site/Camus-SL/index.md rule 12 now reads 'Anonymous Functions Permitted'; Camus.rs/specification.md's Closures section says 'permitted' with updated rationale, its Relation table row 12 updated, and its Exceptions worked example now illustrates rule 11 instead of the no-longer-exceptional rule 12; Camus.rs/SKILL.md's Prohibition Rules, When Writing Code bullet, and Review Checklist updated; .agents/skills/camus/SKILL.md updated (confirmed .opencode/skills/camus/SKILL.md is a hardlink to the same inode, so one edit covers both); cargo-kiss/camus/EXCEPTIONS.md's Rule 12 entry rewritten as 'Resolved -- Rule 12, 2026-09-10' (kept original rationale for context), Rule 11 entry untouched (still proposed); check.rs's closures category/detection loop removed, category_label entry removed, flags_closures test replaced with closures_are_allowed (asserts zero findings); cargo build and cargo test pass (22 lib + 9 integration tests). Camus-PL/ARCHITECTURE.md section 3 (LCR) rewritten: synchronous is now stated as the default for LCR calls too, later is the explicit opt-in mirroring mut, with a note explaining the earlier 'default mechanism' framing was an accidental leftover from an idea already abandoned in the original ChatGPT conversation. No .cam sample code needed to change since later was already written explicitly at every call site.
<!-- SECTION:NOTES:END -->
