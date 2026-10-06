---
id: TASK-43
title: 'langage v1.1: boucles after each et when each (M5)'
status: To Do
assignee: []
created_date: '2026-10-01 20:39'
updated_date: '2026-10-06 14:26'
labels:
  - camus-pl
dependencies: []
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Boucles: after each (séquentiel) et when each (concurrent). Dépend de task-38.

PRÉREQUIS BLOQUANT — OUVERT

La grammaire n a aucun type collection (type_reference: seulement un nom qualifié
et Result<T>). Ne pas inventer List<T>. Si le checker n en définit pas, s arrêter
et demander. M5 ne peut pas démarrer avant une décision sur les collections.

GRAMMAIRE

    each_stmt = ( "after" | "when" ) , "each" , identifier , "in" , identifier ,
                line_break ,
                indent , [ law_block ] , statement_block , dedent ;

    statement = "assert" , expr , line_break
              | matching_stmt
              | each_stmt
              | ? placeholder ? ;

Extensions des lois: une loi de boucle (each_stmt) porte invariant: seulement.

SÉMANTIQUE

| Forme | Tours | Mutation | Effets |
|---|---|---|---|
| after each x in xs | séquentiels, dans l ordre de xs | peut affecter les let mut locaux | enfilés dans l ordre |
| when each x in xs | détachés | lecture seule de valeurs immuables, aucun let mut | enfilés dans l ordre de xs à l atteinte de chaque tour, donc déterministes |

RÈGLES DE CHECKER

1. Un let mut n est affectable que dans sa propre fonction, y compris dans un
   after each.
2. Une boucle n affecte pas directement les champs de self. La mutation de self
   passe par les méthodes mut self.
3. Pas de break ni de continue. La sortie anticipée sur erreur se fait par return
   d un failure depuis after each.
4. Dans after each, le premier failure interrompt la boucle et se propage. Dans
   when each le comportement est OUVERT (annulation des tours frères ou collecte
   de tous les résultats).
5. Un when each dont tous les tours touchent le même handle de LCR ne gagne aucun
   temps (écritures sérialisées): émettre un avertissement.
6. La terminaison de each est automatique (collection finie): aucune loi de
   terminaison n est exigée.

ORDRE DE MISE EN ŒUVRE

after each d abord; when each ensuite; repeat en dernier, après spécification.

RÉPÉTITION CONDITIONNELLE (repeat) — OUVERT

Il n y a pas de while. Intention: chaque tour lie un état nouveau (pas de
variable modifiée), le corps produit un enum Step (Next(État) ou Stop(Résultat))
matché par after, et decreases: est obligatoire. Un repeat sans decreases n est
admis que dans une struct role tcc. Ne pas implémenter avant spécification
complète.

Référentiel: note de synthèse v1.1, section M5. Ticket 78688008.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 le runtime repose sur JoinSet: chaque portee possede un JoinSet, when lance dans celui de la portee
- [ ] #2 concurrence structuree: la fin de la portee joint le JoinSet, rien ne survit a la portee
- [ ] #3 differe a la demande: le compilateur n emet un spawn que si un consommateur when existe
- [ ] #4 after est sequentiel (attend le flux logique), when est detache (ne bloque pas la suite)
- [ ] #5 FIFO par instance de LCR: les appels a une meme instance suivent l ordre du texte
- [ ] #6 aucun Pending<T> ou Future<T> n est expose dans les types Camus
- [ ] #7 les tests de concurrence existants sont portes et la suite workspace reste verte
<!-- AC:END -->
