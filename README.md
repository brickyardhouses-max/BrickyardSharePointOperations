# Brickyard SharePoint Operations

## Overview

**Brickyard SharePoint Operations** is a Power Apps canvas application that serves as the operational interface for Brickyard Houses Home Solutions LLC. The app is the _interface only_. SharePoint Lists, SharePoint Documents, and OneDrive are the authoritative source of truth for all data and files.

> **No Dataverse.** All project, client, scheduling, billing, and document data lives in SharePoint.

---

## Repository Structure

```
BrickyardSharePointOperations/
├── README.md                          ← This file
├── docs/                              ← Planning and reference documentation
│   ├── BRICKYARD-OPERATIONS-HUB-PLAN.md
│   ├── BUILD-PHASES.md
│   ├── SHAREPOINT-DATA-MODEL.md
│   ├── POWER-AUTOMATE-FLOWS.md
│   ├── APP-SCREEN-PLAN.md
│   └── DO-NOT-EDIT-FILES.md
└── FieldOps2/
    └── BrickyardSharePointOperations/ ← Power Platform solution source
        ├── canvasapps/
        │   └── new_fieldops2_6938b/   ← Canvas app: "Field Ops 2"
        ├── publishers/
        └── solutions/
```

---

## Documentation Index

| Document | Purpose |
|---|---|
| [BRICKYARD-OPERATIONS-HUB-PLAN.md](docs/BRICKYARD-OPERATIONS-HUB-PLAN.md) | Master plan, business rules, and architectural decisions |
| [BUILD-PHASES.md](docs/BUILD-PHASES.md) | Ordered build sequence from Phase 0 through final publish |
| [SHAREPOINT-DATA-MODEL.md](docs/SHAREPOINT-DATA-MODEL.md) | All SharePoint lists, columns, and document library structure |
| [POWER-AUTOMATE-FLOWS.md](docs/POWER-AUTOMATE-FLOWS.md) | All Power Automate flows: triggers, actions, and document generation |
| [APP-SCREEN-PLAN.md](docs/APP-SCREEN-PLAN.md) | All Power Apps screens, components, and navigation |
| [DO-NOT-EDIT-FILES.md](docs/DO-NOT-EDIT-FILES.md) | Files that must not be hand-edited and why |

---

## Current State (as of Build Pass 0)

- **App name:** Field Ops 2 (`new_fieldops2_6938b`)
- **SharePoint site:** `https://bhhomesolutions.sharepoint.com/sites/BrickyardHousesHomeSolutionsLLC`
- **Active SharePoint list:** `Projects-Tracker-Primary`
- **Architecture:** Single-screen record viewer and editor
- **Build pass:** Pass 0 (documentation phase — no app source files modified)

---

## Business Rules

1. SharePoint Lists and SharePoint Documents are the **source of truth**.
2. Power Apps is the **interface only**.
3. **Do not use Dataverse** as the primary database.
4. **Do not create duplicate client records.**
5. **Do not create duplicate project records.**
6. **Do not create duplicate document storage.** Documents live once in their SharePoint project folder.
7. Client-facing documents (warranty, maintenance) must **not expose** cost, revenue, profit, margin, labor, or material data.
8. Copilot reads uploaded documents, extracts data, updates trackers, and assists with document generation. Manual corrections must always remain possible.

---

## Project Workflow

```
Pending Bid → Won → Awaiting Contract → Awaiting NOA → Ready For Scheduling
→ Scheduled → Active → Awaiting Completion Signoff → Ready To Bill
→ Awaiting Payment → Paid → Warranty → Closed
```

---

## Scheduling Rules

- **Missing Contract** or **missing NOA** can block scheduling when those documents are required.
- **Consent To Start does NOT block scheduling.** It is a warning/informational flag only.
- A scheduling override request may be submitted. Overrides must record: Requested By, Reason, Approval Status, and Approval Date.

---

## Getting Started (Maker)

1. Open [Power Apps Studio](https://make.powerapps.com)
2. Navigate to the **BrickyardSharePointOperations** solution
3. Open the **Field Ops 2** canvas app
4. All SharePoint list connections are under the SharePoint connector pointed at `bhhomesolutions.sharepoint.com/sites/BrickyardHousesHomeSolutionsLLC`
5. See [BUILD-PHASES.md](docs/BUILD-PHASES.md) before making any changes to app source

---

## Contributing

Before editing any file in the `FieldOps2/` directory, read [DO-NOT-EDIT-FILES.md](docs/DO-NOT-EDIT-FILES.md).

All app changes must be made in Power Apps Studio, then exported and committed via the Azure DevOps pipeline. Do not hand-edit Power Apps YAML source files except for minor property corrections as noted in the canvas apps YAML warning header.