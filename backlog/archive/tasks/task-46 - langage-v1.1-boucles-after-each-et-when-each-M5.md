---
id: task-46
title: 'langage v1.1: boucles after each et when each (M5)'
status: To Do
assignee: []
created_date: '2026-10-01 20:40'
labels:
  - camus-pl
dependencies:
  - task-38
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
- [ ] #1 prerequis: decision sur les types collection (bloquant, OUVERT)
- [ ] #2 each_stmt = ( after | when ) each identifier in identifier
- [ ] #3 after each: sequentiel, peut affecter les let mut locaux
- [ ] #4 when each: detache, lecture seule, aucun let mut reference
- [ ] #5 une loi de boucle porte invariant: seulement
- [ ] #6 pas de break ni de continue; sortie par return d un failure
- [ ] #7 la terminaison de each est automatique (collection finie)
- [ ] #8 apres each en premier, when each ensuite, repeat en dernier
<!-- AC:END -->
