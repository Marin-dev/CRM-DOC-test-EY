# 00 — Inventaire & stack technique

> Phase **Reader**. Faits issus directement du repo `SuiteCRM/`. Toute affirmation est ancrée par `fichier:ligne`.

## 1. Identité applicative

| Élément | Valeur | Preuve |
|---|---|---|
| Nom | SuiteCRM | [SuiteCRM/composer.json:2](../../SuiteCRM/composer.json#L2) |
| Version applicative | 7.15.1 | [SuiteCRM/suitecrm_version.php:6](../../SuiteCRM/suitecrm_version.php#L6) |
| Version « sugar_db_version » | 6.5.25 | [SuiteCRM/sugar_version.json:3](../../SuiteCRM/sugar_version.json#L3) |
| Licence | GPL-3.0 (composer) / AGPLv3 (README) | [SuiteCRM/composer.json:6](../../SuiteCRM/composer.json#L6), [SuiteCRM/README.md:90](../../SuiteCRM/README.md#L90) |
| Plate-forme cible runtime | PHP 8.1 – 8.4, Apache (recommandé) ou IIS | [SuiteCRM/README.md:34-37](../../SuiteCRM/README.md#L34-L37) |
| Base de données supportée | MySQL / MariaDB (recommandée) ou MSSQL | [SuiteCRM/README.md:37](../../SuiteCRM/README.md#L37) |
| Type composer | `project` | [SuiteCRM/composer.json:5](../../SuiteCRM/composer.json#L5) |

## 2. Stack technique

### 2.1 Langages

| Langage | Usage | Preuve |
|---|---|---|
| PHP ≥ 8.1 | Cœur applicatif, plateforme `^8.1` | [SuiteCRM/composer.json:21,35](../../SuiteCRM/composer.json#L21-L35) |
| JavaScript (vanilla + jQuery + YUI) | UI, AJAX, calendrier | [SuiteCRM/jssource/](../../SuiteCRM/jssource/), [SuiteCRM/include/javascript/](../../SuiteCRM/include/javascript/) |
| Smarty (templating) | Vues serveur | dépendance `smarty/smarty: ^4` ([composer.json:62](../../SuiteCRM/composer.json#L62)) |
| HTML / CSS (SCSS) | Thèmes | [SuiteCRM/themes/SuiteP/](../../SuiteCRM/themes/SuiteP/), [SuiteCRM/composer.json:94](../../SuiteCRM/composer.json#L94) (`scssphp/scssphp`) |
| XML | WSDL, métadonnées Studio | [SuiteCRM/service/core/WSDL.tpl](../../SuiteCRM/service/core/WSDL.tpl) |

### 2.2 Frameworks & librairies majeures (extrait composer.json)

| Lib | Version | Rôle | Preuve |
|---|---|---|---|
| `slim/slim` | ^3.8 | Routage de l'**API V8** (REST JSON:API) | [composer.json:61](../../SuiteCRM/composer.json#L61), [Api/Core/app.php:23](../../SuiteCRM/Api/Core/app.php#L23) |
| `league/oauth2-server` | ^8.5 | Serveur **OAuth2** (Authorization, Resource, Password/Client Credentials/RefreshToken/AuthCode grants) | [composer.json:53](../../SuiteCRM/composer.json#L53), [Api/V8/Config/services/middlewares.php:39-91](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L39-L91) |
| `league/oauth2-client` | ^2.6 | Client OAuth2 (intégrations externes) | [composer.json:52](../../SuiteCRM/composer.json#L52) |
| `smarty/smarty` | ^4 | Moteur de template serveur | [composer.json:62](../../SuiteCRM/composer.json#L62) |
| `monolog/monolog` | ^3 | Logging | [composer.json:54](../../SuiteCRM/composer.json#L54) |
| `phpmailer/phpmailer` | ^6 | Envoi e-mail | [composer.json:58](../../SuiteCRM/composer.json#L58) |
| `javanile/php-imap2` | ^0.1.10 | IMAP / inbound email | [composer.json:50](../../SuiteCRM/composer.json#L50) |
| `zbateson/mail-mime-parser` | ^2.4 | Parsing MIME | [composer.json:73](../../SuiteCRM/composer.json#L73) |
| `elasticsearch/elasticsearch` | ^7.13 | Indexation/recherche | [composer.json:44](../../SuiteCRM/composer.json#L44), [lib/Search/ElasticSearch/](../../SuiteCRM/lib/Search/ElasticSearch/) |
| `zf1s/zend-search-lucene` | ^1.15 | Index Lucene (AOD) | [composer.json:75](../../SuiteCRM/composer.json#L75), [modules/AOD_Index/](../../SuiteCRM/modules/AOD_Index/) |
| `tecnickcom/tcpdf` | ^6.10 | Génération PDF | [composer.json:68](../../SuiteCRM/composer.json#L68) |
| `tinymce/tinymce` | ^8 | Éditeur HTML WYSIWYG | [composer.json:70](../../SuiteCRM/composer.json#L70) |
| `nesbot/carbon` | ^3 | Dates | [composer.json:55](../../SuiteCRM/composer.json#L55) |
| `consolidation/robo` | ^4 | Task runner (CLI) | [composer.json:43](../../SuiteCRM/composer.json#L43), [RoboFile.php:8](../../SuiteCRM/RoboFile.php#L8) |
| `google/apiclient` | ^2.18 | Intégration Google Calendar | [composer.json:46](../../SuiteCRM/composer.json#L46), [composer.json:134-136](../../SuiteCRM/composer.json#L134-L136) |
| `google/recaptcha` | ^1.3 | reCAPTCHA | [composer.json:47](../../SuiteCRM/composer.json#L47) |
| `onelogin/php-saml` | ^4 | Auth SAML2 | [composer.json:57](../../SuiteCRM/composer.json#L57), [modules/Users/authentication/SAML2Authenticate/SAML2Authenticate.php:71](../../SuiteCRM/modules/Users/authentication/SAML2Authenticate/SAML2Authenticate.php#L71) |
| `voku/anti-xss`, `ezyang/htmlpurifier` | ^4 / ^4.10 | Sanitization HTML | [composer.json:45,71](../../SuiteCRM/composer.json#L45-L71) |
| `symfony/options-resolver`, `symfony/validator`, `symfony/yaml`, `symfony/polyfill-uuid` | ^6.4 / ^1.32 | Utilitaires | [composer.json:64-67](../../SuiteCRM/composer.json#L64-L67) |
| `wikimedia/composer-merge-plugin` | ^2 | Merge `composer.ext.json` + extensions Studio | [composer.json:72](../../SuiteCRM/composer.json#L72), [composer.json:137-149](../../SuiteCRM/composer.json#L137-L149) |
| `nikic/php-parser` | ^5.7 | Analyse statique de code | [composer.json:56](../../SuiteCRM/composer.json#L56) |
| `tedivm/jshrink` | ^1.3 | Minification JS | [composer.json:69](../../SuiteCRM/composer.json#L69) |

Extensions PHP requises : `curl`, `gd`, `intl`, `json`, `openssl`, `zip` ([composer.json:36-41](../../SuiteCRM/composer.json#L36-L41)). Extension *suggérée* : `imap` (module Emails) ([composer.json:32](../../SuiteCRM/composer.json#L32)).

### 2.3 Outillage qualité & tests

| Outil | Usage | Preuve |
|---|---|---|
| `phpunit/phpunit` ^10.5 | Tests unitaires | [composer.json:93](../../SuiteCRM/composer.json#L93), [tests/unit/phpunit/](../../SuiteCRM/tests/unit/phpunit/) |
| `codeception/codeception` ^5.2 + suites `acceptance`, `api`, `install` | Tests acceptance/api/install | [composer.json:79](../../SuiteCRM/composer.json#L79), [tests/acceptance.suite.yml](../../SuiteCRM/tests/acceptance.suite.yml), [tests/api.suite.yml](../../SuiteCRM/tests/api.suite.yml), [tests/install.suite.yml](../../SuiteCRM/tests/install.suite.yml), [tests/codeception.dist.yml:1-13](../../SuiteCRM/codeception.dist.yml#L1-L13) |
| `mockery/mockery`, `mikey179/vfsstream`, `fakerphp/faker` | Mocking & VFS | [composer.json:91-92](../../SuiteCRM/composer.json#L91-L92) |
| `friendsofphp/php-cs-fixer` ^3.90 | Style PHP | [composer.json:89](../../SuiteCRM/composer.json#L89) |
| PHP_CodeSniffer (`phpcs.xml`) | Conformité PSR-2 (avec exclusions) | [phpcs.xml:1-7](../../SuiteCRM/phpcs.xml#L1-L7) |
| `browserstack/browserstack-local` | Tests BrowserStack | [composer.json:78](../../SuiteCRM/composer.json#L78) |
| `filp/whoops` | Pages d'erreur dev | [composer.json:87](../../SuiteCRM/composer.json#L87) |
| Codacy | Analyse statique (exclusions définies) | [codacy.yml](../../SuiteCRM/codacy.yml) |
| `tests/runtests.sh` | Lanceur tests | [tests/runtests.sh](../../SuiteCRM/tests/runtests.sh) |

### 2.4 Infrastructure / déploiement

Pas de Dockerfile ni `docker-compose.yml` détecté à la racine `SuiteCRM/`. INCONNU : pas de manifest IaC (Terraform / k8s) dans le repo.

Déploiement type d'après le README : LAMP classique (Apache + PHP + MySQL/MariaDB), self-hosted ou hébergement managé SuiteCRM Ltd ([SuiteCRM/README.md:32](../../SuiteCRM/README.md#L32)).

Couche autoloader (PSR-4) :

```json
"SuiteCRM\\":         ["lib/", "include/"]
"SuiteCRM\\Custom\\": ["custom/lib"]
"SuiteCRM\\Modules\\": ["modules/"]
"classmap":            ["Api/"]
```

Preuve : [composer.json:97-116](../../SuiteCRM/composer.json#L97-L116).

## 3. Topologie de répertoires (cartographie haut niveau)

| Répertoire | Rôle | Preuve |
|---|---|---|
| `Api/` | API V8 (Slim + JSON:API + OAuth2) — point d'entrée HTTP secondaire | [Api/index.php](../../SuiteCRM/Api/index.php), [Api/Core/app.php](../../SuiteCRM/Api/Core/app.php), [Api/V8/Controller/](../../SuiteCRM/Api/V8/Controller/) |
| `soap/`, `service/` | Stacks SOAP (NuSOAP/PHP5Soap) + APIs REST historiques v2…v4_1 + JSON-RPC | [service/v4_1/](../../SuiteCRM/service/v4_1/), [service/core/SoapHelperWebService.php](../../SuiteCRM/service/core/SoapHelperWebService.php), [service/JsonRPCServer/](../../SuiteCRM/service/JsonRPCServer/) |
| `modules/` | 120+ modules métier (Accounts, Contacts, Leads, Opportunities, Cases, Bugs, Campaigns, Emails, Quotes, Invoices, Workflow `AOW_*`, Reports `AOR_*`, Knowledge Base `AOK_*`, etc.) | [modules/](../../SuiteCRM/modules/) (123 sous-dossiers) |
| `include/` | Framework Sugar (MVC, ACL, DB, Smarty, SugarObjects, ResourceManager, LogicHook, JSON helpers…) — 108 sous-dossiers | [include/](../../SuiteCRM/include/), [include/MVC/SugarApplication.php:58](../../SuiteCRM/include/MVC/SugarApplication.php#L58) |
| `data/` | Modèle de données : `SugarBean.php`, `BeanFactory.php`, `Link.php`, `Link2.php`, `Relationships/*` | [data/SugarBean.php:62](../../SuiteCRM/data/SugarBean.php#L62), [data/Relationships/](../../SuiteCRM/data/Relationships/) |
| `metadata/` | 80+ tables de relations (M2M) — *« metadata »* SugarCRM (à ne pas confondre avec les vardefs) | [metadata/](../../SuiteCRM/metadata/) (88 fichiers `*MetaData.php`), [modules/TableDictionary.php:45-138](../../SuiteCRM/modules/TableDictionary.php#L45-L138) |
| `install/` | Installeur web + utilitaires CLI | [install.php](../../SuiteCRM/install.php), [install/](../../SuiteCRM/install/) |
| `themes/SuiteP/`, `themes/default/` | Thèmes UI (Smarty + CSS/JS) | [themes/SuiteP/](../../SuiteCRM/themes/SuiteP/), [themes/SuiteP/themedef.php:50-54](../../SuiteCRM/themes/SuiteP/themedef.php#L50-L54) |
| `lib/` | Composants partagés : Search/Elasticsearch, PDF, Robo, Utility, Exception, API, Log | [lib/](../../SuiteCRM/lib/) |
| `tests/` | Suites PHPUnit et Codeception | [tests/](../../SuiteCRM/tests/) |
| `ModuleInstall/`, `custom/`, `upload/`, `build/`, `Zend/`, `XTemplate/`, `jssource/` | Installation de modules, customisations, uploads, build, dépendances historiques Zend, templating legacy | [ModuleInstall/](../../SuiteCRM/ModuleInstall/), [Zend/](../../SuiteCRM/Zend/) |

## 4. Entrypoints (Web + CLI)

### 4.1 Entrypoints PHP racine

Tous les fichiers PHP à la racine `SuiteCRM/` exigent `define('sugarEntry', true)` et `include('include/entryPoint.php')` (bootstrap commun).

| Fichier | Type | Rôle | Preuve |
|---|---|---|---|
| [index.php](../../SuiteCRM/index.php) | HTTP | Front-controller MVC Sugar — instancie `SugarApplication`, session, dispatch module/action | [index.php:41-52](../../SuiteCRM/index.php#L41-L52), [include/MVC/SugarApplication.php:58,74-103](../../SuiteCRM/include/MVC/SugarApplication.php#L58-L103) |
| [Api/index.php](../../SuiteCRM/Api/index.php) | HTTP | Entrée de l'**API V8** ; charge `Api/Core/app.php` (Slim) puis `$app->run()` | [Api/index.php:1-5](../../SuiteCRM/Api/index.php#L1-L5), [Api/Core/app.php:23-26](../../SuiteCRM/Api/Core/app.php#L23-L26) |
| [soap.php](../../SuiteCRM/soap.php) | HTTP | Serveur **SOAP** (NuSOAP) historique, namespace `http://www.sugarcrm.com/sugarcrm` | [soap.php:41-67](../../SuiteCRM/soap.php#L41-L67) |
| [json_server.php](../../SuiteCRM/json_server.php) | HTTP | Serveur **JSON-RPC** Sugar | [json_server.php:42-48](../../SuiteCRM/json_server.php#L42-L48), [service/JsonRPCServer/](../../SuiteCRM/service/JsonRPCServer/) |
| [install.php](../../SuiteCRM/install.php) | HTTP | Installeur web | [install.php:41-42](../../SuiteCRM/install.php#L41-L42) |
| [cron.php](../../SuiteCRM/cron.php) | CLI | Boucle Cron : exécute le driver `SugarCronJobs` (par défaut) sur la file `job_queue` | [cron.php:48-52, 90-101](../../SuiteCRM/cron.php#L48-L101) |
| [run_job.php](../../SuiteCRM/run_job.php) | CLI | Exécution d'un job de scheduler ciblé | [run_job.php](../../SuiteCRM/run_job.php) |
| [emailmandelivery.php](../../SuiteCRM/emailmandelivery.php) | CLI | Distribution e-mailing campagnes | [emailmandelivery.php](../../SuiteCRM/emailmandelivery.php) |
| [campaign_tracker.php](../../SuiteCRM/campaign_tracker.php) | HTTP | Tracker historique de campagnes | [campaign_tracker.php](../../SuiteCRM/campaign_tracker.php) |
| [download.php](../../SuiteCRM/download.php), [export.php](../../SuiteCRM/export.php), [pdf.php](../../SuiteCRM/pdf.php), [vCard.php](../../SuiteCRM/vCard.php), [ical_server.php](../../SuiteCRM/ical_server.php), [vcal_server.php](../../SuiteCRM/vcal_server.php) | HTTP | Endpoints utilitaires (téléchargement / export / PDF / vCard / iCal / vCal) | racine [SuiteCRM/](../../SuiteCRM/) |
| [HandleAjaxCall.php](../../SuiteCRM/HandleAjaxCall.php), [TreeData.php](../../SuiteCRM/TreeData.php) | HTTP | AJAX endpoints internes | racine [SuiteCRM/](../../SuiteCRM/) |
| [maintenance.php](../../SuiteCRM/maintenance.php), [php_version.php](../../SuiteCRM/php_version.php) | HTTP | Mode maintenance / vérif version PHP | racine [SuiteCRM/](../../SuiteCRM/) |
| [SugarSecurity.php](../../SuiteCRM/SugarSecurity.php) | bootstrap | Garde de sécurité avant entrée | racine [SuiteCRM/](../../SuiteCRM/) |

### 4.2 Entrypoints « legacy » via registry

Le fichier [include/MVC/Controller/entry_point_registry.php](../../SuiteCRM/include/MVC/Controller/entry_point_registry.php) déclare **55** entrées (`grep -c "'file' =>"`). Chacune mappe un identifiant `?entryPoint=X` vers un fichier PHP + un flag `auth` :

Exemples : `image`, `campaign_trackerv2`, `WebToLeadCapture`, `WebToPersonCapture`, `ConfirmOptIn`, `acceptDecline`, `SAML`, `download`, `export`, `pdf`, `vCard`, `json_server`, `HandleAjaxCall`, `Changenewpassword`, etc. ([entry_point_registry.php:45-74](../../SuiteCRM/include/MVC/Controller/entry_point_registry.php#L45-L74)).

`SugarApplication::execute()` reconnaît `$_REQUEST['entryPoint']` et applique la règle d'auth en conséquence ([SugarApplication.php:87-94](../../SuiteCRM/include/MVC/SugarApplication.php#L87-L94)).

### 4.3 Routage API V8 (Slim)

Routes exposées sous `/Api/V8/...` ([Api/V8/Config/routes.php:19-131](../../SuiteCRM/Api/V8/Config/routes.php#L19-L131)). Authentification : OAuth2 (ResourceServerMiddleware) — sauf `/access_token` qui passe par le `AuthorizationServerMiddleware`. Détail complet → voir [api/endpoints.md](../api/endpoints.md).

### 4.4 APIs « legacy » service

| Stack | Localisation | Notes |
|---|---|---|
| REST v2 → v4_1 | `service/v2/`, `service/v2_1/`, `service/v3/`, `service/v3_1/`, `service/v4/`, `service/v4_1/` | Endpoint `service/v4_1/rest.php` / `soap.php`, registres `registry.php` héritant les uns des autres ([service/v4_1/registry.php:46-50](../../SuiteCRM/service/v4_1/registry.php#L46-L50)) |
| JSON-RPC | `service/JsonRPCServer/` | Exposé via `json_server.php` |
| SOAP | `service/core/SoapHelperWebService.php`, `service/core/NusoapSoap.php`, `service/core/PHP5Soap.php`, `service/core/WSDL.tpl` | Exposé via `soap.php` |
| SOAP REST helpers | `service/core/REST/` | Sérialisations REST historiques |

## 5. Configuration applicative

| Fichier | Rôle | Preuve |
|---|---|---|
| `config.php` (généré à l'install) | `$sugar_config` (db, site_url, cache_dir, cron, oauth2_encryption_key, etc.) | référencé par [include/utils/file_utils.php:67-70](../../SuiteCRM/include/utils/file_utils.php#L67-L70), [Api/V8/Config/services/middlewares.php:31-37](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L31-L37) ; non présent dans le repo (créé à l'installation) |
| `config_si.php` (silent install) | Préfill d'installation, fusionné dans `config.php` | [install/install_utils.php:936-939, 1653-1654](../../SuiteCRM/install/install_utils.php#L936-L1654) |
| `Api/V8/OAuth2/private.key` / `public.key` | Clés RSA OAuth2 (chemins constants) | [Api/Core/Config/ApiConfig.php:20-21](../../SuiteCRM/Api/Core/Config/ApiConfig.php#L20-L21) |
| `Api/V8/Config/services.php` et services/* | Container DI Slim | [Api/V8/Config/services.php:8-28](../../SuiteCRM/Api/V8/Config/services.php#L8-L28) |
| `composer.ext.json` + `custom/Extension/.../Composer/*.json` | Dépendances additionnelles (merge plugin) | [composer.json:137-149](../../SuiteCRM/composer.json#L137-L149) |
| `install/install_defaults.php` | Defaults installeur | [install/install_defaults.php:42-50](../../SuiteCRM/install/install_defaults.php#L42-L50) |
| `files.md5` | Empreintes pour vérification fichiers | [files.md5:1-5](../../SuiteCRM/files.md5#L1-L5) |

## 6. Données

Modèle ORM-like maison : **SugarBean** (`data/SugarBean.php`, classe à [ligne 62](../../SuiteCRM/data/SugarBean.php#L62)). Chaque module définit ses champs dans un `vardefs.php`. Exemples :

| Module | Table | Preuve |
|---|---|---|
| Accounts | `accounts` | [modules/Accounts/vardefs.php:45-46](../../SuiteCRM/modules/Accounts/vardefs.php#L45-L46) |
| Contacts | `contacts` | [modules/Contacts/vardefs.php:44](../../SuiteCRM/modules/Contacts/vardefs.php#L44) |
| Leads | `leads` | [modules/Leads/vardefs.php:44](../../SuiteCRM/modules/Leads/vardefs.php#L44) |
| Opportunities | `opportunities` | [modules/Opportunities/vardefs.php:45](../../SuiteCRM/modules/Opportunities/vardefs.php#L45) |
| Cases | `cases` | [modules/Cases/vardefs.php:46](../../SuiteCRM/modules/Cases/vardefs.php#L46) |
| Schedulers | `schedulers` | [modules/Schedulers/Scheduler.php:84](../../SuiteCRM/modules/Schedulers/Scheduler.php#L84) |

Relations (M2M) déclarées dans [modules/TableDictionary.php:45-138](../../SuiteCRM/modules/TableDictionary.php#L45-L138) (88 fichiers metadata).

Drivers de base de données :

| Type | Driver class | Fichier |
|---|---|---|
| MySQL | `MysqlManager` / `MysqliManager` (préféré si `mysqli_connect` dispo) | [include/database/DBManagerFactory.php:78-85](../../SuiteCRM/include/database/DBManagerFactory.php#L78-L85), [include/database/MysqliManager.php](../../SuiteCRM/include/database/MysqliManager.php), [include/database/MysqlManager.php](../../SuiteCRM/include/database/MysqlManager.php) |
| MSSQL | `SqlsrvManager` (`sqlsrv_connect`) ou `FreeTDSManager` ou `MssqlManager` | [include/database/DBManagerFactory.php:86-96](../../SuiteCRM/include/database/DBManagerFactory.php#L86-L96), [include/database/SqlsrvManager.php](../../SuiteCRM/include/database/SqlsrvManager.php), [include/database/FreeTDSManager.php](../../SuiteCRM/include/database/FreeTDSManager.php), [include/database/MssqlManager.php](../../SuiteCRM/include/database/MssqlManager.php) |

Détail ERD → voir [data/modele_donnees.md](../data/modele_donnees.md) et [diagrams/mermaid_erd.md](../diagrams/mermaid_erd.md).

## 7. Recherche

| Moteur | Localisation | Preuve |
|---|---|---|
| **Elasticsearch** (`elasticsearch/elasticsearch ^7.13`) | `lib/Search/ElasticSearch/ElasticSearchEngine.php`, `ElasticSearchClientBuilder.php` | [lib/Search/ElasticSearch/ElasticSearchEngine.php:46-53](../../SuiteCRM/lib/Search/ElasticSearch/ElasticSearchEngine.php#L46-L53), [composer.json:44](../../SuiteCRM/composer.json#L44) |
| **AOD** (Zend Lucene legacy) | `modules/AOD_Index/`, `modules/AOD_IndexEvent/` | [composer.json:75](../../SuiteCRM/composer.json#L75) |
| **BasicSearch** / **SqlSearch** | `lib/Search/BasicSearch/`, `lib/Search/SqlSearch/` | [lib/Search/](../../SuiteCRM/lib/Search/) |

Sélecteur de moteur : `lib/Search/SearchEngine.php`, `lib/Search/SearchWrapper.php`, configuration via `lib/Search/SearchConfigurator.php`.

## 8. Tâches planifiées (out-of-the-box)

`Scheduler::rebuildDefaultSchedulers()` ré-initialise 16 schedulers OOTB (statut Active sauf indication) ([modules/Schedulers/Scheduler.php:815-1003](../../SuiteCRM/modules/Schedulers/Scheduler.php#L815-L1003)) :

| Job | Cron (`min::h::dom::m::dow`) | Statut | Preuve |
|---|---|---|---|
| `processAOW_Workflow` | `*::*::*::*::*` | Active | [Scheduler.php:826-834](../../SuiteCRM/modules/Schedulers/Scheduler.php#L826-L834) |
| `aorRunScheduledReports` | `*::*::*::*::*` | Active | [Scheduler.php:838-846](../../SuiteCRM/modules/Schedulers/Scheduler.php#L838-L846) |
| `trimTracker` | `0::2::1::*::*` | Active | [Scheduler.php:850-858](../../SuiteCRM/modules/Schedulers/Scheduler.php#L850-L858) |
| `pollMonitoredInboxesAOP` | `*::*::*::*::*` | Active | [Scheduler.php:862-870](../../SuiteCRM/modules/Schedulers/Scheduler.php#L862-L870) |
| `pollMonitoredInboxesForBouncedCampaignEmails` | `0::2-6::*::*::*` | Active | [Scheduler.php:874-882](../../SuiteCRM/modules/Schedulers/Scheduler.php#L874-L882) |
| `runMassEmailCampaign` | `0::2-6::*::*::*` | Active | [Scheduler.php:886-894](../../SuiteCRM/modules/Schedulers/Scheduler.php#L886-L894) |
| `pruneDatabase` | `0::4::1::*::*` | **Inactive** | [Scheduler.php:898-906](../../SuiteCRM/modules/Schedulers/Scheduler.php#L898-L906) |
| `aodIndexUnindexed` | `0::0::*::*::*` | Active | [Scheduler.php:910-918](../../SuiteCRM/modules/Schedulers/Scheduler.php#L910-L918) |
| `aodOptimiseIndex` | `0::*/3::*::*::*` | Active | [Scheduler.php:922-930](../../SuiteCRM/modules/Schedulers/Scheduler.php#L922-L930) |
| `sendEmailReminders` | `*::*::*::*::*` | Active | [Scheduler.php:934-942](../../SuiteCRM/modules/Schedulers/Scheduler.php#L934-L942) |
| `cleanJobQueue` | `0::5::*::*::*` | Active | [Scheduler.php:946-954](../../SuiteCRM/modules/Schedulers/Scheduler.php#L946-L954) |
| `removeDocumentsFromFS` | `0::3::1::*::*` | Active | [Scheduler.php:958-966](../../SuiteCRM/modules/Schedulers/Scheduler.php#L958-L966) |
| `trimSugarFeeds` | `0::2::1::*::*` | Active | [Scheduler.php:970-978](../../SuiteCRM/modules/Schedulers/Scheduler.php#L970-L978) |
| `calendarSyncJob` | `*/15::*::*::*::*` | Active | [Scheduler.php:982-990](../../SuiteCRM/modules/Schedulers/Scheduler.php#L982-L990) |
| `runElasticSearchIndexerScheduler` | `30::4::*::*::*` | Active | [Scheduler.php:994-1002](../../SuiteCRM/modules/Schedulers/Scheduler.php#L994-L1002) |

Driver d'exécution : `SugarCronJobs` (par défaut) ou `SugarCronRemoteJobs` ([include/SugarQueue/](../../SuiteCRM/include/SugarQueue/)). Configurable via `$sugar_config['cron_class']` ([cron.php:90-91](../../SuiteCRM/cron.php#L90-L91)).

## 9. Authentification — modes disponibles

| Mode | Classe | Preuve |
|---|---|---|
| Sugar natif (BDD) | `SugarAuthenticate` | [modules/Users/authentication/SugarAuthenticate/](../../SuiteCRM/modules/Users/authentication/SugarAuthenticate/) |
| LDAP | `LDAPAuthenticate` + `LDAPAuthenticateUser` | [modules/Users/authentication/LDAPAuthenticate/](../../SuiteCRM/modules/Users/authentication/LDAPAuthenticate/) |
| SAML2 | `SAML2Authenticate extends SugarAuthenticate` | [modules/Users/authentication/SAML2Authenticate/SAML2Authenticate.php:71-73](../../SuiteCRM/modules/Users/authentication/SAML2Authenticate/SAML2Authenticate.php#L71-L73) (lib : `onelogin/php-saml`) |
| OAuth2 (API V8) | `league/oauth2-server` — grants : Password, ClientCredentials, RefreshToken, AuthCode | [Api/V8/Config/services/middlewares.php:39-91](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L39-L91) |
| Email-based | `EmailAuthenticate` | [modules/Users/authentication/EmailAuthenticate/](../../SuiteCRM/modules/Users/authentication/EmailAuthenticate/) |
| Sélecteur | `AuthenticationController::getAuthController()` | [modules/Users/authentication/AuthenticationController.php:47, 66, 76, 99-100, 114](../../SuiteCRM/modules/Users/authentication/AuthenticationController.php#L47-L114) |

## 10. Intégrations externes détectées

| Intégration | Élément | Preuve |
|---|---|---|
| Google Calendar | `google/apiclient ^2.18` ; service calendar uniquement (`google/apiclient-services: ['Calendar']`) | [composer.json:46,134-136](../../SuiteCRM/composer.json#L46-L136) |
| Google reCAPTCHA | `google/recaptcha ^1.3` | [composer.json:47](../../SuiteCRM/composer.json#L47) |
| SAML2 IdP | `onelogin/php-saml ^4` | [composer.json:57](../../SuiteCRM/composer.json#L57) |
| Elasticsearch | indexation full-text (cluster externe) | [composer.json:44](../../SuiteCRM/composer.json#L44), [lib/Search/ElasticSearch/ElasticSearchClientBuilder.php:46-49, 162](../../SuiteCRM/lib/Search/ElasticSearch/ElasticSearchClientBuilder.php#L46-L162) |
| IMAP (mail entrant) | `javanile/php-imap2` | [composer.json:50](../../SuiteCRM/composer.json#L50), [modules/InboundEmail/InboundEmail.php:54](../../SuiteCRM/modules/InboundEmail/InboundEmail.php#L54) |
| SMTP (mail sortant) | `phpmailer/phpmailer` + `OutboundEmailAccounts` | [composer.json:58](../../SuiteCRM/composer.json#L58), [modules/OutboundEmailAccounts/](../../SuiteCRM/modules/OutboundEmailAccounts/) |
| OAuth2 sortant (vers IdP) | `league/oauth2-client ^2.6` + module `ExternalOAuthConnection`, `ExternalOAuthProvider` | [composer.json:52](../../SuiteCRM/composer.json#L52), [modules/ExternalOAuthConnection/](../../SuiteCRM/modules/ExternalOAuthConnection/), [modules/ExternalOAuthProvider/](../../SuiteCRM/modules/ExternalOAuthProvider/) |
| Connecteurs Sugar (legacy) | `include/connectors/` (sources, filters, formatters) | [include/connectors/](../../SuiteCRM/include/connectors/) |
| Calendar Sync (générique) | `include/CalendarSync/` (orchestrator, jobs, providers) | [include/CalendarSync/application/CalendarSyncOrchestrator.php](../../SuiteCRM/include/CalendarSync/application/CalendarSyncOrchestrator.php), [include/CalendarSync/domain/CalendarProviderType.php](../../SuiteCRM/include/CalendarSync/domain/CalendarProviderType.php) |

## 11. Zones à investiguer / INCONNU

- Pas d'IaC ni de Dockerfile dans le repo — environnement d'exécution réel = INCONNU (à fournir par l'opérateur).
- `config.php` réel = INCONNU (généré à l'installation ; voir [install/install_defaults.php](../../SuiteCRM/install/install_defaults.php)).
- Liste exhaustive des **logic hooks** réellement déployés (custom/Extension) : INCONNU dans ce repo (pas de `custom/` peuplé).
- Endpoints REST `service/v2…v4_1` : registres entiers à inventorier — couverture limitée dans la présente passe (voir [api/endpoints.md](../api/endpoints.md)).

