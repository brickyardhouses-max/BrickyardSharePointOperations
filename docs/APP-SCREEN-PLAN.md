# App Screen Plan — Brickyard Operations Hub

## Overview

The Hub replaces the current single-screen (`MainScreen1`) app with a multi-screen canvas app. All screens are built in Power Apps Studio and exported via the Azure DevOps pipeline. No YAML source files are hand-edited.

All screens share:
- A **NavComponent** (left navigation bar, 9 items)
- A consistent header bar with app name, current user, and a back button when inside a project workspace
- The **Projects-Tracker-Primary** SharePoint data source (plus additional lists per screen)

---

## Navigation Menu

| Position | Screen Name | Internal Screen Name | Icon |
|---|---|---|---|
| 1 | Dashboard | `DashboardScreen` | Home |
| 2 | Pending Bids | `PendingBidsScreen` | Money/bid |
| 3 | Projects | `ProjectsScreen` | Clipboard |
| 4 | Scheduling | `SchedulingScreen` | Calendar |
| 5 | Documents | `DocumentsScreen` | Folder |
| 6 | Billing | `BillingScreen` | Invoice |
| 7 | Financial Health | `FinancialHealthScreen` | Chart |
| 8 | Reports | `ReportsScreen` | Table |
| 9 | Settings | `SettingsScreen` | Gear |

**NavComponent behavior:**
- Active screen item is highlighted
- Collapses to icon-only on small/mobile form factor
- Visible on all screens except when inside `ProjectWorkspaceScreen` (replaced by a back-chevron header button)

---

## Screen Designs

---

### Screen 1 — Dashboard (`DashboardScreen`)

**Purpose:** Command center view. First screen users see after launch.

**Data sources:** Projects-Tracker-Primary (read-only aggregations)

**Components:**

| Component | Description |
|---|---|
| KPI Row | Cards showing count of projects at: Pending Bid, Active, Ready To Bill, Awaiting Payment |
| Blocked Projects Panel | Gallery of projects with `WorkflowStatus` in (Awaiting Contract, Awaiting NOA) that are past 14 days with no update |
| Missing Gate Panel | Projects that are `Ready For Scheduling` but have `ContractReceived = No` or `NOAReceived = No` |
| Recent Activity Feed | Last 10 records modified in Projects-Tracker-Primary (ordered by Modified desc) |
| Quick Navigate Buttons | Buttons to jump to Pending Bids, Scheduling, Billing screens |

**App formula notes:**
- KPI counts use `CountRows(Filter('Projects-Tracker-Primary', WorkflowStatus = "Active"))` etc.
- No data is modified from this screen

---

### Screen 2 — Pending Bids (`PendingBidsScreen`)

**Purpose:** Create new bids and track bids that are not yet won.

**Data sources:** Projects-Tracker-Primary

**Components:**

| Component | Description |
|---|---|
| Gallery | Filtered: `WorkflowStatus = "Pending Bid"`; columns: Project Name, Client, Bid Amount, Created Date |
| Search Box | Filters gallery by Title |
| New Bid Button | Opens `NewBidForm` in New mode |
| NewBidForm | `Form` control; `DataSource = 'Projects-Tracker-Primary'`; `DefaultMode = FormMode.New`; On submit: `SubmitForm`; on success: refresh gallery |
| Status Transition Buttons (per row) | **Won** → `Patch` WorkflowStatus to "Won"; **Lost / No Bid** → `Patch` WorkflowStatus to "Closed" |
| Row tap | Navigate to `ProjectWorkspaceScreen`, passing `ThisItem` as context |

**Rules:**
- The New Bid form is the **only** place a new project record is created
- Submitting the form creates exactly one record in Projects-Tracker-Primary
- Won/Lost transitions patch the existing record — they do not create a new record

---

### Screen 3 — Projects (`ProjectsScreen`)

**Purpose:** Full project list browser with status filtering and search.

**Data sources:** Projects-Tracker-Primary, Clients

**Components:**

| Component | Description |
|---|---|
| Gallery | All projects; columns: Project Name, Client, WorkflowStatus, Scheduled Start, Field Manager |
| Search Box | Filters by Title |
| Status Filter Dropdown | Filters gallery by WorkflowStatus; "All" option shows all records |
| Client Filter Dropdown | Filters by ClientLookup |
| Row tap | Navigate to `ProjectWorkspaceScreen`, passing `ThisItem` |
| New Project Button | Same as New Bid (navigates to PendingBidsScreen with new form pre-opened) |

---

### Screen 4 — Project Workspace (`ProjectWorkspaceScreen`)

**Purpose:** Full project detail view and edit hub. Navigated to from Projects, Pending Bids, Scheduling, or Billing screens.

**Data sources:** Projects-Tracker-Primary, Clients, ProjectDocuments, Invoices

**Context variable passed in:** `SelectedProject` (the gallery row from the source screen)

**Layout:** Vertical scrollable panels OR tabbed layout (tabs preferred on tablet form factor)

**Panels:**

---

#### Panel 1 — Project Details

| Field | Source | Editable |
|---|---|---|
| Project Name | Title | Yes |
| Bid Number | BidNumber | Yes |
| Site Address | field_1 | Yes |
| Project Type | field_5 | Yes |
| Agency / Insurer | field_2 | Yes |
| Case Manager | field_3 | Yes |
| Case Mgr Email | field_4 | Yes |
| Notes | field_21 | Yes |
| Created | Created | No |
| Modified | Modified | No |

Save button: `Patch('Projects-Tracker-Primary', SelectedProject, { ...edits })`

---

#### Panel 2 — Workflow Status

| Component | Description |
|---|---|
| Pipeline Visual | Row of 13 stage chips; current stage highlighted in brand color; completed stages shown with checkmark |
| Advance Status Button | Dropdown or forward-arrow button to move to the next stage; requires confirmation |
| Set Custom Status | Dropdown to jump to any stage (admin only) |
| Status History | Optional: last 5 status changes with date and user (if audit list is added in future) |

---

#### Panel 3 — Client Info

| Field | Source | Notes |
|---|---|---|
| Client Name | ClientLookup.Title | Read-only link; tap to open Clients list item |
| Primary Contact | Clients.PrimaryContact | |
| Contact Email | Clients.ContactEmail | |
| Contact Phone | Clients.ContactPhone | |
| Client Folder Link | ClientFolderLink | Opens SharePoint folder in browser |

---

#### Panel 4 — Contract Status

| Component | Description |
|---|---|
| Contract Received | Yes/No toggle → patches `ContractReceived` |
| Contract Date | Date picker → patches `ContractDate` |
| Contract File Link | Hyperlink to file; navigates to SharePoint |
| Upload prompt | "Upload contract to Contracts/ folder" button → opens `ProjectFolderLink/Contracts/` in browser |
| Status Badge | Green "Received" / Red "Missing" |

---

#### Panel 5 — NOA Status

Same structure as Panel 4. Fields: `NOAReceived`, `NOADate`, link to NOA file.

---

#### Panel 6 — Documents

| Component | Description |
|---|---|
| Document Gallery | Filtered from `ProjectDocuments` where `ProjectLookup = SelectedProject.ID`; columns: Document Name, Type, Date, Link |
| Filter by Type | Chips for document type filter |
| Open Folder Button | Opens `ProjectFolderLink` in browser |
| Generate buttons | One button per auto-generated document type (see Panel 14–16) |

---

#### Panel 7 — Photos

| Component | Description |
|---|---|
| Photos Folder Link | Button that opens `{ProjectFolderLink}/Photos/` in browser |
| Recent Photo Count | `CountRows(Filter(ProjectDocuments, ...DocumentType = "Photo"))` |
| Last Photo Date | Most recent photo upload date from ProjectDocuments |

Photos are stored and viewed in SharePoint — not embedded in the app.

---

#### Panel 8 — Receipts

Same structure as Panel 7. Points to `{ProjectFolderLink}/Receipts/`.

---

#### Panel 9 — Invoices

| Component | Description |
|---|---|
| Invoice Gallery | Filtered from `Invoices` where `ProjectLookup = SelectedProject.ID`; columns: Invoice #, Date, Amount, Status |
| New Invoice Button | Opens invoice form (Patch to Invoices list) |
| Total Billed | `Sum(Filter(Invoices, ...), Amount)` |
| Total Paid | `Sum(Filter(Invoices, ..., Status = "Paid"), Amount)` |
| Amount Owed | Total Billed − Total Paid |

---

#### Panel 10 — Financial Snapshot

> **Internal use only — never show this panel in any client-facing output.**

| Field | Source |
|---|---|
| Bid Amount | field_6 |
| Labor Cost | field_7 |
| Materials Cost | field_8 |
| Other Cost | field_9 |
| Total Expenses | field_10 |
| Profit | field_11 |
| Margin % | field_12 |
| Amount Billed | field_17 |
| Paid to Date | field_18 |
| Currently Owed | field_19 |

All fields editable. Save via Patch.

---

#### Panel 11 — Assigned Field Manager

| Field | Source |
|---|---|
| Field Manager | AssignedFieldManager (Person picker) |
| Crew | AssignedCrew (multi-line text) |

---

#### Panel 12 — Scheduling Status

| Component | Description |
|---|---|
| Scheduled Start Date | `ScheduledStartDate` date picker |
| Contract Gate Badge | Red "Contract Missing" if `ContractReceived = No` |
| NOA Gate Badge | Red "NOA Missing" if `NOAReceived = No` |
| Consent Warning | Yellow "Consent To Start Not Signed" if `ConsentToStartSigned = No` (warning only, not a gate) |
| Schedule Button | Enabled when: `ContractReceived = Yes AND NOAReceived = Yes` OR `SchedulingOverrideApprovalStatus = "Approved"` |
| Request Override Button | Opens override form; writes to SchedulingOverrides; triggers Flow 8 |
| Override Status Display | Shows current override approval status if a request is pending |

---

#### Panel 13 — Completion Package

| Component | Description |
|---|---|
| Completion Signoff Status | Signed Y/N |
| Completion Date | `CompletionDate` date picker |
| Generate Completion Signoff Button | Calls Flow 4 via HTTP; on success shows link |
| Completion Signoff Doc Link | `CompletionSignoffDocUrl` hyperlink |

---

#### Panel 14 — Warranty

| Component | Description |
|---|---|
| Warranty Expiration Date | `WarrantyExpirationDate` date picker |
| Generate Warranty Doc Button | Calls Flow 5 via HTTP |
| Warranty Doc Link | `WarrantyDocUrl` hyperlink |

---

#### Panel 15 — Maintenance Guide

| Component | Description |
|---|---|
| Generate Maintenance Guide Button | Calls Flow 6 via HTTP |
| Maintenance Guide Doc Link | `MaintenanceGuideDocUrl` hyperlink |
| Generate Cleaning Instructions Button | Calls Flow 7 via HTTP |
| Cleaning Instructions Doc Link | `CleaningInstructionsDocUrl` hyperlink |

> Buttons for Panels 14 and 15 are grouped under a **"Generate Client Package"** action that calls Flows 5, 6, and 7 in sequence and reports success/failure for each.

---

#### Panel 16 — Consent To Start

| Component | Description |
|---|---|
| Consent To Start Signed | `ConsentToStartSigned` Yes/No toggle |
| Generate Consent To Start Button | Calls Flow 2 via HTTP |
| Consent To Start Doc Link | `ConsentToStartDocUrl` hyperlink |
| Warning Note | Displays: "Consent To Start is not required to schedule this project." |

---

### Screen 5 — Scheduling (`SchedulingScreen`)

**Purpose:** View and manage projects that are ready for scheduling or already scheduled.

**Data sources:** Projects-Tracker-Primary, SchedulingOverrides

**Components:**

| Component | Description |
|---|---|
| Gallery | Projects with WorkflowStatus in ("Ready For Scheduling", "Scheduled"); columns: Project Name, Scheduled Start, Field Manager, Gate Status badges |
| Gate Status Badges (per row) | Red "Contract" if missing; Red "NOA" if missing; Yellow "No Consent" if ConsentToStartSigned = No |
| Schedule Button (per row) | Enabled per scheduling rules above; patches WorkflowStatus to "Scheduled" and ScheduledStartDate |
| Request Override Button (per row) | Opens override form overlay |
| Override Form | Fields: RequestedBy (auto = current user), Reason (text input), Submit → Patch project record + trigger Flow 8 |
| Override Pending Badge | Shown per row if SchedulingOverrideApprovalStatus = "Pending" |
| Override Approved Badge | Shown per row if SchedulingOverrideApprovalStatus = "Approved" |

---

### Screen 6 — Documents (`DocumentsScreen`)

**Purpose:** Unified document viewer across all projects.

**Data sources:** ProjectDocuments, Projects-Tracker-Primary

**Components:**

| Component | Description |
|---|---|
| Project Filter Dropdown | Filter gallery to one project (or "All Projects") |
| Document Type Filter | Chips for each document type |
| Gallery | Filtered ProjectDocuments rows; columns: Document Name, Type, Project, Date, Link |
| Open File Button (per row) | Opens `FileLink` in browser |
| Upload Notice | "To upload new documents, open the project workspace and navigate to the correct subfolder." |

---

### Screen 7 — Billing (`BillingScreen`)

**Purpose:** Invoice creation and payment tracking.

**Data sources:** Projects-Tracker-Primary, Invoices

**Components:**

| Component | Description |
|---|---|
| Gallery | Projects with WorkflowStatus in ("Ready To Bill", "Awaiting Payment"); columns: Project, Billed Amount, Paid, Owed, Status |
| Create Invoice Button (per row) | Opens invoice form; Patch to Invoices list; updates `field_17` (Amount Billed) on project |
| Mark Paid Button (per row) | Sets `PaidDate` on invoice; patches `field_18` (Paid to Date), `field_19` (Currently Owed), advances WorkflowStatus to "Paid" |
| Invoice History Gallery | Filtered Invoices for a selected project |

---

### Screen 8 — Financial Health (`FinancialHealthScreen`)

**Purpose:** Aggregate financial view across all projects. No new data storage.

**Data sources:** Projects-Tracker-Primary (read-only aggregations)

**Components:**

| Component | Description |
|---|---|
| Total Bid Pipeline | Sum of field_6 where WorkflowStatus not in (Closed) |
| Total Revenue | Sum of field_17 (Amount Billed) |
| Total Paid | Sum of field_18 |
| Total Outstanding | Sum of field_19 |
| Avg Margin % | Average of field_12 |
| Profit by Project Type | Grouped bar chart — Sum(field_11) grouped by field_5 |
| Projects by Status | Pie or donut chart — CountRows grouped by WorkflowStatus |
| Date Range Filter | Filter all calculations to a date range (Created >= start AND Created <= end) |

> **Note:** All calculations are in-app using Power Apps formula functions. No aggregation tables are written to SharePoint.

---

### Screen 9 — Reports (`ReportsScreen`)

**Purpose:** Filterable tabular views for reporting and data export.

**Data sources:** Projects-Tracker-Primary, Invoices

**Components:**

| Component | Description |
|---|---|
| Filter Panel | Status filter, date range, client filter, project type filter |
| Projects Table | Sortable columns: Project Name, Client, Status, Bid Amount, Billed, Owed, Completion Date |
| Export Button | Triggers a Power Automate flow (optional) that exports the filtered list to Excel in SharePoint |
| Invoice Table | Sortable invoice list with filters |

---

### Screen 10 — Settings (`SettingsScreen`)

**Purpose:** App configuration. Admin access recommended.

**Data sources:** AppSettings

**Components:**

| Component | Description |
|---|---|
| Settings Gallery/Form | Each AppSettings row displayed as a labeled input field |
| Save Button | Patch changes to AppSettings list |
| Current User Display | Shows the signed-in user (Office 365 Users connector) |
| About Section | App version, last publish date (from PublishInfo.json — display only) |

---

## Context Variable Reference

| Variable | Type | Set On | Used On |
|---|---|---|---|
| `SelectedProject` | Record | ProjectsScreen, PendingBidsScreen, SchedulingScreen, BillingScreen | ProjectWorkspaceScreen |
| `CurrentUser` | Record | App.OnStart | All screens (header, override forms) |
| `AppSettingsCollection` | Collection | App.OnStart (loaded from AppSettings list) | SettingsScreen, Flows |
| `NavSelectedItem` | Text | NavComponent | NavComponent highlight |
| `OverrideFormVisible` | Boolean | SchedulingScreen | Override form overlay |

---

## App.OnStart Formulas

```
// Load app settings into a collection
Collect(AppSettingsCollection, 'AppSettings');

// Set current user
Set(CurrentUser, Office365Users.MyProfileV2());

// Set default navigation context
Set(NavSelectedItem, "Dashboard");
```

---

## Component Library

| Component Name | Purpose | Used On |
|---|---|---|
| NavComponent | 9-item navigation sidebar | All screens |
| HeaderBar | App title, current user, back button | All screens |
| StatusPipelineComponent | Visual 13-step pipeline display | ProjectWorkspaceScreen Panel 2 |
| GateBadgeComponent | Colored gate/warning badge | SchedulingScreen, ProjectWorkspaceScreen Panel 12 |
| DocumentRowComponent | Standard document list row with open-file button | DocumentsScreen, ProjectWorkspaceScreen Panel 6 |
| InvoiceRowComponent | Invoice row with status badge | BillingScreen, ProjectWorkspaceScreen Panel 9 |
