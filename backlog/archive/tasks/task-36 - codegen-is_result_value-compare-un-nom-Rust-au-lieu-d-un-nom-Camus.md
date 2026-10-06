---
id: TASK-36
title: 'codegen: is_result_value compare un nom Rust au lieu d un nom Camus'
status: Done
assignee: []
created_date: '2026-09-30 20:43'
updated_date: '2026-10-06 14:26'
labels:
  - camus-pl
dependencies: []
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Bug dans is_result_value (codegen/src/gen.rs), livre par cherry-pick du check-in 574cd5efaa.

La fonction decide si une expression retournee par une fonction Result est deja une valeur Result, au quel cas elle est transmise telle quelle au lieu d etre re-emballee en Ok.

Elle comparait l entree d environnement au nom RUST "CamusResult". Or l environnement contient des noms de types CAMUS : un binding Result est enregistre comme "Result" ou "Result<T>" (voir Ty::is_result). La comparaison etait donc TOUJOURS FAUSSE.

CONSEQUENCE

    let found: Result<uuidv7> = store.find(name)
    return found

produisait

    return CamusResult::Ok(found);

soit un CamusResult<CamusResult<T>> renvoye par une fonction declaree -> CamusResult<T>. Le code genere ne compile pas.

CORRECTION

    t == "Result" || t.starts_with("Result<")

La branche ExprKind::Later(_) => true est supprimee : elle etait inaccessible (translate_expr refuse later hors d un let) et un handle n est pas une valeur Result.

La fixture resultforward et son test sont reecrits pour cette branche : match n existe plus ici (v0.9 ne garde que when), donc la fixture matche avec when et utilise la syntaxe Ok(id) / Err(error) qu impose ce parseur. Le test est ajoute au generate.rs de cette branche, aligne sur le codegen when-only plutot que d importer le corpus de tests D1.

Fossil: d59a831eb1f348652f389bc69ff425be09257cce
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 is_result_value reconnait les noms de types Camus de l environnement (Result, Result<T>)
- [ ] #2 return found ou found: Result<T> est abaisse en return found, pas en return CamusResult::Ok(found)
- [ ] #3 la branche Later inaccessible et fausse est retiree
- [ ] #4 fixture resultforward : code genere compile sur cette branche (when-only)
- [ ] #5 regression: checker 46, codegen 19
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
DELIVERED (Fossil 0d937587809, cherry-pick de 574cd5efaa).

is_result_value reconnait desormais les noms de types Camus contenus dans l
environnement. La branche ExprKind::Later, inaccessible et fausse, est retiree.

La fixture resultforward et son test sont reecrits pour cette branche : match
n existe plus ici, donc la fixture matche avec when et utilise la syntaxe
Ok(id) / Err(error) qu impose ce parseur.

Tests: checker 46, codegen 19. verus-gen conserve un echec preexistant sur
sample_project_verifies_with_verus, non touche par ce changement.

Cette tache remplace mon ancienne task-35 (meme numero, autre contenu sur la
feuille 9f7a409757), d ou le renumerotage en 36.
<!-- SECTION:NOTES:END -->
