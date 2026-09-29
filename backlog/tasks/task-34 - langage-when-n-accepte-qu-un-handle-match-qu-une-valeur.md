---
id: task-34
title: 'langage: when n accepte qu un handle, match qu une valeur'
status: Done
assignee: []
created_date: '2026-09-29 21:12'
updated_date: '2026-09-29 21:44'
labels:
  - camus-pl
dependencies:
  - task-33
  - task-25
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Application de la decision humaine Fossil 771d3bdc78, qui complete bdd9d01747.

Ce ticket fige une decision, il ne la invente pas. bdd9d01747 (livre par TASK-33) retenait:

  match(valeur) = matcher la valeur MAINTENANT (deja disponible)
  when(valeur)  = matcher la valeur QUAND ELLE DEVIENT DISPONIBLE (issue de later)

Une valeur qui devient disponible est une valeur issue de later, donc un handle. bdd9d01747 annoncait en outre que la regle du checker sur when serait reprise. C est cette reprise.

La grammaire v0.8 est a l origine du desaccord: sa production when acceptait aussi une Result VALUE, faisant de when un sur ensemble de match.

PREUVE (Rust emis par codegen, TASK-25)

  source Camus              sujet      Rust emis
  match found               valeur     match &found
  when created              valeur     match &created
  when found                handle     match camus_runtime::wait(&found)
  when saved                handle     match camus_runtime::wait(&saved)

Les deux premieres lignes sont de meme forme: deux mots differents, un resultat identique, aucune information transmise. L incoherence etait deja visible dans le depot, matchwhen ecrivant match found sur une valeur et resultpayload ecrivant when created sur une valeur.

OBJECTIF

  match n accepte qu une valeur, when n accepte qu un handle. Les deux formes sont complementaires, et la forme EST le timing du sujet.

La distinction handle/valeur cesse d etre une inference du compilateur (table d environnement de noms venus de later) et devient une regle du langage. Le generateur ne peut plus desynchroniser cette table et produire silencieusement du mauvais code.

HORS PERIMETRE

  - le runtime TASK-25, non modifie
  - le bug is_result_value (gen.rs), compare "Result" a un prefixe "CamusResult" et donc jamais vrai: task distincte
  - Concrete::Handle, qui ferait passer la distinction handle/valeur de la table d environnement vers le type: aucun effet observable, traite apres coup pour ne pas toucher deux fois le meme code

Fossil: 771d3bdc783424026f97830f7bf6585ecbbbbaf3
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [x] #1 checker: when sur une valeur Result est refuse, avec un message nommant match comme alternative
- [x] #2 checker: match sur un handle later est refuse (comportement existant, message a confirmer)
- [x] #3 codegen: match_lowering se base sur temporality et non plus sur une table d environnement; le marqueur de handle ne subsiste que pour return h
- [x] #4 fixtures: result_payload (checker) et resultpayload (codegen) passent de when created a match created
- [x] #5 tests: une fixture negative couvre when sur une valeur, et une fixture positive couvre les deux formes sur leurs sujets respectifs
- [x] #6 grammar.ebnf: bump de version, la production when n accepte plus de Result VALUE, avec renvoi au ticket 771d3bdc78
- [x] #7 ARCHITECTURE.md: la section concurrente distingue explicitement match valeur et when handle
- [x] #8 regression: les 33 tests checker, 19 tests codegen et 9 tests runtime passent, clippy sans nouvelle erreur
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
IMPLEMENTED (Fossil 8cd9762f8d, 2026-09-29).

The two matching forms are now complementary, and the form states the timing of
the subject. On a value the two words lowered to the same Rust (match &x), so
neither carried any information; only on a handle does when change the emitted
code.

checker: `when` over a Result value is rejected, pointing at `match`. `match`
over a later handle keeps its message, pointing at `when`. The handle test runs
before the Result test, since a handle is also typed Result. The old "expected a
later handle or a Result value" message named no alternative; both new messages
do.

codegen: match_lowering branches on temporality instead of consulting the
environment for a handle marker. The environment is still read, but only to
reject a `when` whose subject is not a handle: emitting
camus_runtime::wait(&v) would not compile, since wait takes a Task<T> and a
resolved value is a CamusResult<T>. The handle marker now survives only for
`return h`, which must await before propagating.

A body that fails to lower no longer collapses into an unimplemented! placeholder.
Nothing depended on that branch, and a placeholder would have hidden the real
cause while still producing a file that compiles. The error now propagates.

Fixtures: result_payload (checker) and resultpayload (codegen) move from
"when created" to "match created". bad/match_missing_arm.cam binds its subject
with `later`, otherwise the new rule would have masked the exhaustiveness error
it exists to test. bad/when_on_non_result.cam keeps its fixture, with the
assertion updated to the narrowed message. New bad/when_on_value.cam covers the
new error; codegen/tests/fixtures/whenonvalue covers the generator refusing.

grammar.ebnf v0.9 -> v0.10 records the rule and withdraws the v0.6 reading.
ARCHITECTURE.md states the complementarity, keeps the four-row table of emitted
Rust as the evidence, and drops the "two targets" framing.

Tests: checker 34, codegen 20, runtime 9, verus-gen 3. Clippy: no new warning;
the 8 in checker/src/parse.rs are pre-existing. E2E unchanged: camus build
sample-project produces a CLI that persists across invocations, and camus-check
still reports only the 3 pre-existing TCC/LCR boundary errors.

Out of scope, unchanged: the TASK-25 runtime, the is_result_value bug, and
Concrete::Handle.
<!-- SECTION:NOTES:END -->
