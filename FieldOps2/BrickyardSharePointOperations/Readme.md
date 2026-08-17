# Field Ops 2 — Brickyard SharePoint Operations

Power Platform solution containing the **Field Ops 2** canvas app, connected to the
`Projects-Tracker-Primary` SharePoint list.

## Repository layout

| Path | Purpose |
| --- | --- |
| `canvasapps/new_fieldops2_6938b/canvasapp.yml` | Canvas app metadata (connections, display name) |
| `.../new_fieldops2_6938b_DocumentUri/Src/MainScreen1.pa.yaml` | Human-readable screen source (review/edit) |
| `.../new_fieldops2_6938b_DocumentUri/Controls/4.json` | Packed screen definition — this is what ships |
| `.../new_fieldops2_6938b_DocumentUri/Controls/1.json` | Packed App object definition |
| `solutions/BrickyardSharePointOperations/` | Solution manifest and components |

## Standing rule: keep both app files in step

`Src/MainScreen1.pa.yaml` is the reviewable source, but the packer builds the app from
`Controls/4.json`. **Every change must update both files in the same commit.** A change made
only in the YAML looks correct in a pull request yet has no effect on the imported app — that
is what happened in build passes 1B and 1C, and it is why those passes appeared to do nothing.

For each control in the YAML there must be a matching entry in `Controls/4.json` with:

- the same `Parent` and the same property expressions (`Rules[].InvariantScript`);
- layout properties (`FillPortions`, `AlignInContainer`, `LayoutMin*`, `LayoutMax*`) under
  `DynamicProperties` whenever the control sits inside an auto-layout container;
- a `ControlUniqueId` that is unique across `Controls/1.json` and `Controls/4.json`;
- a `ZIndex` that does not collide with its siblings.

Gallery `OnSelect` and `TemplateFill` live on the gallery's `galleryTemplate` child in the
packed definition, not on the gallery control itself. A handful of style-derived properties
(container `Radius*`, text input `Size`/`BorderThickness`) appear only in the YAML because the
packed definition inherits them from the control style.

## Project lifecycle supported by the app

Pre-Estimate → Pending Bid → Awaiting NOA → Ready For Scheduling → Scheduled → Active →
Awaiting Signoff → Ready To Bill → Awaiting Payment → Paid → Warranty → Closed.

The form surfaces scheduling readiness (contract, NOA, consent to start), document guidance,
financial roll-ups and project folder links for the selected record.

## Features by build pass

| Area | What the app does |
| --- | --- |
| Project list | Search, sort (newest / stage / amount owed), a "needs attention" filter (no stage, missing NOA, or money owed), a record count, a stage-coloured accent bar and a one-line scheduling-readiness summary per row |
| Record header | Project title, stage chip tinted by lifecycle group, "Docs attached" / "No docs attached" badge, and a folder icon that opens the project's SharePoint document folder |
| Workflow card | Stage indicator ("Stage n of 12"), a compact readiness chip (Ready / Needs Attention / Blocked) with a one-line blocker reason, **Advance to &lt;next stage&gt;** (blocked with a short reason when readiness rules fail) and **Step back** |
| Financials | Total Expenses, Profit, Margin % and Currently Owed are calculated from their inputs; entered values are never overwritten on view — press **Use Calculated Totals** to apply them. Helper lines show a green tick when the entered value matches and amber text when it differs |
| Documents | Two-column document checklist, "Docs attached" / "No docs attached", the derived project folder path, **Open Project Folder** and **Open List Item** |
| Safety | Success notification and data refresh after save, seeded "No" defaults for NOA Received / Billed / Change Order on new records, and a delete that requires a typed reason |

### Scheduling readiness rule

Only a missing **Contract** or a missing **NOA** can block scheduling. **Consent To Start** is a
warning only and never blocks: the readiness chip drops to *Needs Attention*, but the record can
still advance.

### Visual language

The screen follows Microsoft Lists / SharePoint conventions: white cards, light grey separators
(`RGBA(237, 235, 233, 1)`), dark neutral text (`RGBA(50, 49, 48, 1)`) for headings and field
labels, and grey secondary text (`RGBA(96, 94, 92, 1)`) for supporting lines. Colour is reserved
for small tinted status chips — green for Ready, gold for Needs Attention, red for Blocked.
Brickyard red stays on the app header, the primary **Advance** action and genuine warnings.

### Folder link convention

The folder buttons open
`Shared Documents/Projects/<Project Title>` in the
`BrickyardHousesHomeSolutionsLLC` site. No SharePoint column was added for this — if you would
rather store an explicit folder URL per project, add a "Project Folder" column and the links can
read from it instead.

## Known limitations

- Status is still a free-text SharePoint column; the 12 stages are enforced in the app, and
  legacy values (for example "Won") are mapped onto the closest stage.
- Sorting and the needs-attention filter can raise delegation warnings on very large lists.
