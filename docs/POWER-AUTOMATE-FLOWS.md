# Power Automate Flows — Incremental Usage Policy

## Current Direction

For current improvement passes, the app work is focused on existing Field Ops 2 interface improvements.

No new flow architecture is required in this pass.

---

## Rules for This Stage

1. Do not change existing Power Automate flows unless explicitly approved.
2. Do not introduce new flow-triggered storage systems.
3. Do not create new folder/list structures from flows in this pass.
4. Keep document handling aligned to existing SharePoint/OneDrive locations.

---

## Practical Use in Near-Term Passes

Near-term app passes may:
- surface existing document categories more clearly in UI
- surface existing folder/file link actions if fields already exist
- keep uploads/attachments tied to the current record and existing storage

These improvements do not require creating new flows by default.

---

## Copilot Assistance Direction (Future Pass)

In a later approved pass, Copilot/automation can be used to:
- read uploaded documents,
- suggest tracker updates,
- help draft completion and billing paperwork,
- assist with file organization cues.

All suggestions must remain human-reviewed and manually confirmable before save.

---

## Change Control

Any new flow or flow behavior change must be:
1. explicitly approved,
2. added to a future pass plan,
3. validated against non-duplication and source-of-truth rules.
