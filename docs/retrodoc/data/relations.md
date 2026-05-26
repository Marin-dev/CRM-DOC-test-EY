# Données — Relations & contraintes

> Phase **Writer**. Relations prouvées par les `metadata/*MetaData.php` (chargées par `modules/TableDictionary.php`). Les cardinalités ci-dessous proviennent du champ `relationship_type` (et `true_relationship_type` quand présent).

## 1. Modèle de relations dans SuiteCRM

SuiteCRM implémente 4 grandes catégories ([data/Relationships/](../../SuiteCRM/data/Relationships/)) :

| Classe | Cas d'usage | Fichier |
|---|---|---|
| `M2MRelationship` | Many-to-Many (avec join table) | [data/Relationships/M2MRelationship.php](../../SuiteCRM/data/Relationships/M2MRelationship.php) |
| `One2MBeanRelationship` | One-to-Many porté par une FK sur la table fille | [data/Relationships/One2MBeanRelationship.php](../../SuiteCRM/data/Relationships/One2MBeanRelationship.php) |
| `One2MRelationship` | One-to-Many avec join table | [data/Relationships/One2MRelationship.php](../../SuiteCRM/data/Relationships/One2MRelationship.php) |
| `One2OneBeanRelationship` / `One2OneRelationship` | One-to-One | [data/Relationships/One2OneBeanRelationship.php](../../SuiteCRM/data/Relationships/One2OneBeanRelationship.php) |
| `EmailAddressRelationship` | Cas particulier emails (via `email_addr_bean_rel`) | [data/Relationships/EmailAddressRelationship.php](../../SuiteCRM/data/Relationships/EmailAddressRelationship.php) |

Sélection à l'exécution : `RelationshipFactory` ([data/Relationships/RelationshipFactory.php](../../SuiteCRM/data/Relationships/RelationshipFactory.php)).

## 2. Relations principales (extrait prouvé)

> Cardinalités : valeurs `relationship_type` lues dans les `metadata/*MetaData.php` correspondants.

### 2.1 CRM Core

| Relation | LHS → RHS | Cardinalité | Join table | Preuve |
|---|---|---|---|---|
| `accounts_contacts` | Accounts ↔ Contacts | many-to-many | `accounts_contacts` (`account_id`, `contact_id`) | [metadata/accounts_contactsMetaData.php:44, 58-61](../../SuiteCRM/metadata/accounts_contactsMetaData.php#L44-L61) |
| `accounts_opportunities` | Accounts ↔ Opportunities | many-to-many | `accounts_opportunities` (`account_id`, `opportunity_id`) | [metadata/accounts_opportunitiesMetaData.php:44, 58-61](../../SuiteCRM/metadata/accounts_opportunitiesMetaData.php#L44-L61) |
| `accounts_cases` | Accounts ↔ Cases | many-to-many | `accounts_cases` | [metadata/accounts_casesMetaData.php](../../SuiteCRM/metadata/accounts_casesMetaData.php) |
| `accounts_bugs` | Accounts ↔ Bugs | many-to-many | `accounts_bugs` | [metadata/accounts_bugsMetaData.php:60](../../SuiteCRM/metadata/accounts_bugsMetaData.php#L60) |
| `contacts_cases` | Contacts ↔ Cases | many-to-many | `contacts_cases` | [metadata/contacts_casesMetaData.php](../../SuiteCRM/metadata/contacts_casesMetaData.php) |
| `contacts_bugs` | Contacts ↔ Bugs | many-to-many | `contacts_bugs` | [metadata/contacts_bugsMetaData.php](../../SuiteCRM/metadata/contacts_bugsMetaData.php) |
| `opportunities_contacts` | Opportunities ↔ Contacts | many-to-many | `opportunities_contacts` (avec `opportunity_role`) | [metadata/opportunities_contactsMetaData.php](../../SuiteCRM/metadata/opportunities_contactsMetaData.php), rôle ancré [modules/Contacts/vardefs.php:121-129](../../SuiteCRM/modules/Contacts/vardefs.php#L121-L129) |
| `cases_bugs` | Cases ↔ Bugs | many-to-many | `cases_bugs` | [metadata/cases_bugsMetaData.php](../../SuiteCRM/metadata/cases_bugsMetaData.php) |
| `contacts_users` | Contacts ↔ Users | many-to-many | `contacts_users` | [metadata/contacts_usersMetaData.php](../../SuiteCRM/metadata/contacts_usersMetaData.php) |

### 2.2 Activités

| Relation | Cardinalité | Join table |
|---|---|---|
| `meetings_users`, `meetings_contacts`, `meetings_leads` | many-to-many | tables éponymes |
| `calls_users`, `calls_contacts`, `calls_leads` | many-to-many | tables éponymes |
| `calendar_accounts_meetings` | many-to-many | `calendar_accounts_meetings` ([metadata/calendar_accounts_meetingsMetaData.php](../../SuiteCRM/metadata/calendar_accounts_meetingsMetaData.php)) |

### 2.3 Marketing

| Relation | Cardinalité | Join table |
|---|---|---|
| `prospect_lists_prospects` | many-to-many | `prospect_lists_prospects` |
| `prospect_list_campaigns` | many-to-many | `prospect_list_campaigns` |
| `email_marketing_prospect_lists` | many-to-many | `email_marketing_prospect_lists` ([metadata/email_marketing_prospect_listsMetaData.php](../../SuiteCRM/metadata/email_marketing_prospect_listsMetaData.php)) |

### 2.4 Documents / Notes

| Relation | Cardinalité | Join table |
|---|---|---|
| `documents_accounts` | many-to-many | `documents_accounts` |
| `documents_contacts` | many-to-many | `documents_contacts` |
| `documents_opportunities` | many-to-many | `documents_opportunities` |
| `documents_cases` | many-to-many | `documents_cases` |
| `documents_bugs` | many-to-many | `documents_bugs` |
| `linked_documents` | many-to-many | `linked_documents` ([metadata/linked_documentsMetaData.php](../../SuiteCRM/metadata/linked_documentsMetaData.php)) |
| `notes ↔ parent (poly)` | 1:n (polymorphique via `parent_type`, `parent_id`) | implémenté en colonnes directes sur `notes` |

### 2.5 ACL & Security

| Relation | Cardinalité | Join table | Preuve |
|---|---|---|---|
| `acl_roles_actions` | many-to-many (avec `access_override`) | `acl_roles_actions` | [metadata/acl_roles_actionsMetaData.php:99](../../SuiteCRM/metadata/acl_roles_actionsMetaData.php#L99) |
| `acl_roles_users` | many-to-many | `acl_roles_users` | [metadata/acl_roles_usersMetaData.php:93](../../SuiteCRM/metadata/acl_roles_usersMetaData.php#L93) |
| `securitygroups_users` | many-to-many | `securitygroups_users` | [metadata/securitygroups_usersMetaData.php](../../SuiteCRM/metadata/securitygroups_usersMetaData.php) |
| `securitygroups_acl_roles` | many-to-many | `securitygroups_acl_roles` | [metadata/securitygroups_acl_rolesMetaData.php](../../SuiteCRM/metadata/securitygroups_acl_rolesMetaData.php) |
| `securitygroups_records` | many-to-many polymorphique (`module_name`) | `securitygroups_records` | [metadata/securitygroups_recordsMetaData.php](../../SuiteCRM/metadata/securitygroups_recordsMetaData.php) |
| `securitygroups_defaults` | many-to-many (module → group) | `securitygroups_defaults` | [metadata/securitygroups_defaultsMetaData.php](../../SuiteCRM/metadata/securitygroups_defaultsMetaData.php) |

### 2.6 Workflow

| Relation | Cardinalité | Join table |
|---|---|---|
| `aow_processed ↔ aow_actions` | many-to-many | `aow_processed_aow_actions` ([metadata/aow_processed_aow_actionsMetaData.php](../../SuiteCRM/metadata/aow_processed_aow_actionsMetaData.php)) |

### 2.7 Projets / Tâches

| Relation | Cardinalité | Join table |
|---|---|---|
| `am_projecttemplates_contacts_1` | many-to-many (`true_relationship_type = many-to-many`) | `am_projecttemplates_contacts_1` ([metadata/am_projecttemplates_contacts_1MetaData.php:4](../../SuiteCRM/metadata/am_projecttemplates_contacts_1MetaData.php#L4)) |
| `am_projecttemplates_project_1` | one-to-many (true) ; stocké en M2M | [metadata/am_projecttemplates_project_1MetaData.php:4](../../SuiteCRM/metadata/am_projecttemplates_project_1MetaData.php#L4) |
| `am_projecttemplates_users_1` | many-to-many | [metadata/am_projecttemplates_users_1MetaData.php](../../SuiteCRM/metadata/am_projecttemplates_users_1MetaData.php) |
| `am_tasktemplates_am_projecttemplates` | one-to-many (true) ; stocké en M2M | [metadata/am_tasktemplates_am_projecttemplatesMetaData.php:4](../../SuiteCRM/metadata/am_tasktemplates_am_projecttemplatesMetaData.php#L4) |
| `projects_accounts`, `projects_contacts`, `projects_opportunities` | many-to-many | metadata éponymes |
| `project_bugs`, `project_cases`, `project_products` | many-to-many | metadata éponymes |

### 2.8 Quotes / Contracts / Invoices

| Relation | Cardinalité (true) | Join table |
|---|---|---|
| `aos_quotes_aos_contracts` | many-to-many | `aos_quotes_aos_contracts` ([metadata/aos_quotes_aos_contractsMetaData.php:41](../../SuiteCRM/metadata/aos_quotes_aos_contractsMetaData.php#L41)) |
| `aos_quotes_aos_invoices` | many-to-many | `aos_quotes_aos_invoices` ([metadata/aos_quotes_aos_invoicesMetaData.php:41](../../SuiteCRM/metadata/aos_quotes_aos_invoicesMetaData.php#L41)) |
| `aos_quotes_project` | many-to-many | `aos_quotes_project` ([metadata/aos_quotes_projectMetaData.php](../../SuiteCRM/metadata/aos_quotes_projectMetaData.php)) |
| `aos_contracts_documents` | many-to-many | `aos_contracts_documents` ([metadata/aos_contracts_documentsMetaData.php:41](../../SuiteCRM/metadata/aos_contracts_documentsMetaData.php#L41)) |

### 2.9 Events / Maps / Knowledge

| Relation | Cardinalité | Join table |
|---|---|---|
| `fp_event_locations_fp_events_1` | many-to-many | éponyme ([metadata/fp_event_locations_fp_events_1MetaData.php](../../SuiteCRM/metadata/fp_event_locations_fp_events_1MetaData.php)) |
| `fp_events_contacts`, `fp_events_leads_1`, `fp_events_prospects_1`, `fp_events_fp_event_delegates_1`, `fp_events_fp_event_locations_1` | many-to-many | metadata éponymes |
| `jjwg_maps_jjwg_areas`, `jjwg_maps_jjwg_markers` | many-to-many | metadata éponymes |
| `aok_knowledgebase_categories` | many-to-many | éponyme ([metadata/aok_knowledgebase_categoriesMetaData.php:42](../../SuiteCRM/metadata/aok_knowledgebase_categoriesMetaData.php#L42)) |
| `surveyquestionoptions_surveyquestionresponses` | many-to-many | éponyme ([metadata/surveyquestionoptions_surveyquestionresponsesMetaData.php](../../SuiteCRM/metadata/surveyquestionoptions_surveyquestionresponsesMetaData.php)) |

### 2.10 Email index

`email_addresses` ↔ « tous beans » via `email_addr_bean_rel` :

| Colonne | Sens |
|---|---|
| `id`, `email_address_id`, `bean_id`, `bean_module` | clé composite logique |
| `primary_address` (bool) | flag adresse principale |
| `reply_to_address` (bool) | flag reply-to |
| `date_created`, `date_modified`, `deleted` | std |

Preuve indirecte : SQL forgée par `ModuleService::getRecords` ([Api/V8/Service/ModuleService.php:147-150](../../SuiteCRM/Api/V8/Service/ModuleService.php#L147-L150)).

## 3. Self-référence

Les modules « company-template » et « person-template » exposent des liens auto-référents :

- `Accounts.parent_id` → `Accounts.id` (`members`/`member_of`) — preuve [modules/Accounts/vardefs.php:55-100](../../SuiteCRM/modules/Accounts/vardefs.php#L55-L100).
- `Contacts.reports_to_id` → `Contacts.id` — preuve [modules/Contacts/vardefs.php:137-146](../../SuiteCRM/modules/Contacts/vardefs.php#L137-L146).

## 4. Contraintes (ON DELETE / ON UPDATE / FK)

- **Soft delete** systématique : `deleted = 1` au lieu d'un `DELETE`. Conséquence : pas de cascade physique côté DB. Preuve : le flag `deleted` est défini dans tous les vardefs et les `metadata/*MetaData.php` (`'deleted' => 'bool'`).
- Les `id_name` Sugar ne créent pas systématiquement de **FOREIGN KEY** physiques au sens DDL. Les sub-modèles avec `relationship_type = many-to-many` créent des tables d'association séparées (avec PK + `alternate_key` sur la paire de FKs logiques) — exemple `accounts_opportunities` ([metadata/accounts_opportunitiesMetaData.php:52-55](../../SuiteCRM/metadata/accounts_opportunitiesMetaData.php#L52-L55)).
- INCONNU : présence éventuelle de FK SQL strictes côté MariaDB/MySQL (dépend de la version du moteur et des migrations Studio appliquées). Sur une instance live, vérifier via `SHOW CREATE TABLE ...`.

## 5. Comment lire un fichier `metadata/<rel>MetaData.php`

Structure standard :

```php
$dictionary['<relName>'] = array(
  'table'  => '<joinTable>',
  'fields' => array(
    array('name' =>'id',            'type' =>'varchar', 'len' => 36),
    array('name' =>'<lhs_fk>',      'type' =>'varchar', 'len' => 36),
    array('name' =>'<rhs_fk>',      'type' =>'varchar', 'len' => 36),
    array('name' =>'date_modified', 'type' =>'datetime'),
    array('name' =>'deleted',       'type' =>'bool',    'default' => 0),
  ),
  'indices' => array(
    array('name' => '<relName>pk',     'type' => 'primary',       'fields' => array('id')),
    array('name' => 'idx_<lhs>_<rhs>', 'type' => 'alternate_key', 'fields' => array('<lhs_fk>','<rhs_fk>')),
  ),
  'relationships' => array('<relName>' => array(
    'lhs_module' => '<LhsModule>', 'lhs_table' => '<lhs_table>', 'lhs_key' => 'id',
    'rhs_module' => '<RhsModule>', 'rhs_table' => '<rhs_table>', 'rhs_key' => 'id',
    'relationship_type' => 'many-to-many',
    'join_table' => '<joinTable>', 'join_key_lhs' => '<lhs_fk>', 'join_key_rhs' => '<rhs_fk>',
  )),
);
```
Preuve exemplaire : [metadata/accounts_opportunitiesMetaData.php:44-62](../../SuiteCRM/metadata/accounts_opportunitiesMetaData.php#L44-L62).
