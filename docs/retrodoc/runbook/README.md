# Runbook

> Phase **Writer**. Procédures d'exploitation. Toutes les commandes/chemins sont prouvés dans le repo.

## 1. Prérequis système

Repris du README ([SuiteCRM/README.md:34-39](../../SuiteCRM/README.md#L34-L39)) et de [composer.json](../../SuiteCRM/composer.json) :

| Élément | Valeur |
|---|---|
| PHP | 8.1 → 8.4 (composer cible `^8.1` — [composer.json:35](../../SuiteCRM/composer.json#L35)) |
| Serveur web | Apache (recommandé) ou IIS |
| BDD | MySQL / MariaDB (recommandé) ou MSSQL |
| Extensions PHP requises | `curl`, `gd`, `intl`, `json`, `openssl`, `zip` ([composer.json:36-41](../../SuiteCRM/composer.json#L36-L41)) |
| Extension suggérée | `imap` (module Emails) ([composer.json:32](../../SuiteCRM/composer.json#L32)) |

## 2. Installation (manuelle)

### 2.1 Étapes UI

1. Cloner / déposer le contenu de `SuiteCRM/` à la racine du virtual host (ex. `/var/www/SuiteCRM/`).
2. `composer install` (gère aussi le merge plugin → composer.ext.json + custom/Extension Composer). Preuve : [composer.json:128-149](../../SuiteCRM/composer.json#L128-L149).
3. Ouvrir `http://<host>/install.php` (preuve [install.php:41-42](../../SuiteCRM/install.php#L41-L42)).
4. Remplir l'écran d'installation web ; il génère `config.php` à la racine. INCONNU : valeurs effectives — voir defaults [install/install_defaults.php:42-50](../../SuiteCRM/install/install_defaults.php#L42-L50).

### 2.2 Installation silencieuse (CI)

- Préparer `config_si.php` avec les valeurs (DB, admin user, site_url). Le fichier est fusionné dans `config.php` automatiquement ([install/install_utils.php:936-939, 1653-1654](../../SuiteCRM/install/install_utils.php#L936-L1654)).
- Lancer `php install.php` (preuve installeur disponible en mode CLI : [install.php:41-42](../../SuiteCRM/install.php#L41-L42)).

### 2.3 Post-install — API V8

1. Générer les clés RSA pour OAuth2 :
   ```bash
   openssl genrsa -out Api/V8/OAuth2/private.key 2048
   openssl rsa -in Api/V8/OAuth2/private.key -pubout -out Api/V8/OAuth2/public.key
   chmod 600 Api/V8/OAuth2/*.key
   ```
   Chemins prouvés : [Api/Core/Config/ApiConfig.php:20-21](../../SuiteCRM/Api/Core/Config/ApiConfig.php#L20-L21).
2. Renseigner `$sugar_config['oauth2_encryption_key']` dans `config.php` (clé aléatoire forte). Sans elle, fallback `SCRM-DEFK` + log fatal — [middlewares.php:31-37](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L31-L37).
3. Créer un client OAuth2 via l'UI admin (module `OAuth2Clients`).

### 2.4 Réécriture web pour PHP-FPM

Le code lit `REDIRECT_HTTP_AUTHORIZATION` si `HTTP_AUTHORIZATION` est absent ([Api/Core/app.php:13-18](../../SuiteCRM/Api/Core/app.php#L13-L18)). Le `.htaccess` Apache doit donc forwarder l'`Authorization` header (config Apache spécifique non versionnée dans le repo → INCONNU précis).

## 3. Configuration du cron

Ajouter au crontab du compte web (ex. `www-data`) :

```cron
* * * * * php /var/www/SuiteCRM/cron.php >/dev/null 2>&1
```

Garde-fou : `cron.php` refuse de tourner si l'utilisateur n'est pas dans `$sugar_config['cron']['allowed_cron_users']` ([cron.php:54-77](../../SuiteCRM/cron.php#L54-L77)). En particulier, **`root` est explicitement déconseillé** par le log fatal ([cron.php:63-67](../../SuiteCRM/cron.php#L63-L67)).

## 4. Variables `$sugar_config` clés

Configurables dans `config.php` (généré à l'install) :

| Clé | Sens | Preuve |
|---|---|---|
| `dbconfig` (`db_type`, `db_host_name`, `db_user_name`, `db_password`, `db_name`, `db_port`) | Connexion DB | [include/database/DBManagerFactory.php:144-147](../../SuiteCRM/include/database/DBManagerFactory.php#L144-L147) |
| `db_manager`, `db_manager_class` | Override driver | [DBManagerFactory.php:104-119](../../SuiteCRM/include/database/DBManagerFactory.php#L104-L119) |
| `mysqli_disabled` | Force `MysqlManager` au lieu de `MysqliManager` | [DBManagerFactory.php:80](../../SuiteCRM/include/database/DBManagerFactory.php#L80) |
| `db_mssql_force_driver` | `sqlsrv` ou `freetds` | [DBManagerFactory.php:88-92](../../SuiteCRM/include/database/DBManagerFactory.php#L88-L92) |
| `unique_key` | Anti session hijack | [SugarApplication.php:112-120](../../SuiteCRM/include/MVC/SugarApplication.php#L112-L120) |
| `cron_class` | Driver cron (def `SugarCronJobs`) | [cron.php:90-91](../../SuiteCRM/cron.php#L90-L91) |
| `cron.allowed_cron_users` | Liste d'utilisateurs autorisés à exécuter `cron.php` | [cron.php:59-69](../../SuiteCRM/cron.php#L59-L69) |
| `default_module`, `default_language` | UI | [SugarApplication.php:77-79](../../SuiteCRM/include/MVC/SugarApplication.php#L77-L79), [cron.php:80](../../SuiteCRM/cron.php#L80) |
| `oauth2_encryption_key` | Chiffrement OAuth2 | [middlewares.php:31-37](../../SuiteCRM/Api/V8/Config/services/middlewares.php#L31-L37) |
| `site_url` | URL publique (WSDL SOAP, OAuth2 callbacks) | [soap.php:67](../../SuiteCRM/soap.php#L67) |
| `cache_dir` | Cache | [include/utils/file_utils.php:67-70](../../SuiteCRM/include/utils/file_utils.php#L67-L70) |

INCONNU : la liste complète sera dans `config.php` post-install + admin Configurator (`modules/Configurator/`).

## 5. Procédures courantes

### 5.1 Repair / Rebuild (admin)

Application des vardefs/metadata sur le schéma DB : Admin → **Repair → Quick Repair and Rebuild**. Code : [modules/Administration/](../../SuiteCRM/modules/Administration/) (workflow standard SugarCRM). Effet : génération du DDL ALTER nécessaire à partir des `vardefs.php` + `metadata/*MetaData.php`.

### 5.2 Réinitialiser les schedulers OOTB

`Scheduler::rebuildDefaultSchedulers()` ([Scheduler.php:815-1003](../../SuiteCRM/modules/Schedulers/Scheduler.php#L815-L1003)) — appelable depuis l'UI Admin → Schedulers ou via un script utilisant `BeanFactory::newBean('Schedulers')`.

### 5.3 Vidage du cache

Le cache vit dans `$sugar_config['cache_dir']` (souvent `cache/`). Suppression manuelle : `rm -rf cache/*` (préserver `cache/`). Aucune commande CLI dédiée détectée dans le repo.

### 5.4 Mode maintenance

Endpoint `maintenance.php` ([maintenance.php](../../SuiteCRM/maintenance.php)) à activer côté serveur web (rewriting) pendant les opérations bloquantes.

### 5.5 Mise à niveau

`include/UpgradeWizard/` + `modules/UpgradeWizard/` ([modules/UpgradeWizard/](../../SuiteCRM/modules/UpgradeWizard/)). Le README pointe vers la doc officielle [https://docs.suitecrm.com/admin/installation-guide/upgrading/](../../SuiteCRM/README.md#L53).

### 5.6 Logs

- Application : configurable via `monolog` (canal `$GLOBALS['log']`) — fichier par défaut INCONNU sans config (typiquement `suitecrm.log`).
- Cron : redirigé via crontab `>/dev/null 2>&1` (à enlever en debug).
- Web : journaux Apache/Nginx + PHP-FPM.

## 6. Sauvegarde

Items critiques à sauvegarder :

| Item | Localisation |
|---|---|
| BDD | dump MySQL / MariaDB / MSSQL complet |
| `config.php` | racine SuiteCRM |
| `Api/V8/OAuth2/*.key` | clés RSA OAuth2 |
| `upload/` | pièces jointes utilisateur ([upload/](../../SuiteCRM/upload/)) |
| `custom/` | personnalisations |
| `cache/` | non requis (rebuild possible) |

## 7. Tests automatisés

- PHPUnit : `vendor/bin/phpunit --configuration tests/unit/phpunit.xml.dist` (preuve config [tests/unit/phpunit.xml.dist](../../SuiteCRM/tests/unit/phpunit.xml.dist) — INCONNU contenu exact).
- Codeception : `vendor/bin/codecept run` (config racine [codeception.dist.yml:1-13](../../SuiteCRM/codeception.dist.yml#L1-L13)). Suites : `acceptance`, `api`, `install` ([tests/acceptance.suite.yml](../../SuiteCRM/tests/acceptance.suite.yml), [tests/api.suite.yml](../../SuiteCRM/tests/api.suite.yml), [tests/install.suite.yml](../../SuiteCRM/tests/install.suite.yml)).
- Lanceur all-in-one : `tests/runtests.sh` ([tests/runtests.sh](../../SuiteCRM/tests/runtests.sh)).
- Style PHP : `vendor/bin/phpcs --standard=phpcs.xml` ([phpcs.xml](../../SuiteCRM/phpcs.xml)).
- PHP-CS-Fixer : `vendor/bin/php-cs-fixer` ([composer.json:89](../../SuiteCRM/composer.json#L89)).

## 8. Procédures de récupération

### 8.1 Mot de passe admin perdu

1. Régénérer via `?entryPoint=GeneratePassword` (auth `false`) — [entry_point_registry.php:51](../../SuiteCRM/include/MVC/Controller/entry_point_registry.php#L51).
   *Note de sécurité* : cet entrypoint doit être restreint en prod (modules/Users/GeneratePassword.php).
2. Ou : `UPDATE users SET user_hash = '<bcrypt>' WHERE user_name = 'admin'` (puis vidage cache).

### 8.2 Cron bloqué

1. Vérifier `job_queue` : présence de jobs `status = 'running'` anciens.
2. `cleanJobQueue` (scheduler OOTB, cron `0::5::*::*::*`) nettoie les jobs anormalement bloqués ([Scheduler.php:944-954](../../SuiteCRM/modules/Schedulers/Scheduler.php#L944-L954)).
3. Forcer : `UPDATE job_queue SET status='done' WHERE status='running' AND date_modified < DATE_SUB(NOW(), INTERVAL 1 HOUR)`.

### 8.3 OAuth2 — token rejeté

- Vérifier mtime / permissions `Api/V8/OAuth2/*.key` (`chmod 600` Unix).
- Vérifier `oauth2_encryption_key` (sinon log `fatal` + clé `SCRM-DEFK`).
- Vérifier date système : un décalage trop important fait expirer les JWT.

## 9. Sécurité d'exploitation

- Ne pas laisser `Access-Control-Allow-Origin: *` exposé directement (utiliser un reverse proxy avec une whitelist d'origines) — [Api/Core/app.php:3](../../SuiteCRM/Api/Core/app.php#L3).
- Restreindre les entry points marqués `auth=false` qui ne sont pas indispensables (formulaires marketing, trackers) — [entry_point_registry.php](../../SuiteCRM/include/MVC/Controller/entry_point_registry.php).
- Désactiver les anciennes stacks (`soap.php`, `service/v*/`) si non utilisées par le SI client (suppression côté .htaccess) — INCONNU dans le repo.
- Vérifier que `composer install` n'installe pas les `require-dev` en production (`--no-dev`).
