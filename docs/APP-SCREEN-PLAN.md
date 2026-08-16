# App Screen Plan — Existing Screen First

## Direction

This plan does not replace Field Ops 2 with a new multi-screen app in this pass.

Primary focus is improving the current selected record experience on the existing working screen (`MainScreen1`) and existing form (`Form1`).

---

## Current Working Behavior to Preserve

- `RecordsGallery1` lists records from `Projects-Tracker-Primary`.
- Selecting a row opens the selected project/client record in `Form1`.
- Existing edit/save/cancel/delete behavior remains.
- Existing Notes field remains.
- Existing Attachments behavior remains.
- Existing financial/accounting fields remain.

---

## Pass 1 UI Improvements (Same Screen)

### 1) Sectioned Form Layout

Reorder/label existing fields in `Form1` into clear groups:
- Project / Client Information
- Documents / Attachments
- Scheduling Readiness
- Financials / Billing
- Notes

No new storage required.

### 2) Document Tools (Practical, Small)

Add visible category labels/actions on the selected record for:
- NOA
- Contract
- Consent To Start
- Completion Signoff
- Change Order
- Warranty
- Maintenance Guide

Use existing attachments and existing link/file locations only.

### 3) Folder Link Actions (If Existing Fields Already Exist)

If existing fields already hold folder links, surface obvious actions:
- Open Client Folder
- Open Project Folder

If fields are not available, do not create new fields in this pass.

### 4) Notes by Category (No New Storage)

Keep the existing Notes field and use a lightweight category format, for example:
- `[NOA] ...`
- `[CONTRACT] ...`
- `[CONSENT] ...`
- `[COMPLETION] ...`
- `[CHANGE ORDER] ...`
- `[WARRANTY] ...`
- `[MAINTENANCE] ...`
- `[BILLING] ...`

---

## Scheduling Rule Display Requirement

In UI messaging and labels:
- Contract + NOA may block scheduling when required.
- Consent To Start is warning-only and does not block scheduling.

---

## Out of Scope for This Plan

- New full app structure
- New screen set by default
- New SharePoint lists/libraries/folders
- Dataverse table creation
- Replacing `Projects-Tracker-Primary`
