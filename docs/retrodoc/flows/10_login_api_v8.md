# Flow 10 — Login + appel API V8 (OAuth2 password grant)

> Phase **Writer**. Toutes les étapes sont prouvées par `fichier:ligne`. Diagramme de séquence → [diagrams/mermaid_sequences.md](../diagrams/mermaid_sequences.md).

## 1. Acteurs

- **Client** (mobile, intégrateur, script) qui appelle l'API V8.
- **Apache / Nginx + PHP-FPM** : serveur web. Lecture de `Authorization:` ou `REDIRECT_HTTP_AUTHORIZATION`.
- **Slim 3 App** (`Api/Core/app.php`) + middlewares OAuth2.
- **`league/oauth2-server`** (Authorization & Resource Server).
- **DB** SuiteCRM (tables `users`, `oauth2clients`, `oauth2tokens`).

## 2. Pré-requis vérifiables dans le code

- Clé privée RSA `Api/V8/OAuth2/private.key` accessible avec mode `0600` (sauf Windows). Preuve [Api/V8/Config/services/middlewares.php:49-53, 29](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L29-L53).
- Clé publique RSA `Api/V8/OAuth2/public.key`. Preuve [middlewares.php:105-109](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L105-L109).
- `$sugar_config['oauth2_encryption_key']` défini (sinon fallback `SCRM-DEFK` + log fatal). Preuve [middlewares.php:31-37](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L31-L37).
- Un client OAuth2 créé en BDD (module `OAuth2Clients`).
- Un utilisateur Sugar actif (statut `Active`, `external_auth_only=0`) ([modules/Users/](../../SuiteCRM/modules/Users/)).

## 3. Étapes

### 3.1 Obtention de l'access token

```
Client ──► POST https://<host>/Api/access_token
            Content-Type: application/x-www-form-urlencoded
            grant_type=password
            client_id=<id>&client_secret=<secret>
            username=<user>&password=<pwd>
```

1. Apache/Nginx route vers `Api/index.php` ([Api/index.php:1-5](../../SuiteCRM/Api/index.php#L1-L5)).
2. `Api/Core/app.php` :
   - Pose les CORS headers ([app.php:3-5](../../SuiteCRM/Api/Core/app.php#L3-L5))
   - Reconstruit `HTTP_AUTHORIZATION` si PHP-FPM ([app.php:16-18](../../SuiteCRM/Api/Core/app.php#L16-L18))
   - `require_once 'include/entryPoint.php'` (bootstrap globaux SuiteCRM)
   - Instancie `new \Slim\App(ContainerLoader::configure())` et appelle `RouteLoader::configureRoutes($app)` ([app.php:23-26](../../SuiteCRM/Api/Core/app.php#L23-L26)).
3. Slim matche `POST /access_token` ; **`AuthorizationServerMiddleware`** est ajouté à la route ([routes.php:16-18](../../SuiteCRM/Api/V8/Config/routes.php#L16-L18)). Le middleware appelle le serveur d'autorisation `League\OAuth2\Server\AuthorizationServer`.
4. Le serveur :
   - Lit `client_id` / `client_secret` → `ClientRepository::getClientEntity(...)` ([Api/V8/OAuth2/Repository/ClientRepository.php](../../SuiteCRM/Api/V8/OAuth2/Repository/ClientRepository.php))
   - Authentifie l'utilisateur (`username`/`password`) via `UserRepository::getUserEntityByUserCredentials(...)` ([UserRepository.php](../../SuiteCRM/Api/V8/OAuth2/Repository/UserRepository.php))
   - Crée et signe (JWT RS256, clé privée RSA) un `AccessToken` (TTL `PT1H`) et un `RefreshToken` (TTL `P1M`) ([middlewares.php:64-81](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L64-L81))
   - Persiste via `AccessTokenRepository::persistNewAccessToken` et `RefreshTokenRepository::persistNewRefreshToken` ([Api/V8/OAuth2/Repository/AccessTokenRepository.php](../../SuiteCRM/Api/V8/OAuth2/Repository/AccessTokenRepository.php), [RefreshTokenRepository.php](../../SuiteCRM/Api/V8/OAuth2/Repository/RefreshTokenRepository.php))
5. Réponse 200 :
   ```json
   { "token_type":"Bearer", "expires_in":3600, "access_token":"<jwt>", "refresh_token":"<refresh>" }
   ```

### 3.2 Appel d'une ressource

```
Client ──► GET https://<host>/Api/V8/module/Accounts/<id>
            Authorization: Bearer <jwt>
            Accept: application/vnd.api+json
```

1. Apache → `Api/index.php` → Slim.
2. Slim matche `GET /V8/module/{moduleName}/{id}`.
3. **`ResourceServerMiddleware`** ([routes.php:131](../../SuiteCRM/Api/V8/Config/routes.php#L131)) :
   - Lit le bearer dans `Authorization` (ou `REDIRECT_HTTP_AUTHORIZATION`).
   - Vérifie la signature JWT avec la **clé publique RSA** (`Api/V8/OAuth2/public.key`).
   - Vérifie expiration + revoked via `AccessTokenRepository::isAccessTokenRevoked` ([AccessTokenRepository.php](../../SuiteCRM/Api/V8/OAuth2/Repository/AccessTokenRepository.php)).
   - Injecte `oauth_user_id` dans la requête Slim.
4. `ParamsMiddlewareFactory::bind(GetModuleParams::class)` valide les params via `Symfony\OptionsResolver` ([routes.php:64](../../SuiteCRM/Api/V8/Config/routes.php#L64)).
5. Contrôleur `ModuleController::getModuleRecord` :
   - Appelle `ModuleService::getRecord($params, $request->getUri()->getPath())` ([ModuleController.php:36-43](../../SuiteCRM/Api/V8/Controller/ModuleController.php#L36-L43)).
6. `ModuleService::getRecord` :
   - `BeanManager::getBeanSafe(moduleName, id)` → SugarBean chargé en DB ([Api/V8/Service/ModuleService.php:77-80](../../SuiteCRM/Api/V8/Service/ModuleService.php#L77-L80))
   - `bean->ACLAccess('view')` → si refusé, `AccessDeniedException` ([ModuleService.php:82-84](../../SuiteCRM/Api/V8/Service/ModuleService.php#L82-L84))
   - `getDataResponse(bean, fields, path)` construit l'objet JSON:API (attributes, relationships) via `AttributeObjectHelper`, `RelationshipObjectHelper`.
7. `BaseController::generateResponse` :
   - Status 200
   - Headers `Accept`/`Content-type` `application/vnd.api+json`
   - JSON encodé (`JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES`) ([BaseController.php:19-34](../../SuiteCRM/Api/V8/Controller/BaseController.php#L19-L34))

### 3.3 Échec d'auth

- Bearer absent / invalide : `ResourceServerMiddleware` renvoie 401 (réponse OAuth2 standard de la lib).
- Bearer ok mais ACL refuse : `AccessDeniedException` → bloc `catch (\Exception)` du contrôleur → `generateErrorResponse(..., 400)` avec body JSON:API d'erreur ([BaseController.php:43-51](../../SuiteCRM/Api/V8/Controller/BaseController.php#L43-L51), [Api/V8/JsonApi/Response/ErrorResponse.php:142-158](../../SuiteCRM/Api/V8/JsonApi/Response/ErrorResponse.php#L142-L158)).

### 3.4 Refresh

```
POST /Api/access_token
grant_type=refresh_token
client_id=...&client_secret=...
refresh_token=<refresh>
```
Le `RefreshTokenGrant` est activé ([middlewares.php:72-81](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L72-L81)).

### 3.5 Logout

`POST /Api/V8/logout` → `LogoutController` → `LogoutService` ([routes.php:26](../../SuiteCRM/Api/V8/Config/routes.php#L26), [LogoutService.php](../../SuiteCRM/Api/V8/Service/LogoutService.php)).

## 4. Effets de bord prouvés

- **Logs** : un log `fatal` apparaît dans `monolog` si `oauth2_encryption_key` manque ([middlewares.php:34-36](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L34-L36)).
- **Audit** : aucun audit applicatif du login API détecté ; seul un journal applicatif (`$GLOBALS['log']`) via `LoggerManager` est utilisé ([Api/V8/Service/ModuleService.php:22](../../SuiteCRM/Api/V8/Service/ModuleService.php#L22)).
- **Sessions** : la stack V8 *ne* crée *pas* de session PHP — l'API est stateless (Bearer).

## 5. Schéma simplifié

```
 Client                  Slim/V8                   League/OAuth2-Server          DB
   │  POST /access_token   │                                  │                    │
   ├──────────────────────►│                                  │                    │
   │                       │  AuthorizationServerMiddleware   │                    │
   │                       ├─────────────────────────────────►│                    │
   │                       │                                  │  ClientRepository  │
   │                       │                                  ├───────────────────►│ oauth2clients
   │                       │                                  │  UserRepository    │
   │                       │                                  ├───────────────────►│ users
   │                       │                                  │  AccessToken save  │
   │                       │                                  ├───────────────────►│ oauth2tokens
   │  200 {access_token,…} │                                  │                    │
   ◄───────────────────────┤                                  │                    │
   │                       │                                  │                    │
   │  GET /V8/module/...   │                                  │                    │
   ├──────────────────────►│                                  │                    │
   │                       │  ResourceServerMiddleware        │                    │
   │                       ├─ vérifie JWT + revoked ─────────►│ AccessTokenRepo    │
   │                       │  ModuleController::getModuleRecord                    │
   │                       │  ModuleService::getRecord                             │
   │                       │  BeanManager::getBeanSafe ─────► DBManager ──────────►│ <bean table>
   │                       │  bean->ACLAccess('view')                              │
   │  200 vnd.api+json     │                                                       │
   ◄───────────────────────┤                                                       │
```
