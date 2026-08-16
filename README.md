# Brickyard SharePoint Operations

## Overview

**Brickyard SharePoint Operations** is a Power Apps canvas app for office/admin operations.

This repository now follows an **incremental improvement** approach:
- Improve the existing **Field Ops 2** app
- Preserve existing **Projects-Tracker-Primary** usage
- Preserve existing **SharePoint/OneDrive** file locations
- Avoid rebuilds, clones, or replacement data systems unless explicitly approved

> **No Dataverse replacement database.** SharePoint remains the source of truth.

---

## Current Direction

The app is **not** being rebuilt into a new 10-screen replacement in this pass.

Current focus is small, safe passes:
1. Improve the existing selected project/client edit experience
2. Improve document category visibility using current attachments/file links
3. Add simple scheduling readiness indicators
4. Improve billing/payment visibility from existing financial fields
5. Add Copilot-assisted suggestions with human review

---

## Repository Structure

```
BrickyardSharePointOperations/
├── README.md
├── docs/
│   ├── BRICKYARD-OPERATIONS-HUB-PLAN.md
│   ├── BUILD-PHASES.md
│   ├── SHAREPOINT-DATA-MODEL.md
│   ├── POWER-AUTOMATE-FLOWS.md
│   ├── APP-SCREEN-PLAN.md
│   └── DO-NOT-EDIT-FILES.md
└── FieldOps2/
    └── BrickyardSharePointOperations/
        └── canvasapps/new_fieldops2_6938b/
```

---

## Rules for This Implementation Track

1. Keep **Field Ops 2** working while improving it.
2. Keep **Projects-Tracker-Primary** as the operational center.
3. Keep existing Notes, Attachments, financial fields, and edit/save behavior.
4. Do not create duplicate client/project/document storage.
5. Do not create new lists/folders/flows/screens unless later approved.
6. Power Apps is the interface; SharePoint/OneDrive is the file and data source of truth.

---

## Documentation Index

| Document | Purpose |
|---|---|
| [BRICKYARD-OPERATIONS-HUB-PLAN.md](docs/BRICKYARD-OPERATIONS-HUB-PLAN.md) | Master strategy (improve existing app, no rebuild) |
| [BUILD-PHASES.md](docs/BUILD-PHASES.md) | Small-pass sequence |
| [APP-SCREEN-PLAN.md](docs/APP-SCREEN-PLAN.md) | Existing-screen-first UI improvements |
| [SHAREPOINT-DATA-MODEL.md](docs/SHAREPOINT-DATA-MODEL.md) | Current tracker-centric data model |
| [POWER-AUTOMATE-FLOWS.md](docs/POWER-AUTOMATE-FLOWS.md) | Flow usage approach for incremental rollout |
| [DO-NOT-EDIT-FILES.md](docs/DO-NOT-EDIT-FILES.md) | Protected files and editing rules |
