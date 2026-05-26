# Flow 20 — Création d'un Account via l'UI Sugar

> Phase **Writer**. Trace bout-en-bout de la création d'un enregistrement `Accounts` via l'interface web. Sources ancrées dans le repo.

## 1. URL & entrée

```
POST /index.php
   ?module=Accounts
   &action=Save
   (form-encoded, formulaire EditView)
```

Le routeur `index.php` est l'unique front-controller UI ([index.php:41-52](../../SuiteCRM/index.php#L41-L52)).

## 2. Bootstrap

1. `define('sugarEntry', true)` ([index.php:42](../../SuiteCRM/index.php#L42)).
2. `include 'include/MVC/preDispatch.php'` — chargement des globaux Sugar + autoload Composer ([include/MVC/preDispatch.php](../../SuiteCRM/include/MVC/preDispatch.php)).
3. `require_once 'include/entryPoint.php'` — `$sugar_config`, `$db`, `$log`, `$current_language`, sessions ([include/entryPoint.php](../../SuiteCRM/include/entryPoint.php)).
4. `new SugarApplication()->execute()` ([index.php:50-52](../../SuiteCRM/index.php#L50-L52)).

## 3. Dispatch MVC

`SugarApplication::execute()` ([SugarApplication.php:74-103](../../SuiteCRM/include/MVC/SugarApplication.php#L74-L103)) :

1. `module = 'Accounts'` (lecture `$_REQUEST['module']`).
2. `ControllerFactory::getController('Accounts')` — par défaut → `SugarController` ([SugarApplication.php:86](../../SuiteCRM/include/MVC/SugarApplication.php#L86)).
3. Auth :
   - `loadUser()` — lecture session, vérification `unique_key` ([SugarApplication.php:108-122](../../SuiteCRM/include/MVC/SugarApplication.php#L108-L122)).
   - `ACLFilter()`, `preProcess()`, `controller->preProcess()`, `checkHTTPReferer()` ([SugarApplication.php:89-93](../../SuiteCRM/include/MVC/SugarApplication.php#L89-L93)).
4. `SugarThemeRegistry::buildRegistry()` puis `loadLanguages` / `loadDisplaySettings` / `loadGlobals` / `setupResourceManagement('Accounts')`.
5. `controller->execute()` :
   - `action = 'Save'` → résolu via `include/MVC/Controller/action_file_map.php` ou méthode `action_save` du contrôleur ([action_file_map.php](../../SuiteCRM/include/MVC/Controller/action_file_map.php), [SugarController.php](../../SuiteCRM/include/MVC/Controller/SugarController.php)).

## 4. Création du Bean

Le contrôleur :

1. Instancie le Bean : `BeanFactory::newBean('Accounts')` ([data/BeanFactory.php](../../SuiteCRM/data/BeanFactory.php)).
2. Hydrate depuis `$_POST` (parsing standard `SugarController::handleSave` ou équivalent — preuve générique [data/SugarBean.php:62](../../SuiteCRM/data/SugarBean.php#L62)).
3. Vérifie ACL : `bean->ACLAccess('save')` ; si refus → redirect 401 / message.
4. Applique les *logic hooks* `before_save` ([include/utils/LogicHook.php:44-50](../../SuiteCRM/include/utils/LogicHook.php#L44-L50)).
5. Appelle `$bean->save()` qui :
   - Renseigne `id` (UUID via `create_guid()`) si absent.
   - Renseigne `date_entered` / `date_modified` / `created_by` / `modified_user_id` (champs implicites du template `basic`).
   - Persiste dans `accounts` ([modules/Accounts/vardefs.php:46](../../SuiteCRM/modules/Accounts/vardefs.php#L46)).
   - Persiste les champs personnalisés dans `accounts_cstm` (si présents) — module `DynamicFields`.
   - Émet l'audit `accounts_audit` (champs `audited` modifiés) car `Account.audited = true` ([Accounts/vardefs.php:47](../../SuiteCRM/modules/Accounts/vardefs.php#L47)).
   - Applique les *logic hooks* `after_save`.
6. Mise à jour des relations : `email_addresses` + `email_addr_bean_rel` si une adresse mail est saisie (`EmailAddressRelationship`) — [data/Relationships/EmailAddressRelationship.php](../../SuiteCRM/data/Relationships/EmailAddressRelationship.php).
7. Si workflow actif : Scheduler `processAOW_Workflow` traitera ce Bean au prochain cycle cron ([modules/AOW_WorkFlow/AOW_WorkFlow.php:225-232](../../SuiteCRM/modules/AOW_WorkFlow/AOW_WorkFlow.php#L225-L232)) — le `after_save` peut aussi déclencher des actions immédiates.

## 5. Rendu de la vue

Le contrôleur redirige (`HTTP 302`) vers `index.php?module=Accounts&action=DetailView&record=<id>`. Le second tour :

1. `ControllerFactory::getController('Accounts')` (idem).
2. `action = 'DetailView'` → `ViewFactory::loadView('detail', ...)` ([include/MVC/View/ViewFactory.php](../../SuiteCRM/include/MVC/View/ViewFactory.php)).
3. `SugarView::display()` rend via **Smarty** (templates `themes/SuiteP/include/...`) ([themes/SuiteP/](../../SuiteCRM/themes/SuiteP/)).
4. `sugar_cleanup()` ferme la connexion DB.

## 6. Effets persistants

| Effet | Table | Preuve |
|---|---|---|
| Création de la fiche | `accounts` | [Accounts/vardefs.php:45-46](../../SuiteCRM/modules/Accounts/vardefs.php#L45-L46) |
| Audit | `accounts_audit` | `audited=>true` ([vardefs.php:47](../../SuiteCRM/modules/Accounts/vardefs.php#L47)) |
| Email (si saisi) | `email_addresses` + `email_addr_bean_rel` | [Api/V8/Service/ModuleService.php:147-150](../../SuiteCRM/Api/V8/Service/ModuleService.php#L147-L150) (utilisation des mêmes tables côté V8) |
| Custom fields (si présents) | `accounts_cstm` | module `DynamicFields` |
| Tracker | `tracker`, `tracker_perf`, `tracker_queries` | si Trackers activé — [modules/Trackers/](../../SuiteCRM/modules/Trackers/) |

## 7. Erreurs prévues

- ACL refusé : redirection vers page d'erreur + log application.
- CSRF / Referer invalide : `checkHTTPReferer()` peut interrompre la requête ([SugarApplication.php:93](../../SuiteCRM/include/MVC/SugarApplication.php#L93)).
- Session expirée : redirection vers `?module=Users&action=Authenticate`.

## 8. Variation API V8

Le même flux en mode API est documenté dans [architecture/20_composants.md §5.1](../architecture/20_composants.md#51-creation-dun-enregistrement-via-api-v8) (preuves [Api/V8/Service/ModuleService.php:246-295](../../SuiteCRM/Api/V8/Service/ModuleService.php#L246-L295)).
