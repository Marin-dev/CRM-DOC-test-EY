# Diagrammes de séquence (Mermaid)

> Trois flows prouvés bout-en-bout. Pour les détails et preuves, voir [flows/](../flows/).

## 1. Login + lecture d'un Account via API V8 (OAuth2 password)

```mermaid
sequenceDiagram
    autonumber
    actor Client
    participant Web as Apache/PHP-FPM
    participant Slim as Slim 3 (Api/V8)
    participant Auth as AuthorizationServer<br/>(league/oauth2-server)
    participant Res as ResourceServer
    participant MC as ModuleController
    participant MS as ModuleService
    participant BM as BeanManager
    participant DB as DB (users, oauth2*, accounts)

    Client->>Web: POST /Api/access_token<br/>grant_type=password
    Web->>Slim: route /access_token
    Slim->>Auth: AuthorizationServerMiddleware
    Auth->>DB: ClientRepository (oauth2clients)
    Auth->>DB: UserRepository (users)
    Auth->>DB: persistAccessToken (oauth2tokens)
    Auth-->>Slim: {access_token, refresh_token, expires_in}
    Slim-->>Client: 200 JSON

    Client->>Web: GET /Api/V8/module/Accounts/{id}<br/>Authorization: Bearer ...
    Web->>Slim: route /V8/module/{m}/{id}
    Slim->>Res: ResourceServerMiddleware
    Res->>Res: vérifie JWT (clé publique RSA)
    Res-->>Slim: oauth_user_id
    Slim->>MC: getModuleRecord(params)
    MC->>MS: getRecord(params, path)
    MS->>BM: getBeanSafe(moduleName, id)
    BM->>DB: SELECT * FROM accounts WHERE id=...
    DB-->>BM: row
    BM-->>MS: SugarBean
    MS->>MS: bean->ACLAccess('view')
    MS->>MS: getDataResponse(...) JSON:API
    MS-->>MC: DocumentResponse
    MC-->>Slim: BaseController::generateResponse(200)
    Slim-->>Client: 200 application/vnd.api+json
```

Preuves :

- Routes : [Api/V8/Config/routes.php:16-18, 62-64, 131](../../SuiteCRM/Api/V8/Config/routes.php#L16-L131)
- Middlewares OAuth2 : [Api/V8/Config/services/middlewares.php:39-111](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L39-L111)
- Controller/Service : [Api/V8/Controller/ModuleController.php:36-43](../../SuiteCRM/Api/V8/Controller/ModuleController.php#L36-L43), [Api/V8/Service/ModuleService.php:74-92](../../SuiteCRM/Api/V8/Service/ModuleService.php#L74-L92)
- Réponse : [Api/V8/Controller/BaseController.php:19-34](../../SuiteCRM/Api/V8/Controller/BaseController.php#L19-L34)

## 2. Création d'un Account via l'UI Sugar

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant Apache
    participant App as SugarApplication
    participant CtrlF as ControllerFactory
    participant Ctrl as SugarController/AccountsController
    participant Bean as Account (SugarBean)
    participant Hooks as LogicHook
    participant Theme as SugarView (Smarty)
    participant DB as DB (accounts, accounts_audit, email_*)

    User->>Apache: POST /index.php?module=Accounts&action=Save
    Apache->>App: index.php → SugarApplication::execute()
    App->>App: include preDispatch.php + entryPoint.php
    App->>CtrlF: getController('Accounts')
    CtrlF-->>App: SugarController
    App->>App: loadUser + ACLFilter + checkHTTPReferer
    App->>Ctrl: execute(action=Save)
    Ctrl->>Bean: BeanFactory::newBean('Accounts')
    Ctrl->>Bean: hydrate from $_POST
    Ctrl->>Bean: ACLAccess('save')
    Ctrl->>Hooks: call before_save
    Ctrl->>Bean: save()
    Bean->>DB: INSERT accounts + accounts_audit + email_addr_bean_rel
    Ctrl->>Hooks: call after_save
    Ctrl-->>App: HTTP 302 → DetailView
    App-->>Apache: redirect

    User->>Apache: GET /index.php?module=Accounts&action=DetailView&record=...
    Apache->>App: SugarApplication::execute()
    App->>Theme: ViewFactory::loadView('detail')
    Theme->>Bean: retrieve(id)
    Bean->>DB: SELECT accounts WHERE id=...
    Theme-->>User: HTML (SuiteP theme)
```

Preuves :

- Dispatch UI : [include/MVC/SugarApplication.php:74-103](../../SuiteCRM/include/MVC/SugarApplication.php#L74-L103)
- Logic hooks : [include/utils/LogicHook.php:44-50, 150-154](../../SuiteCRM/include/utils/LogicHook.php#L44-L154)
- Audit : [modules/Accounts/vardefs.php:47](../../SuiteCRM/modules/Accounts/vardefs.php#L47)

## 3. Cron → Workflow AOW → DB

```mermaid
sequenceDiagram
    autonumber
    participant OS as Crontab OS
    participant Cron as cron.php (CLI)
    participant Driver as SugarCronJobs
    participant Sched as Scheduler bean
    participant Queue as job_queue (SchedulersJob)
    participant Wf as AOW_WorkFlow::run_flows
    participant Cond as AOW_Conditions
    participant Act as AOW_Actions
    participant DB

    OS->>Cron: * * * * * php cron.php
    Cron->>Cron: verif SAPI=cli + allowed_cron_users
    Cron->>Cron: $current_user = "system user"
    Cron->>Driver: new SugarCronJobs()->runCycle()
    Driver->>DB: SELECT schedulers WHERE status='Active' AND NOT EXISTS(job_queue running)
    DB-->>Driver: schedulers dûs
    Driver->>Queue: createJob(scheduler)
    Driver->>Queue: run job 'function::processAOW_Workflow'
    Queue->>Wf: processAOW_Workflow()
    Wf->>Wf: $doNotRunInSaveLogic = true
    Wf->>DB: SELECT aow_workflow WHERE status='Active'
    DB-->>Wf: workflows
    loop pour chaque workflow
      Wf->>Cond: évalue aow_conditions
      Cond->>DB: SELECT beans matching
      DB-->>Cond: beans
      Wf->>Act: exécute aow_actions
      Act->>DB: UPDATE bean / INSERT Tasks / Emails
      Wf->>DB: INSERT aow_processed
    end
    Wf->>Wf: $doNotRunInSaveLogic = false
    Wf-->>Queue: ok
    Queue->>DB: UPDATE job_queue SET status='done'
    Driver-->>Cron: end cycle
    Cron->>Cron: sugar_cleanup + disconnect
```

Preuves :

- `cron.php` : [cron.php:48-110](../../SuiteCRM/cron.php#L48-L110)
- Scheduler driver : [include/SugarQueue/SugarCronJobs.php](../../SuiteCRM/include/SugarQueue/SugarCronJobs.php), [include/SugarQueue/SugarJobQueue.php](../../SuiteCRM/include/SugarQueue/SugarJobQueue.php)
- `processAOW_Workflow` & garde anti-cascade : [modules/AOW_WorkFlow/AOW_WorkFlow.php:108-232](../../SuiteCRM/modules/AOW_WorkFlow/AOW_WorkFlow.php#L108-L232)
- Sélection des schedulers dûs : [modules/Schedulers/Scheduler.php:196](../../SuiteCRM/modules/Schedulers/Scheduler.php#L196)
