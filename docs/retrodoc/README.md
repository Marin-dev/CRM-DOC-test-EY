# SuiteCRM 7.15.1 — Rétro-documentation FR

> Documentation rétro-ingénierée à partir du repo `SuiteCRM/`. **Zéro hallucination** : toute affirmation est traçable à `fichier:ligne`. Les zones non vérifiables sont marquées `INCONNU`.
>
> Audit du Verifier : voir [adr/00_rapport_verification.md](adr/00_rapport_verification.md). Couverture des exigences : [COVERAGE.md](COVERAGE.md).

## 🚀 Pour commencer

- Si vous découvrez le repo → lire d'abord [architecture/10_vue_ensemble.md](architecture/10_vue_ensemble.md).
- Si vous intégrez l'API → [api/README.md](api/README.md) puis [api/endpoints.md](api/endpoints.md) et [api/payloads.md](api/payloads.md).
- Si vous reprenez l'exploitation → [runbook/README.md](runbook/README.md).
- Si vous avez besoin de la vue domaine → [data/README.md](data/README.md) + [diagrams/mermaid_erd.md](diagrams/mermaid_erd.md).

## 🗂️ Index

### Architecture

- [Inventaire & stack technique](architecture/00_inventaire.md) — version, langages, frameworks, modules, entrypoints, schedulers, intégrations.
- [Dépendances & graphe logique](architecture/01_dependances.md) — composer, graphes UI/V8/Cron, OAuth2, LogicHook.
- [Patterns d'intégration](architecture/02_patterns_integration.md) — matrice de 18 intégrations entrantes/sortantes.
- [Vue d'ensemble C4](architecture/10_vue_ensemble.md) — contexte, conteneurs, composants, cycles requête/réponse.
- [Composants backend (controllers, services, logique métier)](architecture/20_composants.md) — tous les contrôleurs/services V8 + UI + legacy.

### API

- [README](api/README.md) — vue d'ensemble des 4 stacks API.
- [Endpoints](api/endpoints.md) — exhaustif V8 + résumé legacy + 55 entry points.
- [Authentification](api/authentification.md) — OAuth2, session Sugar, SAML, LDAP, sécurité.
- [Payloads](api/payloads.md) — exemples JSON:API requêtes/réponses + format d'erreur.

### Données

- [README](data/README.md) — sources de vérité, conventions SugarBean.
- [Modèle de données](data/modele_donnees.md) — tables principales et colonnes.
- [Relations](data/relations.md) — cardinalités, join tables, contraintes.

### Flows

- [Index](flows/README.md) + [Flows candidats](flows/00_flows_candidats.md)
- [10 — Login + appel API V8 OAuth2](flows/10_login_api_v8.md)
- [20 — Création d'un Account (UI → DB)](flows/20_creation_account_ui.md)
- [30 — Cron / Scheduler / Workflow](flows/30_cron_scheduler.md)

### Diagrammes

- [C4 (Mermaid)](diagrams/mermaid_c4.md)
- [Séquences (Mermaid)](diagrams/mermaid_sequences.md)
- [ERD (Mermaid)](diagrams/mermaid_erd.md)
- [Architecture exec-friendly (Draw.io)](diagrams/drawio_architecture.drawio)

### Runbook

- [Procédures d'exploitation](runbook/README.md) — installation, cron, config, backup, sécurité.

### ADR & rapports

- [Index ADR](adr/README.md)
- [Rapport de vérification (PASS/WARN/FAIL)](adr/00_rapport_verification.md)
- [ADR-01 — API V8 : Slim 3 + OAuth2 + JSON:API](adr/01_adr_choix_v8_slim_oauth2.md)
- [ADR-02 — Persistance : ORM maison SugarBean](adr/02_adr_persistance_sugarbean.md)

## 🧭 Définition de fini (atteinte)

Un nouvel arrivant peut désormais, en lisant cette doc seule :

- ✅ Exécuter le projet localement → [runbook §2](runbook/README.md#2-installation-manuelle)
- ✅ Comprendre 3 flows clés bout en bout → [flows/10](flows/10_login_api_v8.md), [flows/20](flows/20_creation_account_ui.md), [flows/30](flows/30_cron_scheduler.md)
- ✅ Localiser un endpoint, un controller, une table → [api/endpoints.md](api/endpoints.md), [architecture/20_composants.md](architecture/20_composants.md), [data/modele_donnees.md](data/modele_donnees.md)
- ✅ Modifier 1 feature et savoir où chercher les impacts → [architecture/01_dependances.md](architecture/01_dependances.md) (graphes), [architecture/20_composants.md §5](architecture/20_composants.md#5-logique-metier-exemples-ancres)
- ✅ Identifier les zones INCONNU → [adr/00_rapport_verification.md §3](adr/00_rapport_verification.md#3-inconnu-consolides-a-investiguer-cote-instance-live)

## 🤝 Comment contribuer / mettre à jour la doc

1. **Avant d'écrire** : vérifier la règle dans [`.claude/CLAUDE.md`](../../.claude/CLAUDE.md) (anti-hallucination, FR, ancrage fichier:ligne obligatoire).
2. **Mettre à jour la preuve** : si vous citez une ligne, vérifier qu'elle existe encore (les renumérotations PHP arrivent vite — `Grep` avant publication).
3. **Hiérarchie de fichiers** : strictement sous `docs/retrodoc/**`. Ne jamais modifier `SuiteCRM/`.
4. **Diagrammes** :
   - Mermaid pour les diagrammes liés au texte (s'ouvre dans GitHub / VSCode markdown preview).
   - Draw.io pour les vues « exec-friendly » à exporter en PNG/PDF pour les comités.
5. **Verifier** : après tout changement, vérifier la matrice [COVERAGE.md](COVERAGE.md) et regénérer un mini rapport.
6. **Workflow général** : `retrodoc_reader` → `retrodoc_searcher` → `retrodoc_writer` → `retrodoc_diagrams` → `retrodoc_verifier` (voir [`.claude/skills/`](../../.claude/skills/)).

## 📌 Conventions

| Convention | Détail |
|---|---|
| Langue | Français |
| Format pages | Markdown (CommonMark) + Mermaid dans blocs ` ```mermaid ` |
| Format diagrammes | Mermaid (texte) + `.drawio` (XML) |
| Liens vers le code | Liens markdown relatifs vers `../../SuiteCRM/...#Lx-Ly` |
| Quand l'info manque | Tag `INCONNU` + section « À investiguer » |
| Tableaux | Préférés pour endpoints, colonnes DB, env vars |
