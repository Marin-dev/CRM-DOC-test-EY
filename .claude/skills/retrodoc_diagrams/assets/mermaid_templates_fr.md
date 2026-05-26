# Mermaid Templates FR

## 1. C4 — Context

```mermaid
C4Context
  title Contexte système — <Nom>
  Person(user, "Utilisateur", "Description")
  System(app, "<Nom application>", "Description")
  System_Ext(stripe, "Stripe", "Paiement")
  System_Ext(idp, "IdP", "Authentification")

  Rel(user, app, "Utilise", "HTTPS")
  Rel(app, stripe, "Paiements", "HTTPS / Webhook")
  Rel(app, idp, "OIDC", "HTTPS")
```

## 2. C4 — Container

```mermaid
C4Container
  title Containers — <Nom>
  Person(user, "Utilisateur")
  System_Boundary(app, "Application") {
    Container(web, "Web", "React / Vite", "UI utilisateur")
    Container(api, "API", "Node / Express", "REST /v1")
    Container(worker, "Worker", "Node", "Jobs async")
    ContainerDb(db, "Postgres", "RDS", "Données métier")
    ContainerDb(redis, "Redis", "ElastiCache", "Cache / queues")
  }
  System_Ext(stripe, "Stripe")

  Rel(user, web, "HTTPS")
  Rel(web, api, "REST /v1", "JSON / HTTPS")
  Rel(api, db, "SQL", "TCP")
  Rel(api, redis, "Commands", "TCP")
  Rel(api, worker, "Event", "Redis Streams")
  Rel(api, stripe, "Paiements", "HTTPS")
```

## 3. C4 — Component (zoom sur l'API)

```mermaid
C4Component
  title Composants API
  Container_Boundary(api, "API") {
    Component(router, "Router", "Express", "Routage HTTP")
    Component(authMw, "Auth Middleware", "JWT", "Vérifie tokens")
    Component(usersCtl, "UsersController", "REST", "/users/*")
    Component(usersSvc, "UsersService", "Business logic")
    Component(usersRepo, "UsersRepository", "Prisma")
  }
  ContainerDb(db, "Postgres")

  Rel(router, authMw, "applique")
  Rel(authMw, usersCtl, "transmet requête")
  Rel(usersCtl, usersSvc, "appelle")
  Rel(usersSvc, usersRepo, "appelle")
  Rel(usersRepo, db, "SQL")
```

## 4. Séquence — Flow d'authentification

```mermaid
sequenceDiagram
  autonumber
  participant U as Utilisateur
  participant W as Web
  participant A as API
  participant IDP as IdP

  U->>W: Saisit email + mot de passe
  W->>IDP: POST /token
  IDP-->>W: access_token + refresh_token
  W->>A: GET /v1/me (Authorization: Bearer ...)
  A->>A: Vérifie signature + claims
  A-->>W: 200 OK + profil
  W-->>U: Affiche dashboard
```

## 5. Séquence — Flow de création de commande

```mermaid
sequenceDiagram
  autonumber
  participant W as Web
  participant A as API
  participant DB as Postgres
  participant Q as Queue
  participant WK as Worker
  participant S as Stripe

  W->>A: POST /v1/orders
  A->>DB: INSERT orders, order_items
  A->>S: Create PaymentIntent
  S-->>A: client_secret
  A->>Q: Enqueue "order.created"
  A-->>W: 201 + payment_intent
  Q-->>WK: Consume "order.created"
  WK->>WK: Envoi email confirmation
```

## 6. ERD

```mermaid
erDiagram
  USER ||--o{ ORDER : "passe"
  ORDER ||--|{ ORDER_ITEM : "contient"
  ORDER_ITEM }o--|| PRODUCT : "réfère"

  USER {
    uuid id PK
    text email UK
    text name
    timestamptz created_at
  }
  ORDER {
    uuid id PK
    uuid user_id FK
    text status
    timestamptz created_at
  }
  ORDER_ITEM {
    uuid id PK
    uuid order_id FK
    uuid product_id FK
    int quantity
    numeric unit_price
  }
  PRODUCT {
    uuid id PK
    text sku UK
    text name
    numeric price
  }
```

## 7. Flowchart — Algorithme métier

```mermaid
flowchart TD
  start([Début]) --> check{Stock suffisant ?}
  check -->|Non| err[409 Stock insuffisant]
  check -->|Oui| reserve[Réserve stock]
  reserve --> pay{Paiement OK ?}
  pay -->|Non| rollback[Libère stock]
  pay -->|Oui| confirm[Confirme commande]
  rollback --> end1([Fin échec])
  err --> end1
  confirm --> end2([Fin succès])
```
