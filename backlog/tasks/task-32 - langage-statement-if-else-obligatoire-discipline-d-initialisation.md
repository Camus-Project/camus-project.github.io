---
id: TASK-32
title: 'langage: statement if (else obligatoire) + discipline d initialisation'
status: Done
assignee: []
created_date: '2026-09-29 13:01'
updated_date: '2026-09-29 14:23'
labels:
  - camus-pl
dependencies: []
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Prealable a TASK-27 (refonte du sample) et TASK-31 (gestion des erreurs). Le langage n a aucun branchement conditionnel: le lexer n expose aucun mot-cle de condition, la production statement de la grammaire ne definit que assert, et le seul mecanisme de flux de controle est when (deux branches Ok/Err).

Consequences: Result<T> est un conteneur que l application ne peut ni remplir ni vider utilement; une garde applicative simple n a aucune ecriture possible; requires couvre un cas voisin mais echoue au lieu de router.

Decisions humaines (2026-09-29):
- if est un STATEMENT avec indentation, aligne sur when
- else est OBLIGATOIRE: les deux branches sont ecrites, flot total sur les deux chemins
- le checker EXIGE une initialisation avant toute lecture
- option A: forme minimale; l expression-valeur (let x = if ... else ...) n est pas dans le perimetre
- la gestion des erreurs reste separee: ni constructeur Err ni code de sortie dans cette tache (reste TASK-31)

A trancher avant le code:
1. etiquettes des branches (then/else) ou corps nu
2. categorie de la condition (pure_expr, comme assert et les lois)
3. portee exacte de la discipline d initialisation

Fossil: 29360a7c924851ed3e06b4f58b0b74a8e2609ee4
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [x] #1 design: decisions humaines 1-3 actees et documentees avant toute ecriture de code
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Livré au-delà du seul AC#1 (la tache avait ete creee avec une AC unique, trop grossiere).

Contenu reel:
- grammar.ebnf v0.7: production statement etendue avec if/pure_expr/then/else, production statement_block
- ast.rs: StmtKind::If { cond, then_body, else_body }
- parse.rs: lecture de if, messages explicites si then ou else manque
- check.rs: condition Bool obligatoire; chaque branche dans un env enfant; ends_with_return accepte un if final dont les deux branches retournent
- gen.rs: if_lowering produit un if/else Rust; stmt_uses_self descend dans les branches
- fixtures: good/if_statement.cam, bad/if_missing_else.cam, bad/if_non_boolean.cam, bad/let_without_value.cam, codegen/tests/fixtures/ifstatement/main.cam
- 4 tests checker + 1 test codegen
- ARCHITECTURE.md: section if + liste des regles a jour

Deux bugs du meme antipartie corriges: un ? qui renvoyait None sans message, et l appelant ignorait ce None.
- let x: Int sans valeur etait supprime silencieusement
- if sans else etait accepte silencieusement

Limite assumee: pas de constructeur Err ni code de sortie (reste TASK-31). Pas d if-expression (option A, decision humaine).

Tests: checker 30/30, codegen 14 (2+5+6+1), verus-gen 3/3
<!-- SECTION:NOTES:END -->
