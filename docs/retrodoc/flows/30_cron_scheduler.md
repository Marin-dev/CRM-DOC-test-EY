# Flow 30 — Cron / Scheduler / Workflow

> Phase **Writer**. Cycle complet du planificateur SuiteCRM, du cron OS jusqu'à l'exécution d'un workflow ou d'un envoi de campagne.

## 1. Vue d'ensemble

```
crontab (OS)
   │  ex.: * * * * *  php /var/www/SuiteCRM/cron.php
   ▼
cron.php  (CLI)
   ├─ Vérifie SAPI=cli                           ([cron.php:49-52])
   ├─ Vérifie utilisateur ∈ allowed_cron_users    ([cron.php:54-77])
   ├─ Charge bootstrap (entryPoint.php)           ([cron.php:46])
   ├─ Définit $current_user = "system user"      ([cron.php:84-87])
   └─ Instancie driver = $sugar_config['cron_class']
              (def: SugarCronJobs)               ([cron.php:90-91])
              ├─► require_once include/SugarQueue/SugarCronJobs.php
              └─► (new SugarCronJobs)->runCycle()  ([cron.php:100-101])

runCycle()
   │
   ├─ Charge Schedulers actifs sans job pending
   │      SELECT * FROM schedulers
   │      WHERE status='Active'
   │      AND NOT EXISTS (SELECT id FROM job_queue
   │                      WHERE scheduler_id = schedulers.id
   │                        AND status != 'done')
   │      ([Scheduler.php:196])
   │
   ├─ Pour chaque scheduler "dû" (job_interval matchant l'heure courante) :
   │      Scheduler::createJob() → insère un SchedulersJob dans `job_queue`
   │      ([Scheduler.php:168-176])
   │
   ├─ Boucle d'exécution des jobs queued :
   │      pour chaque job:
   │         SchedulersJob::run()
   │            ├─ Résout `target = function::<name>`
   │            ├─ require_once correspondant (modules concernés)
   │            ├─ Exécute la fonction
   │            └─ Met à jour status (done / failure / partial)
   │
   └─ sugar_cleanup(false) + DBManagerFactory::getInstance()->disconnect()
              ([cron.php:104-110])
```
Preuves : [cron.php:48-110](../../SuiteCRM/cron.php#L48-L110), [modules/Schedulers/Scheduler.php:168-200](../../SuiteCRM/modules/Schedulers/Scheduler.php#L168-L200), [include/SugarQueue/SugarCronJobs.php](../../SuiteCRM/include/SugarQueue/SugarCronJobs.php), [include/SugarQueue/SugarJobQueue.php](../../SuiteCRM/include/SugarQueue/SugarJobQueue.php), [modules/SchedulersJobs/SchedulersJob.php](../../SuiteCRM/modules/SchedulersJobs/SchedulersJob.php).

## 2. Mécanique pas-à-pas — workflow AOW

Le scheduler `processAOW_Workflow` est livré **Active** par défaut, avec `job_interval = '*::*::*::*::*'` (chaque minute) ([Scheduler.php:826-834](../../SuiteCRM/modules/Schedulers/Scheduler.php#L826-L834)).

```
cron.php → SugarCronJobs::runCycle()
   ├─ insère un job target='function::processAOW_Workflow'
   ├─ SchedulersJob::run()
   │     └─ require 'modules/AOW_WorkFlow/aow_utils.php'
   │     └─ processAOW_Workflow()   ([AOW_WorkFlow.php:116, 232])
   │            │
   │            ├─ AOW_WorkFlow::$doNotRunInSaveLogic = true  (anti-boucle)
   │            │       ([AOW_WorkFlow.php:209-217])
   │            │
   │            ├─ Pour chaque workflow actif (table `aow_workflow`) :
   │            │     ├─ Sélectionne les Beans concernés selon `flow_module`
   │            │     ├─ Évalue les conditions (`aow_conditions`)
   │            │     ├─ Exécute les actions (`aow_actions`)
   │            │     │   - Create/Update bean
   │            │     │   - Send Email
   │            │     │   - Modify field
   │            │     │   - Create Task
   │            │     │   - ...
   │            │     └─ Marque le bean traité dans `aow_processed`
   │            │             ([modules/AOW_Processed/])
   │            │
   │            └─ AOW_WorkFlow::$doNotRunInSaveLogic = false
   │
   └─ Job marqué `status = done` dans `job_queue`
```
Preuves : [modules/AOW_WorkFlow/AOW_WorkFlow.php:108-232](../../SuiteCRM/modules/AOW_WorkFlow/AOW_WorkFlow.php#L108-L232), [modules/AOW_Conditions/](../../SuiteCRM/modules/AOW_Conditions/), [modules/AOW_Actions/](../../SuiteCRM/modules/AOW_Actions/), [modules/AOW_Processed/](../../SuiteCRM/modules/AOW_Processed/).

## 3. Tableau complet des Schedulers OOTB

Repris de [architecture/00_inventaire.md §8](../architecture/00_inventaire.md#8-taches-planifiees-out-of-the-box) :

| Job | Cron (Sugar) | Statut OOTB | Module / lib visé |
|---|---|---|---|
| `processAOW_Workflow` | `*::*::*::*::*` | Active | `AOW_*` |
| `aorRunScheduledReports` | `*::*::*::*::*` | Active | `AOR_Reports` |
| `trimTracker` | `0::2::1::*::*` | Active | `Trackers` |
| `pollMonitoredInboxesAOP` | `*::*::*::*::*` | Active | `InboundEmail` + `AOP_*` |
| `pollMonitoredInboxesForBouncedCampaignEmails` | `0::2-6::*::*::*` | Active | `Campaigns` |
| `runMassEmailCampaign` | `0::2-6::*::*::*` | Active | `Campaigns` + `EmailMan` |
| `pruneDatabase` | `0::4::1::*::*` | **Inactive** | DB cleanup |
| `aodIndexUnindexed` | `0::0::*::*::*` | Active | `AOD_Index` |
| `aodOptimiseIndex` | `0::*/3::*::*::*` | Active | `AOD_Index` |
| `sendEmailReminders` | `*::*::*::*::*` | Active | `Reminders` |
| `cleanJobQueue` | `0::5::*::*::*` | Active | `SchedulersJobs` |
| `removeDocumentsFromFS` | `0::3::1::*::*` | Active | `Documents` |
| `trimSugarFeeds` | `0::2::1::*::*` | Active | `SugarFeed` |
| `calendarSyncJob` | `*/15::*::*::*::*` | Active | `CalendarSync` (Google) |
| `runElasticSearchIndexerScheduler` | `30::4::*::*::*` | Active | `lib/Search/ElasticSearch/` |

Preuves : voir [modules/Schedulers/Scheduler.php:824-1002](../../SuiteCRM/modules/Schedulers/Scheduler.php#L824-L1002).

## 4. Mode "remote" (lambda / worker distant)

Si `$sugar_config['cron_class'] = 'SugarCronRemoteJobs'` ([include/SugarQueue/SugarCronRemoteJobs.php](../../SuiteCRM/include/SugarQueue/SugarCronRemoteJobs.php)), les jobs sont publiés pour exécution par un worker externe. Le client peut alors atteindre l'endpoint `index.php?entryPoint=process_queue` (`auth = true`) ([entry_point_registry.php:66](../../SuiteCRM/include/MVC/Controller/entry_point_registry.php#L66)).

## 5. Effets de bord typiques par job

- `processAOW_Workflow` : INSERT/UPDATE dans `aow_processed`, modifications de beans cibles, e-mails via `OutboundEmailAccounts`, création de Tâches/Calls/Meetings.
- `runMassEmailCampaign` : insertion dans `campaign_log` + envoi via `phpmailer`.
- `pollMonitoredInboxesAOP` : INSERT dans `emails`, `email_addr_bean_rel`, `cases` (si AOP), `aop_case_updates`.
- `aodIndexUnindexed` : écriture dans l'index Lucene FS (modules/AOD_Index/).
- `runElasticSearchIndexerScheduler` : indexe les Beans configurés dans l'index ES distant.
- `calendarSyncJob` : appels Google Calendar API (push/pull events).

## 6. Garde anti-cascade

`AOW_WorkFlow::$doNotRunInSaveLogic = true` désactive temporairement le hook `after_save` de Workflow pendant l'exécution d'un workflow — évite les boucles infinies si un workflow modifie le bean qui l'a déclenché. Preuve : [modules/AOW_WorkFlow/AOW_WorkFlow.php:209-232](../../SuiteCRM/modules/AOW_WorkFlow/AOW_WorkFlow.php#L209-L232).

## 7. Configuration côté admin

- UI admin : Modules `Schedulers` + `SchedulersJobs` (interfaces standard SuiteBean).
- Activation : éditer un scheduler dans l'admin, statut `Active`/`Inactive`.
- Format `job_interval` Sugar : `min::h::dom::m::dow` (séparateur `::` au lieu d'un espace cron classique). Preuve : valeurs hard-codées dans `Scheduler::rebuildDefaultSchedulers()` ([Scheduler.php:829, 853, 877, 889, 901, 913, 925, 949, 961, 973, 985, 997](../../SuiteCRM/modules/Schedulers/Scheduler.php#L829)).
