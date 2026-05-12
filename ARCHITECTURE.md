# Nihadh POS - System Architecture

This document outlines the Feature-First Clean Architecture used in the Nihadh POS system. This structure is designed for an offline-first, high-concurrency retail environment, specifically handling Sri Lankan market requirements like MRP collisions.

## 📂 Folder Structure

lib/
├── core/                   # ⚙️ App-wide configurations and services
│   ├── api/                # Dio client, JWT Interceptors, Base URLs
│   ├── hardware/           # Scanner listener (Keyboard Wedge) & Printer services
│   ├── local_db/           # Isar database initialization and schema definitions
│   ├── theme/              # High-contrast colors & typography for touch terminals
│   └── utils/              # Currency (LKR) & Date formatters
│
├── features/               # 🚀 Distinct, isolated business modules
│   ├── auth/               # Cashier login, Session validation & Manager PIN overrides
│   │
│   ├── pos/                # 🛒 THE CORE: Scanning, Cart logic, and Checkout
│   │   ├── data/           # Isar Repositories (High-speed Product & Batch lookup)
│   │   ├── domain/         # Freezed Models (Order, CartItem, Product, BatchInfo)
│   │   └── presentation/   # Riverpod Providers, MRP Collision Modals, UI Layouts
│   │
│   ├── inventory/          # 📦 Back-office: GRN, Batch management, and Stock counts
│   └── sessions/           # 📊 TillSession (Z-Report) opening/closing management
│
├── shared/                 # 🧩 Reusable UI components across multiple features
│   └── widgets/            # Custom Numpads, global buttons, loading overlays
│
└── main.dart               # 🏁 Application entry point and hardware listener injection

## 🏗️ Architectural Rules
1. **Feature Isolation:** A feature (e.g., `pos`) cannot directly import from another feature's `presentation` or `data` layer. If they must share data, it happens via the `domain` models or a shared Riverpod provider.
2. **Offline-First:** All read operations on the POS terminal prioritize the `core/local_db` (Isar). Network calls (Dio) are primarily used for background syncing.
3. **Immutable State:** All state models inside `domain/` must use `freezed` to ensure financial calculations are never accidentally mutated.