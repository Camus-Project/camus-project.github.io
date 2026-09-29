---
id: task-34
title: 'langage: when n accepte qu un handle, match qu une valeur'
status: To Do
assignee: []
created_date: '2026-09-29 21:12'
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
- [ ] #1 checker: when sur une valeur Result est refuse, avec un message nommant match comme alternative
- [ ] #2 checker: match sur un handle later est refuse (comportement existant, message a confirmer)
- [ ] #3 codegen: match_lowering se base sur temporality et non plus sur une table d environnement; le marqueur de handle ne subsiste que pour return h
- [ ] #4 fixtures: result_payload (checker) et resultpayload (codegen) passent de when created a match created
- [ ] #5 tests: une fixture negative couvre when sur une valeur, et une fixture positive couvre les deux formes sur leurs sujets respectifs
- [ ] #6 grammar.ebnf: bump de version, la production when n accepte plus de Result VALUE, avec renvoi au ticket 771d3bdc78
- [ ] #7 ARCHITECTURE.md: la section concurrente distingue explicitement match valeur et when handle
- [ ] #8 regression: les 33 tests checker, 19 tests codegen et 9 tests runtime passent, clippy sans nouvelle erreur
<!-- AC:END -->
