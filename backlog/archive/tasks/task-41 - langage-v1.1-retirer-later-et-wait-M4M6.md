---
id: TASK-41
title: 'langage v1.1: retirer later et wait (M4+M6)'
status: To Do
assignee: []
created_date: '2026-10-01 20:36'
updated_date: '2026-10-06 14:26'
labels:
  - camus-pl
dependencies: []
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Retirer later et wait du lexique, du checker et du lowering. Dépend de task-38
(grammaire) et task-40 (runtime).

SUPPRESSION

Retirer later et wait de la liste des mots réservés (l.308-311) et du checker.
let x = f() remplace let x = later f(). La dépendance de données remplace
wait x: une instruction qui lit une valeur l attend si elle n est pas résolue
(attente par nécessité).

Mettre à jour les commentaires qui les citent: l en-tête (l.14, 20, 77), v0.6
(l.161-165), la note de lexique (l.312-314, l exemple let result: Result = later
...), et les spec function (l.441: pas d effet, ni appel de LCR, ni when / after).

Result seul (forme unitaire) reste valide: son rôle de handle de later
disparaît, pas son existence.

OUVERT — COLLISION DE NOM ignore

ignore désigne aujourd hui l écart d un cas dans une branche (<cas> ignore).
L ancien design prévoyait aussi un ignore d instruction (déclencher sans se
soucier du résultat). Ne pas implémenter l ignore d instruction avant arbitrage;
conserver l ignore de branche tel quel. Piste: renommer l un des deux.

PRÉCONDITION

Le lowering sait produire et propager des valeurs non résolues (task-40 livrée).
Ne pas retirer later / wait du lowering avant cela.

TESTS

- Un Result non consommé (ni lié, ni matché, ni retourné) est une erreur.
- Les anciens exemples later sont réécrits.
- La suite workspace reste verte.

Référentiel: note de synthèse v1.1, section M4. Ticket 78688008.
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
