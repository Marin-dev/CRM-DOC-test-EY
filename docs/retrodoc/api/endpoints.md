# API — Endpoints

> Phase **Writer**. Endpoints prouvés directement dans le repo. Toute route déclarée dans le code est ancrée.

## 1. Conventions

- **Préfixe API V8** : ce dépôt expose le front-controller à `Api/index.php` ([Api/index.php:1-5](../../SuiteCRM/Api/index.php#L1-L5)). Selon la configuration du serveur web, l'URL publique est typiquement `https://<host>/Api/V8/...` ou `https://<host>/legacy/Api/V8/...` (la configuration Apache n'est pas dans ce repo → INCONNU pour la base URL effective).
- **Routes API V8** : déclarées dans [Api/V8/Config/routes.php:12-131](../../SuiteCRM/Api/V8/Config/routes.php#L12-L131).
- **Middlewares** :
  - `/access_token` → `AuthorizationServerMiddleware`
  - `/V8/*` → `ResourceServerMiddleware` (token Bearer obligatoire) ([routes.php:131](../../SuiteCRM/Api/V8/Config/routes.php#L131))
- **Content-type réponses V8** : `application/vnd.api+json` ([BaseController.php:10](../../SuiteCRM/Api/V8/Controller/BaseController.php#L10)).

## 2. API V8 — endpoints exhaustifs

> Source : [Api/V8/Config/routes.php](../../SuiteCRM/Api/V8/Config/routes.php). Codes de retour codés en dur dans les contrôleurs (`200` lecture, `201` écriture, `400` exception). Les codes 401/403 sont gérés par le middleware OAuth2 et par `AccessDeniedException`.

### 2.1 Authentification

| Verbe | Chemin | Description | Auth | Code succès | Code erreur | Preuve |
|---|---|---|---|---|---|---|
| `POST` | `/access_token` | Émission d'un access token OAuth2 (grants : `password`, `client_credentials`, `refresh_token`, `authorization_code`) | aucune (init) | 200 | 4xx OAuth2 | [routes.php:16-18](../../SuiteCRM/Api/V8/Config/routes.php#L16-L18), [middlewares.php:39-91](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L39-L91) |
| `POST` | `/V8/logout` | Révocation de session | OAuth2 Bearer | 200 | 400 | [routes.php:26](../../SuiteCRM/Api/V8/Config/routes.php#L26) |

### 2.2 Méta-informations

| Verbe | Chemin | Description | Paramètres | Réponse | Preuve |
|---|---|---|---|---|---|
| `GET` | `/V8/current-user` | Renvoie l'utilisateur authentifié | — | JSON:API `User` | [routes.php:36](../../SuiteCRM/Api/V8/Config/routes.php#L36) |
| `GET` | `/V8/meta/modules` | Liste des modules accessibles | — | JSON | [routes.php:38](../../SuiteCRM/Api/V8/Config/routes.php#L38) |
| `GET` | `/V8/meta/fields/{moduleName}` | Liste des champs d'un module (vardefs) | path `moduleName` | JSON | [routes.php:40-41](../../SuiteCRM/Api/V8/Config/routes.php#L40-L41), params [Api/V8/Param/GetFieldListParams.php](../../SuiteCRM/Api/V8/Param/GetFieldListParams.php) |
| `GET` | `/V8/meta/swagger.json` | Schéma OpenAPI/Swagger | — | JSON | [routes.php:50](../../SuiteCRM/Api/V8/Config/routes.php#L50) |
| `GET` | `/V8/search-defs/module/{moduleName}` | Définitions de recherche ListView | path `moduleName` | JSON | [routes.php:28-30](../../SuiteCRM/Api/V8/Config/routes.php#L28-L30) |
| `GET` | `/V8/listview/columns/{moduleName}` | Colonnes ListView | path `moduleName` | JSON | [routes.php:32-34](../../SuiteCRM/Api/V8/Config/routes.php#L32-L34) |
| `GET` | `/V8/user-preferences/{id}` | Préférences utilisateur | path `id` | JSON | [routes.php:43-45](../../SuiteCRM/Api/V8/Config/routes.php#L43-L45) |

### 2.3 CRUD Modules

| Verbe | Chemin | Description | Paramètres | Status succès | Preuve |
|---|---|---|---|---|---|
| `GET` | `/V8/module/{moduleName}` | Liste paginée d'enregistrements | path `moduleName` ; query `fields[<module>]`, `filter`, `sort`, `page[size]`, `page[number]`, `deleted` | 200 | [routes.php:56-57](../../SuiteCRM/Api/V8/Config/routes.php#L56-L57), [GetModulesParams](../../SuiteCRM/Api/V8/Param/GetModulesParams.php), [ModuleService::getRecords](../../SuiteCRM/Api/V8/Service/ModuleService.php#L100-L235) |
| `GET` | `/V8/module/{moduleName}/{id}` | Lecture d'un enregistrement | path `moduleName`, `id` ; query `fields[...]` | 200 | [routes.php:62-64](../../SuiteCRM/Api/V8/Config/routes.php#L62-L64), [ModuleService::getRecord](../../SuiteCRM/Api/V8/Service/ModuleService.php#L74-L92) |
| `POST` | `/V8/module` | Création | body JSON:API `{data:{type, attributes, id?}}` | 201 | [routes.php:69-71](../../SuiteCRM/Api/V8/Config/routes.php#L69-L71), [ModuleService::createRecord](../../SuiteCRM/Api/V8/Service/ModuleService.php#L246-L295) |
| `PATCH` | `/V8/module` | Mise à jour | body JSON:API `{data:{type, id, attributes}}` | 201 | [routes.php:76-78](../../SuiteCRM/Api/V8/Config/routes.php#L76-L78), [ModuleService::updateRecord](../../SuiteCRM/Api/V8/Service/ModuleService.php#L297-L) |
| `DELETE` | `/V8/module/{moduleName}/{id}` | Suppression (soft delete) | path `moduleName`, `id` | 200 | [routes.php:83-85](../../SuiteCRM/Api/V8/Config/routes.php#L83-L85), [ModuleService::deleteRecord](../../SuiteCRM/Api/V8/Service/ModuleService.php) |

Codes d'erreur : `400` retourné par les contrôleurs sur `\Exception` ([ModuleController.php:40-43, 60-63, 79-82, 96-101, 114-119](../../SuiteCRM/Api/V8/Controller/ModuleController.php#L40-L119)). Le middleware OAuth2 peut produire `401` (token absent / invalide).

### 2.4 Relations

| Verbe | Chemin | Description | Status | Preuve |
|---|---|---|---|---|
| `GET` | `/V8/module/{moduleName}/{id}/relationships/{linkFieldName}` | Liste les beans liés | 200 | [routes.php:91-95](../../SuiteCRM/Api/V8/Config/routes.php#L91-L95) |
| `POST` | `/V8/module/{moduleName}/{id}/relationships` | Crée une relation (body décrit le lien) | 201 | [routes.php:101-104](../../SuiteCRM/Api/V8/Config/routes.php#L101-L104) |
| `POST` | `/V8/module/{moduleName}/{id}/relationships/{linkFieldName}` | Crée une relation via un linkField précis | 201 | [routes.php:111-114](../../SuiteCRM/Api/V8/Config/routes.php#L111-L114) |
| `DELETE` | `/V8/module/{moduleName}/{id}/relationships/{linkFieldName}/{relatedBeanId}` | Supprime une relation | 200 | [routes.php:121-124](../../SuiteCRM/Api/V8/Config/routes.php#L121-L124) |

### 2.5 Custom routes

Le groupe `/V8/custom` charge les routes personnalisées via `CustomLoader::loadCustomRoutes($app)` ([routes.php:128-130](../../SuiteCRM/Api/V8/Config/routes.php#L128-L130)). Aucune route custom n'est définie dans ce repo (`custom/` non peuplé).

### 2.6 Headers spéciaux

| Header | Rôle | Preuve |
|---|---|---|
| `Authorization: Bearer <token>` | Authentification OAuth2 (sauf `/access_token`) | [Api/Core/app.php:13-18](../../SuiteCRM/Api/Core/app.php#L13-L18) |
| `Accept: application/vnd.api+json` | Négociation media-type JSON:API | [BaseController.php:10, 19-34](../../SuiteCRM/Api/V8/Controller/BaseController.php#L10-L34) |
| `Content-Type: application/vnd.api+json` | Body des POST/PATCH/DELETE | idem |
| `Access-Control-Allow-Origin: *` | CORS (côté serveur) | [Api/Core/app.php:3-5](../../SuiteCRM/Api/Core/app.php#L3-L5) |

## 3. REST legacy v2.1 → v4_1

Endpoints HTTP : `service/v2/rest.php`, `service/v2_1/rest.php`, `service/v3/rest.php`, `service/v3_1/rest.php`, `service/v4/rest.php`, `service/v4_1/rest.php` (chacun expose un `rest.php` et un `soap.php` éponyme dans son dossier).

Mécanisme : `POST` avec champs `method`, `input_type=JSON`, `response_type=JSON`, `rest_data` (JSON encodé en string). La liste des méthodes est définie par les `registry_v*::registerFunction()` (héritage).

**v4_1** ajoute notamment :

| Méthode | Description | Signature (résumée) | Preuve |
|---|---|---|---|
| `get_relationships` (v4_1) | Récupère relations avec pagination | `session, module_name, module_id, link_field_name, related_module_query, related_fields, related_module_link_name_to_fields_array, deleted, order_by, offset, limit` | [service/v4_1/registry.php:60-64](../../SuiteCRM/service/v4_1/registry.php#L60-L64) |
| `get_modified_relationships` | Liste les relations modifiées entre 2 dates | `session, module_name, related_module, from_date, to_date, offset, max_results, deleted, module_user_id, select_fields, relationship_name, deletion_date` | [service/v4_1/registry.php:68-72](../../SuiteCRM/service/v4_1/registry.php#L68-L72) |

Les méthodes héritées (auth, CRUD, search) viennent des registres v2 → v4. Pour la liste exhaustive : ouvrir `service/v*/registry.php` (héritage en chaîne).

> INCONNU : registre par registre, il faudrait dérouler chaque `registerFunction()` pour produire un tableau exhaustif. Hors périmètre de cette passe (les sources sont auto-documentées : `registry_v*::registerFunction()`).

## 4. SOAP (legacy)

- Endpoint : `https://<host>/soap.php` ([soap.php:41-67](../../SuiteCRM/soap.php#L41-L67)).
- Namespace WSDL : `http://www.sugarcrm.com/sugarcrm` ([soap.php:65-67](../../SuiteCRM/soap.php#L65-L67)).
- Stack : NuSOAP via `include/nusoap/nusoap.php` ([soap.php:50](../../SuiteCRM/soap.php#L50)) puis enregistrement des services `SoapSugarUsers`, `SoapData`, `SoapDeprecated`, plus `SoapPortalUsers` si `portal_on` est activé ([soap.php:70-78](../../SuiteCRM/soap.php#L70-L78)).
- WSDL généré dynamiquement par `configureWSDL()` (URL publique = `$sugar_config['site_url'] . '/soap.php'`).

## 5. JSON-RPC

- Endpoint : entrypoint `json_server` (`?entryPoint=json_server`, mappé `json_server.php`) ; `auth = true` ([entry_point_registry.php:55](../../SuiteCRM/include/MVC/Controller/entry_point_registry.php#L55)).
- Implémentation : `JsonRPCServer` ([service/JsonRPCServer/JsonRPCServer.php](../../SuiteCRM/service/JsonRPCServer/JsonRPCServer.php), instanciation [json_server.php:46-48](../../SuiteCRM/json_server.php#L46-L48)).

## 6. Endpoints utilitaires (entry_point_registry)

55 endpoints accessibles via `index.php?entryPoint=<nom>` ([entry_point_registry.php:45-74](../../SuiteCRM/include/MVC/Controller/entry_point_registry.php#L45-L74)). Sélection des plus visibles :

| `entryPoint` | Fichier cible | Auth | Usage | Preuve |
|---|---|---|---|---|
| `download` | `download.php` | oui | Téléchargement fichier | l.47 |
| `export` | `export.php` | oui | Export | l.48 |
| `vCard` | `vCard.php` | oui | Génération vCard | l.52 |
| `pdf` | `pdf.php` | oui | Génération PDF (TCPDF) | l.53 |
| `minify` | `jssource/minify.php` | oui | Minification JS | l.54 |
| `json_server` | `json_server.php` | oui | JSON-RPC | l.55 |
| `HandleAjaxCall` | `HandleAjaxCall.php` | oui | AJAX interne | l.57 |
| `TreeData` | `TreeData.php` | oui | AJAX arbre | l.58 |
| `image` | `modules/Campaigns/image.php` | **non** | Tracker open campagne | l.59 |
| `campaign_trackerv2` | `modules/Campaigns/Tracker.php` | **non** | Tracker click campagne | l.60 |
| `WebToLeadCapture` | `modules/Campaigns/WebToLeadCapture.php` | **non** | Captation lead via formulaire externe | l.61 |
| `WebToPersonCapture` | `modules/Campaigns/WebToPersonCapture.php` | **non** | Idem pour person | l.62 |
| `removeme` | `modules/Campaigns/RemoveMe.php` | **non** | Désinscription | l.63 |
| `ConfirmOptIn` | `include/entryPointConfirmOptInConnector.php` | **non** | Confirm. opt-in | l.64 |
| `acceptDecline` | `modules/Contacts/AcceptDecline.php` | **non** | Accept/refus invitation contact | l.65 |
| `getImage` | `include/SugarTheme/getImage.php` | **non** | Images thèmes | l.69 |
| `SAML` | `modules/Users/authentication/SAMLAuthenticate/index.php` | **non** | Init flow SAML | l.74 |
| *(et 38 autres)* | … | … | … | [entry_point_registry.php](../../SuiteCRM/include/MVC/Controller/entry_point_registry.php) |

Total : **55** entrées (`grep -c "'file' =>" entry_point_registry.php`).

## 7. Pagination & filtres (V8)

Pagination — paramètres query :

- `page[size]` — taille (int)
- `page[number]` — numéro de page (1-based)

Calcul : `offset = (number - 1) * size` ([ModuleService.php:122](../../SuiteCRM/Api/V8/Service/ModuleService.php#L122)). La métadonnée de pagination et les links HATEOAS sont injectés via `PaginationObjectHelper` ([ModuleService.php:224-231](../../SuiteCRM/Api/V8/Service/ModuleService.php#L224-L231)).

Filtres :

- `filter` — chaîne (forme `<field> = '<value>'` ou expressions composées) injectée comme `WHERE`.
- `sort` — `<field>` ou `-<field>` (desc).
- `deleted` — bool ; cf. flux email cité plus bas.
- Cas spécial e-mail : `email1`/`email2` provoque une jointure `email_addresses` + `email_addr_bean_rel` ([ModuleService.php:131-187](../../SuiteCRM/Api/V8/Service/ModuleService.php#L131-L187)).

## 8. Codes de retour observés (V8)

| Code | Cas | Preuve |
|---|---|---|
| `200` | Lecture, delete OK | [ModuleController.php:40, 60, 117](../../SuiteCRM/Api/V8/Controller/ModuleController.php#L40-L117) |
| `201` | Création, mise à jour | [ModuleController.php:79, 98](../../SuiteCRM/Api/V8/Controller/ModuleController.php#L79-L98) |
| `400` | `\Exception` côté service (validation, ACL, conflit, etc.) | [ModuleController.php:42, 62, 81, 100, 119](../../SuiteCRM/Api/V8/Controller/ModuleController.php#L42-L119) |
| `401`/`403` | Token invalide / scope refusé (middleware OAuth2) | [middlewares.php:95-111](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L95-L111) |
| `404` | Route inconnue (Slim default) | INCONNU dans la config locale |
