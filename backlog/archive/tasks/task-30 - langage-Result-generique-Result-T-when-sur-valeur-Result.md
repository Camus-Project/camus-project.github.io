---
id: task-30
title: 'langage: Result generique (Result<T>) + when sur valeur Result'
status: Done
assignee: []
created_date: '2026-09-29 12:05'
updated_date: '2026-09-29 12:12'
labels:
  - camus-pl
  - camus-pl-checker
dependencies: []
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Tache prealable a TASK-27 (refonte sample). Bloquee par le fait que le langage ne sait pas inspecter une valeur Result : le seul destructuring est when, reserve aux handles later. Un App qui retourne Result<uuidv7> est donc inexploitable par le TCC, et AC#1 de TASK-27 (traduire le Result en stdout + code de sortie) n est pas exprimable.

Changements:
- grammaire (grammar.ebnf): type_reference parametrise pour Result<T>
- checker: Ty::is_result reconnait Result et Result<T>; resolve les mappe sur Concrete::Result; when accepte un binding de type Result en plus des handles later
- codegen: map_ty Result<T> -> CamusResult<T>; emit_fn emet le type de retour Result et lower return <expr> en CamusResult::Ok(<expr>); pass-through d une valeur Result existante (propagation d erreur LCR) pour les Result a payload unitaire; when sur binding Result -> match andr

Limite assumee: pas de constructeur Err depuis le code applicatif (pas encore de syntaxe); les Err viennent des LCR ou du runtime.

Decisions humaines: Result generique (porteur de valeur); elargir when plutot qu ajouter un match general.

Fossil: 845da23337
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [x] #1 grammar.ebnf: type_reference accepte Result<T> (type parametre)
- [x] #2 checker: Ty::is_result reconnait Result et Result<T>; resolve mappe les deux sur Concrete::Result
- [x] #3 checker: when accepte un binding de type Result (pas seulement un handle produit par later); un when sur un non-Result reste rejete
- [x] #4 codegen: map_ty Result<T> -> CamusResult<T>; une fonction retournant Result emet le type de retour Rust
- [x] #5 codegen: return <expr> dans une fonction Result genere CamusResult::Ok(<expr>); pass-through d une valeur Result existante
- [x] #6 codegen: when sur un binding Result genere match andr sur CamusResult
- [x] #7 tests: fixtures checker (good/bad when sur Result) + fixture codegen App retournant Result<uuidv7> consomme par un TCC
<!-- AC:END -->
