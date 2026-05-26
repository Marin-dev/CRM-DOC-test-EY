# RetroDoc Kit — Règles Projet (FR)

## Objectif
Générer une rétro-documentation complète, à jour, et vérifiable depuis le code :
- Doc technique : architecture, modules, composants, dépendances
- Doc fonctionnelle : parcours / flows bout en bout
- Doc API : endpoints, authentification, payloads
- Doc Backend : controllers, services, logique métier
- Doc Data : ERD, tables, relations, événements, topics
- Diagrammes : Mermaid + Draw.io
- Patterns d'intégration : type, format, protocole, fonctionnement pas à pas

## Périmètre d'écriture
Écrire **uniquement** sous `docs/retrodoc/**`. Ne jamais modifier le code source applicatif.

## Règles de véracité (anti-hallucination)
- Ne jamais inventer endpoints, tables, events, env vars, dépendances, classes, méthodes.
- Toute affirmation doit être traçable à une preuve : `fichier:ligne` + symbole + snippet court.
- Si incertain : marquer `INCONNU` + lister la preuve manquante dans une section "À investiguer".
- Les diagrammes doivent contenir des nœuds `INCONNU` si besoin plutôt que d'inventer.
- Quand un nom existe dans le code mais que le comportement n'est pas évident : citer la preuve, ne pas extrapoler.

## Style & format
- Documentation en **français**
- Markdown pour les pages
- Mermaid dans des blocs ` ```mermaid `
- Draw.io : fichiers `.drawio` (XML) dans `docs/retrodoc/diagrams/`
- Liens relatifs entre pages
- Tableaux préférés pour endpoints, colonnes DB, env vars

## Workflow standard
1. **Discovery** (Reader) — inventaire, entrypoints, tech stack
2. **Dépendances & flows** (Searcher) — preuve par preuve
3. **Rédaction docs** (Writer) — templates FR
4. **Diagrammes** (Diagrams) — Mermaid + Draw.io
5. **Vérification** (Verifier) — rapport + corrections
6. **Publication** — README index + navigation + COVERAGE.md à jour

## Définition de fini
Un nouvel arrivant peut, en lisant la doc seule :
- Exécuter le projet localement
- Comprendre 3 flows clés bout en bout
- Localiser un endpoint, un controller, une table dans le code
- Modifier 1 feature et savoir où chercher les impacts
- Identifier les zones `INCONNU` à creuser

## Exigences de couverture minimale (cf. COVERAGE.md)
La doc DOIT couvrir, quand l'élément existe dans le repo :
1. **Architecture** : diagramme (front/back/DB/infra), stack, patterns d'intégration
2. **API** : endpoints, authentification, payloads
3. **Backend** : controllers, services, dépendances, logique métier
4. **Base de données** : ERD, clés, relations
