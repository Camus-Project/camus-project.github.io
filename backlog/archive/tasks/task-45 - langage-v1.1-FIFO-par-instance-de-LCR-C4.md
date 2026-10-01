---
id: task-45
title: 'langage v1.1: FIFO par instance de LCR (C4)'
status: To Do
assignee: []
created_date: '2026-10-01 20:40'
labels:
  - camus-pl
dependencies:
  - task-40
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Garantir l ordre des effets par une file FIFO par instance de LCR. Dépend de
task-40 (runtime).

CONTRAT

La ressource est l instance de LCR (le handle obtenu par ctx.get), pas la
fonction. Les appels à une même instance sont exécutés dans l ordre du texte,
c est-à-dire dans l ordre où le flux séquentiel atteint l instruction, même si
l appel est ensuite détaché. Des instances différentes sont concurrentes.

PRÉCISIONS

- L identité est celle de l instance résolue, pas du nom de variable.
- L interface d un LCR déclare quelles opérations lisent et lesquelles écrivent.
  Les lectures consécutives peuvent être regroupées entre deux écritures.
  Interface silencieuse = « tout écrit » (direction sûre).
- Toute implémentation d un LCR (réelle, fichier, mock de test) honore ce contrat.
- after ne sert alors que pour l ordre entre LCR distincts mais liés, ou pour un
  état externe non modélisé. Un after doit se justifier par une relation que les
  signatures n expriment pas.

RÈGLE D OR

L ordre observable des effets ne dépend jamais d un choix d optimisation du
compilateur. Toute exécution respectant les dépendances est légale, y compris
l exécution entièrement synchrone.

TESTS

- Deux écritures sur la même instance conservent l ordre du texte, même détachées.
- Deux instances différentes restent concurrentes.
- Les lectures consécutives sont regroupées entre deux écritures.

Référentiel: note de synthèse v1.1, section C4. Ticket 78688008.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 la ressource est l instance de LCR (ctx.get), pas la fonction
- [ ] #2 les appels a une meme instance suivent l ordre du texte, meme detaches
- [ ] #3 des instances differentes restent concurrentes
- [ ] #4 l identite est celle de l instance resolue, pas du nom de variable
- [ ] #5 l interface d un LCR declare lectures et ecritures; interface silencieuse = tout ecrit
- [ ] #6 apres ne sert que pour l ordre entre LCR distincts mais lies
- [ ] #7 tests: deux ecritures sur la meme instance conservent l ordre; deux instances concurrentes
<!-- AC:END -->
