# SharePoint Data Model — Brickyard Operations Hub

## Overview

All application data lives in SharePoint. Power Apps reads from and writes to these lists and libraries. No data is duplicated across lists. No Dataverse entities are used as the primary database.

**SharePoint Site:** `https://bhhomesolutions.sharepoint.com/sites/BrickyardHousesHomeSolutionsLLC`

---

## Lists

### 1. Projects-Tracker-Primary *(existing — extend only)*

**Purpose:** Single source of truth for all project records. One row per project. Never create a second row for the same project.

**List URL:** `https://bhhomesolutions.sharepoint.com/sites/BrickyardHousesHomeSolutionsLLC/Lists/ProjectsTrackerPrimary/AllItems.aspx`

#### Existing Columns (do not rename or delete)

| Display Name | Internal Name | Type | Notes |
|---|---|---|---|
| Title | Title | Single line | Project name |
| Site Address | field_1 | Single line | |
| Agency/Insurer | field_2 | Single line | |
| Case Manager | field_3 | Single line | |
| Case Mgr Email | field_4 | Single line | |
| Project Type | field_5 | Single line | |
| Bid Amount | field_6 | Number | |
| Labor Cost | field_7 | Number | Internal financial — never expose on client docs |
| Materials Cost | field_8 | Number | Internal financial — never expose on client docs |
| Other Cost | field_9 | Number | Internal financial — never expose on client docs |
| Total Expenses | field_10 | Number | Internal financial — never expose on client docs |
| Profit | field_11 | Number | Internal financial — never expose on client docs |
| Margin % | field_12 | Single line | Internal financial — never expose on client docs |
| Status | field_13 | Single line | Legacy free-text status; keep during transition |
| NOA Received | field_14 | Single line | Legacy; replace with NOAReceived Yes/No |
| Completion Date | field_15 | Single line | Legacy; supplement with new CompletionDate Date column |
| Billed | field_16 | Single line | |
| Amount Billed | field_17 | Number | |
| Paid to Date | field_18 | Number | |
| Currently Owed | field_19 | Number | |
| Change Order | field_20 | Single line | |
| Notes | field_21 | Multiple lines | |

#### New Columns to Add

| Display Name | Type | Allowed Values / Notes |
|---|---|---|
| WorkflowStatus | Choice | Pending Bid; Won; Awaiting Contract; Awaiting NOA; Ready For Scheduling; Scheduled; Active; Awaiting Completion Signoff; Ready To Bill; Awaiting Payment; Paid; Warranty; Closed |
| BidNumber | Single line | Reference number used on client-facing documents |
| ClientLookup | Lookup | → Clients list, Title column |
| ContractReceived | Yes/No | Default: No |
| ContractDate | Date | Date contract was received |
| NOAReceived | Yes/No | Default: No |
| NOADate | Date | Date NOA was received |
| ConsentToStartSigned | Yes/No | Informational only — does NOT block scheduling |
| CompletionDate | Date | Actual project completion date |
| WarrantyExpirationDate | Date | Calculated or manually set |
| ScheduledStartDate | Date | Planned start date when scheduled |
| AssignedFieldManager | Person or Group | |
| AssignedCrew | Multiple lines | Comma-separated crew member names or notes |
| SchedulingOverrideRequested | Yes/No | Default: No; triggers override notification flow |
| SchedulingOverrideRequestedBy | Person or Group | |
| SchedulingOverrideReason | Multiple lines | |
| SchedulingOverrideApprovalStatus | Choice | Pending; Approved; Denied |
| SchedulingOverrideApprovalDate | Date | |
| ClientFolderLink | Hyperlink | Link to `Projects/{ClientName}/` in Shared Documents |
| ProjectFolderLink | Hyperlink | Link to `Projects/{ClientName}/{ProjectName-BidNumber}/` |
| ConsentToStartDocUrl | Hyperlink | Link to generated ConsentToStart.docx |
| ChangeWorkOrderDocUrl | Hyperlink | Link to generated ChangeWorkOrder.docx |
| CompletionSignoffDocUrl | Hyperlink | Link to generated CompletionSignoff.docx |
| WarrantyDocUrl | Hyperlink | Link to generated Warranty.docx |
| MaintenanceGuideDocUrl | Hyperlink | Link to generated MaintenanceGuide.docx |
| CleaningInstructionsDocUrl | Hyperlink | Link to generated CleaningAndMaintenanceInstructions.docx |

> **Rule:** WorkflowStatus replaces the free-text `field_13` (Status) for pipeline tracking. During the transition period, both columns exist. Once all records are migrated to WorkflowStatus, `field_13` is retired (hidden in views, not deleted).

---

### 2. Clients *(new)*

**Purpose:** Single source of truth for client records. One row per client. Projects reference this list via `ClientLookup`. Never copy client data into the Projects list.

| Display Name | Type | Notes |
|---|---|---|
| Title | Single line | Client name (built-in) |
| PrimaryContact | Single line | Main point of contact name |
| ContactEmail | Single line | |
| ContactPhone | Single line | |
| ClientFolderLink | Hyperlink | Link to `Projects/{ClientName}/` in Shared Documents |
| Notes | Multiple lines | |

---

### 3. SchedulingOverrides *(new)*

**Purpose:** Audit log for every scheduling override request. One row per request.

| Display Name | Type | Notes |
|---|---|---|
| Title | Single line | Auto: `{ProjectName} Override {Date}` |
| ProjectLookup | Lookup | → Projects-Tracker-Primary, Title |
| RequestedBy | Person or Group | |
| Reason | Multiple lines | |
| ApprovalStatus | Choice | Pending; Approved; Denied |
| ApprovalDate | Date | |
| ApprovedBy | Person or Group | |

---

### 4. ProjectDocuments *(new)*

**Purpose:** Metadata registry for all documents associated with a project. The actual file lives in the SharePoint document library — this list stores the metadata and a link. Never store duplicate copies of a document.

| Display Name | Type | Notes |
|---|---|---|
| Title | Single line | Document display name |
| ProjectLookup | Lookup | → Projects-Tracker-Primary, Title |
| DocumentType | Choice | Contract; NOA; Consent To Start; Change Work Order; Completion Signoff; Warranty; Maintenance Guide; Cleaning Instructions; Photo; Receipt; Invoice; Other |
| FileLink | Hyperlink | Direct link to the file in SharePoint |
| GeneratedDate | Date | Date the document was generated or uploaded |
| GeneratedBy | Person or Group | |
| IsClientFacing | Yes/No | True for Warranty, Maintenance Guide, Cleaning Instructions — these must not contain financial data |

---

### 5. Invoices *(new)*

**Purpose:** Invoice records for billing and payment tracking. One row per invoice.

| Display Name | Type | Notes |
|---|---|---|
| Title | Single line | Invoice number |
| ProjectLookup | Lookup | → Projects-Tracker-Primary, Title |
| InvoiceDate | Date | |
| Amount | Currency | |
| PaidDate | Date | Blank until payment received |
| Status | Choice | Draft; Sent; Paid; Void |

---

### 6. AppSettings *(new)*

**Purpose:** Key-value store for app configuration. Read by Power App `OnStart` into a collection.

| Display Name | Type | Notes |
|---|---|---|
| Title | Single line | Setting key (e.g., `DefaultWarrantyPeriodMonths`) |
| SettingValue | Single line | Setting value |
| Notes | Multiple lines | Description of the setting |

**Seed records:**

| Title | SettingValue |
|---|---|
| DefaultWarrantyPeriodMonths | 12 |
| CompanyName | Brickyard Houses Home Solutions LLC |
| CompanySupportEmail | *(set on deployment)* |
| OverrideApproverEmail | *(set on deployment)* |

---

## Document Library Structure

**Library:** Shared Documents (default library on the site)

```
Shared Documents/
└── Projects/
    └── {ClientName}/                           ← Created once per client
        └── {ProjectName}-{BidNumber}/          ← Created by flow when project is created
            ├── Contracts/                      ← Uploaded contract PDFs
            ├── Photos/                         ← Site photos
            ├── Receipts/                       ← Material and expense receipts
            ├── Invoices/                       ← Invoice PDFs
            └── GeneratedDocs/                  ← Auto-generated documents
                ├── ConsentToStart.docx
                ├── ChangeWorkOrder.docx
                ├── CompletionSignoff.docx
                ├── Warranty.docx
                ├── MaintenanceGuide.docx
                └── CleaningAndMaintenanceInstructions.docx
```

### Folder Naming Convention

| Folder | Pattern | Example |
|---|---|---|
| Client folder | `{ClientName}` | `Johnson Family` |
| Project folder | `{ProjectName}-{BidNumber}` | `Roof Replacement-BH2024-031` |

---

## Word Templates

Word templates live in a separate `_Templates/` folder in Shared Documents and are **never** modified by the generation flows — flows use them as read-only sources.

```
Shared Documents/
└── _Templates/
    ├── ConsentToStart-Template.docx
    ├── ChangeWorkOrder-Template.docx
    ├── CompletionSignoff-Template.docx
    ├── Warranty-Template.docx
    ├── MaintenanceGuide-Template.docx
    └── CleaningAndMaintenanceInstructions-Template.docx
```

### Template Merge Fields by Document

#### Consent To Start
| Field | Source Column |
|---|---|
| `{{ProjectName}}` | Title |
| `{{ClientName}}` | ClientLookup.Title |
| `{{SiteAddress}}` | field_1 |
| `{{DateGenerated}}` | Today() |

#### Change Work Order
| Field | Source Column |
|---|---|
| `{{ProjectName}}` | Title |
| `{{ClientName}}` | ClientLookup.Title |
| `{{BidNumber}}` | BidNumber |
| `{{ChangeDescription}}` | Passed as parameter from app |
| `{{ChangeAmount}}` | Passed as parameter from app |
| `{{DateGenerated}}` | Today() |

#### Completion Signoff
| Field | Source Column |
|---|---|
| `{{ProjectName}}` | Title |
| `{{ClientName}}` | ClientLookup.Title |
| `{{CompletionDate}}` | CompletionDate |
| `{{FieldManager}}` | AssignedFieldManager |
| `{{Notes}}` | Notes |

#### Warranty *(client-facing — no financial data)*
| Field | Source Column |
|---|---|
| `{{ClientName}}` | ClientLookup.Title |
| `{{ProjectName}}` | Title |
| `{{BidNumber}}` | BidNumber |
| `{{CompletionDate}}` | CompletionDate |
| `{{WarrantyExpirationDate}}` | WarrantyExpirationDate |
| `{{SiteAddress}}` | field_1 |

**Must NOT include:** field_7 (Labor), field_8 (Materials), field_9 (Other), field_10 (Total Expenses), field_11 (Profit), field_12 (Margin %), field_6 (Bid Amount)

#### Maintenance Guide *(client-facing — no financial data)*
| Field | Source Column |
|---|---|
| `{{ClientName}}` | ClientLookup.Title |
| `{{ProjectName}}` | Title |
| `{{BidNumber}}` | BidNumber |
| `{{CompletionDate}}` | CompletionDate |
| `{{WarrantyExpirationDate}}` | WarrantyExpirationDate |

**Must NOT include:** Any financial columns.

#### Cleaning and Maintenance Instructions *(client-facing — no financial data)*
| Field | Source Column |
|---|---|
| `{{ClientName}}` | ClientLookup.Title |
| `{{ProjectName}}` | Title |
| `{{BidNumber}}` | BidNumber |
| `{{CompletionDate}}` | CompletionDate |

**Must NOT include:** Any financial columns.

---

## Scheduling Logic Reference

| Condition | Effect |
|---|---|
| `ContractReceived = No` AND contract is required for this project type | Scheduling blocked; red badge displayed |
| `NOAReceived = No` AND NOA is required for this project type | Scheduling blocked; red badge displayed |
| `ConsentToStartSigned = No` | Warning badge displayed; scheduling NOT blocked |
| `SchedulingOverrideApprovalStatus = "Approved"` | Scheduling gates bypassed for this project |

---

## Data Relationship Diagram

```
Clients (1)
    │
    └──< Projects-Tracker-Primary (many)
              │
              ├──< ProjectDocuments (many)
              │
              ├──< SchedulingOverrides (many)
              │
              └──< Invoices (many)
```

All relationships use SharePoint Lookup columns. No foreign key constraints beyond what SharePoint provides.
