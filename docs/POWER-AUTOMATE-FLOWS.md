# Power Automate Flows — Brickyard Operations Hub

## Overview

Power Automate flows handle all automation: folder creation, document generation, scheduling override notifications, and Copilot-assisted data extraction. Flows run independently of Power Apps source code changes and are built in Phase 2 (see [BUILD-PHASES.md](BUILD-PHASES.md)).

All flows connect to the SharePoint site at:
`https://bhhomesolutions.sharepoint.com/sites/BrickyardHousesHomeSolutionsLLC`

---

## Flow 1 — Create Project Folder

**Name:** `BHO - Create Project Folder`
**Trigger:** When an item is created in `Projects-Tracker-Primary`
**Purpose:** Automatically create the document folder hierarchy for a new project and write the folder links back to the project record.

### Steps

1. Get the ClientName by looking up the `ClientLookup` field → Clients list
2. Compose the folder path: `Projects/{ClientName}/{Title}-{BidNumber}`
3. Create folder: `Projects/{ClientName}/` (if it does not already exist — use "Create new folder" with error handling)
4. Create folder: `Projects/{ClientName}/{Title}-{BidNumber}/`
5. Create subfolders: `Contracts/`, `Photos/`, `Receipts/`, `Invoices/`, `GeneratedDocs/`
6. Get the URL of the `{Title}-{BidNumber}/` folder
7. Update the project item:
   - `ClientFolderLink` = URL of `{ClientName}/`
   - `ProjectFolderLink` = URL of `{Title}-{BidNumber}/`
8. Update the client record:
   - `ClientFolderLink` = URL of `{ClientName}/` (only if blank — do not overwrite)

### Error Handling

- If a folder with the same name already exists, log a note and continue (do not fail)
- If ClientLookup is blank, use a `NoClient` fallback folder and send a notification to the override approver email from AppSettings

---

## Flow 2 — Generate Consent To Start

**Name:** `BHO - Generate Consent To Start`
**Trigger:** HTTP request (called from Power App button)
**Input parameters:**
- `ProjectID` (integer — SharePoint item ID)

**Purpose:** Fill the Consent To Start Word template with project data and save the document to the project's GeneratedDocs folder.

### Steps

1. Get item from `Projects-Tracker-Primary` by `ProjectID`
2. Get the client record via `ClientLookup`
3. Get the template file from `_Templates/ConsentToStart-Template.docx`
4. Populate Word Online (Business) → "Fill in a Word template" action with:
   - `ProjectName` = Title
   - `ClientName` = ClientLookup.Title
   - `SiteAddress` = field_1
   - `DateGenerated` = `formatDateTime(utcNow(), 'MMMM d, yyyy')`
5. Create file in `{ProjectFolderLink}/GeneratedDocs/ConsentToStart.docx` (overwrite if exists)
6. Get the file URL
7. Update the project item: `ConsentToStartDocUrl` = file URL
8. Create a record in `ProjectDocuments`:
   - `Title` = "Consent To Start"
   - `ProjectLookup` = ProjectID
   - `DocumentType` = "Consent To Start"
   - `FileLink` = file URL
   - `GeneratedDate` = today
   - `GeneratedBy` = flow's connection user
   - `IsClientFacing` = No
9. Respond to the Power App with `{"status": "success", "url": "<file URL>"}`

---

## Flow 3 — Generate Change Work Order

**Name:** `BHO - Generate Change Work Order`
**Trigger:** HTTP request (called from Power App button)
**Input parameters:**
- `ProjectID` (integer)
- `ChangeDescription` (string)
- `ChangeAmount` (number)

### Steps

1. Get project item and client record (same as Flow 2)
2. Get template from `_Templates/ChangeWorkOrder-Template.docx`
3. Fill template:
   - `ProjectName`, `ClientName`, `BidNumber`
   - `ChangeDescription`, `ChangeAmount`
   - `DateGenerated`
4. Save to `{ProjectFolderLink}/GeneratedDocs/ChangeWorkOrder-{timestamp}.docx` (timestamped to preserve multiple change orders)
5. Get file URL
6. Update `ChangeWorkOrderDocUrl` on the project item (most recent)
7. Create ProjectDocuments record
8. Respond to Power App

---

## Flow 4 — Generate Completion Signoff

**Name:** `BHO - Generate Completion Signoff`
**Trigger:** HTTP request
**Input parameters:**
- `ProjectID` (integer)
- `Notes` (string, optional)

### Steps

1. Get project item and client record
2. Get template `_Templates/CompletionSignoff-Template.docx`
3. Fill template:
   - `ProjectName`, `ClientName`, `SiteAddress`
   - `CompletionDate`
   - `FieldManager` = AssignedFieldManager.DisplayName
   - `Notes`
   - `DateGenerated`
4. Save to `{ProjectFolderLink}/GeneratedDocs/CompletionSignoff.docx`
5. Update `CompletionSignoffDocUrl` on the project item
6. Create ProjectDocuments record
7. Respond to Power App

---

## Flow 5 — Generate Warranty Document

**Name:** `BHO - Generate Warranty Document`
**Trigger:** HTTP request
**Input parameters:**
- `ProjectID` (integer)

**Purpose:** Generate a client-facing warranty document. This document must NOT contain any financial data.

### Steps

1. Get project item and client record
2. Get template `_Templates/Warranty-Template.docx`
3. Fill template with **client-safe fields only**:
   - `ClientName`
   - `ProjectName`
   - `BidNumber`
   - `SiteAddress`
   - `CompletionDate`
   - `WarrantyExpirationDate`
   - `DateGenerated`
4. **Verify:** Confirm the template does NOT contain any of these merge fields before population — if found, reject and notify: `LaborCost`, `MaterialCost`, `OtherCost`, `TotalExpenses`, `Profit`, `Margin`, `BidAmount`, `Revenue`
5. Save to `{ProjectFolderLink}/GeneratedDocs/Warranty.docx`
6. Update `WarrantyDocUrl` on the project item
7. Create ProjectDocuments record with `IsClientFacing = Yes`
8. Respond to Power App

---

## Flow 6 — Generate Maintenance Guide

**Name:** `BHO - Generate Maintenance Guide`
**Trigger:** HTTP request
**Input parameters:**
- `ProjectID` (integer)

**Purpose:** Client-facing maintenance guide. No financial data.

### Steps

Same pattern as Flow 5. Uses `_Templates/MaintenanceGuide-Template.docx`.
Saves to `{ProjectFolderLink}/GeneratedDocs/MaintenanceGuide.docx`.
Updates `MaintenanceGuideDocUrl`. Sets `IsClientFacing = Yes`.

---

## Flow 7 — Generate Cleaning and Maintenance Instructions

**Name:** `BHO - Generate Cleaning and Maintenance Instructions`
**Trigger:** HTTP request
**Input parameters:**
- `ProjectID` (integer)

**Purpose:** Client-facing cleaning instructions. No financial data.

### Steps

Same pattern as Flows 5 and 6. Uses `_Templates/CleaningAndMaintenanceInstructions-Template.docx`.
Saves to `{ProjectFolderLink}/GeneratedDocs/CleaningAndMaintenanceInstructions.docx`.
Updates `CleaningInstructionsDocUrl`. Sets `IsClientFacing = Yes`.

---

## Flow 8 — Scheduling Override Notification

**Name:** `BHO - Scheduling Override Notification`
**Trigger:** When an item is modified in `Projects-Tracker-Primary` AND `SchedulingOverrideRequested` changed to `Yes`
**Purpose:** Notify the approver when a scheduling override is requested; create an audit record.

### Steps

1. Get the project item
2. Create a record in `SchedulingOverrides`:
   - `Title` = `{ProjectTitle} Override {Date}`
   - `ProjectLookup` = project item ID
   - `RequestedBy` = `SchedulingOverrideRequestedBy`
   - `Reason` = `SchedulingOverrideReason`
   - `ApprovalStatus` = "Pending"
3. Get `OverrideApproverEmail` from `AppSettings`
4. Send an email to the approver with:
   - Subject: `Scheduling Override Requested — {ProjectTitle}`
   - Body: Project name, requested by, reason, a direct link to the project in SharePoint, and a link to the SchedulingOverrides list item
5. Post an adaptive card in Teams (if Teams is configured) to the approver
6. When the approver responds (Approved/Denied via adaptive card or manually in SharePoint):
   - Update `SchedulingOverrideApprovalStatus` and `SchedulingOverrideApprovalDate` on the project item
   - Update the SchedulingOverrides list record

---

## Flow 9 — Copilot Document Extraction

**Name:** `BHO - Copilot Document Extraction`
**Trigger:** When a file is created in a `Contracts/` or `NOA*/` folder anywhere under `Shared Documents/Projects/`
**Purpose:** Copilot reads the uploaded document, extracts relevant data, and posts suggested field updates for user review. All suggestions require manual confirmation before saving.

### Steps

1. Get the file content
2. Send the file to Microsoft Copilot Studio / AI Builder Document Processing model
3. Extract fields:
   - For contracts: party names, effective date, project address, contract value
   - For NOAs: NOA date, project reference, issuing authority
4. Match the file's folder path to the project record (by matching `ProjectFolderLink` pattern)
5. Post an adaptive card to the assigned field manager (or the user who uploaded the file) with:
   - Extracted fields displayed for review
   - "Confirm and Update" button → triggers a child flow that patches the project record
   - "Edit Before Saving" button → opens the project record in the Power App
   - "Dismiss" button → no action taken
6. No updates are written to SharePoint until the user explicitly confirms

### Fields Updated on Confirmation (Contract)

- `ContractReceived` = Yes
- `ContractDate` = extracted date

### Fields Updated on Confirmation (NOA)

- `NOAReceived` = Yes
- `NOADate` = extracted date

---

## Flow Summary Table

| # | Flow Name | Trigger | Output |
|---|---|---|---|
| 1 | Create Project Folder | New item in Projects-Tracker-Primary | Folder hierarchy + links written back |
| 2 | Generate Consent To Start | HTTP (from app) | .docx in GeneratedDocs; URL on project record |
| 3 | Generate Change Work Order | HTTP (from app) | .docx in GeneratedDocs; URL on project record |
| 4 | Generate Completion Signoff | HTTP (from app) | .docx in GeneratedDocs; URL on project record |
| 5 | Generate Warranty Document | HTTP (from app) | Client-safe .docx; URL on project record |
| 6 | Generate Maintenance Guide | HTTP (from app) | Client-safe .docx; URL on project record |
| 7 | Generate Cleaning Instructions | HTTP (from app) | Client-safe .docx; URL on project record |
| 8 | Scheduling Override Notification | Item modified (override requested) | Override record; email/Teams notification to approver |
| 9 | Copilot Document Extraction | File created in Contracts/ or NOA/ folder | Adaptive card to user for review and confirmation |

---

## Notes

- All document generation flows use the **Word Online (Business)** connector → "Fill in a Word template" action.
- All HTTP-triggered flows must use JSON body inputs and return JSON responses so the Power App can parse success/failure and display the document URL.
- Flows 5, 6, and 7 must include a template field validation step to ensure no financial merge fields are present before generating the document.
- Flow 9 suggestions are advisory only. Manual correction is always possible. No data is saved without explicit user confirmation.
