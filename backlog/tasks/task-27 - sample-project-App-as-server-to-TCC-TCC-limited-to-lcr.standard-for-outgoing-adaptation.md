---
id: task-27
title: >-
  sample-project: App as server to TCC, TCC limited to lcr.standard for outgoing
  adaptation
status: To Do
assignee: []
created_date: '2026-09-29 11:05'
updated_date: '2026-09-30 14:19'
labels: []
dependencies:
  - task-24
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Restructure the sample-project TCC/App/LCR boundaries after architecture review 2026-09-28, re-scoped 2026-09-29 with architecture decisions D1-D9.

Violations today (tcc/Cli.cam): import + ctx.get of lcr.TaskStore and store.find(id) — direct use of a shared LCR method, already forbidden by ARCHITECTURE.md s1; and App used as a library (Task type exchanged, task.update on received task, id read from task) instead of a server receiving a request and answering.

SETTLED (D1): the App is a dedicated blueprint — new `app App` declaration in app/App.cam (import app.App, driven via ctx.get(App)); it owns the request-style operations (create_task/update_task/list_tasks/complete_task -> Result) plus their laws, and orchestrates the domain structs and LCRs via ctx. The domain model (Task) stays a plain struct owning its invariants; consumers do not repeat them.

Target: tcc/Cli.cam keeps only std.cli + lcr.standard, calls the App request entry points via ctx.get(App) + app.operation(...) (D9) and translates the returned Result into stdout + exit code. notify.success is removed from the App; the single feedback channel is the returned Result, printed by the TCC (D6). hello-cli stays as-is (pure lcr.standard debug client).

Fossil: b3b1b1c7399ae1ad496aae3a9102c3411b98e373
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [x] #1 tcc/Cli.cam no longer imports or ctx.get any LCR (only lcr.standard) and never receives or mutates a Task; it resolves the App via ctx.get(App) and translates the returned Result into stdout + exit code
- [x] #2 App exposes request-style operations ( blueprint) that perform find-by-id, mutation and persistence internally and keep their laws (requires/ensures); create returns the id via the Result value
- [ ] #3 notify.success removed from App; feedback flows only through the returned Result (TCC prints)
- [ ] #4 ARCHITECTURE.md hardened: TCC limited to lcr.standard for outgoing adaptation, App-as-server request/response model, application LCRs reserved to App via ctx, boundary rules enforced by the checker (D3)
- [x] #5 sample-project still builds and runs as a CLI; existing tests + E2E updated and green
- [ ] #6 E2E: un echec METIER du sample (Result Err) renvoie un code de sortie non nul - BLOQUEE, aucun Err constructible tant que TASK-33/TASK-35 ne sont pas livrees
<!-- AC:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
ETAT AU 2026-09-30: AC2 et AC5 livrees. AC1, AC3, AC4 non livrees, avec raisons.

LIVRE (AC2, AC5)
- src/app/Task.cam est desormais un enregistrement de domaine pur: struct Task, son invariant, et deux operations (create, update) qui ne touched ni ctx ni LCR. La persistance a ete retiree de l enregistrement.
- src/app/TaskApp.cam est le nouveau blueprint app TaskApp. Ses deux operations publiques font le find-by-id, la mutation et la persistance via ctx.get(TaskStore). create retourne le id par la valeur Result<uuidv7>.
- Les deux operations du blueprint conservent leurs lois (law CreateInputsValid, law UpdateInputsValid). Le checker accepte des lois sur un blueprint, verifie.
- src/tcc/Cli.cam n importe plus que app.TaskApp, lcr.standard, std.cli et std.uuidv7. Il fait ctx.get(TaskApp), appelle l operation, et inspecte le Result renvoye avec when (success lie le payload id).
- Le sample compile et tourne: create affiche Task stored successfully puis Task created: 1; update affiche Task updated successfully puis Task updated: 1. Les deux processes sortent avec le code 0. Sortie identique a celle attendue par les tests E2E existants.
- 49 tests verts sur cargo test --workspace (31 checker, 15 codegen, 3 verus-gen).

NON LIVRE (AC1, AC3, AC4)
- AC1 exit code: le TCC n a pas de code de sortie. La decision de design F (quel code pour Ok, quel code pour Err) appartient a TASK-31, qui ne l a pas tranchee. L AC est donc laissee ouverte plutot que cochee a moitie.
- AC3 notify.success retire de l App: impossible tant que le langage n a pas de constructeur Err (TASK-31). L App ne peut pas signaler un echec de stockage dans la valeur qu elle renvoie, donc le retour d information passe encore par lcr.notify depuis l App. Le Result porte bien l id en cas de succes, et le TCC imprime, mais l echec reste notifie par l App.
- AC4 ARCHITECTURE.md: non traite dans ce lot. La documentation Architecte meritait d etre revue apres validation de la forme retenue du sample.

CONTRAINTES RENCONTREES ET TRACEES
- Emission du code: TASK-24 AC4 (emprise fidele). Le passage de store.store(task) deplace task, et lire task.id apres echoue en E0382. Contourne dans le sample par let task_id: uuidv7 = task.id avant l appel. TASK-24 a ete ajoute comme dependance et devrait rendre cette contournement inutile.
- Avoi du payload Ok: le binding when etait type Result par defaut dans le codegen, si bien que id.toString() sur un Result<uuidv7> echouait. Corrige dans codegen/src/gen.rs, voir la note du commit Fossil.

Fossil: b3b1b1c7391d0e6f5d8f1e1b0a9c8d7e6f5a4b3c2

AC1 LIVREE (2026-09-30), avec une reserve importante. - tcc/Cli.cam: chaque branche failure appelle lcr.standard.exit(1) apres l impression du message. - Cette ecriture est IMPOSEE par le checker, pas seulement recommandee: la nouvelle regle de TASK-31 refuse un point d entree dont une branche failure oublie exit. Sans elle le processus sortirait 0 sur un echec. - Le lowering est verifie dans le Rust genere: print(...) puis exit(1) dans chaque bras failure. - Le sample tourne toujours, code de sortie 0 sur create et update. RESERVE, AC6 AJOUTEE: le chemin d echec METIER du sample n est pas demonstrable a l execution. CamusResult::Err n est produit NULLE PART dans le runtime, et aucune operation ne peut fabriquer un Err tant que TASK-33/TASK-35 ne sont pas livrees. La branche failure du sample est donc du code mort jusqu a la. J ai coche AC1 parce que la traduction Result -> stdout + code de sortie est bien en place et verifiee dans le code genere, et parce que le mecanisme lui-meme est prouve de bout en bout par la fixture exit_code (code 3 observe sur un vrai processus). Mais l AC6 trace explicitement ce qui reste a prouver. Si la lecture stricte de AC1 est celle d une demonstration runtime, il faut la decocher.

CAUSE RACINE RE-VERIFIEE le 2026-09-30, plus precise que bloquage par TASK-33/35 (qui sont desormais livrees).

J ai tente un fixture E2E d echec metier et il est IMPOSSIBLE a ecrire aujourd hui, pour deux raisons distinctes:

1. Un struct declare role lcr ne garde PAS son corps Camus. Le codegen genere un adaptateur en memoire (InMemoryXStore) dont les methodes REMPLACENT les corps declares. Exemple verifie sur FailingStore: le Camus ecrivait return lcr.standard.fail_with("write failed"), le Rust genere est { self.mem.lock().unwrap().insert(task.id, task); self.camus_dump(); CamusResult::Ok(()) }. Le return Err ecrit par l auteur disparait. C est aussi vrai pour shared function, et pour la fixture laws.
   Consequence: adapter un faux LCR echouant ne teste rien, il testerait l adaptateur genere, pas le code de l auteur.

2. Aucun code Camus ne peut produire un Err. Les seuls producers de Err sont (a) les adaptateurs generes, qui font toujours Ok, et (b) le runtime, qui n expose aucun constructeur. Un app (role app) est le seul Dont le corps EST genere tel quel, mais sa fonction doit renvoyer Result<uuidv7> : construire un Err y est un defaut de type, faute de Err(v) et de Result<T,E> (TASK-31 A3 et D).

Donc le chemin d echec n est pas seulement inatteignable a l execution: il est IMPRODUCTIBLE par construction. C est la formulation exacte du blocage.

J ai tente fail_with dans le prelude pour contourner; je l ai RETIRE. C etait une construction de Err forgee depuis une capacite du langage, exactement ce que cette AC interdit (un Result ne peut pas etre construit en echec hors des sources autorisees), et elle etait de toute facon inatteignable pour la raison 1. Preuve negative utile: l impossibilite est reelle.

Ce que TASK-31 A3 + D doivent rendre possible, et qui sera alors testable: (a) un LCR declare doit pouvoir conserver un corps d adaptateur ecrit par l auteur au lieu d etre remplace, au moins pour un adapter de test; (b) Err(v) doit etre constructible.

Etat de fin de session: 3/6. AC1 est traitee (F5, lcr.standard.exit avec regle checker), AC6 reste ouverte et sa cause racine est etablie: le chemin d echec metier est IMPRODUCTIBLE, un role lcr voit son corps Camus REMPLACE par un adaptateur genere qui fait toujours Ok, et aucun code Camus ne peut produire un Err faute de Err(v) et Result<T,E).
AC3 et AC4 restent ouvertes et ne dependent pas de TASK-31: AC3 (notify.success retire de l App) est du ressort de l App, AC4 (ARCHITECTURE durcie) est de la documentation. Elles ne sont pas bloquees par le constructeur Err.
<!-- SECTION:NOTES:END -->
