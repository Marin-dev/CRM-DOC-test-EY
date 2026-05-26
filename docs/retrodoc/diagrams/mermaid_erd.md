# Diagramme ERD — Sous-ensemble CRM core

> Le schéma complet (~100 tables) n'est pas représenté ici — il est dérivé des vardefs/metadata (voir [data/modele_donnees.md](../data/modele_donnees.md)). Le diagramme suivant montre le **cœur CRM** prouvé.

## 1. CRM core

```mermaid
erDiagram
    accounts {
      uuid id PK
      string name
      uuid parent_id FK
      string industry
      string sic_code
      bool deleted
      datetime date_entered
      datetime date_modified
      uuid created_by
      uuid modified_user_id
      uuid assigned_user_id
    }

    contacts {
      uuid id PK
      string first_name
      string last_name
      uuid account_id FK
      uuid reports_to_id FK
      string lead_source
      bool deleted
      datetime date_entered
      datetime date_modified
      uuid assigned_user_id
    }

    leads {
      uuid id PK
      string first_name
      string last_name
      string lead_source
      string status
      uuid account_id FK
      uuid contact_id FK
      uuid opportunity_id FK
      bool converted
      uuid campaign_id FK
      bool deleted
    }

    opportunities {
      uuid id PK
      string name
      string sales_stage
      decimal amount
      uuid currency_id FK
      uuid account_id FK
      uuid campaign_id FK
      string opportunity_type
      date date_closed
      int probability
      bool deleted
    }

    cases {
      uuid id PK
      string name
      int case_number
      string priority
      string status
      string state
      uuid account_id FK
      uuid contact_created_by FK
      bool deleted
    }

    bugs {
      uuid id PK
      string name
      string status
      bool deleted
    }

    campaigns {
      uuid id PK
      string name
      string status
      date start_date
      date end_date
      bool deleted
    }

    users {
      uuid id PK
      string user_name
      string user_hash
      string first_name
      string last_name
      bool is_admin
      bool external_auth_only
      string status
      datetime last_login
    }

    accounts_contacts {
      uuid id PK
      uuid account_id FK
      uuid contact_id FK
      bool deleted
    }

    accounts_opportunities {
      uuid id PK
      uuid account_id FK
      uuid opportunity_id FK
      bool deleted
    }

    accounts_cases {
      uuid id PK
      uuid account_id FK
      uuid case_id FK
      bool deleted
    }

    accounts_bugs {
      uuid id PK
      uuid account_id FK
      uuid bug_id FK
      bool deleted
    }

    contacts_cases {
      uuid id PK
      uuid contact_id FK
      uuid case_id FK
      bool deleted
    }

    contacts_bugs {
      uuid id PK
      uuid contact_id FK
      uuid bug_id FK
      bool deleted
    }

    opportunities_contacts {
      uuid id PK
      uuid opportunity_id FK
      uuid contact_id FK
      string contact_role
      bool deleted
    }

    cases_bugs {
      uuid id PK
      uuid case_id FK
      uuid bug_id FK
      bool deleted
    }

    accounts ||--o{ accounts : "members<br/>parent_id"
    accounts ||--o{ accounts_contacts : ""
    contacts ||--o{ accounts_contacts : ""
    accounts ||--o{ accounts_opportunities : ""
    opportunities ||--o{ accounts_opportunities : ""
    accounts ||--o{ accounts_cases : ""
    cases ||--o{ accounts_cases : ""
    accounts ||--o{ accounts_bugs : ""
    bugs ||--o{ accounts_bugs : ""
    contacts ||--o{ contacts_cases : ""
    cases ||--o{ contacts_cases : ""
    contacts ||--o{ contacts_bugs : ""
    bugs ||--o{ contacts_bugs : ""
    opportunities ||--o{ opportunities_contacts : ""
    contacts ||--o{ opportunities_contacts : ""
    cases ||--o{ cases_bugs : ""
    bugs ||--o{ cases_bugs : ""
    contacts }o--|| accounts : "account_id"
    contacts }o--o| contacts : "reports_to_id"
    opportunities }o--|| accounts : "account_id"
    opportunities }o--o| campaigns : "campaign_id"
    leads }o--o| accounts : "account_id"
    leads }o--o| contacts : "contact_id"
    leads }o--o| opportunities : "opportunity_id"
    leads }o--o| campaigns : "campaign_id"
    cases }o--|| accounts : "account_id"
    users ||--o{ accounts : "assigned_user_id"
    users ||--o{ contacts : "assigned_user_id"
    users ||--o{ opportunities : "assigned_user_id"
    users ||--o{ cases : "assigned_user_id"
```

Preuves :

- `accounts` vardefs : [modules/Accounts/vardefs.php:45-200](../../SuiteCRM/modules/Accounts/vardefs.php#L45-L200)
- `contacts` vardefs : [modules/Contacts/vardefs.php:44-200](../../SuiteCRM/modules/Contacts/vardefs.php#L44-L200)
- `opportunities` vardefs : [modules/Opportunities/vardefs.php:45-200](../../SuiteCRM/modules/Opportunities/vardefs.php#L45-L200)
- `cases` vardefs : [modules/Cases/vardefs.php:46-220](../../SuiteCRM/modules/Cases/vardefs.php#L46-L220)
- M2M `accounts_contacts` : [metadata/accounts_contactsMetaData.php:44-62](../../SuiteCRM/metadata/accounts_contactsMetaData.php#L44-L62)
- M2M `accounts_opportunities` : [metadata/accounts_opportunitiesMetaData.php:44-62](../../SuiteCRM/metadata/accounts_opportunitiesMetaData.php#L44-L62)
- M2M `accounts_cases` : [metadata/accounts_casesMetaData.php](../../SuiteCRM/metadata/accounts_casesMetaData.php)
- M2M `accounts_bugs` : [metadata/accounts_bugsMetaData.php:60](../../SuiteCRM/metadata/accounts_bugsMetaData.php#L60)
- M2M `cases_bugs` : [metadata/cases_bugsMetaData.php](../../SuiteCRM/metadata/cases_bugsMetaData.php)

## 2. Marketing

```mermaid
erDiagram
    campaigns {
      uuid id PK
      string name
      string status
      string campaign_type
      date start_date
      date end_date
      bool deleted
    }

    prospect_lists {
      uuid id PK
      string name
      string list_type
      bool deleted
    }

    prospects {
      uuid id PK
      string first_name
      string last_name
      bool deleted
    }

    prospect_list_campaigns {
      uuid id PK
      uuid campaign_id FK
      uuid prospect_list_id FK
      bool deleted
    }

    prospect_lists_prospects {
      uuid id PK
      uuid prospect_list_id FK
      uuid related_id FK
      string related_type
      bool deleted
    }

    campaign_log {
      uuid id PK
      uuid campaign_id FK
      uuid list_id FK
      uuid target_id FK
      string target_type
      string activity_type
      datetime activity_date
      bool deleted
    }

    email_marketing {
      uuid id PK
      uuid campaign_id FK
      string name
      uuid template_id FK
      datetime date_start
      bool deleted
    }

    emailman {
      int id PK
      uuid campaign_id FK
      uuid marketing_id FK
      uuid list_id FK
      uuid related_id FK
      string related_type
      datetime send_date_time
    }

    campaigns ||--o{ prospect_list_campaigns : ""
    prospect_lists ||--o{ prospect_list_campaigns : ""
    prospect_lists ||--o{ prospect_lists_prospects : "polymorphique<br/>related_type"
    campaigns ||--o{ campaign_log : ""
    campaigns ||--o{ email_marketing : ""
    campaigns ||--o{ emailman : ""
```

Preuves : [metadata/prospect_list_campaignsMetaData.php](../../SuiteCRM/metadata/prospect_list_campaignsMetaData.php), [metadata/prospect_lists_prospectsMetaData.php](../../SuiteCRM/metadata/prospect_lists_prospectsMetaData.php), [modules/CampaignLog/](../../SuiteCRM/modules/CampaignLog/), [modules/EmailMarketing/](../../SuiteCRM/modules/EmailMarketing/), [modules/EmailMan/](../../SuiteCRM/modules/EmailMan/).

## 3. ACL & Security

```mermaid
erDiagram
    users {
      uuid id PK
      string user_name
      bool is_admin
    }
    acl_roles {
      uuid id PK
      string name
      bool deleted
    }
    acl_actions {
      uuid id PK
      string category
      string name
      string access_type
      int aclaccess
      bool deleted
    }
    acl_roles_actions {
      uuid id PK
      uuid role_id FK
      uuid action_id FK
      int access_override
      bool deleted
    }
    acl_roles_users {
      uuid id PK
      uuid role_id FK
      uuid user_id FK
      bool deleted
    }
    securitygroups {
      uuid id PK
      string name
      bool deleted
    }
    securitygroups_users {
      uuid id PK
      uuid securitygroup_id FK
      uuid user_id FK
      bool deleted
    }
    securitygroups_acl_roles {
      uuid id PK
      uuid securitygroup_id FK
      uuid role_id FK
      bool deleted
    }
    securitygroups_records {
      uuid id PK
      uuid securitygroup_id FK
      uuid record_id FK
      string module FK
      bool deleted
    }

    users ||--o{ acl_roles_users : ""
    acl_roles ||--o{ acl_roles_users : ""
    acl_roles ||--o{ acl_roles_actions : ""
    acl_actions ||--o{ acl_roles_actions : ""
    users ||--o{ securitygroups_users : ""
    securitygroups ||--o{ securitygroups_users : ""
    securitygroups ||--o{ securitygroups_acl_roles : ""
    acl_roles ||--o{ securitygroups_acl_roles : ""
    securitygroups ||--o{ securitygroups_records : "polymorphique<br/>module"
```

Preuves : [metadata/acl_roles_actionsMetaData.php:99](../../SuiteCRM/metadata/acl_roles_actionsMetaData.php#L99), [metadata/acl_roles_usersMetaData.php:93](../../SuiteCRM/metadata/acl_roles_usersMetaData.php#L93), [metadata/securitygroups_usersMetaData.php](../../SuiteCRM/metadata/securitygroups_usersMetaData.php), [metadata/securitygroups_recordsMetaData.php](../../SuiteCRM/metadata/securitygroups_recordsMetaData.php).

## 4. Scheduler / jobs

```mermaid
erDiagram
    schedulers {
      uuid id PK
      string name
      string job
      datetime date_time_start
      datetime date_time_end
      string job_interval
      string status
      bool catch_up
      bool deleted
    }

    job_queue {
      uuid id PK
      string name
      uuid scheduler_id FK
      string target
      text data
      bool requeue
      int retry_count
      int failure_count
      string status
      text resolution
      text message
      datetime execute_time
      int percent_complete
      bool deleted
    }

    schedulers ||--o{ job_queue : "scheduler_id"
```

Preuves : [modules/Schedulers/Scheduler.php:84, 196](../../SuiteCRM/modules/Schedulers/Scheduler.php#L84-L196), [modules/SchedulersJobs/SchedulersJob.php](../../SuiteCRM/modules/SchedulersJobs/SchedulersJob.php).

## 5. OAuth2 (API V8)

```mermaid
erDiagram
    oauth2clients {
      uuid id PK
      string name
      string secret
      string redirect_url
      bool is_confidential
      bool deleted
    }
    oauth2tokens {
      uuid id PK
      string access_token
      uuid client_id FK
      uuid user_id FK
      datetime expires_at
      bool revoked
      text scopes
      bool deleted
    }
    oauth2authcodes {
      uuid id PK
      string auth_code
      uuid client_id FK
      uuid user_id FK
      datetime expires_at
      string redirect_uri
      bool deleted
    }
    users { uuid id PK }

    oauth2clients ||--o{ oauth2tokens : ""
    users ||--o{ oauth2tokens : ""
    oauth2clients ||--o{ oauth2authcodes : ""
    users ||--o{ oauth2authcodes : ""
```

Preuves : [Api/V8/OAuth2/Repository/](../../SuiteCRM/Api/V8/OAuth2/Repository/), modules [modules/OAuth2Clients/](../../SuiteCRM/modules/OAuth2Clients/), [modules/OAuth2Tokens/](../../SuiteCRM/modules/OAuth2Tokens/), [modules/OAuth2AuthCodes/](../../SuiteCRM/modules/OAuth2AuthCodes/).

> ⚠️ Les colonnes précises des tables OAuth2 sont déclarées dans les vardefs des modules ; les types ci-dessus représentent les conventions Sugar et la structure attendue par `league/oauth2-server`. INCONNU : champ par champ exhaustif sans dérouler chaque `<module>/vardefs.php` du sous-dossier OAuth2.
