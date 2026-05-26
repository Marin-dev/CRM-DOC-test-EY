# Templates FR — RetroDoc Writer

Templates Markdown à respecter pour chaque livrable. Chaque template indique les **sections obligatoires**, les **règles de preuve**, et un **squelette à copier**.

---

## 1. README (index)

```markdown
# Documentation rétro-ingénierie — <Nom du projet>

> Documentation générée par le kit RetroDoc. Toute information est traçable au code (voir section "Sources" de chaque page).

## Contexte & objectif
- Finalité du projet : <à compléter ou INCONNU>
- Périmètre : <ce qui est dedans / dehors>

## Comment exécuter (quickstart)
Voir [Runbook](runbook/README.md).

## Architecture
- [Vue d'ensemble](architecture/10_vue_ensemble.md)
- [Composants backend](architecture/20_composants.md)
- [Dépendances](architecture/01_dependances.md)
- [Patterns d'intégration](architecture/02_patterns_integration.md)
- [Inventaire factuel](architecture/00_inventaire.md)

## API
- [Vue d'ensemble](api/README.md)
- [Endpoints](api/endpoints.md)
- [Authentification](api/authentification.md)
- [Payloads](api/payloads.md)

## Data
- [Vue d'ensemble](data/README.md)
- [Modèle de données (ERD)](data/modele_donnees.md)
- [Relations](data/relations.md)

## Flows
- [Index des flows](flows/README.md)

## Diagrammes
- [Mermaid C4](diagrams/mermaid_c4.md)
- [Mermaid séquences](diagrams/mermaid_sequences.md)
- [Mermaid ERD](diagrams/mermaid_erd.md)
- [Draw.io architecture](diagrams/drawio_architecture.drawio)

## Runbook
- [Build / Run / Test / Deploy](runbook/README.md)

## ADR & rapports
- [Rapport de vérification](adr/00_rapport_verification.md)

## Glossaire
- <terme> : <définition> — preuve : <fichier:ligne>

## Comment contribuer / mettre à jour la doc
1. Relancer le kit RetroDoc via : <commande à documenter>
2. Vérifier la matrice [COVERAGE.md](COVERAGE.md)
3. Lire le rapport de vérification le plus récent
4. Soumettre une PR avec les diffs sous `docs/retrodoc/`
```

---

## 2. Architecture — Vue d'ensemble

`architecture/10_vue_ensemble.md`

```markdown
# Architecture — Vue d'ensemble

## 1. Finalité (problème métier)
<INCONNU si non documenté côté métier>

## 2. Périmètre & frontières
- Dedans : <liste>
- Dehors : <liste>

## 3. Diagramme d'architecture
Voir [diagrams/mermaid_c4.md](../diagrams/mermaid_c4.md) et [diagrams/drawio_architecture.drawio](../diagrams/drawio_architecture.drawio).

Le diagramme doit couvrir :
- **Frontend** : navigateur, mobile app — preuves : <fichiers>
- **Backend** : API, services, workers — preuves : <fichiers>
- **Base de données** : type + nom logique — preuves : <fichiers>
- **Infrastructure** : cloud, CDN, load balancer, file d'attente, cache — preuves : <fichiers IaC>
- **Flux** : flèches étiquetées (REST, event, batch...)

## 4. Sous-systèmes / services
| Sous-système | Responsabilité | Langage / framework | Entrypoint | Preuve |
|---|---|---|---|---|
| api | Expose REST | Node/Express | `apps/api/src/server.ts:12` | … |
| web | UI utilisateur | React/Vite | `apps/web/src/main.tsx:1` | … |

## 5. Dépendances & intégrations
Voir [01_dependances.md](01_dependances.md) et [02_patterns_integration.md](02_patterns_integration.md).

## 6. Data
- Stockages : <Postgres / Mongo / Redis / S3 …> — preuve
- Events / topics : <Kafka / RabbitMQ / SQS / EventBridge …> — preuve
- Contrats : <OpenAPI / Avro / Protobuf / AsyncAPI …> — preuve

## 7. Non-fonctionnel
- Performance : <observée / cible / INCONNU>
- Sécurité : <auth, secrets, scans> — preuve
- Disponibilité : <SLA, redondance> — INCONNU si non documenté

## 8. Points de vigilance
- <ex: legacy, dette, dépendance vulnérable> — preuve

## Sources
- <liste fichiers consultés>

## À investiguer
- <liste INCONNU>
```

---

## 3. Architecture — Composants (Backend)

`architecture/20_composants.md`

```markdown
# Composants Backend

## 1. Controllers / Handlers / Routes
| Controller | Fichier | Routes exposées | Services appelés | Preuve |
|---|---|---|---|---|
| `UsersController` | `apps/api/src/users/users.controller.ts` | `GET /users`, `POST /users` | `UsersService` | ligne 10 |

## 2. Services
| Service | Fichier | Responsabilité | Dépendances (repo / clients) | Preuve |
|---|---|---|---|---|
| `UsersService` | `apps/api/src/users/users.service.ts` | CRUD utilisateur | `UsersRepository`, `MailClient` | ligne 5 |

## 3. Repositories / DAO
| Repository | Fichier | Entité | DB cible | Preuve |
|---|---|---|---|---|
| `UsersRepository` | `…` | User | Postgres | … |

## 4. Workers / Jobs / Consumers
| Composant | Fichier | Déclencheur | Action | Preuve |
|---|---|---|---|---|
| `ExportJob` | `…` | cron 0 2 * * * | Génère export CSV | … |

## 5. Logique métier — fiches "règle"
### Règle : <Nom de la règle>
- **Localisation** : `fichier.ts:ligne` — symbole `xxx()`
- **Énoncé** : <description en français>
- **Inputs** : <données nécessaires>
- **Conditions** : <if/else clés>
- **Sorties** : <effets observables>
- **Exemple concret** :
  - Entrée : `{...}`
  - Sortie attendue : `{...}`
- **Preuve** : extrait court du code
- **À investiguer** : INCONNU si manque de contexte métier

(Répéter pour 3 à 5 règles métier critiques)

## Sources
- <fichiers>

## À investiguer
- <INCONNU>
```

---

## 4. API — README

`api/README.md`

```markdown
# API — Vue d'ensemble

## Style
- REST / GraphQL / gRPC / autre — preuve : <fichier>
- Versioning : <préfixe / header / aucun> — preuve
- Base URL en local : `http://localhost:<port>` — preuve : config
- Base URL prod : INCONNU si non documenté

## Format
- Content-Type : `application/json` (sauf exception)
- Encodage erreurs : <RFC 7807 / format custom> — preuve

## Détails
- [Liste des endpoints](endpoints.md)
- [Authentification](authentification.md)
- [Payloads](payloads.md)

## Spécification OpenAPI / GraphQL schema
- Fichier : <chemin ou INCONNU>
- Génération : <commande si applicable>

## Sources
- <fichiers>
```

---

## 5. API — Endpoints (tableau exhaustif)

`api/endpoints.md`

```markdown
# Endpoints

## Convention de lecture
- **Auth** : Public / Bearer JWT / API Key / Session — preuve par endpoint
- **Codes retour** : codes effectivement renvoyés par le code (pas les codes "standards" théoriques)

## Liste

| Méthode | Route | Controller / Handler | Auth | Params (path/query) | Headers spéciaux | Body (résumé) | Codes retour | Preuve |
|---|---|---|---|---|---|---|---|---|
| GET | `/v1/users` | `UsersController.list` | Bearer JWT | `?page`, `?limit` | — | — | 200, 401 | `users.controller.ts:14` |
| POST | `/v1/users` | `UsersController.create` | Bearer JWT | — | — | `CreateUserDto` | 201, 400, 401, 409 | `users.controller.ts:30` |
| GET | `/v1/users/:id` | `UsersController.findOne` | Bearer JWT | `id` (path) | — | — | 200, 401, 404 | `users.controller.ts:45` |

## Endpoints non documentés / INCONNU
- <route> — raison : <code pas localisable / dynamique>

## Sources
- <fichiers parsés : routeurs, décorateurs, configs>
```

---

## 6. API — Authentification

`api/authentification.md`

```markdown
# Authentification

## Type
- Mécanisme : <Bearer JWT / OAuth2 / API Key / Session cookie / autre> — preuve : <fichier:ligne>
- Émetteur : <auth provider / interne> — preuve
- Durée de vie token : <valeur ou INCONNU>

## Flow

\`\`\`mermaid
sequenceDiagram
  participant U as Utilisateur
  participant W as Web
  participant A as API
  participant IDP as IdP

  U->>W: Saisit credentials
  W->>IDP: POST /token
  IDP-->>W: access_token + refresh_token
  W->>A: GET /resource (Authorization: Bearer ...)
  A->>A: Vérifie signature + claims
  A-->>W: 200 OK
\`\`\`

(Adapter le diagramme aux preuves trouvées. Si certaines étapes sont INCONNU, les marquer.)

## Validation côté serveur
- Bibliothèque : <jose / passport / Spring Security / ASP.NET Identity / …> — preuve
- Middleware d'auth : `fichier:ligne` — symbole `…`
- Claims vérifiés : <iss, aud, exp, scope, …> — preuve

## Refresh / révocation
- Mécanisme : <description ou INCONNU>

## Erreurs courantes
| Code | Cas | Source |
|---|---|---|
| 401 | Token absent ou expiré | … |
| 403 | Scope insuffisant | … |

## Sources
- <fichiers>

## À investiguer
- <INCONNU>
```

---

## 7. API — Payloads

`api/payloads.md`

```markdown
# Payloads (Requêtes / Réponses)

Pour chaque endpoint critique, donner un exemple de **requête** et de **réponse**, ancrés sur les DTO / schémas réels.

## POST /v1/users

### Requête
- DTO source : `CreateUserDto` — `apps/api/src/users/dto/create-user.dto.ts:1`

\`\`\`json
{
  "email": "alice@example.com",
  "name": "Alice"
}
\`\`\`

### Réponse — 201
- DTO source : `UserDto` — preuve

\`\`\`json
{
  "id": "uuid",
  "email": "alice@example.com",
  "name": "Alice",
  "createdAt": "2026-05-18T10:00:00Z"
}
\`\`\`

### Réponses d'erreur
| Code | Body (résumé) | Source |
|---|---|---|
| 400 | `{ "message": "validation_failed", "errors": [...] }` | validator middleware |
| 409 | `{ "message": "email_already_exists" }` | service:42 |

## (Répéter par endpoint critique)

## Sources
- <DTO files, schémas>
```

---

## 8. Data — README

`data/README.md`

```markdown
# Data — Vue d'ensemble

## Stockages détectés
| Type | Nom logique | Sous-projet | Driver / ORM | Preuve |
|---|---|---|---|---|
| Postgres | `app_main` | api | Prisma | `prisma/schema.prisma:1` |
| Redis | `cache` | api | ioredis | `src/cache.ts:3` |
| S3 | `uploads` | api | AWS SDK | `src/storage.ts:10` |

## Stratégie de migration
- Outil : <Prisma migrate / Flyway / Liquibase / Alembic / EF Migrations / …> — preuve
- Localisation des migrations : <dossier>
- Politique de rollback : INCONNU si non documenté

## Documents associés
- [Modèle de données (ERD)](modele_donnees.md)
- [Relations](relations.md)

## Sources
- <fichiers>
```

---

## 9. Data — Modèle de données (ERD)

`data/modele_donnees.md`

```markdown
# Modèle de données

## ERD (Mermaid)
Voir [diagrams/mermaid_erd.md](../diagrams/mermaid_erd.md).

Aperçu :

\`\`\`mermaid
erDiagram
  USER ||--o{ ORDER : "passe"
  ORDER ||--|{ ORDER_ITEM : "contient"
  ORDER_ITEM }o--|| PRODUCT : "réfère"
\`\`\`

## Tables / Collections

### `users`
- Source : `prisma/schema.prisma:10` (ou migration `…sql`)

| Colonne | Type | Nullable | Défaut | Contraintes / Clés | Preuve |
|---|---|---|---|---|---|
| `id` | uuid | non | `gen_random_uuid()` | PK | schema.prisma:11 |
| `email` | text | non | — | UNIQUE | schema.prisma:12 |
| `name` | text | oui | — | — | … |
| `created_at` | timestamptz | non | `now()` | — | … |

### `orders`
(Même format)

### (toutes les tables détectées)

## Index notables
| Table | Colonnes | Type | Preuve |
|---|---|---|---|
| `users` | `(email)` | UNIQUE | … |
| `orders` | `(user_id, created_at)` | INDEX | … |

## INCONNU
- <colonnes / tables sans contexte métier>

## Sources
- <fichiers schémas + migrations>
```

---

## 10. Data — Relations

`data/relations.md`

```markdown
# Relations entre entités

## Tableau récapitulatif
| Table source | Colonne FK | Table cible | Type de relation | ON DELETE | Preuve |
|---|---|---|---|---|---|
| `orders` | `user_id` | `users.id` | n:1 | CASCADE | migration_20251010 |
| `order_items` | `order_id` | `orders.id` | n:1 | CASCADE | … |
| `order_items` | `product_id` | `products.id` | n:1 | RESTRICT | … |

## Cardinalités narratives
- Un `user` possède **0..n** `order` (1:n)
- Un `order` contient **1..n** `order_item` (1:n)
- Un `order_item` réfère **exactement 1** `product` (n:1)
- (Relations n:n explicitées via table de jonction si présente)

## Diagramme
Voir [diagrams/mermaid_erd.md](../diagrams/mermaid_erd.md).

## Sources
- <fichiers>
```

---

## 11. Flow (fiche par flow critique)

`flows/<NN>_<nom_du_flow>.md`

```markdown
# Flow : <Nom>

## Déclencheur
- Type : HTTP / event / cron / CLI / webhook
- Détail : `POST /v1/orders` — `OrdersController.create` — `apps/api/src/orders/orders.controller.ts:22`

## Préconditions
- <ex: utilisateur authentifié, panier non vide>
- Preuve : <fichier:ligne>

## Étapes
1. **Étape 1** — `fichier:ligne` — symbole — description courte
2. **Étape 2** — …
3. …

## Données
- Lectures : tables / caches / services tiers
- Écritures : tables / events / files

## Erreurs & retries
| Erreur | Code / Type | Comportement | Preuve |
|---|---|---|---|
| Stock insuffisant | 409 | Pas de retry | service:55 |
| Timeout paiement | 504 | Retry x3 expo | client:30 |

## Observabilité
- Logs : <pattern / niveau> — preuve
- Metrics : <métrique exposée> — preuve
- Traces : <span name> — preuve

## Diagramme de séquence
Voir [diagrams/mermaid_sequences.md](../diagrams/mermaid_sequences.md) (#flow-<slug>).

## Sources
- <fichiers>

## À investiguer
- <INCONNU>
```

---

## 12. Runbook

`runbook/README.md`

```markdown
# Runbook

## Build
| Sous-projet | Commande | Pré-requis | Source |
|---|---|---|---|
| api | `npm run build` | Node 20 | `package.json` |

## Tests
| Sous-projet | Commande | Type | Source |
|---|---|---|---|
| api | `npm test` | unit | `package.json` |
| api | `npm run test:e2e` | e2e | `package.json` |

## Lint / Format
| Outil | Commande | Source |
|---|---|---|
| ESLint | `npm run lint` | `.eslintrc` |
| Prettier | `npm run format` | `.prettierrc` |

## Lancement local
1. Cloner le repo
2. Installer : <commandes>
3. Configurer les variables d'env (cf. tableau)
4. Démarrer la DB / services tiers : <commande / docker-compose>
5. Lancer : <commande>

## Variables d'environnement (prouvées)
| Nom | Description (si commentée) | Obligatoire ? | Source |
|---|---|---|---|
| `DATABASE_URL` | URL Postgres | oui | `.env.example:3` |
| `JWT_SECRET` | Secret signature JWT | oui | `.env.example:5` |

## Déploiement (si détecté)
- Cible : <cloud / on-prem / INCONNU>
- Pipeline : <fichier CI/CD>
- Étapes : build → test → deploy → smoke test
- Preuve : <fichier workflow>

## Troubleshooting
| Symptôme | Cause probable | Action | Source |
|---|---|---|---|
| 401 sur tous les appels | JWT_SECRET non chargé | Vérifier `.env` | … |

## Sources
- <fichiers>
```

---

## 13. ADR — README

`adr/README.md`

```markdown
# ADR — Architecture Decision Records

Décisions techniques détectées dans le code/historique ou identifiées comme implicites.

## Index
| # | Titre | Statut | Date | Lien |
|---|---|---|---|---|
| 001 | Choix Postgres pour stockage principal | Accepté (implicite) | INCONNU | [001](001_postgres.md) |

## Format d'une ADR
- Contexte
- Décision
- Conséquences (positives / négatives)
- Alternatives envisagées
- Statut

## Rapports
- [Rapport de vérification](00_rapport_verification.md)
```
