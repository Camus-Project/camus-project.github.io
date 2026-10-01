---
id: task-44
title: 'langage v1.1: repenser l observabilite de l asynchrone'
status: To Do
assignee: []
created_date: '2026-10-01 20:40'
updated_date: '2026-10-01 20:43'
labels:
  - camus-pl
dependencies:
  - task-38
  - task-40
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
- [ ] #1 later et wait retires de la liste des mots reserves
- [ ] #2 let x = f() remplace let x = later f(); la dependance de donnees remplace wait x
- [ ] #3 les commentaires citant later/wait sont mis a jour
- [ ] #4 Result seul (forme unitaire) reste valide
- [ ] #5 un Result non consomme (ni lie, ni matche, ni retourne) est une erreur
- [ ] #6 les anciens exemples later sont reecrits
- [ ] #7 la suite workspace reste verte
<!-- AC:END -->
