# Brickyard Operations Hub — Master Plan

## Purpose

This document captures the approved plan for transforming the current **Field Ops 2** single-screen app into a full **Brickyard Operations Hub** — a SharePoint-first, multi-screen Power Apps application that manages the complete lifecycle of every Brickyard project from bid to close.

---

## Architectural Principles

| Principle | Decision |
|---|---|
| Source of truth | SharePoint Lists + SharePoint Documents + OneDrive |
| Interface | Power Apps canvas app (interface only) |
| Database | No Dataverse; SharePoint Lists only |
| Automation | Power Automate flows |
| AI assistance | Microsoft Copilot — reads uploads, extracts data, updates trackers, helps generate documents |
| Manual correction | Always possible; Copilot suggestions are never final without user confirmation |
| Duplicate prevention | One record per project (Projects-Tracker-Primary), one record per client (Clients), one document copy per file (SharePoint library) |

---

## What the Hub Must Do

### Project Lifecycle Management
- Track every project from Pending Bid through Closed using a defined 13-stage workflow
- Allow status transitions to be made from the Power App
- Surface blocked projects (missing contract, missing NOA) before they reach scheduling

### Client Management
- Maintain a single client record per client in the Clients SharePoint list
- Projects reference clients via a lookup column — never copied or duplicated

### Document Management
- All documents (contracts, photos, receipts, invoices, generated docs) live once in their SharePoint project folder
- The Power App navigates to documents via folder links — it does not store document content
- Auto-generated documents are created from Word templates by Power Automate flows and saved directly into the project folder in SharePoint

### Scheduling
- Projects can be scheduled when ContractReceived = Yes AND NOAReceived = Yes (when those documents are required)
- Consent To Start is informational only — it is a warning badge, not a scheduling gate
- Scheduling overrides can be requested and must be tracked (Requested By, Reason, Approval Status, Approval Date)

### Billing and Financial Health
- Invoice records live in the Invoices SharePoint list
- Financial fields on the project record track Bid Amount, Labor Cost, Materials Cost, Other Cost, Total Expenses, Profit, Margin %, Amount Billed, Paid to Date, Currently Owed
- Financial Health screen aggregates across projects — no new data storage needed

### Document Generation
The following documents are auto-generated from Word templates by Power Automate:

| Document | Template Merge Fields | Saved To |
|---|---|---|
| Consent To Start | Project Name, Client Name, Site Address, Date | Project GeneratedDocs folder |
| Change Work Order | Project Name, Client Name, Description of Change, Amount | Project GeneratedDocs folder |
| Completion Signoff | Project Name, Client Name, Completion Date, Notes | Project GeneratedDocs folder |
| Warranty | Client Name, Project Name, Bid Number, Completion Date, Warranty Expiration | Project GeneratedDocs folder |
| Maintenance Guide | Client Name, Project Name, Bid Number, Completion Date, Warranty Expiration | Project GeneratedDocs folder |
| Cleaning and Maintenance Instructions | Client Name, Project Name, Bid Number, Completion Date | Project GeneratedDocs folder |

> **Client-facing documents (Warranty, Maintenance Guide, Cleaning and Maintenance Instructions) must include:**
> Client Name, Project Name, Bid Number, Completion Date, Warranty Expiration date
>
> **Client-facing documents must NOT include:**
> Cost, Revenue, Profit, Margin %, Labor Cost, Material Cost, or any internal financial data

---

## Business Rules

1. **SharePoint is the source of truth.** Power Apps reads from and writes to SharePoint only.
2. **No Dataverse** will be used as the primary database for this solution.
3. **One project record** per project. Workflow status transitions update the existing record — they never create a new record.
4. **One client record** per client. All project records reference the Clients list via lookup.
5. **One document copy** per file. Documents are stored in the SharePoint document library exactly once. The Power App displays links to those documents.
6. **Copilot assists, humans confirm.** Copilot can read uploaded documents (contracts, NOAs, photos, invoices), extract field data, and suggest record updates. All suggestions require manual review and confirmation before saving.
7. **Scheduling gates:** Only missing Contract and missing NOA can block a project from being scheduled (when those items are required for that project type). Consent To Start is never a hard block.
8. **Scheduling overrides** must record: who requested the override, the reason, the current approval status (Pending / Approved / Denied), and the approval date.
9. **Client-facing documents** must never expose internal financial data.

---

## Project Workflow

```
Pending Bid
    ↓
Won
    ↓
Awaiting Contract
    ↓
Awaiting NOA
    ↓
Ready For Scheduling          ← Contract and NOA gates apply here
    ↓
Scheduled
    ↓
Active
    ↓
Awaiting Completion Signoff
    ↓
Ready To Bill
    ↓
Awaiting Payment
    ↓
Paid
    ↓
Warranty
    ↓
Closed
```

### Status Transition Rules

| From | To | Condition |
|---|---|---|
| Any | Any (manual) | Allowed by authorized users; audit log updated |
| Awaiting NOA | Ready For Scheduling | ContractReceived = Yes AND NOAReceived = Yes (or override approved) |
| Active | Awaiting Completion Signoff | Field Manager marks work complete |
| Ready To Bill | Awaiting Payment | Invoice created and sent |
| Awaiting Payment | Paid | Payment recorded |
| Paid | Warranty | Warranty period begins |
| Warranty | Closed | Warranty period expires or is closed manually |

---

## Target App Menu

```
1. Dashboard
2. Pending Bids
3. Projects
4. Scheduling
5. Documents
6. Billing
7. Financial Health
8. Reports
9. Settings
```

---

## Project Workspace Panels

Every project opens a workspace screen with these panels:

```
1.  Project Details          (name, address, client, project type, notes)
2.  Workflow Status          (visual pipeline; current stage highlighted)
3.  Client Folder Link       (link to SharePoint client folder)
4.  Contract Status          (received Y/N, date, link to file)
5.  NOA Status               (received Y/N, date, link to file)
6.  Documents                (filtered view of Documents list for this project)
7.  Photos                   (link to Photos subfolder)
8.  Receipts                 (link to Receipts subfolder)
9.  Invoices                 (filtered view of Invoices list for this project)
10. Financial Snapshot       (bid, expenses, profit, margin, billed, owed)
11. Assigned Field Manager   (person lookup)
12. Assigned Crew            (text or crew lookup)
13. Scheduling Status        (scheduled date, override status if applicable)
14. Completion Package       (Completion Signoff doc link + status)
15. Warranty                 (Warranty doc link + expiration date)
16. Maintenance Guide        (Maintenance Guide + Cleaning Instructions doc links)
```

---

## Copilot Integration Points

| Trigger | Copilot Action | User Action Required |
|---|---|---|
| User uploads a contract PDF | Copilot reads the document, extracts key dates and party names, suggests updating ContractReceived and ContractDate on the project record | User reviews and confirms |
| User uploads an NOA | Copilot reads the NOA, extracts NOA date and project reference, suggests updating NOAReceived and NOADate | User reviews and confirms |
| User uploads a photo batch | Copilot tags photos by date and location if metadata is present | User reviews tags |
| User uploads an invoice | Copilot extracts invoice number, amount, and date; suggests creating a record in Invoices list | User reviews and confirms |
| User requests document generation | Copilot prepopulates the merge fields for the template from the project record | User reviews, edits if needed, triggers generation |

---

## Document Folder Structure

All files live in the SharePoint document library under the following hierarchy:

```
Sites/BrickyardHousesHomeSolutionsLLC/
└── Shared Documents/
    └── Projects/
        └── {ClientName}/
            └── {ProjectName-BidNumber}/
                ├── Contracts/
                ├── Photos/
                ├── Receipts/
                ├── Invoices/
                └── GeneratedDocs/
                    ├── ConsentToStart.docx
                    ├── ChangeWorkOrder.docx
                    ├── CompletionSignoff.docx
                    ├── Warranty.docx
                    ├── MaintenanceGuide.docx
                    └── CleaningAndMaintenanceInstructions.docx
```

The project record stores:
- `ClientFolderLink` — link to `{ClientName}/` folder
- `ProjectFolderLink` — link to `{ProjectName-BidNumber}/` folder
- `ConsentToStartDocUrl`, `ChangeWorkOrderDocUrl`, `CompletionSignoffDocUrl`, `WarrantyDocUrl`, `MaintenanceGuideDocUrl` — links to generated files

---

## Related Documents

- [BUILD-PHASES.md](BUILD-PHASES.md) — ordered build sequence
- [SHAREPOINT-DATA-MODEL.md](SHAREPOINT-DATA-MODEL.md) — complete data model
- [POWER-AUTOMATE-FLOWS.md](POWER-AUTOMATE-FLOWS.md) — automation flows
- [APP-SCREEN-PLAN.md](APP-SCREEN-PLAN.md) — screen-by-screen Power Apps design
- [DO-NOT-EDIT-FILES.md](DO-NOT-EDIT-FILES.md) — protected files
