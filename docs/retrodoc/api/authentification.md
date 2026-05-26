# API — Authentification

> Phase **Writer**. Documente les flux d'authentification *prouvés par le code*.

## 1. Schémas disponibles

| Schéma | Stack consommatrice | Lib | Preuve |
|---|---|---|---|
| **OAuth2** (Bearer) | API V8 | `league/oauth2-server ^8.5` | [composer.json:53](../../SuiteCRM/composer.json#L53), [Api/V8/Config/services/middlewares.php:13-22](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L13-L22) |
| Session Sugar (cookies PHP) | UI Sugar (`index.php`) ; APIs legacy (`soap.php`, `service/v*/rest.php`, `json_server.php`) | natif PHP | [include/MVC/SugarApplication.php:108-122](../../SuiteCRM/include/MVC/SugarApplication.php#L108-L122) |
| SAML 2.0 | UI / SSO | `onelogin/php-saml ^4` | [composer.json:57](../../SuiteCRM/composer.json#L57), [modules/Users/authentication/SAML2Authenticate/SAML2Authenticate.php:71](../../SuiteCRM/modules/Users/authentication/SAML2Authenticate/SAML2Authenticate.php#L71) |
| LDAP / AD | UI | natif PHP LDAP | [modules/Users/authentication/LDAPAuthenticate/](../../SuiteCRM/modules/Users/authentication/LDAPAuthenticate/) |
| Email | UI | natif | [modules/Users/authentication/EmailAuthenticate/](../../SuiteCRM/modules/Users/authentication/EmailAuthenticate/) |
| Sugar natif | UI | hash en BDD | [modules/Users/authentication/SugarAuthenticate/](../../SuiteCRM/modules/Users/authentication/SugarAuthenticate/) |

Le sélecteur central est `AuthenticationController` ([modules/Users/authentication/AuthenticationController.php:47, 66, 76, 114](../../SuiteCRM/modules/Users/authentication/AuthenticationController.php#L47-L114)).

## 2. OAuth2 — détail

### 2.1 Grants activés

| Grant | TTL access_token | TTL refresh_token | Preuve |
|---|---|---|---|
| `client_credentials` | `PT1H` (1h) | — | [middlewares.php:58-61](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L58-L61) |
| `password` | `PT1H` | `P1M` (1 mois) | [middlewares.php:64-81](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L64-L81) |
| `refresh_token` | `PT1H` | rotatif (`P1M`) | [middlewares.php:72-81](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L72-L81) |
| `authorization_code` | `PT1H` | — (auth code valide `PT10M`) | [middlewares.php:83-91](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L83-L91) |

### 2.2 Clés & secrets

| Paramètre | Source | Preuve |
|---|---|---|
| Clé privée RSA | `Api/V8/OAuth2/private.key` (lue via `CryptKey`) | [middlewares.php:49-53](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L49-L53), [ApiConfig::OAUTH2_PRIVATE_KEY](../../SuiteCRM/Api/Core/Config/ApiConfig.php#L20) |
| Clé publique RSA | `Api/V8/OAuth2/public.key` | [middlewares.php:105-109](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L105-L109), [ApiConfig::OAUTH2_PUBLIC_KEY](../../SuiteCRM/Api/Core/Config/ApiConfig.php#L21) |
| Clé chiffrement | `$sugar_config['oauth2_encryption_key']` (fallback **`SCRM-DEFK`** + log `fatal`) | [middlewares.php:31-37](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L31-L37) |
| Vérification permissions FS | `chmod 600` requis sauf Windows | [middlewares.php:29, 98](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L29-L98) (via `OsHelper::OS_WINDOWS`) |

⚠ **Note de sécurité prouvée par le code** : si `oauth2_encryption_key` n'est pas défini en config, SuiteCRM utilise la chaîne `SCRM-DEFK` et émet un log `fatal` — il faut impérativement configurer cette clé en production. Preuve : [middlewares.php:31-37](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L31-L37).

### 2.3 Flow détaillé — Password grant

```
Client                                Slim/OAuth2                    Sugar DB
  │   POST /Api/access_token             │                              │
  │   grant_type=password                │                              │
  │   client_id=&client_secret=          │                              │
  │   username=&password=                │                              │
  ├──────────────────────────────────────►                              │
  │                                       │  ClientRepository::lookup    │
  │                                       ├─────────────────────────────►│  table oauth2clients
  │                                       │  UserRepository::lookup      │
  │                                       ├─────────────────────────────►│  table users
  │                                       │  AccessTokenRepository::save │
  │                                       ├─────────────────────────────►│  table oauth2tokens
  │                                       │  RefreshTokenRepository::save│
  │                                       ├─────────────────────────────►│  idem
  │   200 { access_token, refresh_token,  │                              │
  │         token_type=Bearer, expires_in}│                              │
  ◄───────────────────────────────────────┤                              │
```
Preuves : [middlewares.php:39-91](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L39-L91), [Api/V8/OAuth2/Repository/](../../SuiteCRM/Api/V8/OAuth2/Repository/), modules associés `OAuth2Clients`, `OAuth2Tokens`, `OAuth2AuthCodes` ([modules/](../../SuiteCRM/modules/)).

### 2.4 Flow détaillé — Authorization Code grant

1. Client redirige l'utilisateur vers le serveur d'autorisation (`AuthorizationServer`) avec `response_type=code`, `client_id`, `redirect_uri`, `state` (paramètres standards RFC 6749).
2. SuiteCRM affiche l'écran de consentement (UI Sugar — INCONNU précis dans le repo : la page d'autorisation n'a pas de route Slim dédiée visible dans `routes.php`).
3. Émission d'un `auth_code` stocké via `AuthCodeRepository` ([Api/V8/OAuth2/Repository/AuthCodeRepository.php](../../SuiteCRM/Api/V8/OAuth2/Repository/AuthCodeRepository.php)).
4. Client échange `auth_code` contre `access_token` via `POST /access_token` (grant `authorization_code`).
5. `auth_code` valide `PT10M` ([middlewares.php:88](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L88)).

### 2.5 Utilisation du token

Chaque appel `/Api/V8/...` doit porter `Authorization: Bearer <access_token>`.

Pour PHP-FPM, SuiteCRM transforme `REDIRECT_HTTP_AUTHORIZATION` (passé via .htaccess rewrite) en `HTTP_AUTHORIZATION` :

> *« For php-fpm we pass the "Authorization" header through HTTP_AUTHORIZATION using .htaccess rewrite rules. »* — [Api/Core/app.php:13-18](../../SuiteCRM/Api/Core/app.php#L13-L18).

### 2.6 Révocation / Logout

`POST /Api/V8/logout` → `LogoutController` → `LogoutService` ([routes.php:26](../../SuiteCRM/Api/V8/Config/routes.php#L26), [Api/V8/Service/LogoutService.php](../../SuiteCRM/Api/V8/Service/LogoutService.php)).

## 3. Session Sugar (UI + APIs legacy)

Cycle :

1. UI : `index.php?module=Users&action=Authenticate` (`POST` user/password) → `SugarApplication::loadUser()` ([SugarApplication.php:108](../../SuiteCRM/include/MVC/SugarApplication.php#L108)).
2. Création d'une session PHP avec `$_SESSION['unique_key']` comparée à `$sugar_config['unique_key']` pour détecter le hijacking ([SugarApplication.php:112-120](../../SuiteCRM/include/MVC/SugarApplication.php#L112-L120)).
3. API legacy : `login(user, password)` retourne un `session_id` passé dans chaque appel JSON-RPC / REST / SOAP.

Particularité : `login_error` court-circuite la détruction de session ([SugarApplication.php:119-120](../../SuiteCRM/include/MVC/SugarApplication.php#L119-L120)).

## 4. SAML 2.0

- Entrypoint init : `?entryPoint=SAML` (`auth = false`) → [modules/Users/authentication/SAMLAuthenticate/index.php](../../SuiteCRM/modules/Users/authentication/SAMLAuthenticate/index.php) ([entry_point_registry.php:74](../../SuiteCRM/include/MVC/Controller/entry_point_registry.php#L74)).
- Implémentation : `SAML2Authenticate extends SugarAuthenticate` ([SAML2Authenticate.php:71-73](../../SuiteCRM/modules/Users/authentication/SAML2Authenticate/SAML2Authenticate.php#L71-L73)).
- Métadonnées : `getSAML2Metadata($settingsInfo)` ([SAML2Authenticate.php:50-54](../../SuiteCRM/modules/Users/authentication/SAML2Authenticate/SAML2Authenticate.php#L50-L54)).
- Sélection à l'exécution : `AuthenticationController::getAuthController($type)` accepte explicitement les sous-classes de `SAMLAuthenticate` et `SAML2Authenticate` ([AuthenticationController.php:99-100](../../SuiteCRM/modules/Users/authentication/AuthenticationController.php#L99-L100)).

## 5. LDAP / AD

- `LDAPAuthenticate` + `LDAPAuthenticateUser` ([modules/Users/authentication/LDAPAuthenticate/](../../SuiteCRM/modules/Users/authentication/LDAPAuthenticate/)).
- Configuration : `modules/Users/authentication/LDAPAuthenticate/LDAPConfigs/`.
- Active si `$sugar_config['authenticationClass']` ou `Users::$default_auth_type` = `LDAPAuthenticate` (INCONNU : valeur effective définie à l'install/Admin).

## 6. Sécurité transverses

| Garde | Localisation | Preuve |
|---|---|---|
| `checkHTTPReferer` | UI Sugar (`SugarApplication`) | [SugarApplication.php:93](../../SuiteCRM/include/MVC/SugarApplication.php#L93) |
| `ACLFilter` | UI Sugar | [SugarApplication.php:90](../../SuiteCRM/include/MVC/SugarApplication.php#L90) |
| `unique_key` session vs config | Protection hijack | [SugarApplication.php:112-120](../../SuiteCRM/include/MVC/SugarApplication.php#L112-L120) |
| `ACLAccess($action)` par bean | API V8 (`getRecord`, `getRecords`, `createRecord`, `updateRecord`) | [ModuleService.php:82-84, 117-119, 264-266](../../SuiteCRM/Api/V8/Service/ModuleService.php#L82-L266) |
| Sanitization | HTMLPurifier + `voku/anti-xss` | [composer.json:45, 71](../../SuiteCRM/composer.json#L45-L71), [include/HtmlSanitizer.php](../../SuiteCRM/include/HtmlSanitizer.php) |
| reCAPTCHA | login + entry points exposés | [composer.json:47](../../SuiteCRM/composer.json#L47) |
| CORS API V8 | `Allow-Origin: *` (à restreindre par reverse proxy en prod) | [Api/Core/app.php:3-5](../../SuiteCRM/Api/Core/app.php#L3-L5) |

## 7. Points d'attention sécurité

- La clé `oauth2_encryption_key` doit être positionnée explicitement dans `config.php` (sinon fallback `SCRM-DEFK` + log fatal — [middlewares.php:31-37](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L31-L37)).
- Les fichiers `Api/V8/OAuth2/private.key` / `public.key` doivent exister avec permissions `0600` sur Unix (`OsHelper::OS_WINDOWS` désactive la vérif). Preuve : [middlewares.php:29, 98](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L29-L98).
- L'API V8 met `Access-Control-Allow-Origin: *` ([Api/Core/app.php:3](../../SuiteCRM/Api/Core/app.php#L3)) ; en production : restreindre via reverse proxy.
- Les entry-points marqués `auth = false` ([entry_point_registry.php](../../SuiteCRM/include/MVC/Controller/entry_point_registry.php)) sont volontairement publics : Web-to-Lead, trackers de campagne, opt-in, etc.
