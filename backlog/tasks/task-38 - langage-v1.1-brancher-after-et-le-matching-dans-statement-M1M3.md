---
id: task-38
title: 'langage v1.1: brancher "after" et le matching dans "statement" (M1+M3)'
status: To Do
assignee: []
created_date: '2026-10-01 20:35'
updated_date: '2026-10-01 20:37'
labels:
  - camus-pl
dependencies: []
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Fondation de l évolution v1.1. Aucune dépendance.

La grammaire définit `when_stmt` (l.504) mais `statement` (l.545) ne le
référence pas: `when` est aujourd hui parsé par un test de préfixe textuel
(parse.rs:1385). Ce câblage est à corriger.

MODIFICATIONS

M1 — Ajouter `after` et brancher le matching dans `statement`:

    matching_stmt = ( "when" | "after" ) , identifier , line_break ,
                    indent , { when_branch } , dedent ;

    statement     = "assert" , expr , line_break
                  | matching_stmt
                  | ? placeholder ? ;

Une seule production. La différence `when` / `after` est un attribut du nœud
AST (`timing`), pas un second arbre. `when_branch`, `branch_label`, `pattern`
restent inchangés.

M3 — Sémantique:
- `when v` enregistre les branches pour le moment où `v` est résolue et ne
  bloque pas les instructions suivantes (point de concurrence).
- `after v` fait attendre les instructions suivantes jusqu à la fin de la branche
  exécutée (point de séquence). On attend le flux logique, pas forcément un
  thread. Sur une valeur déjà résolue, `after` est l ancien `match`.
- Une instruction qui lit une valeur dépend d elle et l attend si elle n est pas
  résolue (attente par nécessité). Cela vaut aussi pour `return`.

MIGRATION

Tout `when` existant dont une branche contient `return`, écrit une variable
extérieure, ou dont la suite suppose la fin de la branche, devient `after`.
L exemple de la l.512-523 (`when result` avec `return` dans les branches) est
en réalité un `after`.

RÈGLES DE CHECKER (les mêmes pour les deux formes)

- Exhaustivité obligatoire, `_` explicite, aucun repli implicite.
- `return` est interdit dans un corps de `when` (la fonction a pu déjà
  retourner), autorisé dans `after`.
- Un corps de `when` ne référence aucun `let mut` de la fonction (ni lecture ni
  écriture). Un instantané se fait par un `let` immuable au préalable.
- Un appel dont le résultat est un `Result` et qui n est ni lié, ni matché, ni
  retourné est une erreur.

Référentiel: note de synthèse v1.1, sections M1 et M3. Ticket 78688008.
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
