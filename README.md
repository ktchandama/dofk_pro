# 🏛️ DOFK_Pro – SIGP

## Système Intégré de Gestion de la Préfiscalisation
### Multi-platform taxpayer tracking system for informal sector formalization in Togo

🌐 **Available languages:** [English](README.md) · [Français](readme_fr.md) · [Deutsch](readme_de.md)

---

## 📌 Overview

**DOFK_Pro SIGP (Système Intégré de Gestion de la Préfiscalisation)** is a multi-platform system built to support field tax officers in identifying, monitoring, and formalizing economic operators active in the informal sector.

It gives officers a single, coherent way to manage the taxpayer lifecycle across native mobile, cross-platform, and web clients:

**Identification → Pre-tax Registration → Field Monitoring → Payment Tracking → Tax Registration (TIN)**

The goal is to improve data quality and traceability in the field, strengthen coordination between agents, and support the progressive integration of informal activity into the formal tax base — while keeping the personal and financial data collected proportionate, secured, and access-controlled.

---

## 🎯 Context and Problem Statement

A large share of economic activity in Togo takes place outside the formal tax registry. Operators without a Tax Identification Number (TIN/NIF) are traditionally tracked through paper forms and disconnected spreadsheets, which leads to:

- Loss or duplication of taxpayer records
- No shared view of who has already been visited or invoiced
- Limited visibility on outstanding payments
- No reliable history of interactions with a given taxpayer
- Difficult coordination between field agents and supervisors

SIGP replaces this with a shared digital record per taxpayer, usable offline in the field and synchronized centrally.

---

## 💡 Core Concept

Every taxpayer identified in the field — registered or not — is tracked through a unique identifier.

Taxpayers without a TIN receive a **temporary local identifier** (e.g. `K02-200001`) used until an official TIN is issued; taxpayers who already hold a TIN are tracked and verified under that number.

This identifier guarantees:

- A single record per taxpayer, avoiding duplicates
- Continuity of history from first contact to formal registration
- Consistent tracking across agents and visits

---

## 📱 Platforms and Applications

DOFK_Pro is delivered as a small family of clients sharing the same data model, rather than a single monolithic app:

| Client | Stack | Role |
|---|---|---|
| **Android app** | Kotlin, Jetpack Compose | Primary field tool for agents; offline-capable data entry and sync |
| **iOS app** | Swift, SwiftUI | Feature-equivalent companion app for agents on iPhone |
| **Cross-platform app** | TypeScript, React Native / Expo | Shared mobile/web client for lighter deployments |
| **Legacy connector** | Google Apps Script | Optional lightweight backend for pilot/low-connectivity contexts |

All clients read and write the same taxpayer records through a central backend, so a taxpayer created on one device is immediately visible to every other agent and supervisor with access.

---

## 🚀 Main Features

### 👤 Taxpayer Management
- Registration of non-registered taxpayers under a temporary identifier
- Verification of already-registered taxpayers (TIN validity, declared activity, inconsistencies)
- Capture of activity, sector, contact details, and field observations

### 📍 Field Geolocation
- Optional GPS capture at the point of identification, for mapping and visit planning
- Location data is collected only when relevant to the field mission and is treated as personal data (see *Data Protection* below)

### 💰 Assessments and Payments
- Recording of tax assessments (type, amount, date, deadline)
- Payment tracking with running balance and full history
- Payment schedules (*échéanciers*) for staged settlements

### 📅 Follow-up and Field Missions
- Logging of visits, invitations, reminders (including bulk reminder campaigns), and administrative notes
- SMS-based reminders: sent directly from the device on Android, and through a secured messaging gateway on iOS (Apple does not allow direct SIM-based sending)
- Field mission tracking for agents operating on assigned circuits

### 🛠️ Administration
- Agent/user management with roles and profiles
- Import/export of data (CSV) for bulk operations and reporting
- Per-agent activity statistics

---

## 🔄 Taxpayer Lifecycle

```mermaid
flowchart LR

A[Field Identification] --> B[Temporary ID Generation]
B --> C[Optional GPS Capture]
C --> D[Tax Assessment]
D --> E[Payment Monitoring]
E --> F[Follow-up Actions]
F --> G[TIN Registration]
G --> H[Formal Taxpayer Record]
```

---

## 🏗️ System Architecture

```
        Android App        iOS App        Web / Cross-platform App
        (Kotlin/Compose)   (Swift/SwiftUI)   (React Native / Expo)
              \                 |                    /
               \                |                   /
                \-------------- Backend API --------/
                                 |
                      Managed PostgreSQL Database
                                 |
                    Authentication & Access Control
                                 |
                    (Optional) Legacy Sheet Connector
                        for pilot deployments
```

Messaging (SMS reminders) is routed through a dedicated gateway rather than embedding provider credentials in the client apps, so agent devices never hold long-lived secrets.

---

## 🗄️ Data Model (simplified)

```
Taxpayer (Contribuable)
│
├── local_id            # temporary identifier before TIN issuance
├── tin                 # official Tax ID, once available
├── name, phone
├── activity, sector
├── gps_coordinates     # optional, field-captured
├── status
│
├── Assessment (Acte fiscal)
│      ├── amount, tax_type
│      ├── date
│      └── status
│
├── Payment (Transaction)
│      ├── amount, date
│      └── reference
│
├── Follow-up (Relance / Rendez-vous)
│      ├── date, type
│      └── status
│
└── Field Mission
       ├── agent
       ├── circuit
       └── visit log

User (Utilisateur / Agent)
│
├── login, role/profile
└── assigned section
```

> Field/table names above are illustrative. Actual schema definitions, credentials, and environment-specific identifiers (database URLs, API keys, spreadsheet references) are kept out of this document and out of version control — see *Data Protection* below.

---

## 📊 Indicators and Analytics

**Taxpayer indicators:** number identified, number formally registered, geographic and sector distribution.

**Financial indicators:** total assessed, total collected, collection rate, outstanding balances.

**Operational indicators:** field visits, follow-up actions, conversion rate from temporary registration to TIN, per-agent activity.

---

## 🔒 Data Protection and Privacy

SIGP handles personal data (names, phone numbers, GPS locations) and financial data belonging to taxpayers, so the project follows privacy-by-design principles inspired by GDPR-style good practice, regardless of jurisdiction:

- **Data minimization** — only fields required for the tax-formalization purpose are collected (e.g. GPS is optional and mission-specific, not tracked continuously).
- **Purpose limitation** — taxpayer data is used only for identification, assessment, and follow-up, not repurposed silently.
- **Access control** — access is role-based (agent / supervisor / admin); no anonymous access to taxpayer records.
- **Secrets management** — database URLs, API keys, and messaging-gateway credentials are configured through local/environment configuration files excluded from version control, never hard-coded or committed.
- **No real data in documentation or examples** — this README and related docs use fictitious identifiers, amounts, and coordinates only.
- **Data retention** — historical records are kept only as long as needed for the formalization and audit purpose; retention rules should be defined at deployment level in line with applicable local regulation.

Anyone deploying SIGP is responsible for configuring their own credentials and access policies and for ensuring the deployment complies with the data-protection rules applicable in their jurisdiction.

---

## 🛠️ Technology Stack

**Mobile — Android:** Kotlin, Jetpack Compose

**Mobile — iOS:** Swift, SwiftUI, XcodeGen

**Cross-platform mobile / web:** TypeScript, React Native, Expo Router

**Backend & Database:** Managed PostgreSQL backend with REST access and row-level access policies

**Messaging:** Secured SMS gateway (serverless function) for platforms without direct SIM access

**Legacy / pilot connector:** Lightweight scripted backend for early or low-connectivity pilots

**Build tooling:** Gradle (Android), XcodeGen (iOS), pnpm/Expo CLI (cross-platform app)

---

## 🔮 Future Improvements

- Automated, configurable reminder schedules
- GIS dashboards for field-visit planning
- Risk-based prioritization of follow-ups
- Interoperability with national tax information systems
- Consolidated cross-platform analytics dashboard

---

## 🌍 Impact

✅ Better taxpayer identification and traceability
✅ Cleaner, de-duplicated taxpayer data
✅ Stronger, better-coordinated field operations
✅ Progressive formalization of the informal sector
✅ Modernized, privacy-conscious tax administration tooling

---

## 👨‍💻 Author

Digital transformation project aimed at improving taxpayer monitoring through mobile-first field tools, structured data, and responsible handling of personal information.
