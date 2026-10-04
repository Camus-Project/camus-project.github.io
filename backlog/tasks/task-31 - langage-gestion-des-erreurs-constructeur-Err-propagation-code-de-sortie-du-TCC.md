---
id: TASK-31
title: >-
  langage: gestion des erreurs - constructeur Err, propagation, code de sortie
  du TCC
status: To Do
assignee: []
created_date: '2026-09-29 12:34'
updated_date: '2026-10-02 13:31'
labels:
  - camus-pl
dependencies:
  - task-30
  - task-33
  - task-35
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Tache de suivi de TASK-30 (Result generique). La question des erreurs n est pas videe: TASK-30 a rendu Result<T> utilisable mais n a pas resolu la fabrication ni la propagation d un echec.

Etat de l art:
- Result<T> existe (grammaire, checker, codegen) et porte un payload Ok
- when detruit une valeur Result (success lie le payload, failure lie le message)
- dans une fonction Result, return <expr> construit Ok; une valeur Result deja presente est forwardee
- if existe des TASK-32 (statement, else obligatoire)
- AUCUN constructeur Err: l application ne peut pas fabriquer un echec
- le runtime code en dur Err(String): CamusResult<T> n a qu un parametre
- aucun code de sortie: le processus sort toujours en 0

Consequences:
1. une operation App ne peut pas refuser une entree invalide
2. pas de propagateur ergonomique: un enchainement exige un when par etape
3. le TCC ne peut pas distinguer un echec metier d une panne d adaptateur
4. le code de sortie du processus, revendique par l architecture, n a aucun mecanisme

===================================================================
DECISIONS HUMAINES PRISES LE 2026-09-30
===================================================================

A. CONSTRUCTEUR D ERREUR -> A3, Err comme CONSTRUCTEUR DE VALEUR
Err est un constructeur ordinaire d une somme, au meme titre que Ok. Il ne
s agit PAS d un mot-cle de controle de flux (fail / error / raise):
  return Err(<expr>)
Raison: c est la seule option qui reste dans un seul mecanisme de matching.
Un mot-cle de faute ferait de l erreur un artefact de controle de flux, non
destructurable par le mecanisme unifie que TASK-33 met en place. A3 ne touche
pas non plus return, donc ni ends_with_return ni l analyse d ateignabilite.
Refute: A1 (aucun constructeur, l App ne peut pas refuser une entree) et A2
(un second chemin de sortie a raisonner dans le checker, verus-gen et le
lowering de if).

D. TYPE D ERREUR -> TYPE NOMME, pas String
Consequence directe: le runtime doit passer de CamusResult<T> a deux
parametres (Ok(T) / Err(E)), et la signature doit devenir Result<T, E>.
Refute: Err(String) conserve, qui ne verifie ni ne documente aucun contrat.

F. CODE DE SORTIE -> lcr.standard.exit(code), APPELE PAR LE TCC
Un code de sortie est une adaptation sortante. Il appartient donc a la capacite
du canal standard, pas au langage: aucune nouvelle construction, aucun
changement de signature, aucun retoucher de ends_with_return.
  when created
    success id
      lcr.standard.print("Task created: " + id.toString())
    failure error
      lcr.standard.print(error)
      lcr.standard.exit(1)
Refute: F1 (statu quo, le shell ne peut pas agir), F2 (l entry point derive 0/1,
qui impose de faire renvoyer un Result par toute entree TCC et perd la
distinction Unix), F3 (table de correspondance par issue, prematuree tant
qu aucun TCC n a deux codes), F4 (le code dans la valeur d erreur, qui laisse
l App dicter une convention de processus et contredit ARCHITECTURE.md section 9).

REGLE DU CHECKER ASSOCIEE A F (decidee en meme temps)
Une fonction public d un struct role tcc est un point d entree. TOUTE branche
failure d un when doit contenir un appel a lcr.standard.exit(<expr>).
Une operation App n est pas concernee: elle renvoie un Result, elle ne fixe
pas le code du processus.
Consequence assumee: un TCC ne peut pas traiter un echec de maniere recuperable
sans quitter le processus. C est la garantie demandee: impossible de sortir en 0
par oubli.
A noter: lcr.standard est aujourd hui un prelude code en dur (print,
print_success, print_error sont des fonctions litterales du Rust genere), donc
exit tient en quelques lignes. TASK-26 fera de lcr.standard un canal
reconfigurable; exit y trouvera sa place.

===================================================================
OBSTACLE BLOQUANT DECOUVERT EN VERIFIANT LES FAITS
===================================================================
La decision D n est pas realisable en l etat du langage. La grammaire ne declare
que deux formes de type:
  struct_decl = "struct" , identifier , ...
  app_decl    = "app" , identifier , ...
Il n existe NI enum, NI type, NI alias. Et type_reference n accepte qu un seul
parametre generique:
  type_reference = qualified_identifier | "Result" , "<" , type_reference , ">" ;
Donc: aucun type d erreur nomme ne peut etre DECLARE, et une signature
Result<T, E> ne peut pas etre ECRITE.

DECISION SOUS-JACENTE PRISE LE 2026-09-30 -> OPTION 2, GENERALISER L ENUM
L option 1 (declaration dediee error Name) et l option 3 (reutiliser struct)
sont ecartees. On generalise l enum, adosse au mecanisme de matching unifie.

Raison: tout l interet de A3 est que l erreur soit une VALEUR MATCHABLE. Un type
d erreur nomme qu on ne peut pas destructurer par when n est qu une etiquette.
C est l enum qui rend cela vrai.

Etat reel verifie dans le code avant decision:
- l AST n a AUCUN noeud Pattern. StmtKind::When est fige sur
  { handle: String, success: Branch, failure: Branch }: un handle, deux branches
  nommees, sans pattern.
- le parser appelle en dur parse_branch("success") puis parse_branch("failure").
- grammar.ebnf (lignes 8 a 36) dit deja que match et when sont deux formes d un
  seul mecanisme, et que le support success/failure livre en v0.6/v0.7 est
  PROVISOIRE et doit etre repris quand le mecanisme general arrivera.

Consequence: la chaine d implementation est imposee, dans cet ordre.
  1. declaration d enum + noeud Pattern (constructeur + liaison de variable)
  2. refonte de when en liste de branches (Pattern, Branch) + exhaustivite
  3. seulement ensuite Result<T, E> et Err(v) sur un type d erreur nomme
TASK-33 devient le VERROU de A et D: ce n est plus seulement du pattern matching,
c est un chantier d enums. Un decoupage en deux taches est propose.

============================================================
DECISION P2 PRISE LE 2026-09-30 : PAYLOADS SUR LES CAS
============================================================
Les cas d un enum PORTENT des donnees.
  enum TaskError {
    NotFound(uuidv7)
    InvalidName(String)
  }
P1 (tags nus) est ecarte: Err(NotFound) ne lierait rien, le TCC ne pourrait
pas afficher "tache 42 introuvable", et tout cas exigeant un contexte (entree
rejetee, identifiant, delai) serait inexprimable. Ce serait le probleme de
l ETIQUETTE rejete ci-dessus, deplace d un cran.
Consequence: P2 impose la syntaxe de construction et les sous-patterns dans
when, et rend le matching recursif.

============================================================
DECISION D EXHAUSTIVITE PRISE LE 2026-09-30
============================================================
when est EXHAUSTIF sur les cas de l enum matche, avec un catch-all EXPLICITE
(_) si l on veut une branche de repli.
Coherence: if impose deja else (TASK-32), le when actuel impose success et
failure. L exhaustivite contrainte est donc dans la discipline du langage.
Valeur de certification: ajouter un cas a l enum devient une ERREUR DE
COMPILATION partout ou l enum est matchee. C est precisement la propriete qui
rend un type d erreur nomme superieur a String.
Le catch-all est explicite et non un else implicite: un repli silencieux
annulerait le benefice.

C (propagateur) et E (metier vs technique) restent ouvertes.

Fossil: 44622fa8a5e072ee0d99272c5cf44bea6c2b8084
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [x] #1 design: decision humaine actee et documentee sur A (constructeur explicite ou valeur d erreur seulement) - pas de code avant
- [ ] #2 design: decision humaine actee et documentee sur C (propagateur), en tenant compte du mecanisme de matching unifie (TASK-33) et non plus d une alternative when contre match
- [x] #3 design: decision humaine actee et documentee sur D (type d erreur: Err(String) vs type nomme)
- [ ] #4 checker: un Result ne peut pas etre construit en echec hors des sources autorisees (LCR, runtime, constructeur si adoption)
- [ ] #5 grammar.ebnf: syntaxe de construction d erreur, si adoption (version bump + notes de version)
- [ ] #6 codegen: une valeur Err construite par l application genere CamusResult::Err(..) de facon sound (pas de Err<String> forge sur un Result<T> non String)
- [ ] #7 codegen: propagation d erreur (operateur ? ou forwarding explicite) avec lowering correct et test
- [ ] #8 sample-project: une operation App refuse effectivement une entree invalide et le TCC rend le code de sortie attendu
- [ ] #9 tests: fixtures checker (bon + mauvais), fixture codegen, E2E sur stdout ET code de sortie du processus
- [ ] #10 ARCHITECTURE.md: section gestion des erreurs (construction, propagation, codes de sortie) et regles checker a jour
- [x] #11 design: question de declaration d un type d erreur nomme TRANCHEE (enum generalise vs declaration dediee error vs struct) - elle bloque A et D, aucune implementation avant
- [x] #12 F: lcr.standard.exit(code) implemente dans le prelude, exitant par le processus, et REGLE CHECKER: toute branche failure d un point d entree (fonction public d un struct role tcc) doit contenir un appel a lcr.standard.exit
- [ ] #13 design: decision humaine actee et documentee sur E (erreur metier de l App vs erreur technique d adaptateur) - ENCORE OUVERTE
- [x] #14 P2: declaration d enum avec payload sur les cas, syntaxe de construction, et sous-patterns; fixtures checker bon et mauvais
- [x] #15 exhaustivite: when refuse un match qui ne couvre pas tous les cas de l enum (erreur de compilation), et accepte un catch-all explicite _ ; test des deux
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
F5 LIVRE (2026-09-30). - codegen/src/gen.rs: exit(code: i32) ajoute au prelude (le prelude est une chaine brute, pas du Rust: les guillemets doivent rester echappes). - gen.rs: lowering de lcr.standard.exit -> exit(<expr>), branche dediee dans le match des services natifs, a cote de print/notify.success/notify.error. - checker/src/check.rs: check_entry_point_failure_exits. Une fonction public d un struct role tcc est un point d entree; toute branche failure d un when y doit contenir un appel lcr.standard.exit, a n importe quelle profondeur de >. Les branches sont parcouries recursivement, donc un when imbrique est verifie lui aussi. Une operation App est exemptee: elle renvoie un Result et ne fixe pas le code du processus. - tests: fixture bad/tcc_failure_without_exit.cam + test checker (32 tests, +1). - E2E: fixture codegen exit_code + test lcr_standard_exit_yields_the_real_process_exit_code qui COMPILE le binaire genere, l EXECUTE, et verifie le code 3 choisi par le TCC puis 0 sur le chemin nominal (7 tests codegen, +1). - 51 tests verts sur cargo test --workspace (49 avant). - ARCHITECTURE.md section 9: sous-section exit codes, avec la regle du checker et la consequence assumee. CE QUI RESTE OUVERT: A et D (constructeur Err + type nomme) sont bloques par l option 2 (generaliser l enum), donc par TASK-33 puis TASK-35. C (propagateur) et E (metier vs technique) sont toujours ouverts. AC4 et AC10 non cochees: le Result peut encore etre construit en echec par l adaptateur, et la section ARCHITECTURE ne decrit que les codes de sortie, pas encore construction ni propagation.

AC14 et AC15 livrees: P2 (payloads sur cas d enum) implemente dans TASK-33, exhaustivite + catch-all implementes dans TASK-35. Les fixtures checker bon/mauvais couvrent les deux.

Etat de fin de session: 6/15. Ce qui est livre: F5 (exit + regle checker), la decision A3 (Err est un constructeur de valeur), la decision D (type d erreur nomme), le choix option 2 (enum generalise), P2 (payloads) par TASK-33, et l exhaustivite avec catch-all explicite par TASK-35.
Ce qui bloque le reste, et qui est etabli plutot que suppose: AC4, AC5, AC6, AC7 et AC9 dependent du constructeur Err et de Result<T,E>, absents. AC8 (E2E) depend de la meme chose: le chemin d echec est IMPRODUCTIBLE tant que rien ne produit Err, ce que TASK-27 AC6 consigne en detail.
AC2 (propagateur) et AC13 (erreur metier vs erreur technique) sont des questions de design qui attendent le decideur, et ne dependent d aucune ecriture de code.
Consequence a assumer: un Result ne peut aujourd hui pas etre construit en echec hors des adaptateurs generes, qui font toujours Ok.

REPARATION 2026-10-02, frontmatter duplique dans la Description. Cette tache avait sa section Description encadree par TROIS marqueurs SECTION:DESCRIPTION:BEGIN pour un seul END, et deux copies completes du frontmatter (id, title, status, dependencies) imbriquees avant le vrai texte. Consequence concrete et non cosmétique: backlog task view 31 présentait le YAML de la tache comme Description, et la description reelle (decisions A3/D/F, obstacle enums, P2, exhaustivite) ne commençait qu a la ligne 59 du fichier. C est le shifted content signale par la session du 2026-10-01, isole a cette tache: les 43 autres ont un seul marqueur BEGIN et un seul id. Repare via backlog task edit 31 --description, avec le texte extrait des lignes 59 a 193 du fichier corrompu. Verifie: un seul BEGIN et un seul END, description identique octet pour octet a l extrait, 15 criteres d acceptation intacts, notes intactes.

CASSURE DU CHAMP id, CONSTATEE ET NON CORRIGEE. Ce fichier porte desormais un id en majuscules alors que les 43 autres portent un id en minuscules. Ce n est pas une regression de la reparation mais une revelation: task_prefix ne gouverne QUE le nom de fichier, et la casse du champ id est ecrite par le CLI selon l option employee. L option --description ecrit une forme majuscule, les autres operations ecritent une forme minuscule. La casse n est donc pas stable sous les editions CLI, et la forcer a la main serait a la fois interdit (edition directe interdite) et futile (l edition suivante la relancerait). Consequence: aucune. La resolution est verifiee dans les deux sens: backlog task view 31 trouve la tache et resout ses dependances task-30, task-33 et task-35. A ne pas remettre en cause sans corriger d abord le CLI.

RESIDU DOCUMENTAIRE. Les references en majuscules qui subsistent dans les corps de tache ne sont pas des identifiants resolus, seulement de la prose. En revanche la revision Fossil 31a88c64 affirme que les 44 fichiers portent un id en minuscules, ce qui est devenu faux pour ce fichier. La correction est ici.
<!-- SECTION:NOTES:END -->
