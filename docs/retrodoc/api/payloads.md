# API — Payloads

> Phase **Writer**. Exemples de requêtes/réponses **ancrés** dans le code (params + helpers JSON:API + structure d'erreur).

## 1. Media type & structure JSON:API

- Toutes les requêtes/réponses V8 utilisent `application/vnd.api+json` ([Api/V8/Controller/BaseController.php:10](../../SuiteCRM/Api/V8/Controller/BaseController.php#L10)).
- Encodage : `JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES` ([BaseController.php:28-32](../../SuiteCRM/Api/V8/Controller/BaseController.php#L28-L32)).
- Helpers de construction :
  - `AttributeObjectHelper` ([Api/V8/JsonApi/Helper/AttributeObjectHelper.php](../../SuiteCRM/Api/V8/JsonApi/Helper/AttributeObjectHelper.php))
  - `RelationshipObjectHelper` ([Api/V8/JsonApi/Helper/RelationshipObjectHelper.php](../../SuiteCRM/Api/V8/JsonApi/Helper/RelationshipObjectHelper.php))
  - `PaginationObjectHelper` ([Api/V8/JsonApi/Helper/PaginationObjectHelper.php](../../SuiteCRM/Api/V8/JsonApi/Helper/PaginationObjectHelper.php))

## 2. Tous les paramètres typés (Slim params)

Source : [Api/V8/Param/](../../SuiteCRM/Api/V8/Param/). Chaque classe `*Params` est validée par `Symfony\OptionsResolver` au niveau du `ParamsMiddlewareFactory::bind(...)` ([Api/V8/Factory/ParamsMiddlewareFactory.php](../../SuiteCRM/Api/V8/Factory/ParamsMiddlewareFactory.php)).

| Endpoint | Classe params | Champs reconnus |
|---|---|---|
| `GET /V8/module/{moduleName}/{id}` | `GetModuleParams` | `moduleName`, `id`, `fields` | [GetModuleParams.php:1-48](../../SuiteCRM/Api/V8/Param/GetModuleParams.php#L1-L48) |
| `GET /V8/module/{moduleName}` | `GetModulesParams` | `moduleName`, `fields`, `filter`, `sort`, `page` (size, number), `deleted` | [GetModulesParams.php](../../SuiteCRM/Api/V8/Param/GetModulesParams.php) |
| `POST /V8/module` | `CreateModuleParams` → `CreateModuleDataParams` | `data: { type, id?, attributes }` | [CreateModuleParams.php](../../SuiteCRM/Api/V8/Param/CreateModuleParams.php), [CreateModuleDataParams.php](../../SuiteCRM/Api/V8/Param/CreateModuleDataParams.php) |
| `PATCH /V8/module` | `UpdateModuleParams` → `UpdateModuleDataParams` | `data: { type, id, attributes }` | [UpdateModuleParams.php](../../SuiteCRM/Api/V8/Param/UpdateModuleParams.php), [UpdateModuleDataParams.php](../../SuiteCRM/Api/V8/Param/UpdateModuleDataParams.php) |
| `DELETE /V8/module/{moduleName}/{id}` | `DeleteModuleParams` | `moduleName`, `id` | [DeleteModuleParams.php](../../SuiteCRM/Api/V8/Param/DeleteModuleParams.php) |
| `GET /V8/module/{m}/{id}/relationships/{linkFieldName}` | `GetRelationshipParams` → `GetRelationshipDataParams` | `moduleName`, `id`, `linkFieldName` | [GetRelationshipParams.php](../../SuiteCRM/Api/V8/Param/GetRelationshipParams.php), [GetRelationshipDataParams.php](../../SuiteCRM/Api/V8/Param/GetRelationshipDataParams.php) |
| `POST /V8/module/{m}/{id}/relationships` | `CreateRelationshipParams` | `data: { type, id }` | [CreateRelationshipParams.php](../../SuiteCRM/Api/V8/Param/CreateRelationshipParams.php) |
| `POST /V8/module/{m}/{id}/relationships/{linkFieldName}` | `CreateRelationshipByLinkParams` | + `linkFieldName` | [CreateRelationshipByLinkParams.php](../../SuiteCRM/Api/V8/Param/CreateRelationshipByLinkParams.php) |
| `DELETE /V8/module/{m}/{id}/relationships/{linkFieldName}/{relatedBeanId}` | `DeleteRelationshipParams` | + `relatedBeanId` | [DeleteRelationshipParams.php](../../SuiteCRM/Api/V8/Param/DeleteRelationshipParams.php) |
| `GET /V8/user-preferences/{id}` | `GetUserPreferencesParams` | `id` | [GetUserPreferencesParams.php](../../SuiteCRM/Api/V8/Param/GetUserPreferencesParams.php) |
| `GET /V8/listview/columns/{moduleName}` | `ListViewColumnsParams` | `moduleName` | [ListViewColumnsParams.php](../../SuiteCRM/Api/V8/Param/ListViewColumnsParams.php) |
| `GET /V8/search-defs/module/{moduleName}` | `ListViewSearchParams` | `moduleName` | [ListViewSearchParams.php](../../SuiteCRM/Api/V8/Param/ListViewSearchParams.php) |
| `GET /V8/meta/fields/{moduleName}` | `GetFieldListParams` | `moduleName` | [GetFieldListParams.php](../../SuiteCRM/Api/V8/Param/GetFieldListParams.php) |

## 3. Exemples concrets

### 3.1 Obtenir un access token (Password grant)

**Requête**

```http
POST /Api/access_token HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded

grant_type=password&client_id=sugar&client_secret=s3cret&username=admin&password=admin
```

**Réponse (200)** — structure standard de `league/oauth2-server` :

```json
{
  "token_type": "Bearer",
  "expires_in": 3600,
  "access_token": "<jwt>",
  "refresh_token": "<refresh>"
}
```

> Détails TTL → [authentification.md §2.1](authentification.md#21-grants-actives). Preuve serveur : [middlewares.php:64-71](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L64-L71).

### 3.2 Lire un Account

**Requête**

```http
GET /Api/V8/module/Accounts/9b...uuid HTTP/1.1
Authorization: Bearer <jwt>
Accept: application/vnd.api+json
```

**Réponse (200)** — pattern produit par `ModuleService::getDataResponse(...)` ([Api/V8/Service/ModuleService.php:74-92](../../SuiteCRM/Api/V8/Service/ModuleService.php#L74-L92)) :

```json
{
  "data": {
    "type": "Accounts",
    "id": "9b...uuid",
    "attributes": {
      "name": "ACME",
      "industry": "Technology",
      "billing_address_country": "France"
    },
    "relationships": {
      "contacts": { "links": { "related": "/V8/module/Accounts/9b...uuid/relationships/contacts" } }
    }
  }
}
```

### 3.3 Lister des Contacts (pagination + filtre)

**Requête**

```http
GET /Api/V8/module/Contacts?page[size]=20&page[number]=2&filter[account_id]=<uuid>&sort=last_name HTTP/1.1
Authorization: Bearer <jwt>
Accept: application/vnd.api+json
```

**Réponse (200)** — avec `meta` (pagination) et `links` (HATEOAS) ([ModuleService.php:224-231](../../SuiteCRM/Api/V8/Service/ModuleService.php#L224-L231)) :

```json
{
  "data": [
    { "type": "Contacts", "id": "...", "attributes": { "first_name": "Jane", "last_name": "Doe", "email1": "jane@acme.com" } },
    "..."
  ],
  "meta": { "total-pages": 5 },
  "links": {
    "self":  "/Api/V8/module/Contacts?page[number]=2",
    "first": "/Api/V8/module/Contacts?page[number]=1",
    "prev":  "/Api/V8/module/Contacts?page[number]=1",
    "next":  "/Api/V8/module/Contacts?page[number]=3",
    "last":  "/Api/V8/module/Contacts?page[number]=5"
  }
}
```

### 3.4 Créer un Lead

**Requête**

```http
POST /Api/V8/module HTTP/1.1
Authorization: Bearer <jwt>
Content-Type: application/vnd.api+json

{
  "data": {
    "type": "Leads",
    "attributes": {
      "first_name": "John",
      "last_name": "Smith",
      "email1": "john@example.com",
      "status": "New",
      "lead_source": "Web Site"
    }
  }
}
```

**Réponse (201)** ([ModuleService::createRecord](../../SuiteCRM/Api/V8/Service/ModuleService.php#L246-L295)) :

```json
{
  "data": {
    "type": "Leads",
    "id": "<new uuid>",
    "attributes": { "first_name": "John", "last_name": "Smith", "email1": "john@example.com", "status": "New", "lead_source": "Web Site" },
    "relationships": { ... }
  }
}
```

### 3.5 Mettre à jour (PATCH)

```http
PATCH /Api/V8/module HTTP/1.1
Authorization: Bearer <jwt>
Content-Type: application/vnd.api+json

{
  "data": {
    "type": "Opportunities",
    "id": "<uuid>",
    "attributes": { "sales_stage": "Closed Won", "amount": 25000 }
  }
}
```
Réponse `201` avec data complet (cf. createRecord, [ModuleController.php:98](../../SuiteCRM/Api/V8/Controller/ModuleController.php#L98)).

### 3.6 Supprimer (DELETE)

```http
DELETE /Api/V8/module/Cases/<uuid> HTTP/1.1
Authorization: Bearer <jwt>
```
Réponse `200`. Le bean est soft-deleted (`deleted = 1`) — convention SugarBean.

### 3.7 Créer une relation par link

```http
POST /Api/V8/module/Accounts/<accId>/relationships/contacts HTTP/1.1
Authorization: Bearer <jwt>
Content-Type: application/vnd.api+json

{
  "data": { "type": "Contacts", "id": "<contactId>" }
}
```
Routé via `CreateRelationshipByLinkParams` ([routes.php:111-114](../../SuiteCRM/Api/V8/Config/routes.php#L111-L114)).

## 4. Format d'erreur (V8)

Source : [Api/V8/JsonApi/Response/ErrorResponse.php:142-158](../../SuiteCRM/Api/V8/JsonApi/Response/ErrorResponse.php#L142-L158).

**Exemple — sans debug** (`ApiConfig::getDebugExceptions() === false`, par défaut) :

```json
{
  "errors": {
    "status": 400,
    "title": null,
    "detail": "Bean Accounts with id 9b...uuid is already exist"
  }
}
```

**Exemple — avec debug** (clé `exception` ajoutée) :

```json
{
  "errors": {
    "status": 400,
    "title": null,
    "detail": "...",
    "exception": {
      "code": 0,
      "file": "...",
      "line": 256,
      "message": "...",
      "previous": null,
      "trace": [ ... ],
      "traceAsString": "..."
    }
  }
}
```

> Le flag de debug est statique : `ApiConfig::$debugExceptions = false` par défaut ([ApiConfig.php:27](../../SuiteCRM/Api/Core/Config/ApiConfig.php#L27)). Aucune route ne le change publiquement — il faut modifier le code pour activer.

## 5. Exceptions levées explicitement

| Exception | Cause | Preuve |
|---|---|---|
| `\InvalidArgumentException` | Création d'un Bean avec `id` déjà existant | [ModuleService.php:253-260](../../SuiteCRM/Api/V8/Service/ModuleService.php#L253-L260) |
| `SuiteCRM\Exception\AccessDeniedException` | ACL refuse l'accès | [ModuleService.php:82-84, 117-119, 264-266](../../SuiteCRM/Api/V8/Service/ModuleService.php#L82-L266) |
| `\Exception` (générique) | Catch global du contrôleur → 400 | [ModuleController.php:42, 62, 81, 100, 119](../../SuiteCRM/Api/V8/Controller/ModuleController.php#L42-L119) |

## 6. Notes sur les types de champs (vardefs)

Les `attributes` JSON:API correspondent aux clés des `fields` du vardefs (`<module>/vardefs.php`). Les types les plus fréquents :

| Type vardef | Sens | Exemple |
|---|---|---|
| `id` | UUID v4 (formaté) | `id`, `assigned_user_id`, foreign keys |
| `varchar` | string de longueur configurable | `name`, `first_name`, `email1` |
| `text` | string longue | `description` |
| `enum` | string contraint à une liste (`options => '<dom_list>'`) | `status`, `lead_source`, `sales_stage` |
| `bool` | bool stocké en `0/1` | `deleted`, `do_not_call` |
| `date` / `datetime` | date / datetime | `date_entered`, `date_modified` |
| `int` / `currency` | numériques | `amount`, `number` |
| `relate` / `link` | non-DB, traduit en jointure | `account_name`, `campaign_opportunities` ([modules/Opportunities/vardefs.php:73-138](../../SuiteCRM/modules/Opportunities/vardefs.php#L73-L138)) |

Source : voir vardefs des modules — exemple `Opportunities` ([vardefs.php:47-138](../../SuiteCRM/modules/Opportunities/vardefs.php#L47-L138)), `Accounts` ([vardefs.php:45-200](../../SuiteCRM/modules/Accounts/vardefs.php#L45-L200)).

## 7. Cas particuliers — recherche par e-mail

Si un `filter` contient `email1` ou `email2`, le service bascule sur une SQL avec jointure `email_addresses + email_addr_bean_rel` ([ModuleService.php:131-187](../../SuiteCRM/Api/V8/Service/ModuleService.php#L131-L187)). Côté client, la requête est identique :

```http
GET /Api/V8/module/Contacts?filter[email1]=jane@acme.com
```
Mais en interne SuiteCRM exécute :

```sql
SELECT contacts.id
FROM email_addresses
JOIN email_addr_bean_rel ON email_addresses.id = email_addr_bean_rel.email_address_id
JOIN contacts ON contacts.id = email_addr_bean_rel.bean_id
WHERE email_addresses.email_address = 'jane@acme.com' AND contacts.deleted = 0
```
Preuve : [ModuleService.php:141-170](../../SuiteCRM/Api/V8/Service/ModuleService.php#L141-L170).
