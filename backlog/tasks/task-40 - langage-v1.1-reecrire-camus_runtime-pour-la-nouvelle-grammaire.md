---
id: task-40
title: 'langage v1.1: reecrire camus_runtime pour la nouvelle grammaire'
status: To Do
assignee: []
created_date: '2026-10-01 20:36'
updated_date: '2026-10-01 20:43'
labels:
  - camus-pl
dependencies:
  - task-38
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Réécrire le crate camus_runtime pour appliquer la nouvelle grammaire. Dépend de
task-38.

CONTEXTE

Le crate camus_runtime et rtlink.rs (réintégrés de 7ce2235c6c) sont construits sur
later/wait/when/ignore avec des JoinHandle tokio. La nouvelle grammaire utilise
JoinSet + spawn et le différé à la demande. Le runtime actuel est donc
supersedé et doit être réécrit.

MEILLEURE IMPLÉMENTATION

Le runtime repose sur trois idées:

1. CONCURRENCE STRUCTURÉE. Chaque portée (fonction, interaction TCC) possède un
   JoinSet. when lance le corps de la branche dans le JoinSet de la portée. La
   fin de la portée joint le JoinSet: rien ne survit à la portée qui l a créé.
   La racine est une interaction de TCC.

2. DIFFÉRÉ À LA DEMANDE. Le compilateur n émet un spawn que si un consommateur
   when existe. Sinon l appel s exécute en place, en séquentiel. C est le signal
   qui rend un appel différable (décision C1, ticket 78688008 §12). Le
   producteur n a pas à savoir comment son résultat est consommé.

3. after = SÉQUENTIEL, when = DÉTACHÉ. after attend le flux logique (pas
   forcément un thread) et bloque la suite. when ne bloque pas la suite. Sur une
   valeur déjà résolue, after est un match simple.

FIFO PAR INSTANCE DE LCR

La ressource est l instance de LCR (le handle obtenu par ctx.get), pas la
fonction. Les appels à une même instance sont exécutés dans l ordre du texte,
même si l appel est ensuite détaché. Des instances différentes sont concurrentes.
L identité est celle de l instance résolue, pas du nom de variable.

PAS DE TYPE EN ATTENTE

Aucun type Pending<T> ou Future<T> n est exposé dans les types Camus (C7). Le
runtime manipule des JoinHandle en interne, mais ils ne traversent jamais la
frontière du langage.

LIVRABLES

- Un crate camus_runtime réécrit sur JoinSet + différé à la demande.
- rtlink.rs adapté au nouveau modèle (ou remplacé).
- Les tests de concurrence existants (concurrent_waiters_all_observe_the_outcome,
  later_runs_the_work_off_the_calling_thread) portés et faisant sens dans le
  nouveau modèle.
- La suite workspace reste verte.

Référentiel: note de synthèse v1.1, sections C1, C5, C7 et §5. Ticket 78688008.
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
