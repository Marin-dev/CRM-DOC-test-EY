# 20 — Composants Backend (Controllers, Services, Dépendances, Logique métier)

> Phase **Writer**. Inventaire des contrôleurs, services et dépendances par stack. Tout symbole listé est ancré par `fichier:ligne`.

## 1. Contrôleurs

### 1.1 Contrôleurs UI Sugar (MVC interne)

Le dispatcher est `ControllerFactory` → `SugarController` ([include/MVC/Controller/SugarController.php](../../SuiteCRM/include/MVC/Controller/SugarController.php)). Chaque module peut surcharger via un `controller.php` dans `modules/<Module>/`.

| Module (sélection) | Controller | Preuve |
|---|---|---|
| Tous modules par défaut | `SugarController` | [include/MVC/Controller/SugarController.php](../../SuiteCRM/include/MVC/Controller/SugarController.php) |
| Map module → fichier | `entry_point_registry.php`, `action_file_map.php`, `action_view_map.php` | [include/MVC/Controller/entry_point_registry.php](../../SuiteCRM/include/MVC/Controller/entry_point_registry.php), [action_file_map.php](../../SuiteCRM/include/MVC/Controller/action_file_map.php), [action_view_map.php](../../SuiteCRM/include/MVC/Controller/action_view_map.php) |
| Sélection à l'exécution | `ControllerFactory::getController($module)` | [include/MVC/Controller/ControllerFactory.php](../../SuiteCRM/include/MVC/Controller/ControllerFactory.php), invoqué [SugarApplication.php:86](../../SuiteCRM/include/MVC/SugarApplication.php#L86) |

### 1.2 Contrôleurs API V8 (Slim)

Tous étendent `BaseController` ([Api/V8/Controller/BaseController.php:8](../../SuiteCRM/Api/V8/Controller/BaseController.php#L8)) et exposent un media type unique `application/vnd.api+json` ([BaseController.php:10](../../SuiteCRM/Api/V8/Controller/BaseController.php#L10)).

| Controller | Routes (verbe + URL) | Service utilisé | Preuve route | Preuve service |
|---|---|---|---|---|
| `ModuleController` | `GET /V8/module/{moduleName}` ; `GET /V8/module/{moduleName}/{id}` ; `POST /V8/module` ; `PATCH /V8/module` ; `DELETE /V8/module/{moduleName}/{id}` | `ModuleService` | [routes.php:54-86](../../SuiteCRM/Api/V8/Config/routes.php#L54-L86) | [Api/V8/Service/ModuleService.php:28-66](../../SuiteCRM/Api/V8/Service/ModuleService.php#L28-L66) |
| `RelationshipController` | `GET /V8/module/{m}/{id}/relationships/{linkFieldName}` ; `POST /V8/module/{m}/{id}/relationships` ; `POST /V8/module/{m}/{id}/relationships/{linkFieldName}` ; `DELETE /V8/module/{m}/{id}/relationships/{linkFieldName}/{relatedBeanId}` | `RelationshipService` | [routes.php:90-125](../../SuiteCRM/Api/V8/Config/routes.php#L90-L125) | [Api/V8/Service/RelationshipService.php](../../SuiteCRM/Api/V8/Service/RelationshipService.php) |
| `MetaController` | `GET /V8/meta/modules` ; `GET /V8/meta/fields/{moduleName}` ; `GET /V8/meta/swagger.json` | `MetaService` | [routes.php:38-50](../../SuiteCRM/Api/V8/Config/routes.php#L38-L50) | [Api/V8/Service/MetaService.php](../../SuiteCRM/Api/V8/Service/MetaService.php) |
| `UserController` | `GET /V8/current-user` | `UserService` | [routes.php:36](../../SuiteCRM/Api/V8/Config/routes.php#L36) | [Api/V8/Service/UserService.php](../../SuiteCRM/Api/V8/Service/UserService.php) |
| `UserPreferencesController` | `GET /V8/user-preferences/{id}` | `UserPreferencesService` | [routes.php:43-45](../../SuiteCRM/Api/V8/Config/routes.php#L43-L45) | [Api/V8/Service/UserPreferencesService.php](../../SuiteCRM/Api/V8/Service/UserPreferencesService.php) |
| `ListViewController` | `GET /V8/listview/columns/{moduleName}` | `ListViewService` | [routes.php:32-34](../../SuiteCRM/Api/V8/Config/routes.php#L32-L34) | [Api/V8/Service/ListViewService.php](../../SuiteCRM/Api/V8/Service/ListViewService.php) |
| `ListViewSearchController` | `GET /V8/search-defs/module/{moduleName}` | `ListViewSearchService` | [routes.php:28-30](../../SuiteCRM/Api/V8/Config/routes.php#L28-L30) | [Api/V8/Service/ListViewSearchService.php](../../SuiteCRM/Api/V8/Service/ListViewSearchService.php) |
| `LogoutController` | `POST /V8/logout` | `LogoutService` | [routes.php:26](../../SuiteCRM/Api/V8/Config/routes.php#L26) | [Api/V8/Service/LogoutService.php](../../SuiteCRM/Api/V8/Service/LogoutService.php) |
| *(implicite Slim)* `/access_token` | `POST /access_token` géré par `AuthorizationServerMiddleware` | — (middleware uniquement) | [routes.php:16-18](../../SuiteCRM/Api/V8/Config/routes.php#L16-L18) | [middlewares.php:39-93](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L39-L93) |

### 1.3 Endpoints REST historiques (`service/`)

Chaque registre hérite du précédent (v2 ← v2_1 ← v3 ← v3_1 ← v4 ← v4_1) ; v4_1 ajoute notamment `get_relationships` paginé et `get_modified_relationships` ([service/v4_1/registry.php:46-79](../../SuiteCRM/service/v4_1/registry.php#L46-L79)). Le point d'entrée HTTP est `service/v4_1/rest.php` (et homologues `v*/rest.php`).

| Stack | Implémentation | Helper | Preuve |
|---|---|---|---|
| REST v4_1 | `SugarRestService` + `SugarWebServiceImplv4_1` | `SugarRestUtils` | [service/v4_1/rest.php](../../SuiteCRM/service/v4_1/rest.php), [service/v4_1/SugarWebServiceImplv4_1.php](../../SuiteCRM/service/v4_1/SugarWebServiceImplv4_1.php), [service/core/SugarRestUtils.php](../../SuiteCRM/service/core/SugarRestUtils.php) |
| REST helpers | base `SugarRestService` | — | [service/core/SugarRestService.php](../../SuiteCRM/service/core/SugarRestService.php), [service/core/SugarRestServiceImpl.php](../../SuiteCRM/service/core/SugarRestServiceImpl.php) |
| SOAP | `SugarSoapService` (NuSOAP `NusoapSoap.php`, PHP5 `PHP5Soap.php`) | `SoapHelperWebService` | [service/core/SugarSoapService.php](../../SuiteCRM/service/core/SugarSoapService.php), [service/core/NusoapSoap.php](../../SuiteCRM/service/core/NusoapSoap.php), [service/core/PHP5Soap.php](../../SuiteCRM/service/core/PHP5Soap.php), [service/core/SoapHelperWebService.php](../../SuiteCRM/service/core/SoapHelperWebService.php), [service/core/WSDL.tpl](../../SuiteCRM/service/core/WSDL.tpl) |
| JSON-RPC | `JsonRPCServer` | — | [service/JsonRPCServer/JsonRPCServer.php](../../SuiteCRM/service/JsonRPCServer/JsonRPCServer.php), [json_server.php:46-48](../../SuiteCRM/json_server.php#L46-L48) |

## 2. Services & dépendances (API V8)

Diagramme d'injection résumé (services concrets) :

```
ModuleService  ──► BeanManager  ──► DBManager   (via DI container)
              ├──► AttributeObjectHelper
              ├──► RelationshipObjectHelper
              └──► PaginationObjectHelper

RelationshipService ──► BeanManager + helpers
MetaService         ──► BeanManager + module list
UserService         ──► BeanFactory (Users)
ListViewService     ──► BeanManager + ListView defs
ListViewSearchService ──► BeanManager + search defs
LogoutService       ──► AccessTokenRepository (révocation)
```

Constructeurs prouvés :

- `ModuleService::__construct(BeanManager, AttributeObjectHelper, RelationshipObjectHelper, PaginationObjectHelper)` — [Api/V8/Service/ModuleService.php:56-66](../../SuiteCRM/Api/V8/Service/ModuleService.php#L56-L66).
- `ModuleController::__construct(ModuleService)` — [Api/V8/Controller/ModuleController.php:22-26](../../SuiteCRM/Api/V8/Controller/ModuleController.php#L22-L26).
- Wirings DI : voir [Api/V8/Config/services/controllers.php](../../SuiteCRM/Api/V8/Config/services/controllers.php), [services.php](../../SuiteCRM/Api/V8/Config/services/services.php), [helpers.php](../../SuiteCRM/Api/V8/Config/services/helpers.php), [factories.php](../../SuiteCRM/Api/V8/Config/services/factories.php).
- `BeanManager::__construct(DBManager, beanAliases)` — [Api/V8/Config/services.php:13-18](../../SuiteCRM/Api/V8/Config/services.php#L13-L18).

## 3. Services & dépendances (legacy)

| Couche | Service / impl. | Dépendances |
|---|---|---|
| SOAP | `SoapHelperWebService` | `SugarBean`, `BeanFactory`, `DBManagerFactory` |
| REST v4_1 | `SugarWebServiceImplv4_1` | `SugarBean`, `BeanFactory`, `RelationshipFactory`, `SugarBean::process_list_query` |
| JSON-RPC | `JsonRPCServer` | `SugarBean` (mêmes facades que REST) |
| Cron | `SugarCronJobs`, `SugarCronRemoteJobs` | `SugarJobQueue`, `SchedulersJob` ([modules/SchedulersJobs/SchedulersJob.php](../../SuiteCRM/modules/SchedulersJobs/SchedulersJob.php)) |

## 4. Domaine — modules métier (sélection)

Tous héritent (directement ou indirectement) de `SugarBean` ([data/SugarBean.php:62](../../SuiteCRM/data/SugarBean.php#L62)).

| Module | Classe | Table principale | Preuve |
|---|---|---|---|
| Accounts | `Account` | `accounts` | [modules/Accounts/Account.php](../../SuiteCRM/modules/Accounts/Account.php), [modules/Accounts/vardefs.php:45-46](../../SuiteCRM/modules/Accounts/vardefs.php#L45-L46) |
| Contacts | `Contact` | `contacts` | [modules/Contacts/Contact.php](../../SuiteCRM/modules/Contacts/Contact.php), [modules/Contacts/vardefs.php:44](../../SuiteCRM/modules/Contacts/vardefs.php#L44) |
| Leads | `Lead` | `leads` | [modules/Leads/Lead.php](../../SuiteCRM/modules/Leads/Lead.php), [modules/Leads/vardefs.php:44](../../SuiteCRM/modules/Leads/vardefs.php#L44) |
| Opportunities | `Opportunity` | `opportunities` | [modules/Opportunities/Opportunity.php](../../SuiteCRM/modules/Opportunities/Opportunity.php), [modules/Opportunities/vardefs.php:45](../../SuiteCRM/modules/Opportunities/vardefs.php#L45) |
| Cases | `aCase` | `cases` | [modules/Cases/](../../SuiteCRM/modules/Cases/), [modules/Cases/vardefs.php:46](../../SuiteCRM/modules/Cases/vardefs.php#L46) |
| Bugs | `Bug` | `bugs` | [modules/Bugs/](../../SuiteCRM/modules/Bugs/) |
| Campaigns | `Campaign extends SugarBean` | `campaigns` | [modules/Campaigns/Campaign.php:50, 88-89](../../SuiteCRM/modules/Campaigns/Campaign.php#L50-L89) |
| Emails | `Email` | `emails` | [modules/Emails/Email.php](../../SuiteCRM/modules/Emails/Email.php) |
| InboundEmail | `InboundEmail extends SugarBean` | `inbound_email` | [modules/InboundEmail/InboundEmail.php:54](../../SuiteCRM/modules/InboundEmail/InboundEmail.php#L54) |
| Quotes / Invoices / Contracts | `AOS_Quotes`, `AOS_Invoices`, `AOS_Contracts` | `aos_quotes`, `aos_invoices`, `aos_contracts` | [modules/AOS_Quotes/](../../SuiteCRM/modules/AOS_Quotes/), [modules/AOS_Invoices/](../../SuiteCRM/modules/AOS_Invoices/), [modules/AOS_Contracts/](../../SuiteCRM/modules/AOS_Contracts/) |
| Products | `AOS_Products` | `aos_products` | [modules/AOS_Products/](../../SuiteCRM/modules/AOS_Products/) |
| Reports | `AOR_Report extends Basic` | `aor_reports` | [modules/AOR_Reports/AOR_Report.php:48](../../SuiteCRM/modules/AOR_Reports/AOR_Report.php#L48) |
| Workflow | `AOW_WorkFlow extends Basic` | `aow_workflow` | [modules/AOW_WorkFlow/AOW_WorkFlow.php:42-46](../../SuiteCRM/modules/AOW_WorkFlow/AOW_WorkFlow.php#L42-L46) |
| Knowledge Base | `AOK_KnowledgeBase` | `aok_knowledgebase` | [modules/AOK_KnowledgeBase/](../../SuiteCRM/modules/AOK_KnowledgeBase/) |
| Schedulers | `Scheduler extends SugarBean` | `schedulers` | [modules/Schedulers/Scheduler.php:48, 84](../../SuiteCRM/modules/Schedulers/Scheduler.php#L48-L84) |
| Users | `User extends SugarBean` | `users` | [modules/Users/User.php](../../SuiteCRM/modules/Users/User.php) |
| ACL | `ACLRole`, `ACLAction` | `acl_roles`, `acl_actions` | [modules/ACL/](../../SuiteCRM/modules/ACL/), [modules/ACLRoles/](../../SuiteCRM/modules/ACLRoles/), [modules/ACLActions/](../../SuiteCRM/modules/ACLActions/) |
| Security Groups | `SecurityGroup` | `securitygroups` | [modules/SecurityGroups/](../../SuiteCRM/modules/SecurityGroups/) |
| Calendar | `Meeting`, `Call`, `Task`, `Reminder` + provider sync | `meetings`, `calls`, `tasks`, `reminders` + `calendar_accounts*` | [modules/Calendar/](../../SuiteCRM/modules/Calendar/), [modules/CalendarAccount/](../../SuiteCRM/modules/CalendarAccount/), [include/CalendarSync/](../../SuiteCRM/include/CalendarSync/) |
| Surveys | `Surveys`, `SurveyQuestions`, `SurveyResponses` | `surveys`, `surveyquestions`, `surveyresponses` | [modules/Surveys/](../../SuiteCRM/modules/Surveys/), [modules/SurveyQuestions/](../../SuiteCRM/modules/SurveyQuestions/), [modules/SurveyResponses/](../../SuiteCRM/modules/SurveyResponses/) |

(Liste complète : 123 sous-dossiers de `modules/` — voir [modules/](../../SuiteCRM/modules/).)

## 5. Logique métier — exemples ancrés

### 5.1 Création d'un enregistrement via API V8

`ModuleService::createRecord` :

1. Lit `$params->getData()->getType()` (module), `getId()`, `getAttributes()`.
2. Si `id` fourni *et* qu'un Bean existe : `InvalidArgumentException('Bean %s with id %s is already exist')`.
3. `BeanManager::newBeanSafe($module)`.
4. **ACL** : `bean->ACLAccess('save')` ; sinon `AccessDeniedException`.
5. Si `id` fourni : `bean->id = $id; bean->new_with_id = true`.
6. `setRecordUpdateParams($bean, $attributes)` + `processAttributes`.
7. `$bean->save()`.
8. Si attachment et `module_dir === 'Notes'` → `addFileToNote(...)`.
9. Si attachment et `module_dir === 'Documents'` → `addFileToDocument(...)`.
10. `bean->retrieve($bean->id)` (refresh) puis `getDataResponse(...)` → JSON:API.

Preuve : [Api/V8/Service/ModuleService.php:246-295](../../SuiteCRM/Api/V8/Service/ModuleService.php#L246-L295).

Exemple **entrée** (POST `application/vnd.api+json` → `/Api/V8/module`) :

```json
{
  "data": {
    "type": "Accounts",
    "attributes": {
      "name": "ACME",
      "industry": "Technology",
      "billing_address_country": "France"
    }
  }
}
```

Exemple **sortie** (status 201) :

```json
{
  "data": {
    "type": "Accounts",
    "id": "9b...uuid...",
    "attributes": { "name": "ACME", "industry": "Technology", ... },
    "relationships": { ... }
  }
}
```

(Schémas formels → [api/payloads.md](../api/payloads.md).)

### 5.2 Lecture paginée avec recherche par e-mail

`ModuleService::getRecords` détecte si le filtre porte sur `email1` ou `email2` : il bascule sur une requête SQL avec jointure `email_addresses` + `email_addr_bean_rel` au lieu de la query par défaut sur le module. Cela traduit `<module>.email1 = '...'` → `email_addresses.email_address = '...'`.

Preuve : [Api/V8/Service/ModuleService.php:131-187](../../SuiteCRM/Api/V8/Service/ModuleService.php#L131-L187).

Conséquence métier : la recherche par e-mail traverse la relation E-mail ↔ Bean (`email_addr_bean_rel`) plutôt que d'utiliser des colonnes dénormalisées.

### 5.3 Workflow auto-désactivable

Quand `processAOW_Workflow` exécute un workflow qui modifie un Bean (`after_save`), il pose `AOW_WorkFlow::$doNotRunInSaveLogic = true` pour ne pas re-déclencher de workflow en cascade lors du `save()`. Le flag est levé après la passe.

Preuve : [modules/AOW_WorkFlow/AOW_WorkFlow.php:209-232](../../SuiteCRM/modules/AOW_WorkFlow/AOW_WorkFlow.php#L209-L232).

### 5.4 Cron : sélection des schedulers à exécuter

```sql
SELECT * FROM schedulers
WHERE schedulers.status = 'Active'
  AND NOT EXISTS (
    SELECT id FROM job_queue
    WHERE scheduler_id = schedulers.id
      AND status != 'done'
  )
```
Preuve : [modules/Schedulers/Scheduler.php:196](../../SuiteCRM/modules/Schedulers/Scheduler.php#L196). Cela évite d'empiler des jobs si le précédent n'est pas terminé.

### 5.5 Sélection du driver DB à la connexion

`DBManagerFactory::getInstance()` :

- charge `$sugar_config['dbconfig']` puis appelle `getTypeInstance($type)` ;
- selon `$type` (`mysql` / `mssql` / autre), choisit la classe driver ;
- pour `mysql` : préfère `MysqliManager` si `mysqli_connect` existe et `mysqli_disabled` est faux ;
- pour `mssql` : essaie successivement `SqlsrvManager`, `FreeTDSManager`, `MssqlManager` ;
- option d'override par `db_manager_class` ;
- charge `custom/include/database/{driver}.php` en priorité avant `include/database/{driver}.php`.

Preuves : [include/database/DBManagerFactory.php:60-159](../../SuiteCRM/include/database/DBManagerFactory.php#L60-L159).

## 6. Sécurité d'accès — flux ACL

Tous les services V8 vérifient l'ACL **avant** de répondre :

- `ACLAccess('view')` — `getRecord`, `getRecords` ([ModuleService.php:82-84, 117-119](../../SuiteCRM/Api/V8/Service/ModuleService.php#L82-L119))
- `ACLAccess('save')` — `createRecord`, `updateRecord` ([ModuleService.php:264-266](../../SuiteCRM/Api/V8/Service/ModuleService.php#L264-L266))
- `AccessDeniedException` ([Api/V8/Service/ModuleService.php:25](../../SuiteCRM/Api/V8/Service/ModuleService.php#L25)) si refusé.

L'UI applique la même garde via `SugarApplication::ACLFilter()` ([SugarApplication.php:90](../../SuiteCRM/include/MVC/SugarApplication.php#L90)).

## 7. Notes & conventions

- **Module ↔ table** : par convention Sugar, le bean est singulier (`Account`) et la table plurielle (`accounts`). Confirmé dans [data/SugarBean.php:54-58](../../SuiteCRM/data/SugarBean.php#L54-L58).
- **Champs** : déclarés dans `<module>/vardefs.php`. Types ancrés : `id`, `varchar`, `text`, `enum`, `bool`, `date`, `datetime`, `int`, `currency`, `relate` (lien externe), `link` (lien vers une relation).
- **Relations** : non-nommables côté ORM ; déclarées dans `metadata/<rel>MetaData.php` et chargées via `TableDictionary.php`.
- **Custom fields** : portés par `modules/DynamicFields/` (table `<module>_cstm`).

