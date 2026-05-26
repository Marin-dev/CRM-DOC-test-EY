# API — Vue d'ensemble

> Phase **Writer**. SuiteCRM expose **plusieurs stacks d'API** historiquement cumulées. Ce dossier détaille principalement la stack moderne **V8 (REST JSON:API, OAuth2)** et liste les stacks legacy pour mémoire.

## 1. Stacks d'API en cohabitation

| Stack | Préfixe / fichier | Format | Auth | Statut | Documentation |
|---|---|---|---|---|---|
| **API V8** (recommandée) | `/Api/V8/...` (front-controller `Api/index.php`) | JSON:API (`application/vnd.api+json`) | OAuth2 Bearer | Active | [endpoints.md](endpoints.md), [authentification.md](authentification.md), [payloads.md](payloads.md) |
| REST legacy v2 → v4_1 | `service/v4_1/rest.php` (et v2…v4) | JSON (POST `method=...`) | Session Sugar (`login` → session_id) | Active mais legacy | [endpoints.md §3](endpoints.md#3-rest-legacy-v21--v41) |
| SOAP | `soap.php` (NuSOAP) | XML SOAP 1.1, namespace `http://www.sugarcrm.com/sugarcrm` | Session Sugar | Legacy | [endpoints.md §4](endpoints.md#4-soap-legacy) |
| JSON-RPC | `json_server.php` (entrypoint `json_server`) | JSON-RPC | Session Sugar | Legacy | [endpoints.md §5](endpoints.md#5-json-rpc) |

Preuves de cohabitation : [Api/Core/Config/ApiConfig.php:7-14](../../SuiteCRM/Api/Core/Config/ApiConfig.php#L7-L14), [soap.php:50-67](../../SuiteCRM/soap.php#L50-L67), [json_server.php:46-48](../../SuiteCRM/json_server.php#L46-L48), [service/v4_1/registry.php:46-79](../../SuiteCRM/service/v4_1/registry.php#L46-L79).

## 2. Documents

- [endpoints.md](endpoints.md) — endpoints V8 (et legacy) avec verbes, paramètres, statuts de retour.
- [authentification.md](authentification.md) — OAuth2 (V8) + session Sugar (legacy) + SAML, LDAP.
- [payloads.md](payloads.md) — exemples requête / réponse JSON:API + erreurs.
- Sequence diagrams → [diagrams/mermaid_sequences.md](../diagrams/mermaid_sequences.md).

## 3. Particularités utiles

- **CORS** large (`Access-Control-Allow-Origin: *`) côté API V8 — [Api/Core/app.php:3-5](../../SuiteCRM/Api/Core/app.php#L3-L5).
- **PHP-FPM** : SuiteCRM lit `REDIRECT_HTTP_AUTHORIZATION` quand `HTTP_AUTHORIZATION` est absent (configurable côté Apache rewrite) — [Api/Core/app.php:16-18](../../SuiteCRM/Api/Core/app.php#L16-L18).
- **Swagger** : un schema JSON est exposé via `GET /V8/meta/swagger.json` — [routes.php:50](../../SuiteCRM/Api/V8/Config/routes.php#L50). Il est généré par `MetaController::getSwaggerSchema`.
- **Erreurs API V8** : structure JSON:API d'erreur via `ErrorResponse` — [Api/V8/Controller/BaseController.php:43-51](../../SuiteCRM/Api/V8/Controller/BaseController.php#L43-L51).
