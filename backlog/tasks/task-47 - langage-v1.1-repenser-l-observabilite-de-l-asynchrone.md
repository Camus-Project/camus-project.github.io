---
id: task-47
title: 'langage v1.1: repenser l observabilite de l asynchrone'
status: To Do
assignee: []
created_date: '2026-10-01 20:40'
labels:
  - camus-pl
dependencies: []
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Reprendre la réflexion sur l observabilité de l asynchrone. Indépendante.

CONTEXTE

La proposition de rendre visible l asynchrone par un rendu annoté du compilateur
(graphe de dépendances, points when/after, points d attente inférés) a été
rejetée par l auteur (ticket 78688008 §12). La question reste ouverte.

LE PROBLÈME

C7 retire le handle des types: aucun Pending<T> ou Future<T> n est exposé. En
conséquence, la source ne dit plus où le programme peut bloquer. Pour un langage
dont le but est l auditabilité, c est un coût réel: lire le source ne suffit plus
à savoir où une suspension est possible.

CONTRAINTES

- L observation ne modifie jamais l ordre des effets, ne crée pas de dépendance,
  ne fait pas échouer le programme.
- Elle passe par un canal hors-bande, pas par un LCR ordinaire.
- Une valeur classée secrète n est jamais émise: sa forme et son identité
  seulement.
- Le défaut sûr reste: after plutôt que when, séquentiel plutôt que concurrent.

QUESTIONS À TRANCHER

- Faut-il un marqueur dans la source, ou accepte-t-on que la source ne montre pas
  les points de suspension?
- Si un marqueur, quelle forme? Un mot-clé, une annotation, une convention de nom?
- Le rendu annoté est-il vraiment écarté, ou seulement sa forme actuelle?
- Comment l outillage de revue (pas le compilateur) rend-il les dépendances
  visibles sans bruit?

LIVRABLE

Une position écrite sur l observabilité, avec une recommandation. Pas
d implémentation avant décision.

Référentiel: note de synthèse v1.1, section C7 et §6. Ticket 78688008 §12.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 le rendu annote du compilateur est ecarte; la question reste ouverte
- [ ] #2 le probleme est formule: C7 retire le handle, la source ne dit plus ou le programme peut bloquer
- [ ] #3 les contraintes sont listees (pas d effet sur l ordre, canal hors-bande, secret)
- [ ] #4 les questions a trancher sont posees (marqueur? forme? rendu vraiment ecarte?)
- [ ] #5 livrable: une position ecrite avec recommandation, pas d implementation avant decision
<!-- AC:END -->
