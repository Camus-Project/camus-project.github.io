---
id: TASK-31
title: >-
  langage: gestion des erreurs - constructeur Err, propagation, code de sortie
  du TCC
status: To Do
assignee: []
created_date: '2026-09-29 12:34'
updated_date: '2026-09-29 15:37'
labels:
  - camus-pl
dependencies:
  - TASK-30
  - TASK-33
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Tache de suivi de TASK-30 (Result generique). La question des erreurs n est pas videe: TASK-30 a rendu Result<T> utilisable mais n a pas resolu la fabrication ni la propagation d un echec.

Etat de l art:
- Result<T> existe (grammaire, checker, codegen) et porte un payload Ok
- when detruit une valeur Result (success lie le payload, failure lie le message)
- dans une fonction Result, return <expr> construit Ok; une valeur Result deja presente est forwardee, ce qui propage un echec LCR
- if existe des TASK-32 (statement, else obligatoire)
- AUCUN constructeur Err: l application ne peut pas fabriquer un echec

Consequences:
1. une operation App ne peut pas refuser une entree invalide, seulement retourner Ok ou relayer un echec LCR
2. pas de propagateur ergonomique: un enchainement exige un when par etape, avec corps duplique dans chaque branche
3. le TCC ne peut pas distinguer un echec metier d une panne d adaptateur
4. le code de sortie du processus (stdout + exit code), revendique par l architecture, n a aucun mecanisme
5. Err ne porte qu un String: aucun contrat d echec verifie ni documente

DECISION HUMAINE DEJA PRISE (2026-09-29) - la question quand contre match est TRANCHEE
match et when sont des formes d un SEUL mecanisme de pattern matching, qui differe par la temporalite: match immediat, when differe jusqu a disponibilite de la valeur. Ils partagent syntaxe des patterns, typage, exhaustivite, AST, et verification des branches; seule la semantique d execution et le lowering different. Le langage n expose aucune distinction de resolution (pending/succes/echec): ce qui compte est la valeur produite. Une distinction plus fine doit passer par le TYPE de l erreur, pas par un second mecanisme implicite.
Cette decision est portee par TASK-33 (implementation du mecanisme general). Cette tache en depend.

Questions de design ENCORE a trancher avec le humain AVANT implementation (ne pas les decider dans le code):
A. constructeur d erreur explicite (faute / raise / reject) ou valeur d erreur seulement
B. syntaxe de construction (return error msg, fail msg, reject msg)
C. propagateur: ? explicite, ou when imbrique par etape
D. type d erreur: Err(String) conserve, ou type d erreur nomme par operation
E. metier vs technique: un Result par operation, ou deux canaux distincts
F. code de sortie: quel code pour Ok, quel code pour Err, et qui le fixe

Ces points doivent desormais etre articules au mecanisme de matching unifie (TASK-33) plutot qu a l alternative when elargi contre match, qui n existe plus.

Fossil: 44622fa8a5e072ee0d99272c5cf44bea6c2b8084
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 design: decision humaine actee et documentee sur A (constructeur explicite ou valeur d erreur seulement) - pas de code avant
- [ ] #2 design: decision humaine actee et documentee sur C (propagateur), en tenant compte du mecanisme de matching unifie (TASK-33) et non plus d une alternative when contre match
- [ ] #3 design: decision humaine actee et documentee sur D (type d erreur: Err(String) vs type nomme)
- [ ] #4 design: decision humaine actee et documentee sur E (metier vs technique) et F (codes de sortie)
- [ ] #5 checker: un Result ne peut pas etre construit en echec hors des sources autorisees (LCR, runtime, constructeur si adoption)
- [ ] #6 grammar.ebnf: syntaxe de construction d erreur, si adoption (version bump + notes de version)
- [ ] #7 codegen: une valeur Err construite par l application genere CamusResult::Err(..) de facon sound (pas de Err<String> forge sur un Result<T> non String)
- [ ] #8 codegen: propagation d erreur (operateur ? ou forwarding explicite) avec lowering correct et test
- [ ] #9 sample-project: une operation App refuse effectivement une entree invalide et le TCC rend le code de sortie attendu
- [ ] #10 tests: fixtures checker (bon + mauvais), fixture codegen, E2E sur stdout ET code de sortie du processus
- [ ] #11 ARCHITECTURE.md: section gestion des erreurs (construction, propagation, codes de sortie) et regles checker a jour
<!-- AC:END -->
