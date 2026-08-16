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
