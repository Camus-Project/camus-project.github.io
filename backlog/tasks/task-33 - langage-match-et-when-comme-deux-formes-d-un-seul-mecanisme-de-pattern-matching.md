---
id: TASK-33
title: >-
  langage: match et when comme deux formes d un seul mecanisme de pattern
  matching
status: To Do
assignee: []
created_date: '2026-09-29 15:33'
updated_date: '2026-09-29 15:37'
labels:
  - camus-pl
dependencies:
  - TASK-30
  - TASK-32
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Decision humaine du 2026-09-29. Cette tache NE IMPLEMENTE RIEN pour l instant: elle fige le modele avant que le code soit ecrit, afin que when ne soit PAS construit comme un cas particulier du matching de Result.

MODELE RETENU
  match(valeur) = matcher la valeur MAINTENANT (deja disponible)
  when(valeur)  = matcher la valeur QUAND ELLE DEVIENT DISPONIBLE (issue de later)

Au moment ou when se declenche il effectue EXACTEMENT le meme matching que match. Les deux partagent le maximum possible: syntaxe des patterns, regles de typage des patterns, verification d exhaustivite, representation AST du matching, regles de verification des branches. La seule difference est portee par la semantique d execution et le lowering. when conserve sa semantique historique liee a later.

PAS DE MODELE DISTINCT POUR LES ETATS DE RESOLUTION
Le langage n expose PAS pending / succes / echec de resolution / erreur produite comme etats distincts. Ces etats peuvent exister dans le runtime mais ne sont pas des valeurs ni des cas Camus. Ce qui compte est la valeur produite a la fin. Une distinction plus fine doit etre exprimese dans le TYPE de l erreur et son matching, pas par un second mecanisme implicite de resolution des later.

CONSEQUENCE SUR L EXISTANT
Le travail TASK-30 (when sur une valeur Result) emploie des etiquettes success/failure codes en dur: forme TRANSITOIRE, pas le modele cible. Le travail sur la charge utile Result<T> reste valable; etiquettes et regle du checker seront reprises.

PERIMETRE
Mecanisme de pattern matching generalise et articulation avec when. Ne traite pas la fabrication d Err ni les codes de sortie (task-31), ni la propagation de bout en bout.

Fossil: bdd9d01747c7cc141d36bca75a5da66dd654f9f7
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [ ] #1 patterns: syntaxe des patterns fixee et documentee (au minimum Ok/Err, etendu aux enums plus tard)
- [ ] #2 AST: representation unique du matching, partagee par match et when; la difference est portee par la semantique, pas par la forme
- [ ] #3 checker: typage des patterns + verification d exhaustivite, commun a match et when
- [ ] #4 checker: when conserve sa semantique liee a later (matching differe), match effectue un matching immediat
- [ ] #5 checker: aucune distinction de resolution (pending/succes/echec) exposee comme valeur ou cas du langage
- [ ] #6 grammar.ebnf: bump de version + production match et patterns, avec renvoi a la section Pattern matching de ARCHITECTURE.md
- [ ] #7 tests: fixtures quand une valeur Result est matchee par match ET par when, etant donnees que le resultat est identique
- [ ] #8 ARCHITECTURE.md: la section existante est completee par le comportement reellement implemente
- [ ] #9 codegen: lowering partage, seule la temporite differe; le lowering de when herite de TASK-30 est repris
<!-- AC:END -->
