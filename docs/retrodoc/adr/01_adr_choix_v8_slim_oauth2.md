# ADR 01 — Stack API V8 : Slim 3 + League OAuth2 + JSON:API

## Statut
Acceptée *(décision lisible dans le code livré, version 7.15.1)*.

## Contexte

SuiteCRM possédait historiquement :

- une UI MVC Smarty (`index.php` + `include/MVC/`),
- des APIs REST/SOAP/JSON-RPC anciennes héritées de SugarCRM (`service/v2…v4_1/`, `soap.php`, `json_server.php`),
- une authentification par session PHP (cookies) ou tokens zf1-OAuth (legacy).

La version 7.15.1 expose **en plus** une API moderne « V8 » sous `Api/V8/`.

Preuves : [Api/Core/Config/ApiConfig.php:7-21](../../SuiteCRM/Api/Core/Config/ApiConfig.php#L7-L21) ; coexistence des stacks dans [service/v4_1/](../../SuiteCRM/service/v4_1/), [soap.php:50-67](../../SuiteCRM/soap.php#L50-L67), [json_server.php:46-48](../../SuiteCRM/json_server.php#L46-L48).

## Décision implicite

L'équipe SuiteCRM a choisi :

| Choix | Détail | Preuve |
|---|---|---|
| Routeur **Slim 3** | `^3.8` dans composer | [composer.json:61](../../SuiteCRM/composer.json#L61), instancié [Api/Core/app.php:23-26](../../SuiteCRM/Api/Core/app.php#L23-L26) |
| Spec **JSON:API** | Media type `application/vnd.api+json` | [Api/V8/Controller/BaseController.php:10](../../SuiteCRM/Api/V8/Controller/BaseController.php#L10) |
| Serveur OAuth2 standard | `league/oauth2-server ^8.5` | [composer.json:53](../../SuiteCRM/composer.json#L53), [middlewares.php:13-22](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L13-L22) |
| Grants activés | `password`, `client_credentials`, `refresh_token`, `authorization_code` | [middlewares.php:58-91](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L58-L91) |
| Container DI | injection via `Api\Core\Loader\ContainerLoader` + 8 fichiers thématiques | [Api/V8/Config/services.php:8-28](../../SuiteCRM/Api/V8/Config/services.php#L8-L28) |
| Conservation des stacks legacy | Aucun retrait : SOAP, REST v2…v4_1, JSON-RPC restent actifs | [composer.json:75](../../SuiteCRM/composer.json#L75) (legacy `zf1s/zend-oauth`) |

## Conséquences observées

✅ Avantages constatables :

- **Spec d'auth standard** (RFC 6749) : interopérabilité avec n'importe quel client OAuth2.
- **JSON:API** : structure formalisée → moins d'allers-retours sur le format des réponses.
- **Container DI** + tests unitaires possibles (`tests/unit/`).
- **Découplage** : le code domaine (SugarBean) reste inchangé ; seule la couche API V8 ajoute des Service + Helper.

⚠️ Coûts constatables :

- **Cohabitation** de 4 stacks d'API → surface d'attaque accrue (CORS large + entry-points `auth=false`) — [Api/Core/app.php:3-5](../../SuiteCRM/Api/Core/app.php#L3-L5), [entry_point_registry.php](../../SuiteCRM/include/MVC/Controller/entry_point_registry.php).
- **Slim 3** est une version ancienne du framework (`^3.8`). Migration Slim 4 non amorcée dans ce repo.
- **Clé `oauth2_encryption_key`** : fallback hard-codé `SCRM-DEFK` si non configurée — risque silencieux ([middlewares.php:31-37](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L31-L37)).
- Double persistance OAuth (modules `OAuth2*` pour V8 + `OAuthKeys`/`OAuthTokens` pour zf1) — complexité opérationnelle.

## Recommandations issues du code

- Restreindre CORS via reverse proxy (cf. [runbook §9](../runbook/README.md#9-securite-dexploitation)).
- Définir explicitement `oauth2_encryption_key` en `config.php`.
- Désactiver l'accès HTTP aux stacks legacy non utilisées (`.htaccess` deny pour `soap.php`, `service/`, `json_server.php`).
