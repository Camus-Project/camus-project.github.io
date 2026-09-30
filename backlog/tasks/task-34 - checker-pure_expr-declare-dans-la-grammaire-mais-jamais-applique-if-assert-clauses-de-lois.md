---
id: task-34
title: >-
  checker: pure_expr declare dans la grammaire mais jamais applique (if, assert,
  clauses de lois)
status: To Do
assignee: []
created_date: '2026-09-30 07:55'
updated_date: '2026-09-30 14:19'
labels:
  - camus-pl
dependencies:
  - task-32
  - task-14
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Constat verifie le 2026-09-30 par relecture du code, pas d apres la documentation.

CE QUI EST AFFIRME
grammar.ebnf:56-57: the condition is a pure_expr, the same category used by assert and by law clauses; a method call in the condition is rejected. La production pure_expr existe (grammar.ebnf:358).

CE QUI EST IMPLEMENTE
Rien: aucun controle de purete dans le checker (aucune occurrence de purity, side_effect, is_pure). Les trois emplacements qui devraient utiliser la categorie passent tous par le meme parseur general parse_expr_line: clause if (parse.rs:1242), assert (parse.rs:1304), requires/ensures/invariant (parse.rs:773).

Consequence: un appel de methode est accepte partout ou la documentation annonce une expression pure. La fixture good/if_statement.cam contient if task.is_valid(""), exactement le cas que la grammaire dit rejeter, et le checker l accepte (verifie: exit 0).

POURQUOI CE N EST PAS UNE REGRESSION
pure_expr a ete introduit avec TASK-14 et n a jamais eu d implementation. TASK-32 a etendu le concept a if sans le rendre reel. Il n y a pas de regression, mais une decision jamais executee, presentee comme acquise.

POURQUOI C EST IMPORTANT
La purete est le socle de la certification: une clause de loi pouvant appeler une methode impure peut dependre d un etat non specifie, et verus-gen recoit une expression dont il ne peut pas garantir la correction.

Fossil: 6424ea80ffb372f276a521855cc085a3e92ca75d
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [x] #1 design: decision humaine actee sur la categorie de purete (liste blanche syntaxique, ou semantique recursive verifiant que les predicats appeles sont purs) - pas de code avant
- [x] #2 design: decision humaine actee sur le sort des appels de predicats de spec functions, purs par construction ou verifies
- [x] #3 checker: un predicat de purete partage, utilise par les trois emplacements (condition if, assert, clauses de lois)
- [x] #4 checker: un appel de methode dans une de ces trois positions est refuse, avec diagnostic explicite
- [x] #5 grammar.ebnf: le statut de pure_expr est corrige: soit il est dit enforceable, soit sa portee est reecrite
- [ ] #6 verus-gen: une expression impure est refusee par le backend au lieu d etre transmise a Verus
- [x] #7 fixtures: la fixture good/if_statement.cam est reecrite pour ne pas placer un appel de methode en condition; une fixture bad/ dediee est ajoutee
- [x] #8 ARCHITECTURE.md: la sectionpure_expr decrit le comportement reellement applique
- [x] #9 tests: les trois emplacements (if, assert, lois) ont chacun un test d acceptation et un test de rejet
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
DECISION HUMAINE du 2026-09-30, puis implementation.

PORTEE REECRITE: les trois emplacements ne se valent pas.
- Clause de loi: EXIGENCE DE SOUTENETE. Une clause devient une obligation de preuve, donc un appel qui lit ou change un etat non specifier la rend vide. Le checker l applique, avec UNE seule regle (check_law_forms).
- Condition de if et assert: CONVENTION, plus une regle verifiee. Les deux sont evaluees a l execution (assert devient un debug_assert), donc un appel est attrape a l execution sans etre non sound. Les imposer coutait de l expressivite reelle pour aucun gain de surete.

AC1 (liste blanche syntaxique) et AC2 (spec purs par construction) sont tranchees comme suit: liste blanche syntaxique, et un appel a une spec function du struct est ADMIS tel quel. Pas d analyse recursive: la regle n admet que les deux builtins et les spec functions du struct, et la forme spec exige deja un unique return d une expression pure. Une verification recursive coutait plus et ne terminerait pas sur un graphe d appels cyclique.

AC3 est cochee avec une RESERVE importante: le predicat de purete est partage, mais il couvre le seul site ou il est une exigence de surete, la clause de loi. if et assert en sont explicitement exclus, par decision, et la doc le dit. Cocher AC3 tel quel, qui demande les trois emplacements, aurait ete faux.

AC4: un appel de methode dans une clause de loi est refuse, avec un diagnostic qui nomme le correctif (declarer une spec function). Un appel a une fonction ordinaire est refuse aussi.

AC5: la portee est reecrite, pas la promesse supprimee. grammar.ebnf passe en v1.0: la clause de loi reste un pure_expr et devient enforceable; if et assert passent a expr, avec une note qui dit pourquoi.

AC6: le backend ne decide plus de son cote. tr_cond n est plus une deuxieme liste blanche privee: il abaisse exactement ce que le checker admet. Il a meme ete etendu, parce qu il refusait jusqu ici tout appel sauf old et empty.

TROU FERMEE AU PASSAGE, dans emit_spec_fn: le corps d une spec function etait abaisse avec .ok(), et un echec devenait silencieusement la constante true. Toute loi appelant cette spec function etait alors trivialement satisfaite, sans aucun rapport. C est maintenant une erreur, avec le nom de la fonction et la cause.

CE QUI RESTE A VERIFIER, et que je ne pretend pas avoir fait: le typage Verus de l abaissement d un appel a une spec function. Aucun binaire verus n est disponible dans cet environnement, donc le texte genere est asserté mais jamais soumis au verificateur. La forme spec fn est donc PRISE PAR RECEPTION (recv: &Vault), avec &*self en requires et &*final(self) en ensures. A confirmer des que Verus est disponible.

FINDINGS CONCRETS RENCONTRES EN CHEMIN, a garder en tete:
- Une spec function ne porte PAS de ligne intention, seulement sa signature. Le parseur rejette toute autre ligne.
- Le parseur d expressions n a PAS de moins unaire sur une expression arbitraire, seulement sur un litteral numerique. -self.x est rejete; ecrire self.balance + self.limit >= 0.
- Une spec function du struct s appelle self.g(..) ou g(..): les deux passent. Une methode ordinaire s appelle aussi self.g(..), et c est la que la regule tranche.
- Les fixtures d un meme repertoire shares un espace de noms via le World: deux fixtures ne peuvent pas reutiliser un nom de struct. J ai perdu du temps sur une collision Account, signale ici pour eviter de le refaire.

AC7: la fixture good/if_statement.cam n a pas besoin d etre reecrite, puisque if n est plus soumis a une regle de purete. C est un point notable: la reecriture qu AC7 appelait disparait avec la decision, parce que l appel de methode qu elle visait a retirer est desormais legal. Les fixtures ajoutees sont 4 bad/ (appel de methode, appel a une fonction ordinaire, construction, litteral chaine) et 1 good/ (appel a une spec function).

AC9: les tests d acceptation et de rejet couvrent le site qui compte, les clauses de loi: 4 tests de rejet (un cas chacun) et 1 test d acceptation (appel a une spec function), plus 1 test verus-gen qui asserte la forme generee pour requires et pour ensures. Pour if et assert il n y a PAS de test de rejet de purete, par decision: aucun n existe a ecrire. Ce qui est teste pour eux reste le type Bool de la condition, deja couvert ailleurs.

AC6 LAISSEE OUVERTE, et c est volontaire: elle est supersedee par la decision, pas satisfaite telle quelle.

AC6 demandait que verus-gen refuse une expression impure. Ce n est plus le role du backend: le refus est fait par le checker, plus tot et avec un meilleur diagnostic, donc la garantie est plus forte, mais elle n est plus portee par verus-gen. Cocher AC6 affirmerait que le backend refuse encore, ce qui est faux.

Il reste un cas ou le backend refuse: un assert que Verus ne sait pas traduire. Ce n est pas une regle de purete, c est une limite de traduction, signalee comme telle. Un assert n est de toute facon pas une obligation de preuve.

Etat final des AC: 1,2,3,4,5,7,8,9 cochees. 6 supersedee, laissee ouverte avec cette note.

TEST QUI FIGE LA DECISION: good/if_assert_need_not_be_pure.cam accepte un appel de methode dans une condition de if ET dans un assert. Si quelqu un remet une regle de purete la, ce test echoue et le choix devra etre refait volontairement. C est la seule maniere honnete de garder une decision qui refuse de contraindre.

Etat de fin de session: 8/9. AC6 reste ouverte et le reste volontairement: elle est supersedee (le refus est porte par le checker, plus par verus-gen), et l ouvrir evite d affirmer que le backend refuse encore.

Ce qui reste reellement a faire sur cette tache, et qui n est pas une ecriture de ticket: verifier le typage Verus de l abaissement d un appel a une spec function. Aucun binaire verus n est disponible dans cet environnement. C est le meme manque que pour 9cca051f, et c est ce manque qui avait laisse passer une obligation de preuve silencieusement fausse.
<!-- SECTION:NOTES:END -->
