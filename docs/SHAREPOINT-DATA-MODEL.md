# SharePoint Data Model — Current-State Preservation

## Overview

This track preserves the existing working model:
- `Projects-Tracker-Primary` remains the active center
- existing SharePoint/OneDrive locations remain in use
- Power Apps remains the interface

No replacement database is introduced.

---

## Current Operational List (Primary)

### Projects-Tracker-Primary

This list remains the primary operational tracker and source of truth for current app passes.

Existing known fields in active use include:
- Title (project name)
- field_1 (site address)
- field_2 (agency/insurer)
- field_3 (case manager)
- field_4 (case manager email)
- field_5 (project type)
- field_6 through field_12 (financial values)
- field_13 (status text / workflow indicator in current state)
- field_14 (NOA received in current state)
- field_15 (completion date in current state)
- field_16 through field_19 (billing/payment values)
- field_20 (change order text/value field in current state)
- field_21 (notes)
- Attachments

---

## Storage Rules (Strict)

1. Do not create duplicate client/project records.
2. Do not create duplicate document storage.
3. Do not create new folders/libraries in this pass.
4. Continue using existing SharePoint and OneDrive file locations.
5. Continue using existing Attachments and existing file links where present.

---

## Pass-Scoped Guidance

For current passes, improvements should prioritize:
- clearer use of existing fields
- clearer document category handling in UI
- clearer scheduling readiness indicators using existing record values
- clearer billing/financial grouping using existing tracker fields

No schema expansion is required for this pass.

---

## Future Changes Policy

If future list or schema changes are needed, they must be:
1. explicitly approved,
2. documented in a dedicated pass,
3. validated for no duplication and no disruption to existing app behavior.
