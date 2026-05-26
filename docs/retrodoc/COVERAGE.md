# Matrice de couverture — exigences de documentation technique

> Maintenue par le Writer · auditée par le Verifier ([adr/00_rapport_verification.md](adr/00_rapport_verification.md)).
> Légende statut : ✅ Couvert (avec preuves) · ⚠️ Partiel · ❌ Non couvert · 🚫 Non applicable · ❓ INCONNU
>
> Dernière mise à jour : 2026-05-26.

---

## 1. Architecture

| # | Exigence | Page cible | Statut | Notes / preuves |
|---|---|---|---|---|
| 1.1 | **Diagramme d'architecture** — Frontend (UI Sugar, thèmes) | `diagrams/mermaid_c4.md`, `diagrams/drawio_architecture.drawio` | ✅ | UI Sugar (Smarty, SuiteP) prouvée dans [architecture/00_inventaire.md §3](architecture/00_inventaire.md), [themes/SuiteP/themedef.php:50-54](../../SuiteCRM/themes/SuiteP/themedef.php#L50-L54) |
| 1.2 | **Diagramme d'architecture** — Backend (API, services, workers) | idem | ✅ | API V8 (Slim) + APIs legacy + Cron — composants détaillés dans [diagrams/mermaid_c4.md §3](diagrams/mermaid_c4.md#3-niveau-3-composants-du-conteneur-web) |
| 1.3 | **Diagramme d'architecture** — Base de données | idem + `diagrams/mermaid_erd.md` | ✅ | C4 niveau 2 + ERD complet ([diagrams/mermaid_erd.md](diagrams/mermaid_erd.md)) |
| 1.4 | **Diagramme d'architecture** — Infrastructure (cloud, CDN, LB) | `diagrams/mermaid_c4.md`, `diagrams/drawio_architecture.drawio` | ⚠️ | Pas d'IaC dans le repo — architecture LAMP générique citée + `INC-01` |
| 1.5 | **Diagramme d'architecture** — Flux entre composants | idem | ✅ | Flèches typées Mermaid + Draw.io |
| 1.6 | **Stack technique** — Langages | `architecture/00_inventaire.md` | ✅ | [architecture/00_inventaire.md §2.1](architecture/00_inventaire.md#21-langages) |
| 1.7 | **Stack technique** — Frameworks | idem | ✅ | [architecture/00_inventaire.md §2.2](architecture/00_inventaire.md#22-frameworks--librairies-majeures-extrait-composerjson) |
| 1.8 | **Stack technique** — Infra | idem | ⚠️ | Stack runtime (PHP/Apache) prouvée, IaC = INCONNU [§2.4](architecture/00_inventaire.md#24-infrastructure--deploiement) |
| 1.9 | **Stack technique** — Outils (lint, test, build) | idem | ✅ | [architecture/00_inventaire.md §2.3](architecture/00_inventaire.md#23-outillage-qualite--tests) |
| 1.10 | **Patterns d'intégration** — Type de flux (REST, event, batch, file…) | `architecture/02_patterns_integration.md` | ✅ | Matrice 18 intégrations + tableaux types/formats/protocoles |
| 1.11 | **Patterns d'intégration** — Format (JSON, CSV, XML…) | idem | ✅ | [§3](architecture/02_patterns_integration.md#3-formats--recapitulatif) |
| 1.12 | **Patterns d'intégration** — Protocole (HTTPS, AMQP, SFTP, gRPC…) | idem | ✅ | [§4](architecture/02_patterns_integration.md#4-protocoles--recapitulatif) |
| 1.13 | **Patterns d'intégration** — Fonctionnement pas à pas | idem + `flows/*` | ✅ | [§2](architecture/02_patterns_integration.md#2-detail-pas-a-pas-flux-cles) + 3 flows détaillés |

## 2. API

| # | Exigence | Page cible | Statut | Notes / preuves |
|---|---|---|---|---|
| 2.1 | **Endpoints** — Liste des routes | `api/endpoints.md` | ✅ | V8 exhaustif depuis [Api/V8/Config/routes.php](../../SuiteCRM/Api/V8/Config/routes.php) ; legacy v4_1 résumé + WARN INCONNU sur v2…v4 (`INC-05`) |
| 2.2 | **Endpoints** — Paramètres (path/query/body) | idem + `api/payloads.md` | ✅ | Classes `*Params` ancrées ([api/payloads.md §2](api/payloads.md#2-tous-les-parametres-types-slim-params)) |
| 2.3 | **Endpoints** — Headers spéciaux | `api/endpoints.md` | ✅ | [api/endpoints.md §2.6](api/endpoints.md#26-headers-speciaux) |
| 2.4 | **Endpoints** — Codes de retour réels | idem | ✅ | [api/endpoints.md §8](api/endpoints.md#8-codes-de-retour-observes-v8) |
| 2.5 | **Authentification** — Type (JWT/OAuth2/API Key/Session) | `api/authentification.md` | ✅ | OAuth2 V8 + session Sugar + SAML + LDAP |
| 2.6 | **Authentification** — Flow (diagramme + étapes) | idem + `diagrams/mermaid_sequences.md` | ✅ | [flows/10_login_api_v8.md](flows/10_login_api_v8.md) + séquence Mermaid |
| 2.7 | **Payloads** — Requêtes (DTO ancrés) | `api/payloads.md` | ✅ | Exemples + preuves vardefs |
| 2.8 | **Payloads** — Réponses (succès + erreurs) | idem | ✅ | Format JSON:API + `ErrorResponse` ([api/payloads.md §4](api/payloads.md#4-format-derreur-v8)) |

## 3. Backend

| # | Exigence | Page cible | Statut | Notes / preuves |
|---|---|---|---|---|
| 3.1 | **Controllers** — Liste exhaustive | `architecture/20_composants.md` §1 | ✅ | UI Sugar + V8 (8 contrôleurs) + legacy |
| 3.2 | **Services associés** par controller | idem §1-2 | ✅ | Tableau Controller ↔ Service V8 |
| 3.3 | **Dépendances** par service (repo / clients / libs) | idem §2-3 | ✅ | Injection détaillée + OAuth2 repos |
| 3.4 | **Logique métier** — Règles métier explicitées | idem §5 | ✅ | 5 exemples ancrés ([§5](architecture/20_composants.md#5-logique-metier-exemples-ancres)) |
| 3.5 | **Logique métier** — Exemples concrets (entrée/sortie) | idem §5 | ✅ | Payloads in/out + cas email1 + workflow garde |

## 4. Base de données

| # | Exigence | Page cible | Statut | Notes / preuves |
|---|---|---|---|---|
| 4.1 | **Modèle de données (ERD)** — Tables/collections | `data/modele_donnees.md` + `diagrams/mermaid_erd.md` | ✅ | Tables principales + ERD CRM core + Marketing + ACL + Scheduler + OAuth2 |
| 4.2 | **Clés primaires et secondaires** | `data/modele_donnees.md` | ✅ | Convention `id` PK (UUID 36), indexes vardefs/metadata détaillés |
| 4.3 | **Relations** — Types (1:1, 1:n, n:n) | `data/relations.md` | ✅ | 4 classes (`M2MRelationship`, `One2M*`, `One2One*`, `EmailAddressRelationship`) + tableaux par module |
| 4.4 | **Relations** — Contraintes (ON DELETE / ON UPDATE, FK) | idem | ⚠️ | Soft delete prouvé ; FK SQL strictes = INCONNU intentionnel (`INC-04`) |

---

## Synthèse

| Section | Total exigences | ✅ | ⚠️ | ❌ | 🚫 | ❓ |
|---|---|---|---|---|---|---|
| Architecture | 13 | 11 | 2 | 0 | 0 | 0 |
| API | 8 | 8 | 0 | 0 | 0 | 0 |
| Backend | 5 | 5 | 0 | 0 | 0 | 0 |
| Base de données | 4 | 3 | 1 | 0 | 0 | 0 |
| **Total** | **30** | **27** | **3** | **0** | **0** | **0** |

90 % des exigences ✅. 10 % en ⚠️ avec justification (zones intentionnellement INCONNUES, listées dans [adr/00_rapport_verification.md §3](adr/00_rapport_verification.md#3-inconnu-consolides-a-investiguer-cote-instance-live)).

## Procédure de mise à jour

- Le Writer met à jour le statut au fur et à mesure des pages produites.
- Le Verifier audite cette matrice et signale les ❌ qui auraient dû passer ✅, ainsi que les ✅ douteux.
- Quand un repo n'a pas d'API ou pas de DB, basculer les lignes correspondantes en 🚫 (justifier dans la colonne notes).
- Quand une zone est explicitement INCONNUE (non vérifiable depuis le repo seul), passer en ⚠️ et tagger l'identifiant INCONNU correspondant (`INC-XX`).
