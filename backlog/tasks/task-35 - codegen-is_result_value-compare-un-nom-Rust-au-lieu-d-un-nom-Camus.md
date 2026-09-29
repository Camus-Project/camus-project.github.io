---
id: task-35
title: 'codegen: is_result_value compare un nom Rust au lieu d un nom Camus'
status: Done
assignee: []
created_date: '2026-09-29 21:59'
updated_date: '2026-09-29 22:00'
labels:
  - camus-pl
dependencies:
  - task-34
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Bug dans is_result_value (codegen/src/gen.rs), releve hors de TASK-34.

La fonction decide si une expression retournee par une fonction Result est deja une valeur Result, au quel cas elle est transmise telle quelle au lieu d etre re emballee en Ok.

Elle compare l entree d environnement au nom RUST "CamusResult". Or l environnement contient des noms de types CAMUS : un binding Result est enregistre comme "Result" ou "Result<T>" (voir Ty::is_result). La comparaison etait donc TOUJOURS FAUSSE.

CONSEQUENCE

    let found: Result<uuidv7> = store.find(name)
    return found

produisait

    return CamusResult::Ok(found);

soit un CamusResult<CamusResult<T>> renvoye par une fonction declaree -> CamusResult<T>. Le code genere ne compile pas.

CORRECTION

    t == "Result" || t.starts_with("Result<")

La branche ExprKind::Later(_) => true est en meme temps supprimee : elle etait inaccessible (translate_expr refuse later hors d un let, et un handle n est pas une valeur Result) et etait fausse.

HORS PERIMETRE

Concrete::Handle, laisse pour apres.

Fossil: d59a831eb1f348652f389bc69ff425be09257cce
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [x] #1 is_result_value reconnait les types Result et Result<T> contenus dans l environnement
- [x] #2 return found ou found: Result<T> est abaisse en return found, pas en return CamusResult::Ok(found)
- [x] #3 la branche Later inaccessible et fausse est retiree
- [x] #4 une fixture de regression resultforward genere du code qui compile
- [x] #5 regression: checker 34, codegen 21, runtime 9, verus-gen 3, clippy sans nouvelle alerte
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
FIXED (Fossil 574cd5efa, 2026-09-29).

is_result_value now recognises the Camus type names the environment actually
holds: "Result" and "Result<T>", the same predicate as Ty::is_result. Before,
it compared against the Rust name "CamusResult", which the environment never
contains, so it was always false and `return found` where `found: Result<T>`
was lowered to `return CamusResult::Ok(found)` — a CamusResult<CamusResult<T>>
returned from a -> CamusResult<T> function, which does not compile.

The ExprKind::Later(_) => true arm is removed: it was unreachable
(translate_expr rejects `later` outside a `let` right-hand side) and a handle
is not a Result value.

New fixture codegen/tests/fixtures/resultforward exercises the forwarding path
end to end and compiles the generated code.

Tests: checker 34, codegen 21, runtime 9, verus-gen 3. Clippy: no new warning.
E2E unchanged.
<!-- SECTION:NOTES:END -->
