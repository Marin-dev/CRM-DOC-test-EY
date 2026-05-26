# Modèle de données — tables principales

> Phase **Writer**. Tables prouvées par les vardefs et les metadata. Pour le diagramme ERD, voir [diagrams/mermaid_erd.md](../diagrams/mermaid_erd.md).

## 1. Conventions

- PK = `id` (varchar 36, UUID v4) sur toutes les tables métier.
- Soft delete via colonne `deleted` (bool).
- Timestamps : `date_entered`, `date_modified`.
- Auditing : `<module>_audit` si `audited => true`.
- Champs custom : `<module>_cstm` (relation 1:1, `id_c = parent.id`).
- FK exprimées par convention `<entity>_id` (FK logique, sans contrainte DB stricte — voir [README.md §9](README.md#9-limites--inconnu)).

## 2. Tables principales — colonnes (extraits prouvés)

### 2.1 `accounts`

Colonnes principales (vardefs) — preuves : [modules/Accounts/vardefs.php](../../SuiteCRM/modules/Accounts/vardefs.php).

| Colonne | Type vardef | Notes | Preuve |
|---|---|---|---|
| `id` | id | PK UUID | template basic |
| `name` | name | obligatoire (importable required) | implicite (manque dans extrait mais standard) |
| `parent_id` | id | self-FK (parent account) | [vardefs.php:55-58](../../SuiteCRM/modules/Accounts/vardefs.php#L55-L58) |
| `sic_code` | varchar | code SIC | [vardefs.php:66](../../SuiteCRM/modules/Accounts/vardefs.php#L66) |
| `email_opt_out`, `invalid_email` | bool | non-DB (calculés via emails) | [vardefs.php:111-119](../../SuiteCRM/modules/Accounts/vardefs.php#L111-L119) |
| `parent_name` (relate via `parent_id`) | relate | non-DB | [vardefs.php:74-75](../../SuiteCRM/modules/Accounts/vardefs.php#L74-L75) |
| `members` / `member_of` (links) | link | self-referencing 1:n via `parent_id` | [vardefs.php:91-100](../../SuiteCRM/modules/Accounts/vardefs.php#L91-L100) |
| `cases`, `tasks`, `notes`, `meetings`, `calls`, `emails` (links) | link | relations sortantes | [vardefs.php:127-188](../../SuiteCRM/modules/Accounts/vardefs.php#L127-L188) |
| `deleted`, `date_entered`, `date_modified`, `created_by`, `modified_user_id` | bool / datetime / id | templates `basic` + `company` | template |

Flags : `audited`, `unified_search`, `full_text_search`, `unified_search_default_enabled`, `duplicate_merge`. Preuve : [vardefs.php:46-52](../../SuiteCRM/modules/Accounts/vardefs.php#L46-L52).

### 2.2 `contacts`

Preuve : [modules/Contacts/vardefs.php:44-200](../../SuiteCRM/modules/Contacts/vardefs.php#L44-L200).

| Colonne | Sens |
|---|---|
| `id`, `first_name`, `last_name`, `salutation`, `title` | identité |
| `phone_home`, `phone_mobile`, `phone_work`, `phone_other`, `phone_fax` | contacts |
| `email1`, `email2` (non-DB via `email_addresses`) | emails |
| `account_name` (relate vers `accounts`), `account_id` | rattachement compte (relate non-DB + link) |
| `reports_to_id`, `report_to_name` | hiérarchie | [vardefs.php:137-146](../../SuiteCRM/modules/Contacts/vardefs.php#L137-L146) |
| `lead_source`, `campaign_id`, `campaign_name` | marketing |
| `do_not_call`, `birthdate` | préférences |
| `opportunity_role_id`, `opportunity_role` | M2M Opportunities | [vardefs.php:121-129](../../SuiteCRM/modules/Contacts/vardefs.php#L121-L129) |

### 2.3 `leads`

Preuve : [modules/Leads/vardefs.php:44](../../SuiteCRM/modules/Leads/vardefs.php#L44).

| Colonne | Sens |
|---|---|
| `id`, `salutation`, `first_name`, `last_name`, `title` | identité |
| `phone_*`, `email1`, `email2` | contacts |
| `account_name`, `account_id` (lead → account potentiel) | conversion |
| `lead_source`, `lead_source_description`, `status`, `status_description` | qualification |
| `contact_id`, `account_id`, `opportunity_id`, `opportunity_amount` | conversion |
| `campaign_id`, `campaign_name` | marketing |
| `converted` (bool) | flag conversion |

### 2.4 `opportunities`

Preuve : [modules/Opportunities/vardefs.php:45-200](../../SuiteCRM/modules/Opportunities/vardefs.php#L45-L200).

| Colonne | Sens |
|---|---|
| `id`, `name` (50 char, required) | clé fonctionnelle |
| `opportunity_type` (enum `opportunity_type_dom`) | typologie |
| `account_id`, `account_name` (relate vers `accounts`, required) | rattachement compte |
| `campaign_id`, `campaign_name` | source marketing |
| `lead_source` (enum) | origine |
| `amount`, `amount_usdollar`, `currency_id` (vers `currencies`) | valorisation |
| `date_closed` (date) | échéance |
| `sales_stage` (enum) | étape |
| `probability` (int) | proba % |
| `next_step` | actions |
| `assigned_user_id`, `assigned_user_name` | propriétaire |

Indexes : `name`, `assigned_user_id`, `id+deleted` ([vardefs.php:407-422](../../SuiteCRM/modules/Opportunities/vardefs.php#L407-L422)).

### 2.5 `cases`

Preuve : [modules/Cases/vardefs.php:46](../../SuiteCRM/modules/Cases/vardefs.php#L46).

| Colonne | Sens |
|---|---|
| `id`, `case_number` (auto-incrément), `name`, `description` | identification |
| `priority`, `status`, `state`, `resolution` (enum) | gestion |
| `account_id`, `account_name` | rattachement |
| `contact_created_by`, `contact_created_by_name` (relate Contacts) | demandeur |
| `display_case_attachments`, `display_update_form` | UI flags |
| `assigned_user_id` | propriétaire |

### 2.6 `emails`

Preuve : [modules/Emails/Email.php](../../SuiteCRM/modules/Emails/Email.php) (champs via vardefs `modules/Emails/vardefs.php`).

| Colonne | Sens |
|---|---|
| `id`, `name` (subject), `description` (body texte), `description_html` (body HTML) | contenu |
| `from_addr`, `from_name`, `to_addrs`, `cc_addrs`, `bcc_addrs`, `reply_to_addr` | adressage (champs combinés) |
| `date_sent`, `message_id` | en-têtes |
| `parent_type`, `parent_id` (polymorphique vers bean lié) | rattachement à Account/Contact/… |
| `mailbox_id` (FK `inbound_email.id`) | compte source |
| `type` (`inbound`/`out`/`archived`/…), `status` | classification |

### 2.7 `email_addresses` (index emails)

| Colonne | Sens |
|---|---|
| `id` | PK |
| `email_address` | normalisée |
| `email_address_caps` | capitalized |
| `invalid_email`, `opt_out` | flags |
| `confirm_opt_in`, `confirm_opt_in_date`, `confirm_opt_in_sent_date` | RGPD opt-in |

Relation N:N à n'importe quel bean via `email_addr_bean_rel(email_address_id, bean_id, bean_module, primary_address, reply_to_address)` — preuve indirecte dans [Api/V8/Service/ModuleService.php:147-150](../../SuiteCRM/Api/V8/Service/ModuleService.php#L147-L150).

### 2.8 `schedulers` & `job_queue`

Preuves : [modules/Schedulers/Scheduler.php:84](../../SuiteCRM/modules/Schedulers/Scheduler.php#L84), [modules/SchedulersJobs/SchedulersJob.php](../../SuiteCRM/modules/SchedulersJobs/SchedulersJob.php).

| Table | Colonnes notables |
|---|---|
| `schedulers` | `id`, `name`, `job` (ex `function::processAOW_Workflow`), `date_time_start`, `date_time_end`, `job_interval` (`min::h::dom::m::dow`), `status` (Active/Inactive), `catch_up`, `last_run` |
| `job_queue` | `id`, `name`, `scheduler_id`, `target` (function::), `data`, `requeue`, `retry_count`, `failure_count`, `status` (queued/running/done/failure/partial), `resolution`, `message`, `execute_time`, `percent_complete` |

### 2.9 OAuth2 (V8)

| Table | Rôle | Preuve |
|---|---|---|
| `oauth2clients` | Clients enregistrés | module [modules/OAuth2Clients/](../../SuiteCRM/modules/OAuth2Clients/), repo [Api/V8/OAuth2/Repository/ClientRepository.php](../../SuiteCRM/Api/V8/OAuth2/Repository/ClientRepository.php) |
| `oauth2tokens` | Access tokens émis | module [modules/OAuth2Tokens/](../../SuiteCRM/modules/OAuth2Tokens/), repo [Api/V8/OAuth2/Repository/AccessTokenRepository.php](../../SuiteCRM/Api/V8/OAuth2/Repository/AccessTokenRepository.php) |
| `oauth2authcodes` | Auth codes (grant `authorization_code`) | module [modules/OAuth2AuthCodes/](../../SuiteCRM/modules/OAuth2AuthCodes/), repo [AuthCodeRepository.php](../../SuiteCRM/Api/V8/OAuth2/Repository/AuthCodeRepository.php) |
| `oauthkeys` (legacy OAuth1) | clés zf1 | module [modules/OAuthKeys/](../../SuiteCRM/modules/OAuthKeys/) |
| `oauthtokens` (legacy OAuth1) | tokens zf1 | module [modules/OAuthTokens/](../../SuiteCRM/modules/OAuthTokens/) |

### 2.10 `users`

Preuve : [modules/Users/User.php](../../SuiteCRM/modules/Users/User.php).

| Colonne | Sens |
|---|---|
| `id`, `user_name`, `first_name`, `last_name`, `email1` | identité |
| `user_hash` | hash mot de passe |
| `is_admin`, `status` | flags |
| `default_team`, `default_role` | RBAC |
| `external_auth_only` | SSO uniquement |
| `last_login` | trace |

### 2.11 `acl_roles`, `acl_actions`, `acl_roles_actions`, `acl_roles_users`

| Table | Rôle | Preuve |
|---|---|---|
| `acl_roles` | Rôle nommé | [modules/ACLRoles/](../../SuiteCRM/modules/ACLRoles/) |
| `acl_actions` | Action protégée (par module + access type) | [modules/ACLActions/](../../SuiteCRM/modules/ACLActions/) |
| `acl_roles_actions` | M2M rôle ↔ action (avec level d'accès) | [metadata/acl_roles_actionsMetaData.php:99](../../SuiteCRM/metadata/acl_roles_actionsMetaData.php#L99) |
| `acl_roles_users` | M2M rôle ↔ utilisateur | [metadata/acl_roles_usersMetaData.php:93](../../SuiteCRM/metadata/acl_roles_usersMetaData.php#L93) |
| `securitygroups`, `securitygroups_users`, `securitygroups_acl_roles`, `securitygroups_records` | Groupes de sécurité (record-level) | [modules/SecurityGroups/](../../SuiteCRM/modules/SecurityGroups/), metadata `securitygroups_*MetaData.php` |

### 2.12 Marketing / Campagnes

| Table | Sens |
|---|---|
| `campaigns` | `id`, `name`, `status`, `start_date`, `end_date`, `expected_cost`, `budget`, `campaign_type` (enum) | [modules/Campaigns/](../../SuiteCRM/modules/Campaigns/) |
| `prospects` | `id`, `first_name`, `last_name`, `email1`, `account_name`, … | [modules/Prospects/](../../SuiteCRM/modules/Prospects/) |
| `prospect_lists` | listes ciblées | [modules/ProspectLists/](../../SuiteCRM/modules/ProspectLists/) |
| `prospect_list_campaigns` | M2M | [metadata/prospect_list_campaignsMetaData.php](../../SuiteCRM/metadata/prospect_list_campaignsMetaData.php) |
| `prospect_lists_prospects` | M2M | [metadata/prospect_lists_prospectsMetaData.php](../../SuiteCRM/metadata/prospect_lists_prospectsMetaData.php) |
| `campaign_log` | suivi (envoi, click, opt-out, view…) | [modules/CampaignLog/](../../SuiteCRM/modules/CampaignLog/) |
| `campaign_trackers` | URL/codes de tracking | [modules/CampaignTrackers/](../../SuiteCRM/modules/CampaignTrackers/) |
| `emailman` | file d'envoi | [modules/EmailMan/](../../SuiteCRM/modules/EmailMan/) |
| `email_marketing` | définitions e-mailings | [modules/EmailMarketing/](../../SuiteCRM/modules/EmailMarketing/), [modules/EmailTemplates/](../../SuiteCRM/modules/EmailTemplates/) |

### 2.13 Sales (commerce)

| Table | Sens | Preuve module |
|---|---|---|
| `aos_quotes` | Devis | [modules/AOS_Quotes/](../../SuiteCRM/modules/AOS_Quotes/) |
| `aos_invoices` | Factures | [modules/AOS_Invoices/](../../SuiteCRM/modules/AOS_Invoices/) |
| `aos_contracts` | Contrats | [modules/AOS_Contracts/](../../SuiteCRM/modules/AOS_Contracts/) |
| `aos_products`, `aos_product_categories` | Catalogue | [modules/AOS_Products/](../../SuiteCRM/modules/AOS_Products/), [modules/AOS_Product_Categories/](../../SuiteCRM/modules/AOS_Product_Categories/) |
| `aos_products_quotes` | Lignes de devis | [modules/AOS_Products_Quotes/](../../SuiteCRM/modules/AOS_Products_Quotes/) |
| `aos_line_item_groups` | Groupes de lignes | [modules/AOS_Line_Item_Groups/](../../SuiteCRM/modules/AOS_Line_Item_Groups/) |
| `aos_pdf_templates` | Templates PDF | [modules/AOS_PDF_Templates/](../../SuiteCRM/modules/AOS_PDF_Templates/) |

### 2.14 Workflows & Reports

| Table | Sens |
|---|---|
| `aow_workflow` | Définition workflow | [modules/AOW_WorkFlow/](../../SuiteCRM/modules/AOW_WorkFlow/) |
| `aow_conditions` | Conditions | [modules/AOW_Conditions/](../../SuiteCRM/modules/AOW_Conditions/) |
| `aow_actions` | Actions | [modules/AOW_Actions/](../../SuiteCRM/modules/AOW_Actions/) |
| `aow_processed` | Trace par bean traité | [modules/AOW_Processed/](../../SuiteCRM/modules/AOW_Processed/) |
| `aow_processed_aow_actions` | M2M trace × action | [metadata/aow_processed_aow_actionsMetaData.php](../../SuiteCRM/metadata/aow_processed_aow_actionsMetaData.php) |
| `aor_reports`, `aor_fields`, `aor_conditions`, `aor_charts`, `aor_scheduled_reports` | Reporting | [modules/AOR_Reports/](../../SuiteCRM/modules/AOR_Reports/), siblings |

### 2.15 Calendar / Activities

| Table | Sens |
|---|---|
| `meetings`, `meetings_contacts`, `meetings_users`, `meetings_leads` | RDV + invités |
| `calls`, `calls_contacts`, `calls_users`, `calls_leads` | Appels |
| `tasks` | Tâches |
| `reminders`, `reminders_invitees` | Rappels |
| `calendar_accounts` | Comptes calendrier (external sync) |
| `calendar_accounts_meetings` | Mapping events ↔ calendars (M2M) — [metadata/calendar_accounts_meetingsMetaData.php](../../SuiteCRM/metadata/calendar_accounts_meetingsMetaData.php) |

### 2.16 Documents / Notes

| Table | Sens |
|---|---|
| `documents`, `document_revisions` | Documents versionnés (table 1:n) | [modules/Documents/](../../SuiteCRM/modules/Documents/), [modules/DocumentRevisions/](../../SuiteCRM/modules/DocumentRevisions/) |
| `documents_accounts`, `documents_contacts`, `documents_opportunities`, `documents_cases`, `documents_bugs` | M2M document ↔ entité | metadata `documents_*MetaData.php` |
| `notes` | Notes/attachments | [modules/Notes/](../../SuiteCRM/modules/Notes/) |

### 2.17 Multi-tenant / système

| Table | Sens |
|---|---|
| `config` | Config admin (clé/valeur) | [metadata/configMetaData.php](../../SuiteCRM/metadata/configMetaData.php) |
| `tracker`, `tracker_sessions`, `tracker_perf`, `tracker_queries` | Activité utilisateur | [modules/Trackers/](../../SuiteCRM/modules/Trackers/) |
| `audit` (`<module>_audit`) | Trace changements champs audités | template basic |
| `import_maps` | Import CSV mappings | [metadata/import_mapsMetaData.php](../../SuiteCRM/metadata/import_mapsMetaData.php) |
| `users_signatures` | Signatures e-mail | [metadata/users_signaturesMetaData.php](../../SuiteCRM/metadata/users_signaturesMetaData.php) |
| `inbound_email`, `inbound_email_autoreply`, `inbound_email_cache` | Comptes IMAP entrants | [metadata/inboundEmail_*MetaData.php](../../SuiteCRM/metadata/) |
| `email_cache` | Cache des en-têtes IMAP | [metadata/email_cacheMetaData.php](../../SuiteCRM/metadata/email_cacheMetaData.php) |
| `outbound_email` | Comptes SMTP sortants | [metadata/outboundEmailMetaData.php](../../SuiteCRM/metadata/outboundEmailMetaData.php) |
| `oauth_nonces` | Nonces zf1-OAuth | [metadata/oauth_nonce.php](../../SuiteCRM/metadata/oauth_nonce.php) |
| `currencies` | Devises | [modules/Currencies/](../../SuiteCRM/modules/Currencies/) |
| `releases`, `versions` | Versions logicielles | [modules/Releases/](../../SuiteCRM/modules/Releases/) |

## 3. Repérage rapide

Pour une table donnée, retrouver son schéma :

1. Trouver le module correspondant dans [modules/](../../SuiteCRM/modules/) (nom proche).
2. Ouvrir `<module>/vardefs.php` — toutes les colonnes y sont déclarées (`fields`).
3. Pour les relations M2M : chercher la table de liaison correspondante dans [metadata/](../../SuiteCRM/metadata/) (nom = `<lhs>_<rhs>` ou `<lhs>_<rhs>_NMetaData.php`).
