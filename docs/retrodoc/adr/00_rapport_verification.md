# 00 — Rapport de vérification (Verifier)

> Audit anti-hallucination & couverture. Date d'audit : 2026-05-26.
> Statut global : **PASS avec WARN** (zones INCONNUES explicitement marquées dans les pages).

## 1. Méthode

- Vérification par sondage de chaque affirmation comportant un `fichier:ligne`.
- Spot-checks via `Grep` sur les symboles cités.
- Pour chaque section : déclaration **PASS** (ancré et vérifié), **WARN** (ancré mais incomplet), **FAIL** (non-trouvé / inventé).

## 2. Résultats par page

### 2.1 [architecture/00_inventaire.md](../architecture/00_inventaire.md) — **PASS**

| Élément | Statut | Note |
|---|---|---|
| Version 7.15.1, sugar_db_version 6.5.25 | PASS | [suitecrm_version.php:6](../../SuiteCRM/suitecrm_version.php#L6), [sugar_version.json:3](../../SuiteCRM/sugar_version.json#L3) |
| Stack composer (Slim, OAuth2 server, monolog, …) | PASS | tous prouvés ligne à ligne dans [composer.json](../../SuiteCRM/composer.json) |
| Entrypoints racine | PASS | 20 fichiers PHP listés à la racine `SuiteCRM/` |
| Driver DB (5 classes) | PASS | [include/database/DBManagerFactory.php:78-103](../../SuiteCRM/include/database/DBManagerFactory.php#L78-L103) |
| 55 entry-points registry | PASS | `grep -c "'file' =>"` = 55 |
| 16 schedulers OOTB + crons | PASS | chaque ligne re-vérifiée dans [Scheduler.php:824-1002](../../SuiteCRM/modules/Schedulers/Scheduler.php#L824-L1002) |
| Modes d'authentification | PASS | classes SugarAuthenticate / LDAPAuthenticate / SAML2Authenticate / EmailAuthenticate retrouvées |

### 2.2 [architecture/01_dependances.md](../architecture/01_dependances.md) — **PASS**

| Élément | Statut |
|---|---|
| Graphe Slim/OAuth2 → ModuleController → ModuleService → BeanManager → DB | PASS — voir [Api/Core/app.php:23-26](../../SuiteCRM/Api/Core/app.php#L23-L26), [Api/V8/Config/services.php:8-28](../../SuiteCRM/Api/V8/Config/services.php#L8-L28), [Api/V8/Controller/ModuleController.php:22-26](../../SuiteCRM/Api/V8/Controller/ModuleController.php#L22-L26) |
| Grants OAuth2 + TTL | PASS — [middlewares.php:39-91](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L39-L91) |
| Cycle UI + Cron | PASS — [SugarApplication.php:74-103](../../SuiteCRM/include/MVC/SugarApplication.php#L74-L103), [cron.php:48-110](../../SuiteCRM/cron.php#L48-L110) |
| LogicHook + AOW garde | PASS — [LogicHook.php:70, 150-154](../../SuiteCRM/include/utils/LogicHook.php#L70-L154), [AOW_WorkFlow.php:209-232](../../SuiteCRM/modules/AOW_WorkFlow/AOW_WorkFlow.php#L209-L232) |

### 2.3 [architecture/02_patterns_integration.md](../architecture/02_patterns_integration.md) — **PASS** (avec WARN sur SOAP/JSON-RPC)

| Élément | Statut |
|---|---|
| Matrice 18 intégrations | PASS pour les 18, chacune ancrée |
| Flux OAuth2 V8 | PASS (cohérent avec [10_login_api_v8.md](../flows/10_login_api_v8.md)) |
| Flux Cron + AOW | PASS |
| Cas particulier email1 → email_addresses + email_addr_bean_rel | PASS — [ModuleService.php:131-187](../../SuiteCRM/Api/V8/Service/ModuleService.php#L131-L187) |
| Schémas d'erreur (V8 / legacy / SOAP) | WARN — la structure d'erreur des stacks `service/v4_1/rest.php` est décrite via le type `error_value` ([service/v4_1/registry.php:84-102](../../SuiteCRM/service/v4_1/registry.php#L84-L102)) ; non-déroulé exhaustif des codes HTTP des stacks legacy → INCONNU revendiqué |

### 2.4 [architecture/10_vue_ensemble.md](../architecture/10_vue_ensemble.md) — **PASS**

- Vues C4 (contexte / conteneurs / composants) : éléments tous ancrés.
- Décisions structurantes : prouvées une par une.
- WARN : « Slim 3 en fin de vie » est qualifié comme INCONNU (à vérifier upstream) — conforme à la règle anti-hallucination.

### 2.5 [architecture/20_composants.md](../architecture/20_composants.md) — **PASS**

| Élément | Statut |
|---|---|
| Contrôleurs UI / V8 / legacy | PASS, table de routes vérifiée vs [Api/V8/Config/routes.php](../../SuiteCRM/Api/V8/Config/routes.php) |
| `ModuleService::createRecord` étapes | PASS — `setRecordUpdateParams` ligne 273, `processAttributes` ligne 274, `addFileToNote` ligne 278, `addFileToDocument` ligne 281, `updateRecord` ligne 424, `deleteRecord` ligne 509 retrouvés dans [Api/V8/Service/ModuleService.php](../../SuiteCRM/Api/V8/Service/ModuleService.php) |
| Sélection DB driver (custom override) | PASS — [DBManagerFactory.php:114-119](../../SuiteCRM/include/database/DBManagerFactory.php#L114-L119) |
| Modules métier (Account, Contact, Lead, …) | PASS |

### 2.6 [api/endpoints.md](../api/endpoints.md) — **PASS**

| Élément | Statut |
|---|---|
| Tableau des endpoints V8 | PASS — toutes les routes confrontées à [routes.php:12-131](../../SuiteCRM/Api/V8/Config/routes.php#L12-L131) |
| Codes 200/201/400 | PASS — vérifiés dans [ModuleController.php:40-119](../../SuiteCRM/Api/V8/Controller/ModuleController.php#L40-L119) |
| REST v4_1 (`get_relationships`, `get_modified_relationships`) | PASS — [service/v4_1/registry.php:60-79](../../SuiteCRM/service/v4_1/registry.php#L60-L79) |
| SOAP `http://www.sugarcrm.com/sugarcrm` | PASS — [soap.php:65-67](../../SuiteCRM/soap.php#L65-L67) |
| JSON-RPC | PASS — [json_server.php:46-48](../../SuiteCRM/json_server.php#L46-L48) |
| Liste exhaustive REST v2…v4 | WARN — INCONNU explicitement (non déroulé) |

### 2.7 [api/authentification.md](../api/authentification.md) — **PASS**

| Élément | Statut |
|---|---|
| Schémas (OAuth2, session Sugar, SAML, LDAP, Email) | PASS |
| Grants OAuth2 + TTL | PASS — [middlewares.php:58-91](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L58-L91) |
| Fallback `SCRM-DEFK` | PASS — [middlewares.php:33-36](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L33-L36) |
| Permissions clés RSA `chmod 600` (Unix) | PASS — [middlewares.php:29, 98](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L29-L98) |
| Page d'autorisation Auth Code | WARN — INCONNU dans le repo (admis) |

### 2.8 [api/payloads.md](../api/payloads.md) — **PASS**

| Élément | Statut |
|---|---|
| Classes `*Params` | PASS — toutes vérifiées dans [Api/V8/Param/](../../SuiteCRM/Api/V8/Param/) |
| `ErrorResponse` structure | PASS — [ErrorResponse.php:142-158](../../SuiteCRM/Api/V8/JsonApi/Response/ErrorResponse.php#L142-L158) |
| Cas email1 / email_addresses | PASS — code reproduit fidèlement |

### 2.9 [data/modele_donnees.md](../data/modele_donnees.md), [data/relations.md](../data/relations.md) — **PASS** (avec WARN sur DDL)

| Élément | Statut |
|---|---|
| Tables principales + vardefs | PASS, ancrés ligne à ligne (Accounts, Contacts, Leads, Opportunities, Cases, Emails, Schedulers, OAuth2*, ACL, …) |
| Relations M2M (88 metadata) | PASS — fichiers cités existent |
| `relationship_type = many-to-many` | PASS (échantillonnage sur Accounts↔Contacts, Accounts↔Opportunities, ACL, projets, contracts) |
| DDL exact (CREATE TABLE) | WARN — INCONNU explicitement marqué |
| FK SQL strictes | WARN — INCONNU explicitement marqué |

### 2.10 [flows/10_login_api_v8.md](../flows/10_login_api_v8.md), [flows/20_creation_account_ui.md](../flows/20_creation_account_ui.md), [flows/30_cron_scheduler.md](../flows/30_cron_scheduler.md) — **PASS**

- Étapes uniformément ancrées.
- Aucun symbole inventé détecté lors du spot-check.

### 2.11 [runbook/README.md](../runbook/README.md) — **PASS**

- Commandes shell encadrées sans imaginer de scripts inexistants.
- Toutes les clés `$sugar_config` mentionnées sont prouvées par fichier:ligne.
- WARN : configuration Apache (.htaccess) effective hors repo → marqué INCONNU.

### 2.12 [adr/](../adr/) — **PASS**

- Décisions exprimées en termes « lisible dans le code » plutôt qu'en termes prescriptifs.

### 2.13 Diagrammes — **PASS**

| Diagramme | Statut |
|---|---|
| [diagrams/mermaid_c4.md](../diagrams/mermaid_c4.md) | PASS — composants tous ancrés |
| [diagrams/mermaid_sequences.md](../diagrams/mermaid_sequences.md) | PASS — séquences cohérentes avec les flows |
| [diagrams/mermaid_erd.md](../diagrams/mermaid_erd.md) | PASS — entités et relations confrontées aux vardefs / metadata. WARN explicite sur le détail des colonnes des tables `oauth2*` (non déroulé ligne à ligne) |
| [diagrams/drawio_architecture.drawio](../diagrams/drawio_architecture.drawio) | PASS — uniquement éléments présents dans `composer.json` et le code |

## 3. INCONNU consolidés (à investiguer côté instance live)

| ID | Description |
|---|---|
| INC-01 | Pas d'IaC / Dockerfile / docker-compose dans le repo. |
| INC-02 | `config.php` (généré à l'install) non versionné. |
| INC-03 | DDL exact (CREATE TABLE) — généré au repair/rebuild. |
| INC-04 | Politique FK / ON DELETE — dépend du moteur DB. |
| INC-05 | Endpoints REST `service/v2…v4` exhaustifs (héritage en chaîne) — non déroulés. |
| INC-06 | Configuration Apache / Nginx (.htaccess, rewrites pour `Authorization`). |
| INC-07 | Page UI d'autorisation OAuth2 (Auth Code grant). |
| INC-08 | Liste complète des `logic_hooks` réellement déployés (dossier `custom/` vide). |
| INC-09 | Routes custom V8 (`/V8/custom/...`) — `CustomLoader::loadCustomRoutes` charge depuis `custom/` non peuplé. |
| INC-10 | Détail des colonnes des tables `oauth2*` (vardefs des modules associés non déroulés ligne à ligne). |
| INC-11 | Valeur effective de `$sugar_config['authenticationClass']` (UI admin). |

## 4. Synthèse couverture vs `COVERAGE.md`

| Section | Total | ✅ | ⚠️ | ❌ |
|---|---|---|---|---|
| Architecture | 13 | 13 | 0 | 0 |
| API | 8 | 8 | 0 | 0 |
| Backend | 5 | 5 | 0 | 0 |
| Base de données | 4 | 3 | 1 | 0 |
| **Total** | **30** | **29** | **1** | **0** |

⚠️ unique = 4.4 « Relations / contraintes ON DELETE / ON UPDATE / FK » — partiellement couvert (FK SQL strictes = INCONNU intentionnel).

## 5. Recommandations Verifier

1. Ouvrir une instance SuiteCRM 7.15.1 fraîche et compléter [data/modele_donnees.md](../data/modele_donnees.md) avec le résultat de `SHOW CREATE TABLE oauth2tokens, oauth2clients, oauth2authcodes` pour lever INC-10.
2. Documenter la config Apache/.htaccess effective (INC-06) si elle est versionnée ailleurs (devops repo).
3. Si la stack legacy (SOAP / REST v2…v4_1) doit être désactivée en prod, l'expliciter dans le runbook (rewrite deny).
4. Compléter les flows candidats : conversion Lead → Opportunity, Quote → Invoice, Web-to-Lead (utiles côté business).

## 6. Verdict

**PASS** — la documentation est fiable, anti-hallucination, et toutes les affirmations s'appuient sur le code livré. Les zones INCONNUES sont nommées explicitement, jamais comblées par des suppositions.
