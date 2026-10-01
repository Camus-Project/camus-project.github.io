---
id: task-39
title: 'langage v1.1: patterns booleens et suppression de if (M2)'
status: To Do
assignee: []
created_date: '2026-10-01 20:35'
updated_date: '2026-10-01 20:43'
labels:
  - camus-pl
dependencies:
  - task-38
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Supprimer if et traiter le booléen comme un cas de matching. Dépend de task-38.

MÉCANISME BOOLÉEN

Un pattern peut être un littéral booléen:

    pattern = identifier , [ "(" , pattern , ")" ]
            | boolean ;

Bool est traité comme un enum à deux cas true / false. Il est donc exhaustif
par construction. when / after travaillent toujours sur un identifier: une
condition est d abord liée à une variable.

    let big = n > 3
    after big
      true
        notify()
      false ignore

L obligation d avoir un else (v0.7) est subsumée par l exhaustivité: les deux cas
doivent apparaître, false ignore suffit pour « rien à faire ».

RÉÉCRITURE MÉCANIQUE

if C then A else B devient:

    let cond_N = C
    after cond_N
      true
        A
      false
        B

cond_N est un nom frais qui n entre en collision avec aucun nom visible.
Retirer ensuite la mention de if dans le commentaire de statement.

La discipline d initialisation (v0.7) est inchangée: une variable est initialisée
à son let, ou pas du tout.

OUVERT

Une garde sur valeur (Int if n > 3) est une alternative possible. Ne pas l
implémenter dans cette itération.

Référentiel: note de synthèse v1.1, section M2. Ticket 78688008.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 after est une production de statement via matching_stmt
- [ ] #2 l AST distingue when et after par un attribut timing
- [ ] #3 les memes regles d exhaustivite s appliquent aux deux formes
- [ ] #4 return dans un corps de when est rejete par le checker
- [ ] #5 un let mut reference dans un corps de when est rejete
- [ ] #6 les when existants contenant return sont migres vers after
- [ ] #7 tests: meme exhaustivite pour les deux; return dans when rejete; let mut dans when rejete
<!-- AC:END -->
