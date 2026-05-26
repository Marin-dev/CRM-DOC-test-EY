# 01 — Dépendances & graphe logique

> Phase **Searcher**. Toutes les dépendances ci-dessous sont prouvées par `require`/`use`/`new`/inclusion.

## 1. Dépendances composer (`require`)

Source unique : [composer.json:34-76](../../SuiteCRM/composer.json#L34-L76).

```
suitecrm/suitecrm  ──► PHP ^8.1  +  ext-curl/gd/intl/json/openssl/zip
                  ├── slim/slim ^3.8
                  ├── league/oauth2-server ^8.5
                  ├── league/oauth2-client ^2.6
                  ├── smarty/smarty ^4
                  ├── monolog/monolog ^3
                  ├── phpmailer/phpmailer ^6
                  ├── javanile/php-imap2 ^0.1.10
                  ├── zbateson/mail-mime-parser ^2.4
                  ├── elasticsearch/elasticsearch ^7.13
                  ├── zf1s/zend-search-lucene ^1.15
                  ├── tecnickcom/tcpdf ^6.10
                  ├── tinymce/tinymce ^8
                  ├── nesbot/carbon ^3
                  ├── google/apiclient ^2.18  (services Calendar)
                  ├── google/recaptcha ^1.3
                  ├── onelogin/php-saml ^4
                  ├── voku/anti-xss ^4 + ezyang/htmlpurifier ^4.10
                  ├── symfony/{options-resolver,validator,yaml,polyfill-uuid} ^6.4 / ^1.32
                  ├── consolidation/{log ^3.1, robo ^4}
                  ├── nikic/php-parser ^5.7
                  ├── tedivm/jshrink ^1.3
                  ├── wikimedia/composer-merge-plugin ^2
                  ├── psr/{container ^1.0, log ^3.0}
                  ├── justinrainbow/json-schema ^6
                  ├── soundasleep/html2text ^2
                  ├── gymadarasz/{ace ^1.2, imagesloaded ^4.1}
                  └── zf1s/zend-oauth ^1.15
```

Dépendances **dev uniquement** ([composer.json:77-96](../../SuiteCRM/composer.json#L77-L96)) : `codeception/*`, `phpunit/phpunit ^10.5`, `mockery/mockery`, `fakerphp/faker`, `mikey179/vfsstream`, `friendsofphp/php-cs-fixer`, `filp/whoops`, `flow/jsonpath`, `browserstack/browserstack-local`, `jeroendesloovere/vcard`, `scssphp/scssphp`, `vlucas/phpdotenv`.

## 2. Composer merge-plugin (extensions)

[composer.json:137-149](../../SuiteCRM/composer.json#L137-L149) — Le plugin `wikimedia/composer-merge-plugin` fusionne :

- `composer.ext.json`
- `custom/Extension/application/Ext/Composer/*/*.json` (récursif, sans remplacement, sans duplicate, **inclut `require-dev`**)

Conséquence pratique : les modules tiers peuvent enrichir `require` sans modifier `composer.json` principal.

## 3. Dépendances internes — graphe de modules clés

```
index.php ──► include/MVC/preDispatch.php
          ──► include/entryPoint.php (bootstrap global)
          ──► include/MVC/SugarApplication.php (class SugarApplication)
                       │
                       ├─► include/MVC/Controller/ControllerFactory.php
                       │           └─► SugarController (dispatch action_view_map / action_file_map)
                       │
                       ├─► include/MVC/View/ViewFactory.php (Smarty)
                       │
                       ├─► data/SugarBean.php  (ACLAccess, retrieve, save, list)
                       │           └─► data/Relationships/RelationshipFactory.php
                       │                       ├─► M2MRelationship / One2M* / One2One* / EmailAddressRelationship
                       │                       └─► SugarRelationship (base)
                       │
                       ├─► include/database/DBManagerFactory.php
                       │           └─► MysqliManager / MysqlManager / SqlsrvManager / FreeTDSManager / MssqlManager
                       │
                       ├─► modules/Users/authentication/AuthenticationController.php
                       │           └─► Sugar | LDAP | SAML2 | Email
                       │
                       └─► include/resource/ResourceManager.php (observers)
```
Preuves : [index.php:45-52](../../SuiteCRM/index.php#L45-L52), [include/MVC/SugarApplication.php:51-103](../../SuiteCRM/include/MVC/SugarApplication.php#L51-L103), [data/SugarBean.php:45-62](../../SuiteCRM/data/SugarBean.php#L45-L62), [include/database/DBManagerFactory.php:60-126](../../SuiteCRM/include/database/DBManagerFactory.php#L60-L126).

## 4. API V8 — graphe d'injection

```
Api/index.php
   └─► Api/Core/app.php
            └─► new Slim\App( Api\Core\Loader\ContainerLoader::configure() )
                              │  (services.php → beanAliases / controllers / factories /
                              │   globals / helpers / middlewares / params / services / validators)
                              ▼
                         Api/V8/Config/routes.php  (groupes /  +  /V8)
                              ├─► AuthorizationServerMiddleware  →  POST /access_token
                              └─► ResourceServerMiddleware       →  /V8/*
                                       ├── ModuleController     ──► ModuleService     ──► BeanManager (DB + ACL)
                                       ├── RelationshipController ─► RelationshipService
                                       ├── MetaController       ──► MetaService (modules, fields, swagger)
                                       ├── UserController       ──► UserService
                                       ├── UserPreferencesController ─► UserPreferencesService
                                       ├── ListViewController / ListViewSearchController
                                       └── LogoutController     ──► LogoutService
```
Preuves : [Api/Core/app.php:23-26](../../SuiteCRM/Api/Core/app.php#L23-L26), [Api/V8/Config/services.php:8-28](../../SuiteCRM/Api/V8/Config/services.php#L8-L28), [Api/V8/Config/routes.php:12-131](../../SuiteCRM/Api/V8/Config/routes.php#L12-L131), [Api/V8/Controller/BaseController.php:1-52](../../SuiteCRM/Api/V8/Controller/BaseController.php#L1-L52), [Api/V8/Controller/ModuleController.php:13-122](../../SuiteCRM/Api/V8/Controller/ModuleController.php#L13-L122), [Api/V8/Service/ModuleService.php:28-66](../../SuiteCRM/Api/V8/Service/ModuleService.php#L28-L66).

## 5. Stack OAuth2 — composants

| Composant | Class | Preuve |
|---|---|---|
| Authorization Server | `League\OAuth2\Server\AuthorizationServer` | [Api/V8/Config/services/middlewares.php:39-93](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L39-L93) |
| Resource Server | `League\OAuth2\Server\ResourceServer` | [middlewares.php:95-111](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L95-L111) |
| Grant types | `ClientCredentialsGrant`, `PasswordGrant`, `RefreshTokenGrant`, `AuthCodeGrant` | [middlewares.php:58-91](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L58-L91) |
| TTL access token | 1 heure (`PT1H`) | idem |
| TTL refresh token | 1 mois (`P1M`) | [middlewares.php:76](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L76) |
| TTL auth code | 10 minutes (`PT10M`) | [middlewares.php:88](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L88) |
| Repositories | `AccessTokenRepository`, `AuthCodeRepository`, `ClientRepository`, `RefreshTokenRepository`, `ScopeRepository`, `UserRepository` | [Api/V8/OAuth2/Repository/](../../SuiteCRM/Api/V8/OAuth2/Repository/) |
| Entities | `AccessTokenEntity`, `AuthCodeEntity`, `ClientEntity`, `RefreshTokenEntity`, `UserEntity` | [Api/V8/OAuth2/Entity/](../../SuiteCRM/Api/V8/OAuth2/Entity/) |
| Clé chiffrement | `$sugar_config['oauth2_encryption_key']` (fallback `SCRM-DEFK` + log fatal) | [middlewares.php:31-37](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L31-L37) |
| Clé RSA privée/publique | `Api/V8/OAuth2/private.key` / `Api/V8/OAuth2/public.key` | [Api/Core/Config/ApiConfig.php:20-21](../../SuiteCRM/Api/Core/Config/ApiConfig.php#L20-L21) |

Module Sugar associé (stockage des tokens en BDD) : `modules/OAuth2Tokens/`, `modules/OAuth2Clients/`, `modules/OAuth2AuthCodes/`, `modules/OAuthKeys/`, `modules/OAuthTokens/` (legacy zf1).

## 6. Cycle de vie d'une requête HTTP

### 6.1 Front Sugar (UI) — `index.php`

1. `define('sugarEntry')` puis `include 'include/MVC/preDispatch.php'` ([index.php:42-45](../../SuiteCRM/index.php#L42-L45)).
2. `require_once 'include/entryPoint.php'` : bootstrap globaux (`$sugar_config`, `$db`, `$log`, locale…).
3. `SugarApplication::execute()` :
   - lecture `$_REQUEST['module']` (fallback `default_module` = `Home`),
   - `ControllerFactory::getController($module)`,
   - si `entryPoint` requiert auth ([SugarApplication.php:87-94](../../SuiteCRM/include/MVC/SugarApplication.php#L87-L94)) : `loadUser()` + `ACLFilter()` + `preProcess()` + `checkHTTPReferer()`,
   - `SugarThemeRegistry::buildRegistry()` puis `controller->execute()`,
   - `sugar_cleanup()`.

### 6.2 API V8 — `Api/index.php`

1. CORS headers (`Allow-Origin: *`, methods, headers) — [Api/Core/app.php:3-5](../../SuiteCRM/Api/Core/app.php#L3-L5).
2. Récupération `HTTP_AUTHORIZATION` (fallback `REDIRECT_HTTP_AUTHORIZATION` pour PHP-FPM) — [app.php:16-18](../../SuiteCRM/Api/Core/app.php#L16-L18).
3. Bootstrap Sugar : `require_once 'include/entryPoint.php'`.
4. `new Slim\App(ContainerLoader::configure())` + `RouteLoader::configureRoutes($app)` ([app.php:23-26](../../SuiteCRM/Api/Core/app.php#L23-L26)).
5. Slim dispatch : `ResourceServerMiddleware` valide le bearer token (sauf `/access_token`) → contrôleur → service → `BeanManager` → DB.
6. Réponse JSON:API (`Content-type: application/vnd.api+json`) — [Api/V8/Controller/BaseController.php:10, 19-34](../../SuiteCRM/Api/V8/Controller/BaseController.php#L10-L34).

### 6.3 CLI Cron — `cron.php`

1. Refus si SAPI ≠ CLI ([cron.php:49-52](../../SuiteCRM/cron.php#L49-L52)).
2. Vérif utilisateur Unix vs `sugar_config['cron']['allowed_cron_users']` ([cron.php:54-77](../../SuiteCRM/cron.php#L54-L77)).
3. `current_user` = utilisateur système Sugar ([cron.php:84-87](../../SuiteCRM/cron.php#L84-L87)).
4. Driver `sugar_config['cron_class']` (def. `SugarCronJobs`) — instancie + `runCycle()` ([cron.php:90-101](../../SuiteCRM/cron.php#L90-L101)).
5. `sugar_cleanup(false)` + `DBManagerFactory::getInstance()->disconnect()` + `session_destroy()`.

## 7. SugarBean — point central du domaine

[data/SugarBean.php:62](../../SuiteCRM/data/SugarBean.php#L62) — classe **API publique** (`@api`). C'est la « base de tous les business objects » avec CRUD, recherche, listes, relations. Chaque module définit une sous-classe (ex. `Account extends SugarBean`, `Contact extends SugarBean`, …).

L'API V8 manipule des Beans via `BeanManager` :

- `getRecord` → `ACLAccess('view')` + sérialisation JSON:API ([Api/V8/Service/ModuleService.php:74-92](../../SuiteCRM/Api/V8/Service/ModuleService.php#L74-L92)),
- `getRecords` → filtre/pagination/orderBy ; cas particulier `email1/email2` jointure `email_addresses` + `email_addr_bean_rel` ([ModuleService.php:100-235](../../SuiteCRM/Api/V8/Service/ModuleService.php#L100-L235)),
- `createRecord` → `ACLAccess('save')` + `setRecordUpdateParams` + traitement attachments pour `Notes`/`Documents` ([ModuleService.php:246-295](../../SuiteCRM/Api/V8/Service/ModuleService.php#L246-L295)),
- `updateRecord` / `deleteRecord` (status 200/201/400).

## 8. Hooks logiques (`logic_hooks`)

Implémentation : [include/utils/LogicHook.php:70](../../SuiteCRM/include/utils/LogicHook.php#L70). Charge `<custom>/logic_hooks.php` ([LogicHook.php:150-154](../../SuiteCRM/include/utils/LogicHook.php#L150-L154)). Events prédéfinis (en commentaire de la classe) : `before_save`, `after_save`, `after_ui_frame`, `after_ui_footer`, … ([LogicHook.php:44-50](../../SuiteCRM/include/utils/LogicHook.php#L44-L50)).

Usage côté workflow : `AOW_WorkFlow` peut s'auto-désactiver via `AOW_WorkFlow::$doNotRunInSaveLogic = true` pour éviter les boucles dans les `after_save` ([modules/AOW_WorkFlow/AOW_WorkFlow.php:209-232](../../SuiteCRM/modules/AOW_WorkFlow/AOW_WorkFlow.php#L209-L232)).

## 9. Synthèse — dépendances « critiques » (single points of failure)

| Risque | Composant | Pourquoi critique |
|---|---|---|
| Persistance | `data/SugarBean.php` + drivers DB (`include/database/*`) | Toutes les opérations CRUD passent par là |
| Auth API | `league/oauth2-server` + clés RSA Api/V8/OAuth2/ | Sans clés ni `oauth2_encryption_key` configuré, fallback non sécurisé ([middlewares.php:33-36](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L33-L36)) |
| Cron | `cron.php` + `SugarCronJobs` | Pas de cron = workflows, reports, e-mails, indexation à l'arrêt |
| Bootstrap | `include/entryPoint.php` | Bootstrap commun à tous les entrypoints |
| Config | `config.php` généré | Contient creds DB / site_url / clés ; non versionné |

