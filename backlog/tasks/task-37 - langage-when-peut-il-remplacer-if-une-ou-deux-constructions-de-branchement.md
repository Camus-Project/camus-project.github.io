---
id: task-37
title: 'langage: when peut-il remplacer if (une ou deux constructions de branchement)'
status: To Do
assignee: []
created_date: '2026-09-30 20:52'
updated_date: '2026-09-30 21:38'
labels:
  - camus-pl
dependencies:
  - task-32
  - task-35
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Question de conception ouverte par l abandon de D1 (when sur handle, match sur
valeur) et par le retrait de match au profit d un when unique et total. Aucun
code tant qu une decision humaine n est pas actee.

LA QUESTION

Le langage a-t-il UNE ou DEUX constructions de branchement ?

Une : `when` couvre le type discrimine (Result, enum) ET le booleen.
Deux : `if` pour le booleen, `when` pour le type discrimine, avec des formes
     unifiees (indentation, discipline d initialisation, noms de branches).

L intuition pour une seule construction: `when` total + `later` suffisent a
basculer du sync a l async, donc un seul mot couvre les deux temporalites.

CE QUI BLOQUE L EXEMPLE (when result > 1 / < 1 / _)

1. `if` raisonne sur un booleen, `when` sur un type a cas.
2. `>1` / `<1` sont des predicats, pas des motifs du type. Les accepter comme
   motifs n est pas du pattern matching: c est une troisieme forme, qui
   reintroduit la derive que `when` unique voulait supprimer.
3. `let result = later blabla()` donne un handle en cours, pas un nombre:
   `wait` / `ignore` est requis avant de comparer.

L ARBITRAGE REEL

Purete du pattern matching contre commodite du langage. Une construction qui
parle au langage (une temporalite, un seul mot) contre une construction qui
reste un vrai pattern matching sur un type discrimine.

OPTIONS

A. Deux constructions, roles distincts, formes unifiees. C est l etat actuel de
   la branche trunk (avec match en trop, retire).
B. Une construction etendue aux predicats. `when` devient un fourre-tout. Non
   recommande.
C. Une construction pure (motifs types seulement) + `if` pour le booleen.
   Equivalent a A renomme.

QUESTION CONNEXE

La comparaison de seuils (>1 / <1) est-elle un besoin reel qui merite un sucre
dedie, ou un artefact de l exemple?

Ne rien coder avant la decision. Si la decision touche la grammaire, mettre a
jour grammar.ebnf (version) et ARCHITECTURE.md, et bump du ticket.

Fossil: 78688008d6451c725c431205690a07524550b80e
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 decision humaine actee et ecrite: une ou deux constructions de branchement
- [ ] #2 le role de chaque construction est ecrit (booleen vs type discrimine)
- [ ] #3 les formes des deux constructions sont unifiees si on garde les deux (indentation, discipline d initialisation, noms de branches)
- [ ] #4 la question du sucre de comparaison de seuils (>1/<1) est tranchee: besoin reel ou artefact
- [ ] #5 si la decision change le langage: grammar.ebnf bumpe et ARCHITECTURE.md mis a jour

- [ ] #6 livrable 1: sort de if tranche, avec grammaire des etiquettes de branche et sort du controle d exhaustivite
- [ ] #7 livrable 2: preuve ou contre-exemple du caractere observable ou non du choix async/sync par le transpiler
- [ ] #8 livrable 3: tension async universelle vs purete des lois tranchee, en ecrivant ce qui cede
<!-- AC:END -->
