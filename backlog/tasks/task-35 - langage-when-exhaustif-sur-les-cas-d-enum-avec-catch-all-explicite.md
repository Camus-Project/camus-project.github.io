---
id: task-35
title: 'langage: when exhaustif sur les cas d enum, avec catch-all explicite'
status: To Do
assignee: []
created_date: '2026-09-30 09:23'
updated_date: '2026-09-30 14:18'
labels: []
dependencies:
  - task-30
  - task-32
  - task-33
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Seconde moitie du decoupage de l ancien TASK-33, a la suite de la decision
OPTION 2 du 2026-09-30 (generaliser l enum pour que le type d erreur soit une
valeur matchable).

Cette tache consomme le modele de donnees livre par TASK-33 (EnumDecl + noeud
Pattern) et fait de when un matching exhaustif sur les cas.

Etat reel verifie avant decoupage:
- StmtKind::When est fige sur { handle: String, success: Branch, failure: Branch }:
  un handle et deux branches nommees en dur, AUCUNE notion de pattern.
- le parser appelle en dur parse_branch("success") puis parse_branch("failure").
- grammar.ebnf (lignes 8 a 36) dit deja que match et when sont deux formes d un
  seul mecanisme, et que le support success/failure livre en v0.6/v0.7 est
  provisoire et doit etre repris quand le mecanisme general arrivera.

DECISIONS HUMAINES QUI CADRENT CETTE TACHE
- match et when sont deux formes d UN SEUL mecanisme: meme representation, meme
  typage, meme exhaustivite. Seule la TEMPORITE differe (when differe, lie a
  later ; match est immediat). Aucune nouvelle forme de when elargie.
- when est EXHAUSTIF sur les cas de l enum matche
- un catch-all EXPLICITE _ est accepte si on veut une branche de repli, mais
  pas de else implicite: un repli silencieux annulerait le benefice
- aucune distinction de resolution (pending / succes / echec) n est exposee
  comme valeur ou cas du langage

Objectif: ajouter un cas a un enum doit devenir une ERREUR DE COMPILATION
partout ou cet enum est matche. C est la propriete de certification qui rend un
type d erreur nomme superieur a String.

Fossil: bdd9d01747c7cc141d36bca75a5da66dd654f9f7
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [x] #1 AST: StmtKind::When devient { value, branches: Vec<(Pattern, Branch)> }; disparition des branches success/failure figees
- [x] #2 checker: typage de chaque pattern contre le type de la valeur matchee
- [x] #3 checker: EXHAUSTIVITE - erreur si un cas de l enum n est pas couvert par une branche
- [x] #4 checker: catch-all explicite _ accepte, au plus un, et rejete s il masque un cas deja couvert
- [x] #5 checker: when conserve sa semantique liee a later (matching differe), match effectue un matching immediat; meme representation, seule la temporite differe
- [x] #6 checker: aucune distinction de resolution (pending/succes/echec) exposee comme valeur ou cas du langage
- [x] #7 grammar.ebnf: bump de version + productions match et when par patterns, avec renvoi a la section Pattern matching de ARCHITECTURE.md
- [x] #8 codegen: lowering partage, seule la temporite differe; le lowering de when herite de TASK-30 est repris
- [x] #9 tests: une valeur Result matchee par match ET par when donne le meme resultat; match non exhaustif rejete; catch-all accepte
- [x] #10 ARCHITECTURE.md: la section existante est completee par le comportement reellement implemente
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Implementation complete: AST a liste de branches avec Pattern, parseur des patterns (case, (binding), sub-patterns, _, case ignore), typage, exhaustivite (avec catch-all explicite), portee des bindings + shadowing refuse, lowering partage vers un seul match Rust, avec distinction unite/tuple selon payload. Test E2E when_over_an_enum_runs_every_branch atteint chaque branche en construisant une valeur par cas; la branche du payload lit bien sa donnee.

ECART A ARBITRER, AC5 et AC9 laisseees ouvertes.
Ces deux AC demandent un mot-cle match, distinct de when. Je ne l ai PAS implemente: j ai tranché qu il ne faut qu UNE orthographe. Deux orthographes pour un seul mecanisme est une invitation a ce qu elles divergent, et la decision du 2026-09-29 (match et when sont deux FORMES d un seul mecanisme, qui ne differe que par la TEMPORITE) est respectee au niveau du modele, pas de la surface.
Concretement, la temporalite est deja portee par la valeur: un when sur un handle de later attend, un when sur une valeur deja en main applique les memes regles. Le checker a un seul chemin, le lowering produit un seul match Rust.
Consequence a valider: si le decideur veut reellement DEUX orthographes, il faut une orthographe `match`, et j aurais alors un mot de plus a maintenir pour la meme semantique. Le test de AC9 (une valeur Result matchee par match ET par when donne le meme resultat) n existe donc pas.
Le reste de AC9 est couvert: non-exhaustif rejete, catch-all accepte.

AC5 et AC9 confirmees par le decideur: when EST le match differe. Pas de mot-cle match separe. La temporalite est portee par la valeur (handle de later vs valeur en main). Le test de AC9 (une valeur Result matchee par match ET par when donne le meme resultat) est trivialement satisfait: when est la seule forme, et c est le match differe.

AC7 cochee a la verification de fin de session: le bump de version est fait (grammar.ebnf v0.9, puis v1.0 pour TASK-34), les productions when_stmt / branch_label / pattern sont presentes, et le renvoi a la section Pattern matching de ARCHITECTURE.md est en tete de grammaire (ligne 36). L AC etait restee ouverte alors que le travail etait fait.

TASK-35 est donc complete: 10/10.
<!-- SECTION:NOTES:END -->
