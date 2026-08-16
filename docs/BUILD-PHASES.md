# Build Phases — Small Pass Sequence

## Overview

Build Pass 0 is complete (documentation baseline).

All next work follows a **small-pass-only** rule and improves the existing app incrementally.

---

## Global Guardrails (Apply to Every Pass)

1. Preserve existing working behavior in Field Ops 2.
2. Preserve `Projects-Tracker-Primary` as the core data source.
3. Preserve current SharePoint/OneDrive file locations.
4. Do not create new storage systems by default (no new lists, folders, libraries, Dataverse).
5. Do not create new screens unless explicitly approved later.
6. Do not hand-edit protected Power Apps solution files.

---

## Pass 1 — Improve Existing Selected Project/Client Form

**Goal:** Improve the selected record experience in current screen only.

Scope:
- clearer sectioning/grouping on existing selected record form
- better labels for Project/Client info, Documents/Attachments, Scheduling Readiness, Financials/Billing, Notes
- retain existing edit/save/cancel/delete behavior
- retain existing Notes and Attachments behavior

Out of scope:
- new screens
- new lists/folders/libraries
- replacing tracker or app architecture

---

## Pass 2 — Improve Document Category Display (Existing Storage Only)

**Goal:** Make document operations more practical without new storage.

Scope:
- category labels/actions for NOA, Contract, Consent To Start, Completion Signoff, Change Order, Warranty, Maintenance Guide
- rely on existing Attachments and existing SharePoint/OneDrive file links/locations
- surface existing folder link fields (if already available) as obvious open-folder actions

Out of scope:
- new document libraries
- new folder structures
- duplicated file storage

---

## Pass 3 — Add Simple Scheduling Readiness Indicators

**Goal:** Add clear gate visibility while preserving current data model.

Scope:
- clear indicator text/badges for Contract and NOA readiness
- explicit warning-only indicator for Consent To Start
- enforce/communicate rule that Consent To Start does not block scheduling

Out of scope:
- full override workflow storage additions unless explicitly approved

---

## Pass 4 — Improve Billing/Payment Tracking from Existing Financial Fields

**Goal:** Make current financial/accounting fields easier to use.

Scope:
- better grouping/labeling of existing bid, billed, paid, owed and related financial fields
- improve operator readability and consistency

Out of scope:
- replacing financial storage model
- introducing new accounting backend

---

## Pass 5 — Copilot-Assisted Reading and Suggested Updates

**Goal:** Introduce AI assistance safely on top of existing workflow.

Scope:
- Copilot reads uploaded docs and proposes updates
- suggestions remain human-reviewed and manually confirmed
- no automatic blind writes to core records

Out of scope:
- replacing manual controls
- replacing SharePoint as source of truth

---

## Validation Rule per Pass

Each pass must verify:
1. Existing record selection and save behavior still works.
2. Existing Notes and Attachments still work.
3. Existing financial fields still display and save correctly.
4. No duplicate data/storage patterns are introduced.
