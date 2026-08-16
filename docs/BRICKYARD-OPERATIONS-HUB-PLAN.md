# Brickyard Operations Hub — Master Plan (Incremental)

## Purpose

This plan defines how to improve the existing **Field Ops 2** app without rebuilding it.

This is an enhancement track, not a replacement project.

---

## Core Direction

1. Improve what already works in Field Ops 2.
2. Preserve **Projects-Tracker-Primary** as the operational center.
3. Preserve existing SharePoint and OneDrive file locations.
4. Keep existing app behavior stable during each pass.
5. Ship in small, safe, reversible passes.

---

## Non-Negotiable Rules

1. Do not rebuild Field Ops 2 from scratch.
2. Do not create duplicate client/project records.
3. Do not create duplicate document storage.
4. Do not create Dataverse tables as a replacement system.
5. Do not create new SharePoint lists/folders/libraries in this track unless explicitly approved.
6. Do not replace existing app structure unless explicitly approved.
7. Preserve existing selected-record behavior (select/edit/save/cancel/delete/attachments/notes).
8. SharePoint remains source of truth; app remains interface.

---

## Business Rules to Preserve

- Scheduling may be blocked only by missing **Contract** or missing **NOA** when required.
- **Consent To Start** is warning/informational only; it does not block scheduling.
- Financial fields remain internal and must not leak into client-facing outputs.
- Copilot suggestions require human review and manual confirmation.

---

## Current Working Base (Must Be Preserved)

- `RecordsGallery1` displays `Projects-Tracker-Primary`.
- `Form1` opens selected project/client record.
- Existing fields are editable and save back to SharePoint.
- Existing Notes field remains in use.
- Existing Attachments remain in use.
- Existing financial/accounting fields remain in use.

---

## Incremental Outcome

The target outcome is a better office/admin operating experience on the same app and same tracker:
- clearer project/client layout
- clearer document handling cues
- clearer scheduling readiness visibility
- clearer billing/payment tracking
- Copilot-assisted suggestions with manual approval

No replacement database, no duplicate storage, and no forced architecture migration.
