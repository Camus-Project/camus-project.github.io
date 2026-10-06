---
id: TASK-33
title: 'langage: declaration d enum et noeud Pattern (construction et sous-patterns)'
status: To Do
assignee: []
created_date: '2026-09-29 15:33'
updated_date: '2026-10-06 14:26'
labels:
  - camus-pl
dependencies: []
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Tache extraite du decoupage de l ancien TASK-33, a la suite de la decision
OPTION 2 du 2026-09-30 (generaliser l enum pour que le type d erreur soit une
valeur matchable, et non une etiquette).

Cette tache ne traite QUE le modele de donnees: la declaration d un enum et le
noeud Pattern. La consommation de ce modele par when (exhaustivite, catch-all)
est dans TASK-35. Err(v) et Result<T, E> dependent des deux et restent dans
TASK-31.

Etat reel verifie avant decoupage (c est ce qui justifie ce decoupage):
- l AST n a AUCUN noeud Pattern. StmtKind::When est fige sur
  { handle: String, success: Branch, failure: Branch }: un handle et deux
  branches nommees en dur.
- le parser appelle en dur parse_branch("success") puis parse_branch("failure").
- grammar.ebnf (lignes 8 a 36) dit deja que match et when sont deux formes d un
  seul mecanisme, et que le support success/failure livre en v0.6/v0.7 est
  provisoire et doit etre repris quand le mecanisme general arrivera.
- la grammaire ne declare que deux formes de type: struct_decl et app_decl.
  Il n existe ni enum, ni type, ni alias. type_reference n accepte qu un seul
  parametre generique.

DECISIONS HUMAINES QUI CADRENT CETTE TACHE
- declaration d enum generalisee (option 2, ecarte: declaration dediee error,
  et reutilisation de struct comme type d erreur)
- les cas d un enum PORTENT des payloads (P2, ecarte: tags nus P1, qui
  produirait des erreurs non affichables et rexigerait de revenir a String)
- when sera EXHAUSTIF sur les cas, avec un catch-all EXPLICITE _ ; pas de else
  implicite, sinon le benefice de certification disparait

Objectif: un pattern est une valeur de premier classe, partageable par toutes
les formes de matching, et un enum est un type somme dont les cas sont
destructurables.

Fossil: bdd9d01747c7cc141d36bca75a5da66dd654f9f7
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [x] #1 grammar.ebnf: production enum_decl (nom, liste de cas, type de payload optionnel par cas), bump de version, renvoi a la section Pattern matching de ARCHITECTURE.md
- [x] #2 AST: EnumDecl (nom, cas, payload optionnel par cas) et un noeud Pattern unique partage (constructeur, sous-pattern, wildcard, binding de variable)
- [x] #3 parser: parse d une declaration enum; parse d un pattern (constructeur nu, constructeur charge, wildcard, binding)
- [x] #4 checker: resolution du type enum, verification du nombre et du type des payloads a la construction, typage des patterns contre le type de la valeur matchee
- [x] #5 checker: un binding de pattern n existe que dans sa branche (portee lexicale), et shadowing refuse
- [x] #6 tests: fixtures checker - enum valide, payload de mauvais type rejete, nombre d arguments incorrect rejete, binding hors portee rejete
- [x] #7 codegen: lowering d une valeur enum, et matching executable sur un enum
- [x] #8 ARCHITECTURE.md: section enums et patterns decrite, coherent avec la decision du 2026-09-30
- [x] #9 HORS PERIMETRE (garde-fou): la refonte de when en branches exhaustives est dans TASK-35; Result<T,E> et Err(v) restent dans TASK-31
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
PARTIE 1 LIVREE (2026-09-30), sur les 9 AC. - grammar.ebnf v0.8: enum_decl + enum_case, un cas portant au plus UNE charge utile. La construction reutilise `new` avec un type QUALIFIE: new TaskError.NotFound(id). Choix motive dans ARCHITECTURE.md: un nom qualifie apres new est non ambigu, alors qu une syntaxe dediee aurait concurrence l appel de methode pointu receiver.method(...). - AST: EnumDecl, EnumCase, et le noeud Pattern (Constructor / Wildcard / Binding) qui sera consomme par TASK-35. - parser: parse_enum, dispatch `enum ` au niveau superieur, et `new` accepte desormais un chemin qualifie. Parentheses vides sur un cas sont REJETEES: un cas sans payload s ecrit nu, donc les deux formes ne peuvent pas se confondre. - checker: World porte les enums, Concrete::Enum, resolution, et check_enum_construction qui refuse nom d enum inconnu, cas inexistant, arguments nommes, nombre d arguments incorrect, et payload de mauvais type. Un enum ne peut pas non plus partager son nom avec un struct. - codegen: Project porte les enums, emit_enum produit un enum Rust derive(Clone, Debug, PartialEq), un cas a payload devient une variante tuple et un cas nu une variante unite; translate_new traduit new Enum.Case(x) en Enum::Case(x). - map_ty: un nom d enum passe par last_segment, donc un payload uuidv7 donne u128 et String donne String. - ARCHITECTURE.md: section dedicated, avec le raisonnement des trois decisions. TESTS: 54 verts sur cargo test --workspace (51 avant), dont checker +2 et codegen +1. Les deux fixtures negatives sont calees sur UNE SEULE erreur chacune (assert_eq sur diags.len()), pour qu un diagnostic ne puisse pas masquer un autre. Le test codegen compile et EXECUTE le binaire: les trois formes de cas sont construites a l execution. AC4, AC5 et AC9 restent ouvertes: le typage des patterns et la portee des bindings ne sont pas encore exerables, car rien ne consomme encore un Pattern. C est le travail de TASK-35. Detail de syntaxe constate: une fonction membre doit porter sa visibilite (private/public) sur sa ligne de nommage, sinon le parseur rejette le membre; les fixtures ont ete ecrites avec `private function x`.

Correction d honnetete: AC3 decochee. Le noeud Pattern existe dans l AST mais son PARSEUR n est pas ecrit: rien dans la syntaxe actuelle ne peut encore ecrire un pattern, puisque seule la forme a success/failure est acceptee. L ecriture du parseur de patterns appartient a TASK-35, la ou les branches deviennent une liste de patterns. AC9 cochee: garde-fou respectee, la refonte de when n a pas ete touchee ici.

AC3, AC4 et AC5 livrees avec TASK-35, qui devait les rendre exerables.
- Parseur de patterns: un nom nu en tete de branche est un CAS (Conflict), le meme nom nu entre parentheses est un BINDING qui accepte tout (NotFound(id)). La position suffit, donc aucun lookahead n est requis. Les sous-patterns s imbriquent.
- Reprise de 30 occurrences de  /  en  /  dans les fixtures, le sample-project et l ARCHITECTURE. C est une rupture de syntaxe VOLONTAIRE: garder les deux orthographe aurait laisse deux mecanismes se diverger, ce que la decision unificatrice exclut.
-  a ete preserve: un mot-cle de CORPS de branche, pas un binding. La migration l avait d abord casse en , transformant un discard en un binding nomme ignore.
- Typage des patterns: cas inconnu, sous-pattern sur un cas sans payload, et sous-pattern manquant sur un cas qui en a un sont tous refuses. Un nom nu entre parentheses ne pretend rien du payload.
- Portee: un binding appartient a sa branche et le shadowing est refuse.
- Les fixtures good/when_exhaustive.cam couvrent les trois formes: tous les cas, catch-all, et payloads discartes.

Etat de fin de session: 9/9, complete. Le ticket Fossil bdd9d017 peut etre clos.
<!-- SECTION:NOTES:END -->
