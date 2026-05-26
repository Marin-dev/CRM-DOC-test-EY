# 10 — Vue d'ensemble architecturale

> Phase **Writer**. Synthèse haut niveau qui rassemble inventaire, dépendances et patterns d'intégration. Aucun élément ici n'est inventé ; toutes les briques sont prouvées dans [00_inventaire.md](00_inventaire.md), [01_dependances.md](01_dependances.md), [02_patterns_integration.md](02_patterns_integration.md).

## 1. Positionnement produit

SuiteCRM 7.15.1 est une application CRM PHP « full-stack monolithe » : un seul codebase qui sert (a) une UI web rendue serveur (Smarty), (b) plusieurs APIs (V8 JSON:API + REST historiques + SOAP + JSON-RPC), (c) une file de jobs CLI (cron), (d) des intégrations e-mail entrant/sortant, calendrier, IdP et moteur de recherche.

> *« SuiteCRM 7 is a mature and stable CRM with a large community and regular releases »* — [SuiteCRM/README.md:26](../../SuiteCRM/README.md#L26).

## 2. Vue C4 — Contexte

Voir le diagramme dans [diagrams/mermaid_c4.md](../diagrams/mermaid_c4.md). Acteurs principaux et systèmes externes prouvés :

| Acteur / système | Rôle | Preuve |
|---|---|---|
| Utilisateur final (commercial, support, admin) | Consomme l'UI Smarty et l'API V8 | [include/MVC/SugarApplication.php:58](../../SuiteCRM/include/MVC/SugarApplication.php#L58) |
| Client API externe (mobile, intégrateur) | Appelle `/Api/V8/*` via OAuth2 | [Api/V8/Config/routes.php:12-131](../../SuiteCRM/Api/V8/Config/routes.php#L12-L131) |
| IdP SAML | SSO entrant | [modules/Users/authentication/SAML2Authenticate/SAML2Authenticate.php:71](../../SuiteCRM/modules/Users/authentication/SAML2Authenticate/SAML2Authenticate.php#L71) |
| Annuaire LDAP / AD | Authentification | [modules/Users/authentication/LDAPAuthenticate/](../../SuiteCRM/modules/Users/authentication/LDAPAuthenticate/) |
| Serveur IMAP | Mails entrants | [composer.json:50](../../SuiteCRM/composer.json#L50), [modules/InboundEmail/InboundEmail.php:54](../../SuiteCRM/modules/InboundEmail/InboundEmail.php#L54) |
| Serveur SMTP | Mails sortants | [composer.json:58](../../SuiteCRM/composer.json#L58), [modules/OutboundEmailAccounts/](../../SuiteCRM/modules/OutboundEmailAccounts/) |
| Google Calendar | Sync calendrier | [composer.json:46,134-136](../../SuiteCRM/composer.json#L46-L136) |
| Cluster Elasticsearch | Indexation full-text | [composer.json:44](../../SuiteCRM/composer.json#L44), [lib/Search/ElasticSearch/](../../SuiteCRM/lib/Search/ElasticSearch/) |
| Web-to-Lead / Web-to-Person (formulaires tiers) | Captation prospects | [modules/Campaigns/WebToLeadCapture.php](../../SuiteCRM/modules/Campaigns/WebToLeadCapture.php), [entry_point_registry.php:61](../../SuiteCRM/include/MVC/Controller/entry_point_registry.php#L61) |
| Base MySQL/MariaDB (ou MSSQL) | Stockage canonique | [include/database/DBManagerFactory.php:60-126](../../SuiteCRM/include/database/DBManagerFactory.php#L60-L126), [README.md:37](../../SuiteCRM/README.md#L37) |

## 3. Vue C4 — Conteneurs

Conteneurs identifiés (au sens C4 — unités déployables ou processus distincts) :

1. **Web Server (Apache/IIS) + PHP-FPM**
   - Sert `index.php` (UI Smarty MVC), `Api/index.php` (Slim/OAuth2), `soap.php`, `json_server.php`, et les 55 entrypoints `?entryPoint=...` ([entry_point_registry.php:45-74](../../SuiteCRM/include/MVC/Controller/entry_point_registry.php#L45-L74)).
2. **Base de données** (MySQL ≥ recommandé)
   - Pilote choisi à l'exécution via `DBManagerFactory` ([include/database/DBManagerFactory.php:60-126](../../SuiteCRM/include/database/DBManagerFactory.php#L60-L126)).
3. **CLI Cron host** (système d'exploitation)
   - Crontab système → `php cron.php` ([cron.php:48-101](../../SuiteCRM/cron.php#L48-L101)) — peut être le même hôte que le web ou séparé (config-driven).
4. **Cluster Elasticsearch** *(optionnel mais référencé)*
   - Client : `Elasticsearch\Client` ([lib/Search/ElasticSearch/ElasticSearchEngine.php:46](../../SuiteCRM/lib/Search/ElasticSearch/ElasticSearchEngine.php#L46)).
5. **Stockage fichiers** (FS local)
   - `upload/` (uploads + cache), `cache_dir` du `$sugar_config` ([include/utils/file_utils.php:67-70](../../SuiteCRM/include/utils/file_utils.php#L67-L70)).

INCONNU : positionnement effectif des conteneurs (mono-VM, multi-VM, k8s…) — non spécifié dans ce repo.

## 4. Vue C4 — Composants (Web Server)

Composants applicatifs principaux au sein du conteneur PHP :

```
┌──────────────────────────────────────────────────────────────────────────┐
│                            PHP / SuiteCRM                                │
│                                                                          │
│  ┌──────────────────┐   ┌──────────────────────────────────────────────┐ │
│  │   UI / MVC       │   │           API HTTP (multiple stacks)         │ │
│  │   index.php      │   │                                              │ │
│  │                  │   │   /Api/index.php  → Slim 3 + OAuth2 + JSON:API│ │
│  │  SugarApplication│   │   /soap.php       → NuSOAP / PHP5Soap        │ │
│  │  SugarController │   │   /service/v*/    → REST historiques         │ │
│  │  SugarView/Smarty│   │   /json_server.php→ JSON-RPC                 │ │
│  │  Themes (SuiteP) │   │   ?entryPoint=X   → 55 endpoints utilitaires │ │
│  └──────┬───────────┘   └───────────────┬──────────────────────────────┘ │
│         │                               │                                │
│         ▼                               ▼                                │
│  ┌──────────────────────────────────────────────────────────────────────┐│
│  │  Domain : SugarBean + 120+ modules métier                            ││
│  │  (Accounts, Contacts, Leads, Opportunities, Cases, Bugs,             ││
│  │   Campaigns, Emails, Quotes, Invoices, Reports AOR_*, Workflow AOW_*,││
│  │   Knowledge Base AOK_*, Calendar, Schedulers, ACL, Users, ...)       ││
│  └──────┬─────────────────────────────────────────────────────────────┬─┘│
│         │ ACL / DynamicFields / Vardefs / TableDictionary             │  │
│         ▼                                                             ▼  │
│  ┌──────────────────────┐   ┌──────────────────────┐   ┌───────────────┐ │
│  │  Persistence         │   │  Search              │   │  Integrations │ │
│  │  DBManagerFactory    │   │  Elasticsearch       │   │  IMAP, SMTP,  │ │
│  │  Mysqli/Sqlsrv/...   │   │  AOD (Lucene)        │   │  Google API,  │ │
│  └──────────────────────┘   │  BasicSearch/SqlSearch│   │  SAML, LDAP,  │ │
│                             └──────────────────────┘   │  OAuth2 ext.  │ │
│                                                        └───────────────┘ │
│                                                                          │
│  ┌──────────────────────────────────────────────────────────────────────┐│
│  │  Job Queue : SugarCronJobs / SugarCronRemoteJobs (CLI cron.php)      ││
│  │  16 Schedulers OOTB → table `schedulers` + `job_queue`               ││
│  └──────────────────────────────────────────────────────────────────────┘│
└──────────────────────────────────────────────────────────────────────────┘
```

Preuves : voir tableaux dans [00_inventaire.md](00_inventaire.md) §4 et §8, [01_dependances.md](01_dependances.md) §3 et §4.

## 5. Vue dynamique — cycle requête / réponse

### 5.1 Requête UI Sugar (`index.php?module=Accounts&action=DetailView&record=...`)

```
Apache → PHP → include/MVC/preDispatch.php
            → include/entryPoint.php (bootstrap)
            → SugarApplication::execute()
                ├─ ControllerFactory::getController('Accounts')
                ├─ loadUser() + ACLFilter() + checkHTTPReferer()
                ├─ SugarThemeRegistry::buildRegistry()
                ├─ Controller::execute() → action 'DetailView'
                │     └─ ViewFactory → Smarty render (theme SuiteP)
                └─ sugar_cleanup()
```
Preuves : [index.php:45-52](../../SuiteCRM/index.php#L45-L52), [SugarApplication.php:74-103](../../SuiteCRM/include/MVC/SugarApplication.php#L74-L103).

### 5.2 Requête API V8 (`GET /Api/V8/module/Accounts/{id}`)

```
Apache → Api/index.php → Api/Core/app.php
                      ├─ ContainerLoader::configure() (DI Slim)
                      ├─ RouteLoader::configureRoutes($app)
                      └─ $app->run()
                            ├─ ResourceServerMiddleware (valide Bearer)
                            ├─ ModuleController::getModuleRecord
                            │     └─ ModuleService::getRecord
                            │           ├─ BeanManager::getBeanSafe(module, id)
                            │           ├─ bean->ACLAccess('view')
                            │           └─ getDataResponse(...) → JSON:API
                            └─ BaseController::generateResponse → 200 application/vnd.api+json
```
Preuves : [Api/index.php:1-5](../../SuiteCRM/Api/index.php#L1-L5), [Api/Core/app.php:20-26](../../SuiteCRM/Api/Core/app.php#L20-L26), [Api/V8/Controller/ModuleController.php:36-43](../../SuiteCRM/Api/V8/Controller/ModuleController.php#L36-L43), [Api/V8/Service/ModuleService.php:74-92](../../SuiteCRM/Api/V8/Service/ModuleService.php#L74-L92), [Api/V8/Controller/BaseController.php:19-34](../../SuiteCRM/Api/V8/Controller/BaseController.php#L19-L34).

### 5.3 Cron — exécution batch

Voir [02_patterns_integration.md §2.2](02_patterns_integration.md) et [flows/30_cron_scheduler.md](../flows/30_cron_scheduler.md).

## 6. Décisions structurantes lisibles dans le code

| Décision | Conséquence | Preuve |
|---|---|---|
| **Cohabitation de plusieurs APIs** (V8 OAuth2 + REST v2…v4_1 + SOAP + JSON-RPC) | Backward compat ; surface d'attaque accrue ; doc d'API par stack | Cohabitation visible : [Api/Core/Config/ApiConfig.php:7-14](../../SuiteCRM/Api/Core/Config/ApiConfig.php#L7-L14), [service/v4_1/registry.php:46-50](../../SuiteCRM/service/v4_1/registry.php#L46-L50), [soap.php:50-67](../../SuiteCRM/soap.php#L50-L67) |
| **Slim 3 + JSON:API** pour V8 | Spec REST formelle, content-type `application/vnd.api+json` | [Api/V8/Controller/BaseController.php:10](../../SuiteCRM/Api/V8/Controller/BaseController.php#L10) |
| **DI Container** pour V8 uniquement | UI Sugar reste 100% « SugarBean + factories » | [Api/V8/Config/services.php:8-28](../../SuiteCRM/Api/V8/Config/services.php#L8-L28) |
| **Multi-DB** (4 drivers) sélection runtime | Portabilité MySQL/MSSQL ; pas d'ORM tiers | [include/database/DBManagerFactory.php:78-103](../../SuiteCRM/include/database/DBManagerFactory.php#L78-L103) |
| **`SugarBean` comme racine du domaine** | Tout module hérite, ACL/DynamicFields uniformes | [data/SugarBean.php:62](../../SuiteCRM/data/SugarBean.php#L62) |
| **Configuration et clés en dehors du repo** | `config.php`, `oauth2_encryption_key`, RSA keys créés à l'installation | [middlewares.php:31-37](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L31-L37), [Api/Core/Config/ApiConfig.php:20-21](../../SuiteCRM/Api/Core/Config/ApiConfig.php#L20-L21) |
| **Cron piloté par DB** (table `schedulers` + `job_queue`) | Scaling vertical ; pas de broker externe | [cron.php:90-101](../../SuiteCRM/cron.php#L90-L101), [modules/Schedulers/Scheduler.php:84](../../SuiteCRM/modules/Schedulers/Scheduler.php#L84) |
| **Customisation par `custom/` + Studio + composer-merge-plugin** | Modifs persistent à travers upgrades | [composer.json:108-150](../../SuiteCRM/composer.json#L108-L150), [modules/TableDictionary.php:136-138](../../SuiteCRM/modules/TableDictionary.php#L136-L138) |

## 7. Capacités cross-cutting

- **ACL & rôles** : `modules/ACL/`, `modules/ACLActions/`, `modules/ACLRoles/`. Garde-fou systématique via `$bean->ACLAccess($action)` (preuves dans `ModuleService` cité plus haut).
- **Sécurité HTTP** : `SugarApplication::checkHTTPReferer()` ([SugarApplication.php:93](../../SuiteCRM/include/MVC/SugarApplication.php#L93)), CORS large pour API V8 ([Api/Core/app.php:3-5](../../SuiteCRM/Api/Core/app.php#L3-L5)), sanitization HTML par `voku/anti-xss` + HTMLPurifier ([composer.json:45,71](../../SuiteCRM/composer.json#L45-L71)).
- **Audit** : flag `audited => true` sur les vardefs (Accounts, Contacts, Leads, Opportunities…) ; table `*_audit`.
- **Workflow** : `AOW_*` ; **Reports** : `AOR_*` ; **Knowledge base** : `AOK_*`.
- **Multi-langue** : `include/Localization/`, `include/SugarObjects/LanguageManager.php`.
- **Logging** : `monolog/monolog ^3` ([composer.json:54](../../SuiteCRM/composer.json#L54)), `psr/log ^3.0` ([composer.json:60](../../SuiteCRM/composer.json#L60)), wrapper `LoggerManager` (utilisé par exemple dans `ModuleService` — [Api/V8/Service/ModuleService.php:22](../../SuiteCRM/Api/V8/Service/ModuleService.php#L22)).

## 8. Limites & zones d'incertitude

- L'API V8 utilise Slim 3 (`^3.8`) qui est en fin de vie chez le mainteneur (information non vérifiable depuis le repo seul → INCONNU à confirmer côté upstream).
- Les chemins « custom » (`custom/Extension/application/Ext/...`) sont vides dans ce repo : la composition réelle d'un déploiement client peut donc différer significativement.
- Pas d'observabilité applicative outillée détectée (pas de traces OTLP, pas de métrique Prometheus exposée).
- INCONNU : politique de rotation des clés OAuth2 RSA, gestion HA de l'index ES, sharding DB.

