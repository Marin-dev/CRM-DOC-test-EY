# Flows candidats

> Phase **Searcher**. Liste des flows métier identifiables dans le code, classés par criticité. Les trois flows **[*]** ont été choisis comme « représentatifs » et sont détaillés dans des pages dédiées.

## A. Flows authentification / session

| Flow | Statut | Détail |
|---|---|---|
| **[*]** Login web Sugar + appel API V8 OAuth2 | détaillé | [10_login_api_v8.md](10_login_api_v8.md) |
| Login SSO SAML2 | candidat | preuves [api/authentification.md §4](../api/authentification.md#4-saml-20) |
| Login LDAP / AD | candidat | preuves [api/authentification.md §5](../api/authentification.md#5-ldap--ad) |
| Login Sugar natif (legacy session) | candidat | preuves [api/authentification.md §3](../api/authentification.md#3-session-sugar-ui--apis-legacy) |

## B. Flows CRUD UI ↔ DB

| Flow | Statut | Détail |
|---|---|---|
| **[*]** Création d'un Account (UI → DB) | détaillé | [20_creation_account_ui.md](20_creation_account_ui.md) |
| Création / Update via API V8 | candidat | déjà partiellement décrit dans [api/payloads.md §3](../api/payloads.md#3-exemples-concrets) et [architecture/20_composants.md §5.1](../architecture/20_composants.md#51-creation-dun-enregistrement-via-api-v8) |
| Recherche par email (V8) | candidat | déjà décrit dans [api/payloads.md §7](../api/payloads.md#7-cas-particuliers--recherche-par-email) |
| Conversion Lead → Account + Contact + Opportunity | candidat | module [modules/Leads/](../../SuiteCRM/modules/Leads/) ; entrée `Leads/ConvertLead.php`/`controller.php` |
| Création / vente d'un Devis (Quote → Invoice) | candidat | modules `AOS_Quotes`, `AOS_Invoices`, [modules/AOS_Quotes/](../../SuiteCRM/modules/AOS_Quotes/) |

## C. Flows asynchrones (cron / batch)

| Flow | Statut | Détail |
|---|---|---|
| **[*]** Cron `cron.php` → SugarCronJobs → workflows AOW | détaillé | [30_cron_scheduler.md](30_cron_scheduler.md) |
| Indexation Elasticsearch (`runElasticSearchIndexerScheduler`) | candidat | preuves [architecture/00_inventaire.md §8](../architecture/00_inventaire.md#8-taches-planifiees-out-of-the-box) |
| Indexation AOD/Lucene (`aodIndexUnindexed`, `aodOptimiseIndex`) | candidat | idem |
| Envoi mass-email campagne (`runMassEmailCampaign`) | candidat | [modules/Campaigns/](../../SuiteCRM/modules/Campaigns/), [modules/EmailMan/](../../SuiteCRM/modules/EmailMan/) |
| Polling IMAP entrant (`pollMonitoredInboxesAOP`) | candidat | [modules/InboundEmail/InboundEmail.php:54](../../SuiteCRM/modules/InboundEmail/InboundEmail.php#L54), [modules/AOP_Case_Updates/](../../SuiteCRM/modules/AOP_Case_Updates/) |
| Bounces e-mailing (`pollMonitoredInboxesForBouncedCampaignEmails`) | candidat | [modules/Schedulers/Scheduler.php:874-882](../../SuiteCRM/modules/Schedulers/Scheduler.php#L874-L882) |
| Reports planifiés (`aorRunScheduledReports`) | candidat | [modules/AOR_Scheduled_Reports/](../../SuiteCRM/modules/AOR_Scheduled_Reports/) |
| Reminders e-mail (`sendEmailReminders`) | candidat | [modules/Reminders/](../../SuiteCRM/modules/Reminders/) |
| Sync calendar (`calendarSyncJob`) | candidat | [include/CalendarSync/application/CalendarSyncOrchestrator.php](../../SuiteCRM/include/CalendarSync/application/CalendarSyncOrchestrator.php) |
| Trim tracker / sugar feeds / clean job queue | candidat | [modules/Schedulers/Scheduler.php:850-978](../../SuiteCRM/modules/Schedulers/Scheduler.php#L850-L978) |

## D. Flows captation entrants (anonyme)

| Flow | Statut | Détail |
|---|---|---|
| Web-to-Lead | candidat | [architecture/02_patterns_integration.md §2.4](../architecture/02_patterns_integration.md#24-flux-web-to-lead) |
| Web-to-Person | candidat | [entry_point_registry.php:62](../../SuiteCRM/include/MVC/Controller/entry_point_registry.php#L62) |
| Tracker open / click / removeme | candidat | [entry_point_registry.php:59-63](../../SuiteCRM/include/MVC/Controller/entry_point_registry.php#L59-L63) |
| Confirm opt-in | candidat | [entry_point_registry.php:64](../../SuiteCRM/include/MVC/Controller/entry_point_registry.php#L64), [include/EntryPointConfirmOptInHandler.php](../../SuiteCRM/include/EntryPointConfirmOptInHandler.php) |
| Accept / Decline invitations contact | candidat | [entry_point_registry.php:65](../../SuiteCRM/include/MVC/Controller/entry_point_registry.php#L65) |
| Génération PDF (devis, facture, …) via `pdf.php` | candidat | [pdf.php](../../SuiteCRM/pdf.php), [modules/AOS_PDF_Templates/](../../SuiteCRM/modules/AOS_PDF_Templates/) |

## E. Intégrations externes

| Flow | Statut | Détail |
|---|---|---|
| Google Calendar sync | candidat | [include/CalendarSync/](../../SuiteCRM/include/CalendarSync/), `google/apiclient` [composer.json:46](../../SuiteCRM/composer.json#L46) |
| OAuth2 sortant vers IdP tiers | candidat | [modules/ExternalOAuthConnection/](../../SuiteCRM/modules/ExternalOAuthConnection/), [modules/ExternalOAuthProvider/](../../SuiteCRM/modules/ExternalOAuthProvider/) |
| Elasticsearch (lecture/indexation) | partiellement détaillé | [architecture/02_patterns_integration.md §2.5](../architecture/02_patterns_integration.md#25-flux-recherche-elasticsearch) |

