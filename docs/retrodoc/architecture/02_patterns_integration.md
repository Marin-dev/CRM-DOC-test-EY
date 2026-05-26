# 02 — Patterns d'intégration

> Phase **Searcher**. Recense les flux d'intégration *entrants* et *sortants*, leur type, format, protocole, et le détail pas-à-pas avec preuves.

## 1. Synthèse — matrice des intégrations

| # | Intégration | Type | Direction | Format | Protocole | Auth | Preuve |
|---|---|---|---|---|---|---|---|
| 1 | API V8 (REST JSON:API) | API | entrant | JSON (`application/vnd.api+json`) | HTTP/1.1 | OAuth2 Bearer | [Api/V8/Controller/BaseController.php:10](../../SuiteCRM/Api/V8/Controller/BaseController.php#L10) ; [Api/V8/Config/routes.php:131](../../SuiteCRM/Api/V8/Config/routes.php#L131) |
| 2 | API REST legacy v2…v4_1 | API | entrant | JSON (POST `method=...`) | HTTP/1.1 | session token Sugar | [service/v4_1/rest.php](../../SuiteCRM/service/v4_1/rest.php) ; [service/v4_1/registry.php:46](../../SuiteCRM/service/v4_1/registry.php#L46) |
| 3 | SOAP (NuSOAP) | API | entrant | XML SOAP 1.1 | HTTP | session token Sugar | [soap.php:50-67](../../SuiteCRM/soap.php#L50-L67) |
| 4 | JSON-RPC | API | entrant | JSON-RPC | HTTP | session Sugar (entry `json_server`, auth=true) | [json_server.php:46-48](../../SuiteCRM/json_server.php#L46-L48), [entry_point_registry.php:55](../../SuiteCRM/include/MVC/Controller/entry_point_registry.php#L55) |
| 5 | Cron / file de jobs | Batch | interne | PHP serialized in DB (`job_queue`) | CLI / OS | utilisateur OS contrôlé via `allowed_cron_users` | [cron.php:54-77, 90-101](../../SuiteCRM/cron.php#L54-L101) ; drivers [include/SugarQueue/](../../SuiteCRM/include/SugarQueue/) |
| 6 | E-mail entrant (IMAP) | File / API | entrant | MIME / RFC-822 | IMAP4 | login + password (compte e-mail) | [modules/InboundEmail/InboundEmail.php:54](../../SuiteCRM/modules/InboundEmail/InboundEmail.php#L54), lib `javanile/php-imap2` ([composer.json:50](../../SuiteCRM/composer.json#L50)) |
| 7 | E-mail sortant (SMTP) | API | sortant | MIME | SMTP / SMTPS / TLS | par compte `OutboundEmailAccounts` | [modules/OutboundEmailAccounts/](../../SuiteCRM/modules/OutboundEmailAccounts/), lib `phpmailer/phpmailer` ([composer.json:58](../../SuiteCRM/composer.json#L58)) |
| 8 | Elasticsearch | API | sortant | JSON | HTTP(S) | définie côté config Sugar (`$sugar_config`) | [lib/Search/ElasticSearch/ElasticSearchClientBuilder.php:46-49, 162](../../SuiteCRM/lib/Search/ElasticSearch/ElasticSearchClientBuilder.php#L46-L162) |
| 9 | Google Calendar (sync) | API | sortant | JSON (Google API) | HTTPS | OAuth2 (`google/apiclient`) | [composer.json:46](../../SuiteCRM/composer.json#L46) ; [include/CalendarSync/application/CalendarSyncOrchestrator.php](../../SuiteCRM/include/CalendarSync/application/CalendarSyncOrchestrator.php) |
| 10 | SAML2 IdP | API | bidirectionnel | XML SAML | HTTPS (Redirect/POST) | signé / chiffré côté IdP | [modules/Users/authentication/SAML2Authenticate/SAML2Authenticate.php:71-73](../../SuiteCRM/modules/Users/authentication/SAML2Authenticate/SAML2Authenticate.php#L71-L73), lib `onelogin/php-saml` ([composer.json:57](../../SuiteCRM/composer.json#L57)) |
| 11 | LDAP / AD | API | sortant | LDAP | LDAP / LDAPS | bind DN + password | [modules/Users/authentication/LDAPAuthenticate/](../../SuiteCRM/modules/Users/authentication/LDAPAuthenticate/) |
| 12 | reCAPTCHA Google | API | sortant | Form-encoded | HTTPS | secret key | [composer.json:47](../../SuiteCRM/composer.json#L47) |
| 13 | Web-to-Lead / Web-to-Person | API | entrant | HTML form POST | HTTP(S) | aucune (entry-point public, `auth=false`) | [modules/Campaigns/WebToLeadCapture.php](../../SuiteCRM/modules/Campaigns/WebToLeadCapture.php) ; [entry_point_registry.php:61-62](../../SuiteCRM/include/MVC/Controller/entry_point_registry.php#L61-L62) |
| 14 | Campaign trackers (open / click / removeme) | API | entrant | URL avec param tracker | HTTP(S) | aucune (`auth=false`) | [entry_point_registry.php:59-63](../../SuiteCRM/include/MVC/Controller/entry_point_registry.php#L59-L63) ; [modules/Campaigns/Tracker.php](../../SuiteCRM/modules/Campaigns/Tracker.php) |
| 15 | OAuth2 sortant vers IdP tiers | API | sortant | JSON OAuth2 | HTTPS | client_id/secret | `league/oauth2-client` ([composer.json:52](../../SuiteCRM/composer.json#L52)), [modules/ExternalOAuthConnection/](../../SuiteCRM/modules/ExternalOAuthConnection/), [modules/ExternalOAuthProvider/](../../SuiteCRM/modules/ExternalOAuthProvider/) |
| 16 | vCard / iCal / vCal | File | sortant | text/vcard, text/calendar | HTTP | session Sugar | [vCard.php](../../SuiteCRM/vCard.php), [ical_server.php](../../SuiteCRM/ical_server.php), [vcal_server.php](../../SuiteCRM/vcal_server.php), [entry_point_registry.php:52](../../SuiteCRM/include/MVC/Controller/entry_point_registry.php#L52) |
| 17 | PDF generation | File | sortant | PDF | HTTP | session Sugar | [pdf.php](../../SuiteCRM/pdf.php), [entry_point_registry.php:53](../../SuiteCRM/include/MVC/Controller/entry_point_registry.php#L53), lib `tecnickcom/tcpdf` ([composer.json:68](../../SuiteCRM/composer.json#L68)) |
| 18 | Module Install (.zip) | File | entrant | ZIP (manifest.php) | HTTP upload | session Sugar (admin) | [ModuleInstall/](../../SuiteCRM/ModuleInstall/) |

## 2. Détail pas-à-pas — flux clés

### 2.1 Flux REST API V8 (OAuth2 password grant)

Endpoint : `POST /Api/access_token` (sans préfixe `/V8`) puis appels CRUD sur `/Api/V8/module/{moduleName}/...`.

Étapes :

1. **Demande de token** — `POST /Api/access_token` avec `grant_type=password&client_id=...&client_secret=...&username=...&password=...`.
   - Middleware : `AuthorizationServerMiddleware` ([routes.php:16-18](../../SuiteCRM/Api/V8/Config/routes.php#L16-L18)).
   - Server : `League\OAuth2\Server\AuthorizationServer` configuré avec `PasswordGrant` ([middlewares.php:64-71](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L64-L71)).
   - Réponse : `access_token` (TTL 1h, `PT1H`) + `refresh_token` (TTL 1 mois, `P1M`).
2. **Appel API** — `GET /Api/V8/module/Accounts/{id}` avec `Authorization: Bearer <access_token>`.
   - Middleware : `ResourceServerMiddleware` (validation token) ([routes.php:131](../../SuiteCRM/Api/V8/Config/routes.php#L131)).
   - Slim → `ModuleController::getModuleRecord` → `ModuleService::getRecord` → `BeanManager::getBeanSafe` → `bean->ACLAccess('view')`.
   - Si pas le droit : `AccessDeniedException` → 400 + body JSON:API d'erreur ([BaseController.php:43-51](../../SuiteCRM/Api/V8/Controller/BaseController.php#L43-L51), [ModuleService.php:82-84](../../SuiteCRM/Api/V8/Service/ModuleService.php#L82-L84)).
3. **Réponse** — JSON encodé `JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES`, content-type `application/vnd.api+json`, status `200` (`GET`) / `201` (`POST`/`PATCH`) / `400` (erreur).

Détail des routes → [api/endpoints.md](../api/endpoints.md). Détails auth → [api/authentification.md](../api/authentification.md).

### 2.2 Flux Cron — exécution d'un job planifié

```
Crontab système ──> php cron.php
                     │
                     ├─ Vérif SAPI=cli  ([cron.php:49-52])
                     ├─ Vérif user système ∈ allowed_cron_users  ([cron.php:54-77])
                     ├─ $current_user = utilisateur "system"
                     │
                     └─ new $cron_driver  (default: SugarCronJobs)
                              └─ runCycle()
                                   ├─ Sélectionne Schedulers actifs sans job pending
                                   │   ([Scheduler.php:196])
                                   ├─ Crée job(s) dans `job_queue` (SchedulersJob)
                                   ├─ Exécute chaque job (`function::xxxx`)
                                   │   ex.: processAOW_Workflow, aorRunScheduledReports,
                                   │        pollMonitoredInboxesAOP, runMassEmailCampaign,
                                   │        sendEmailReminders, calendarSyncJob, ...
                                   └─ Marque DONE / FAIL
```
Preuves : [cron.php:90-101](../../SuiteCRM/cron.php#L90-L101), [modules/Schedulers/Scheduler.php:84, 196](../../SuiteCRM/modules/Schedulers/Scheduler.php#L84), [modules/Schedulers/Scheduler.php:815-1003](../../SuiteCRM/modules/Schedulers/Scheduler.php#L815-L1003), drivers [include/SugarQueue/SugarCronJobs.php](../../SuiteCRM/include/SugarQueue/SugarCronJobs.php), [include/SugarQueue/SugarJobQueue.php](../../SuiteCRM/include/SugarQueue/SugarJobQueue.php).

Voir [flows/30_cron_scheduler.md](../flows/30_cron_scheduler.md) pour le détail.

### 2.3 Flux E-mail entrant (Inbound Email)

1. Scheduler OOTB `pollMonitoredInboxesAOP` toutes les minutes ([Scheduler.php:862-870](../../SuiteCRM/modules/Schedulers/Scheduler.php#L862-L870)).
2. Job lit chaque compte `inbound_email` (table `inbound_email`).
3. Pour chaque compte : connexion IMAP via `javanile/php-imap2`, parse MIME via `zbateson/mail-mime-parser`.
4. Création d'enregistrements `Email` (table `emails`) + `EmailAddress` (`email_addresses`) + relations `emails_beans` (`emails_beans`).
5. Si `Cases AOP` activé : création/auto-update de `Cases` et `AOP_Case_Updates`.

Preuves : [modules/InboundEmail/InboundEmail.php:54, 131-133](../../SuiteCRM/modules/InboundEmail/InboundEmail.php#L54-L133), modules [modules/AOP_Case_Events/](../../SuiteCRM/modules/AOP_Case_Events/), [modules/AOP_Case_Updates/](../../SuiteCRM/modules/AOP_Case_Updates/).

### 2.4 Flux Web-to-Lead

1. Formulaire HTML hébergé tiers `POST` vers `index.php?entryPoint=WebToLeadCapture` ([entry_point_registry.php:61](../../SuiteCRM/include/MVC/Controller/entry_point_registry.php#L61), `auth = false`).
2. `WebToLeadCapture.php` valide le `campaign_id` + signature, mappe le formulaire → champs `Lead`.
3. Sauvegarde via `BeanFactory::newBean('Leads')->save()`.
4. Si campagne associée : alimente `campaign_log` (table `campaign_log`).

Preuves : [modules/Campaigns/WebToLeadCapture.php](../../SuiteCRM/modules/Campaigns/WebToLeadCapture.php), [entry_point_registry.php:61-62](../../SuiteCRM/include/MVC/Controller/entry_point_registry.php#L61-L62), module [modules/Campaigns/](../../SuiteCRM/modules/Campaigns/).

### 2.5 Flux Recherche Elasticsearch

1. Scheduler `runElasticSearchIndexerScheduler` (cron `30::4::*::*::*`) indexe les beans configurés ([Scheduler.php:994-1002](../../SuiteCRM/modules/Schedulers/Scheduler.php#L994-L1002)).
2. Requête utilisateur (UnifiedSearch) → `lib/Search/SearchEngine.php` → délègue au moteur configuré (`ElasticSearchEngine` ou `BasicSearch` ou `SqlSearch` ou `AOD`) ([lib/Search/](../../SuiteCRM/lib/Search/)).
3. `ElasticSearchEngine::search()` traduit la `SearchQuery` en requête ES ([lib/Search/ElasticSearch/ElasticSearchEngine.php:105-163](../../SuiteCRM/lib/Search/ElasticSearch/ElasticSearchEngine.php#L105-L163)).

### 2.6 Flux Workflow (AOW)

1. Scheduler `processAOW_Workflow` (chaque minute) ([Scheduler.php:826-834](../../SuiteCRM/modules/Schedulers/Scheduler.php#L826-L834)).
2. `AOW_WorkFlow::run_flows()` parcourt les workflows actifs et leurs `AOW_Conditions`.
3. Pour chaque match, exécute les `AOW_Actions` (création de tâche, e-mail, modification de champ, …).
4. Évite la boucle via `AOW_WorkFlow::$doNotRunInSaveLogic` ([AOW_WorkFlow.php:209-232](../../SuiteCRM/modules/AOW_WorkFlow/AOW_WorkFlow.php#L209-L232)).
5. Trace : table `aow_processed`.

### 2.7 Flux SAML2 (login SSO)

1. Utilisateur clique sur le bouton SAML → `index.php?entryPoint=SAML` (`auth=false`) → [modules/Users/authentication/SAMLAuthenticate/index.php](../../SuiteCRM/modules/Users/authentication/SAMLAuthenticate/index.php).
2. Redirection vers IdP avec `AuthnRequest` (lib `onelogin/php-saml`).
3. Retour `SAMLResponse` POST → `SAML2Authenticate->loginAuthenticate()` ([SAML2Authenticate.php:71-73](../../SuiteCRM/modules/Users/authentication/SAML2Authenticate/SAML2Authenticate.php#L71-L73)).
4. Création/mise à jour de l'utilisateur Sugar, ouverture session.

## 3. Formats — récapitulatif

| Famille | Endpoints | Format |
|---|---|---|
| JSON:API | API V8 (`/Api/V8/...`) | `application/vnd.api+json` |
| JSON brut | API REST v4_1 (`service/v4_1/rest.php`), JSON-RPC | `application/json` |
| XML SOAP | `soap.php` | `text/xml; charset=utf-8` |
| HTML form | Web-to-Lead, campaign trackers | `application/x-www-form-urlencoded` |
| MIME e-mail | IMAP / SMTP | RFC-822 / RFC-5322 |
| CSV | Import / Export | [modules/Import/](../../SuiteCRM/modules/Import/), [include/CleanCSV.php](../../SuiteCRM/include/CleanCSV.php) |
| iCal / vCal | `ical_server.php`, `vcal_server.php` | `text/calendar` |
| vCard | `vCard.php` | `text/vcard` |
| PDF | `pdf.php` (TCPDF) | `application/pdf` |

## 4. Protocoles — récapitulatif

| Protocole | Composant | Sens |
|---|---|---|
| HTTP/HTTPS | tous les entrypoints HTTP | entrant + sortant |
| IMAP / IMAPS | `javanile/php-imap2` | sortant (vers serveur IMAP) |
| SMTP / SMTPS / STARTTLS | `phpmailer/phpmailer` | sortant |
| LDAP / LDAPS | LDAPAuthenticate | sortant |
| OAuth2 (RFC 6749) | `league/oauth2-server` et `league/oauth2-client` | entrant + sortant |
| SAML 2.0 | `onelogin/php-saml` | bidirectionnel |
| SOAP 1.1 | `service/core/NusoapSoap.php` + `PHP5Soap.php` | entrant |
| JSON-RPC | `service/JsonRPCServer/` | entrant |

## 5. Schémas d'erreur

- **API V8** : tous les contrôleurs encapsulent dans `try / catch (\Exception)` et retournent `generateErrorResponse(..., 400)` ([ModuleController.php:39-43, 60-63, 79-82, 96-101, 114-119](../../SuiteCRM/Api/V8/Controller/ModuleController.php#L39-L119)) avec un body JSON:API (`ErrorResponse`) — [Api/V8/Controller/BaseController.php:43-51](../../SuiteCRM/Api/V8/Controller/BaseController.php#L43-L51).
- **API legacy v4_1** : status 200 + payload contenant un champ `error` (struct `error_value` enregistrée dans `registry_v4_1::registerTypes`) — [service/v4_1/registry.php:84-102](../../SuiteCRM/service/v4_1/registry.php#L84-L102).
- **SOAP** : faults SOAP standard (NuSOAP) — [soap.php:50, 66](../../SuiteCRM/soap.php#L50-L66).
