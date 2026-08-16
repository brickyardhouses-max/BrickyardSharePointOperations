# Build Phases — Brickyard Operations Hub

## Overview

The Hub is built in ordered phases. Each phase produces a working, stable state before the next phase begins. No phase modifies files listed in [DO-NOT-EDIT-FILES.md](DO-NOT-EDIT-FILES.md) by hand.

All Power App changes are made in **Power Apps Studio**, exported via the Azure DevOps pipeline, and committed to this repository. Do not hand-edit YAML source files in `FieldOps2/` except for minor property corrections explicitly allowed by the canvas apps YAML header warning.

---

## Phase 0 — Documentation (Build Pass 0)

**Goal:** Create the complete approved plan as documentation. No app source files are modified.

**Deliverables:**
- [ ] `README.md` — updated project readme
- [ ] `docs/BRICKYARD-OPERATIONS-HUB-PLAN.md` — master plan
- [ ] `docs/BUILD-PHASES.md` — this file
- [ ] `docs/SHAREPOINT-DATA-MODEL.md` — data model
- [ ] `docs/POWER-AUTOMATE-FLOWS.md` — automation design
- [ ] `docs/APP-SCREEN-PLAN.md` — screen design
- [ ] `docs/DO-NOT-EDIT-FILES.md` — file protection list

**Files that change:** `README.md`, all new `docs/` files only.
**Files that must NOT change:** All files under `FieldOps2/`.

---

## Phase 1 — SharePoint Foundation

**Goal:** Build the complete SharePoint data model before touching the Power App. The existing app continues to work unchanged throughout this phase.

### 1.1 — Extend Projects-Tracker-Primary

Add new columns to the existing list. Do not rename, delete, or repurpose any existing column.

Columns to add:
- `WorkflowStatus` (Choice, 13 values)
- `ContractReceived` (Yes/No)
- `ContractDate` (Date)
- `NOAReceived` (Yes/No)
- `NOADate` (Date)
- `ConsentToStartSigned` (Yes/No — informational, not a scheduling gate)
- `SchedulingOverrideRequested` (Yes/No)
- `SchedulingOverrideRequestedBy` (Person or Group)
- `SchedulingOverrideReason` (Multi-line text)
- `SchedulingOverrideApprovalStatus` (Choice: Pending / Approved / Denied)
- `SchedulingOverrideApprovalDate` (Date)
- `ScheduledStartDate` (Date)
- `AssignedFieldManager` (Person or Group)
- `AssignedCrew` (Multi-line text)
- `ClientLookup` (Lookup → Clients list — add after Clients list is created)
- `ClientFolderLink` (Hyperlink)
- `ProjectFolderLink` (Hyperlink)
- `ConsentToStartDocUrl` (Hyperlink)
- `ChangeWorkOrderDocUrl` (Hyperlink)
- `CompletionSignoffDocUrl` (Hyperlink)
- `WarrantyDocUrl` (Hyperlink)
- `MaintenanceGuideDocUrl` (Hyperlink)
- `CleaningInstructionsDocUrl` (Hyperlink)
- `WarrantyExpirationDate` (Date)
- `CompletionDate` (Date — rename field_15 in title if desired; keep internal name)
- `BidNumber` (Single line of text — used as reference on client-facing documents)

### 1.2 — Create Clients List

New list. Name: `Clients`

Columns:
- `Title` (Client Name — built-in)
- `PrimaryContact` (Single line of text)
- `ContactEmail` (Single line of text)
- `ContactPhone` (Single line of text)
- `ClientFolderLink` (Hyperlink)
- `Notes` (Multi-line text)

After creating this list, add `ClientLookup` to `Projects-Tracker-Primary` pointing to `Clients`.

### 1.3 — Create SchedulingOverrides List

New list. Name: `SchedulingOverrides`

Columns:
- `Title` (Auto: ProjectName + date)
- `ProjectLookup` (Lookup → Projects-Tracker-Primary)
- `RequestedBy` (Person or Group)
- `Reason` (Multi-line text)
- `ApprovalStatus` (Choice: Pending / Approved / Denied)
- `ApprovalDate` (Date)
- `ApprovedBy` (Person or Group)

### 1.4 — Create Documents List

New list. Name: `ProjectDocuments`

Columns:
- `Title` (Document name)
- `ProjectLookup` (Lookup → Projects-Tracker-Primary)
- `DocumentType` (Choice: Contract / NOA / Consent To Start / Change Work Order / Completion Signoff / Warranty / Maintenance Guide / Cleaning Instructions / Photo / Receipt / Invoice / Other)
- `FileLink` (Hyperlink)
- `GeneratedDate` (Date)
- `GeneratedBy` (Person or Group)
- `IsClientFacing` (Yes/No)

### 1.5 — Create Invoices List

New list. Name: `Invoices`

Columns:
- `Title` (Invoice number)
- `ProjectLookup` (Lookup → Projects-Tracker-Primary)
- `InvoiceDate` (Date)
- `Amount` (Currency)
- `PaidDate` (Date)
- `Status` (Choice: Draft / Sent / Paid / Void)

### 1.6 — Create AppSettings List

New list. Name: `AppSettings`

Columns:
- `Title` (Setting name — acts as key)
- `SettingValue` (Single line of text)
- `Notes` (Multi-line text)

Seed entries:
- `DefaultWarrantyPeriodMonths` = 12
- `CompanyName` = Brickyard Houses Home Solutions LLC
- `CompanySupportEmail` = (company email)
- `OverrideApproverEmail` = (approver email)

### 1.7 — Set Up Document Library Folder Structure

In the `Shared Documents` library on the site, create the top-level `Projects/` folder. The Power Automate flow in Phase 2 will create per-project subfolders automatically.

**Manual structure to create once:**
```
Shared Documents/
└── Projects/
```

**Per-project structure created by flow:**
```
Projects/
└── {ClientName}/
    └── {ProjectName-BidNumber}/
        ├── Contracts/
        ├── Photos/
        ├── Receipts/
        ├── Invoices/
        └── GeneratedDocs/
```

### 1.8 — Validate Phase 1

- Open the existing Field Ops 2 app and confirm it still loads and the gallery still shows project records
- Confirm new columns appear in the SharePoint list view
- Confirm the Clients, SchedulingOverrides, ProjectDocuments, Invoices, and AppSettings lists exist

---

## Phase 2 — Power Automate Flows

**Goal:** Build automation flows before touching the Power App. Flows run independently of app changes.

See [POWER-AUTOMATE-FLOWS.md](POWER-AUTOMATE-FLOWS.md) for detailed design of each flow.

Flows to build (in order):

1. **Create Project Folder** — triggers when a new item is added to Projects-Tracker-Primary; creates the folder structure and writes back `ClientFolderLink` and `ProjectFolderLink`
2. **Generate Consent To Start** — triggered by button in app (HTTP request trigger); fills Word template; saves to GeneratedDocs; writes back `ConsentToStartDocUrl`
3. **Generate Change Work Order** — same pattern
4. **Generate Completion Signoff** — same pattern
5. **Generate Warranty Doc** — same pattern; uses client-safe merge fields only
6. **Generate Maintenance Guide** — same pattern; uses client-safe merge fields only
7. **Generate Cleaning and Maintenance Instructions** — same pattern; uses client-safe merge fields only
8. **Scheduling Override Notification** — triggered when `SchedulingOverrideRequested` is set to Yes; notifies approver; creates record in SchedulingOverrides list
9. **Copilot Document Extraction** — triggered when a file is added to a Contracts/ or NOA/ subfolder; Copilot reads the file and posts suggested field updates back to the app via an adaptive card or notification

### Phase 2 Validation

- Test Create Project Folder by adding a test project record; confirm folders appear in SharePoint
- Test each document generation flow manually using the flow test runner
- Confirm generated documents appear in the correct GeneratedDocs subfolder
- Confirm the Warranty and Maintenance Guide documents do not contain any financial fields

---

## Phase 3 — Power App Navigation Scaffold

**Goal:** Replace the single-screen layout with a multi-screen app. All changes made in Power Apps Studio only.

Steps:
1. Create a reusable **NavComponent** (canvas component) containing the 9-item left navigation bar
2. Create 9 blank screens: Dashboard, PendingBids, Projects, Scheduling, Documents, Billing, FinancialHealth, Reports, Settings
3. Rename `MainScreen1` to `DashboardScreen` — or keep the internal name and change the display name only
4. Add the NavComponent to every screen
5. Wire nav buttons to `Navigate()` calls
6. Export and commit; verify all 9 screens are present and navigation works with no data wired yet

### Phase 3 Validation

- All 9 screens accessible from nav
- No data errors in App Checker
- Existing SharePoint connection is intact

---

## Phase 4 — Projects Screen and Project Workspace

**Goal:** Build the core project browser and workspace.

Steps:
1. **Projects screen:** Gallery connected to Projects-Tracker-Primary; search by Title; filter by WorkflowStatus (dropdown); tap row → Navigate to ProjectWorkspace, passing the selected project
2. **ProjectWorkspace screen:** 16-panel scrollable or tabbed layout (see [APP-SCREEN-PLAN.md](APP-SCREEN-PLAN.md)); all panels read from/write to SharePoint
3. **Document generation buttons:** Each button calls the corresponding Power Automate flow via `Power Automate` connector; on success, refresh the project record
4. **Workflow Status panel:** Visual 13-step pipeline; current step highlighted; Patch button to advance status
5. **Financial Snapshot panel:** Display only — no financial data in client-facing documents
6. Export and commit

### Phase 4 Validation

- Open a project; all panels display correct data
- Edit a field; confirm patch saves to SharePoint
- Advance workflow status; confirm SharePoint record updates
- Generate one document; confirm it appears in the SharePoint folder and the URL updates on the project record

---

## Phase 5 — Pending Bids Screen

**Goal:** Dedicated screen for new bids.

Steps:
1. Gallery filtered to `WorkflowStatus = "Pending Bid"`
2. **New Bid** button opens a form to create a new record in Projects-Tracker-Primary — this is the only place new project records are created
3. Status transition buttons: Won / Lost (sets status and closes if lost)

### Phase 5 Validation

- Creating a new bid creates exactly one record in Projects-Tracker-Primary
- The new record has WorkflowStatus = "Pending Bid"
- Won transition changes status without creating a new record

---

## Phase 6 — Scheduling Screen

**Goal:** Scheduling management with gate enforcement.

Steps:
1. Gallery of projects in `Ready For Scheduling` and `Scheduled` statuses
2. For each project, display scheduling gate badges:
   - Red badge: Missing Contract (when required)
   - Red badge: Missing NOA (when required)
   - Yellow warning (not a gate): Consent To Start not yet signed
3. **Schedule** button: only enabled when gates pass (or override is approved)
4. **Request Override** form: writes RequestedBy, Reason to SchedulingOverrides list and sets `SchedulingOverrideRequested = Yes` on the project record; triggers override notification flow
5. Override status displayed on the project row

### Phase 6 Validation

- A project with missing contract cannot be scheduled without an approved override
- Consent To Start missing shows a warning but does not disable the Schedule button
- Submitting an override request creates a record in SchedulingOverrides and notifies the approver

---

## Phase 7 — Documents Screen

**Goal:** Unified document browser.

Steps:
1. Gallery from ProjectDocuments list; filter by project (if navigated from project workspace) or show all
2. Document type filter chips
3. Each row links to the actual file in SharePoint — no file content in the app
4. **Upload** button navigates the user to the correct SharePoint folder (opens in browser)
5. Copilot extraction status shown per document

---

## Phase 8 — Billing Screen

**Goal:** Invoice and payment tracking.

Steps:
1. Gallery of projects in `Ready To Bill` and `Awaiting Payment`
2. **Create Invoice** form: writes to Invoices list; patches Amount Billed on the project record
3. **Mark Paid** button: writes PaidDate, advances WorkflowStatus to Paid

---

## Phase 9 — Dashboard

**Goal:** Command center view.

Steps:
1. KPI cards: count of projects at each key status (Pending Bid, Active, Ready To Bill, Awaiting Payment)
2. Blocked projects panel: projects with missing Contract or NOA that are past Awaiting NOA stage
3. Recent activity: last 10 modified project records
4. Quick-navigate buttons to each main screen

---

## Phase 10 — Financial Health and Reports

**Goal:** Aggregate financial views.

Steps:
1. **Financial Health screen:** Sum and average calculations across Projects-Tracker-Primary using `Sum()`, `Average()`, `CountRows()` — no new data storage
2. **Reports screen:** Filterable gallery views (by status, date range, client, project type); optional export using Power Automate export flow

---

## Phase 11 — Settings Screen

**Goal:** App configuration.

Steps:
1. Gallery/form connected to AppSettings list
2. Fields: default warranty period, company name, support email, override approver email
3. Accessible to admin users only (role check via Office 365 Users connector if needed)

---

## Phase 12 — Final Polish and Publish

1. Run App Checker; resolve all warnings and errors
2. Remove unused data source connections (MicrosoftCopilotStudio, CopilotforFinance if not wired to features)
3. Accessibility review: keyboard nav, color contrast, screen reader labels
4. Increment solution version in `solution.yml`
5. Publish app via Power Apps Studio
6. Export solution; commit to Azure DevOps → sync to GitHub
7. Tag the commit with the release version

---

## Phase Summary Table

| Phase | Scope | Touches App YAML? |
|---|---|---|
| 0 | Documentation only | No |
| 1 | SharePoint lists and library structure | No |
| 2 | Power Automate flows | No |
| 3 | Navigation scaffold in Studio | Yes (Studio only) |
| 4 | Projects + Workspace screens | Yes (Studio only) |
| 5 | Pending Bids screen | Yes (Studio only) |
| 6 | Scheduling screen | Yes (Studio only) |
| 7 | Documents screen | Yes (Studio only) |
| 8 | Billing screen | Yes (Studio only) |
| 9 | Dashboard | Yes (Studio only) |
| 10 | Financial Health + Reports | Yes (Studio only) |
| 11 | Settings screen | Yes (Studio only) |
| 12 | Final polish and publish | Yes (Studio only) |
