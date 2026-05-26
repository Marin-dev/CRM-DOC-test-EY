# ADR 02 — Persistance : ORM maison `SugarBean` plutôt qu'ORM tiers

## Statut
Acceptée *(décision héritée, lisible dans le code 7.15.1)*.

## Contexte

SuiteCRM hérite de SugarCRM Community Edition. Le cœur du domaine est `SugarBean` ([data/SugarBean.php:62](../../SuiteCRM/data/SugarBean.php#L62)) qui sert à la fois :

- de **mapping** vers la table principale (via `vardefs.php`),
- de **service CRUD** (`save()`, `retrieve()`, `process_list_query()`),
- de **moteur de relations** (via `data/Relationships/*` et `RelationshipFactory`),
- de **garde-fou ACL** (`ACLAccess()`).

Aucun ORM tiers de PHP moderne (Doctrine, Eloquent…) n'est utilisé.

Preuves :

- Pas de dépendance `doctrine/orm`, `illuminate/database` dans [composer.json:34-76](../../SuiteCRM/composer.json#L34-L76).
- `data/SugarBean.php` annoté `@api` ([data/SugarBean.php:61](../../SuiteCRM/data/SugarBean.php#L61)).
- Choix du driver DB à l'exécution via `DBManagerFactory` ([include/database/DBManagerFactory.php:60-126](../../SuiteCRM/include/database/DBManagerFactory.php#L60-L126)).

## Décision implicite

| Choix | Détail | Preuve |
|---|---|---|
| ORM maison « SugarBean » | Base unique pour le domaine ; chaque module y hérite | [data/SugarBean.php:62](../../SuiteCRM/data/SugarBean.php#L62), exemples : `Campaign extends SugarBean` ([modules/Campaigns/Campaign.php:50](../../SuiteCRM/modules/Campaigns/Campaign.php#L50)), `Scheduler extends SugarBean` ([modules/Schedulers/Scheduler.php:48](../../SuiteCRM/modules/Schedulers/Scheduler.php#L48)), `InboundEmail extends SugarBean` ([modules/InboundEmail/InboundEmail.php:54](../../SuiteCRM/modules/InboundEmail/InboundEmail.php#L54)) |
| Métadonnées en **PHP** (vardefs) | Pas de DDL SQL versionné ; le schéma se déduit des vardefs | exemple [modules/Accounts/vardefs.php:45-52](../../SuiteCRM/modules/Accounts/vardefs.php#L45-L52) |
| Relations dans `metadata/*MetaData.php` | 88 fichiers chargés par [modules/TableDictionary.php:45-138](../../SuiteCRM/modules/TableDictionary.php#L45-L138) |
| Multi-driver (MySQL/MSSQL) avec sélection runtime | `DBManagerFactory::getTypeInstance` | [include/database/DBManagerFactory.php:78-103](../../SuiteCRM/include/database/DBManagerFactory.php#L78-L103) |
| Soft delete | colonne `deleted` (bool) sur (quasi) toutes les tables | preuve générique : flag `deleted` dans tous les vardefs métier |
| Audit | `<module>_audit` si `audited => true` | [modules/Accounts/vardefs.php:47](../../SuiteCRM/modules/Accounts/vardefs.php#L47) |
| Custom fields | table `<module>_cstm` + module `DynamicFields` | [modules/DynamicFields/](../../SuiteCRM/modules/DynamicFields/) |

## Conséquences observées

✅ Avantages constatables :

- **Portabilité DB** : un même codebase tourne sur MySQL/MariaDB ou MSSQL via 5 drivers (`MysqliManager`, `MysqlManager`, `SqlsrvManager`, `FreeTDSManager`, `MssqlManager`).
- **Studio / ModuleBuilder** : la métadonnée pilotant le schéma permet la création de modules au runtime (admin UI), sans coder.
- **Customisation persistante aux upgrades** : `custom/Extension/...` étend les vardefs et metadata sans toucher au cœur.

⚠️ Coûts constatables :

- **Pas de migrations SQL versionnées** : le schéma vit dans le code (vardefs + metadata). Diffs entre versions plus difficiles à auditer.
- **FK relâchées** : les `id_name` ne créent pas systématiquement de `FOREIGN KEY` côté DDL ; la cohérence est portée par l'app (cf. [data/relations.md §4](../data/relations.md#4-contraintes-on-delete--on-update--fk)).
- **Couplage fort** au design SugarCRM 6.x : exemples — `Api/V8/Service/ModuleService::getRecords` connait directement les tables `email_addresses` + `email_addr_bean_rel` au lieu d'utiliser un repository ([ModuleService.php:131-187](../../SuiteCRM/Api/V8/Service/ModuleService.php#L131-L187)).
- **Tests unitaires** : difficiles à isoler du framework Sugar (les tests Codeception couvrent surtout l'acceptance et l'API).
- **Performance** : `process_list_query` construit du SQL dynamique sensible aux conventions vardefs ; tuning par DBA limité.

## Recommandations issues du code

- Pour des intégrations modernes, privilégier l'API V8 (JSON:API + OAuth2) qui encapsule SugarBean — ne pas écrire de SQL direct dans les intégrations.
- Pour la BI / reporting, utiliser les modules `AOR_Reports` ou des vues SQL en *lecture seule* sur les tables principales.
- Toujours appliquer un « Repair → Quick Repair and Rebuild » après modification d'un `vardefs.php` ou d'un `metadata/*MetaData.php` (cf. [runbook §5.1](../runbook/README.md#51-repair--rebuild-admin)).
