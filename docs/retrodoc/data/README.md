# Données — Vue d'ensemble

> Phase **Writer**. SuiteCRM utilise un ORM maison basé sur `SugarBean`. Le schéma se décrit par les **vardefs** (champs) et les **metadata** (relations).

## 1. Sources de vérité du schéma

| Source | Rôle | Preuve |
|---|---|---|
| `<module>/vardefs.php` | Champs, types, contraintes, indexes au niveau bean | ex. [modules/Accounts/vardefs.php:45-52](../../SuiteCRM/modules/Accounts/vardefs.php#L45-L52) |
| `metadata/*MetaData.php` (88 fichiers) | Tables de liaison M2M (`relationship_type` + `join_table`) | ex. [metadata/accounts_contactsMetaData.php:44-62](../../SuiteCRM/metadata/accounts_contactsMetaData.php#L44-L62) |
| `modules/TableDictionary.php` | Inclut tous les `*MetaData.php` | [modules/TableDictionary.php:45-138](../../SuiteCRM/modules/TableDictionary.php#L45-L138) |
| `modules/BeanDictionary.php` | Recense la liste des beans connus | [modules/BeanDictionary.php](../../SuiteCRM/modules/BeanDictionary.php) |
| `data/Relationships/` | Implémentations des cardinalités (M2M, 1:n, 1:1, EmailAddress) | [data/Relationships/](../../SuiteCRM/data/Relationships/) |

## 2. Documents du dossier

- [modele_donnees.md](modele_donnees.md) — tables principales + clés
- [relations.md](relations.md) — relations et contraintes
- ERD visuel → [diagrams/mermaid_erd.md](../diagrams/mermaid_erd.md)

## 3. Conventions universelles SugarBean

Champs implicites présents sur (quasi) toutes les tables métier (issus des templates `include/SugarObjects/templates/`) :

| Colonne | Type | Sens | Preuve |
|---|---|---|---|
| `id` | varchar(36) UUID | PK | présent dans tous les vardefs métier (vardefs `id`) |
| `name` | varchar | libellé court | ex. [Accounts/vardefs.php:46-72](../../SuiteCRM/modules/Accounts/vardefs.php#L46-L72) |
| `date_entered` | datetime | création | template `basic` ([include/SugarObjects/templates/basic/](../../SuiteCRM/include/SugarObjects/templates/basic/)) |
| `date_modified` | datetime | dernière modification | idem |
| `modified_user_id` | id | auteur dernière modif | idem |
| `created_by` | id | auteur création | idem |
| `description` | text | description longue | idem |
| `deleted` | bool | soft delete (`0/1`) | idem |
| `assigned_user_id` | id | propriétaire | template `person` / `sale` |

## 4. Modules / Tables principaux (extrait)

| Module | Bean | Table | Catégorie |
|---|---|---|---|
| Accounts | `Account` | `accounts` | CRM core |
| Contacts | `Contact` | `contacts` | CRM core |
| Leads | `Lead` | `leads` | CRM core |
| Opportunities | `Opportunity` | `opportunities` | CRM core |
| Cases | `aCase` | `cases` | Support |
| Bugs | `Bug` | `bugs` | Support |
| Emails | `Email` | `emails` | Communication |
| InboundEmail | `InboundEmail` | `inbound_email` | Communication |
| EmailAddresses | `EmailAddress` | `email_addresses` | Index emails |
| Notes | `Note` | `notes` | Pièces jointes |
| Documents | `Document` + `DocumentRevisions` | `documents`, `document_revisions` | GED |
| Campaigns | `Campaign` | `campaigns` | Marketing |
| Prospects, ProspectLists | `Prospect`, `ProspectList` | `prospects`, `prospect_lists` | Marketing |
| AOS_Quotes / AOS_Invoices / AOS_Contracts | … | `aos_quotes`, `aos_invoices`, `aos_contracts` | Sales (commerce) |
| AOS_Products | `AOS_Products` | `aos_products` | Catalogue produit |
| AOR_Reports | `AOR_Report` | `aor_reports` | Reporting |
| AOW_WorkFlow | `AOW_WorkFlow` | `aow_workflow` | Workflows |
| AOK_KnowledgeBase | `AOK_KnowledgeBase` | `aok_knowledgebase` | KB |
| Users | `User` | `users` | IAM |
| ACLRoles / ACLActions | … | `acl_roles`, `acl_actions` | RBAC |
| SecurityGroups | `SecurityGroup` | `securitygroups` | RBAC |
| Schedulers | `Scheduler` | `schedulers` | Cron |
| SchedulersJobs | `SchedulersJob` | `job_queue` | Cron |
| OAuth2Clients / OAuth2Tokens / OAuth2AuthCodes | … | `oauth2clients`, `oauth2tokens`, `oauth2authcodes` | API auth |
| Calendar (Calls, Meetings, Tasks, Reminders) | `Call`, `Meeting`, `Task`, `Reminder` | `calls`, `meetings`, `tasks`, `reminders` | Calendrier |
| Surveys, SurveyQuestions, SurveyResponses | … | `surveys`, `surveyquestions`, `surveyresponses` | Sondages |
| FP_events, FP_Event_Locations | `FP_events`, `FP_Event_Locations` | `fp_events`, `fp_event_locations` | Events |
| jjwg_Maps, jjwg_Markers, jjwg_Areas | … | `jjwg_maps`, `jjwg_markers`, `jjwg_areas` | Maps |

Preuves : voir vardefs respectifs dans [modules/](../../SuiteCRM/modules/) et `TableDictionary.php` ([45-138](../../SuiteCRM/modules/TableDictionary.php#L45-L138)).

## 5. Conventions des champs **relate** & **link**

- `type: relate` — colonne « affichage » (`account_name`) pointant vers `<table>.name` via `id_name` (ex `account_id`) ; *non-DB* (`source => 'non-db'`) sauf override. Exemple : [Opportunities/vardefs.php:73-92](../../SuiteCRM/modules/Opportunities/vardefs.php#L73-L92).
- `type: link` — relation déclarative (vers une `relationship` enregistrée par `RelationshipFactory`) ; *non-DB*. Exemple : [Opportunities/vardefs.php:131-138](../../SuiteCRM/modules/Opportunities/vardefs.php#L131-L138) (`campaign_opportunities`).

## 6. Custom fields

Les champs personnalisés (Studio) sont stockés dans une table `<module>_cstm` jointe par PK = `id_c = parent.id`. Gestion : `modules/DynamicFields/` ([modules/DynamicFields/](../../SuiteCRM/modules/DynamicFields/)). Le bean parent fait un `JOIN <module>_cstm ON id = <module>.id` à la lecture.

## 7. Indexation

Indexes définis dans les `indices` :

- des vardefs (ex. [modules/Opportunities/vardefs.php:407-422](../../SuiteCRM/modules/Opportunities/vardefs.php#L407-L422))
- ou dans les `metadata/*MetaData.php` (ex. [metadata/accounts_opportunitiesMetaData.php:52-55](../../SuiteCRM/metadata/accounts_opportunitiesMetaData.php#L52-L55))

Types observés : `primary`, `alternate_key`, `index`.

## 8. Audit

Les modules ayant `audited => true` dans leurs vardefs (Accounts, Contacts, Leads, Opportunities…) déclenchent la création d'une table `<module>_audit` qui stocke chaque changement de champs `audited`. Preuve : [modules/Accounts/vardefs.php:47](../../SuiteCRM/modules/Accounts/vardefs.php#L47), [modules/Opportunities/vardefs.php:45](../../SuiteCRM/modules/Opportunities/vardefs.php#L45).

## 9. Limites / INCONNU

- Le DDL exact (CREATE TABLE) n'est pas dans le repo : il est *généré* par SuiteCRM via les vardefs/metadata lors de l'installation et des « repair / rebuild ». Le repo ne contient pas de migrations SQL versionnées au sens classique.
- Les contraintes FK ne sont pas systématiquement appliquées au niveau base : SuiteCRM utilise des `id_name` (FK logique) sans `FOREIGN KEY` côté DB par défaut (à l'exception possible de certaines tables ; INCONNU exhaustif sans inspecter le DDL généré sur une instance live).
- Les politiques `ON DELETE / ON UPDATE` ne sont donc pas garanties côté DB : la « suppression » est un soft delete (`deleted = 1`) côté SugarBean — preuve dans tous les vardefs (`'deleted'`).
