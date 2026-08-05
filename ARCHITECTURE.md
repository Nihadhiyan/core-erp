# Clausis Apox — Edge POS Client

This repository contains the **Flutter Edge/POS client** for Clausis Apox, one of the four
pillars of the Clausis Global ecosystem (Apox / Eigen / Micron / Larmora TMS). It does **not**
contain the Clausis Apox backend — the Spring Boot microservices that own the Catalog & Pricing,
Inventory & WMS, Sales & CRM, Finance & Accounting, Identity & Access, and Event Streaming Core
bounded contexts live in a separate repository.

This client talks to whichever server is in front of it — a local `Clausis Apox Edge` instance in
`offlineOnly`/`hybrid` mode, or the cloud endpoint directly in `cloudOnly` mode — over REST. It
never needs to know how cross-branch synchronization, conflict resolution, or ledger posting work;
that complexity is entirely server-side.

## Folder Structure

```
lib/
├── core/                   # App-wide configuration and services
│   ├── api/                # Dio client, JWT interceptors, base URLs
│   ├── hardware/           # Barcode scanner listener, receipt printer services
│   ├── local_db/           # Isar database initialization and schema definitions
│   ├── theme/              # Clausis design system (teal/white/charcoal, typography)
│   └── utils/              # Currency and date formatters
│
├── features/               # Isolated business modules, one per client-facing capability
│   ├── auth/                   # Cashier login, session validation, manager PIN overrides
│   │
│   ├── pos/                    # Scanning, cart, checkout — the core POS flow
│   │   ├── data/                   # Isar repositories (local product/batch lookup + sync outbox)
│   │   ├── domain/                 # Freezed models (Order, CartItem, Product, BatchInfo)
│   │   └── presentation/           # Riverpod providers, MRP-collision modal, screens
│   │
│   ├── inventory/               # GRN intake, batch management, stock counts (warehouse-facing)
│   └── sessions/                # Till session (Z-report) opening/closing
│
├── shared/                 # Reusable UI components across features
│   └── widgets/                 # Numpads, buttons, loading overlays
│
└── main.dart                # Application entry point and hardware listener injection
```

## Architectural Rules

1. **Feature isolation.** A feature (e.g. `pos`) never imports another feature's `data` or
   `presentation` layer directly. Cross-feature sharing happens only through `domain` models or a
   shared Riverpod provider — this keeps each feature independently testable and prevents the kind
   of tangled dependency graph that makes a Flutter app slow to build.
2. **Offline-first.** All reads on the POS terminal are served from `core/local_db` (Isar) first.
   The network (`Dio`, via `core/api`) is used for background synchronization to whichever backend
   is currently configured, never for anything on the interactive checkout path.
3. **Immutable state.** Every model in a feature's `domain/` layer is a `Freezed` class. A price or
   quantity value must never be silently mutated in memory by an unrelated code path — this is a
   financial application, not a form.
4. **No business logic in `presentation/`.** Tax math, discount evaluation, and promotion stacking
   live in `domain`/`data`, never inside a widget build method — this mirrors the same
   separation-of-concerns principle the backend enforces between controllers and services.

## Why this repo is scoped to the client only

Clausis Apox's six bounded contexts are being built as independent backend microservices, each
owning its own database. Bundling that backend into this repository would blur the deployment
boundary between "code that ships to a tablet" and "code that runs in Kubernetes" — two artifacts
with entirely different build tooling, release cadences, and CI pipelines. Keeping them in separate
repositories lets each evolve, version, and deploy on its own schedule.
