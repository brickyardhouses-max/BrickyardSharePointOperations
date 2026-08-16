# Do Not Edit Files — Current Pass Rules

## Purpose

This document defines what must not be manually edited while executing small documentation and planning passes.

Current scope is incremental planning and docs updates, not app-source modification.

---

## Strict No-Edit During Current Pass

Do not manually edit Power Apps solution/app source files under:

- `FieldOps2/BrickyardSharePointOperations/canvasapps/**`
- `FieldOps2/BrickyardSharePointOperations/solutions/**`
- `FieldOps2/BrickyardSharePointOperations/publishers/**`

This includes (but is not limited to):
- `Src/*.pa.yaml`
- `Controls/*.json`
- `Header.json`
- `Properties.json`
- `Resources/PublishInfo.json`
- `References/*.json`
- `canvasapp.yml`
- `solution.yml`, `solutioncomponents.yml`, `rootcomponents.yml`, `missingdependencies.yml`

---

## Why

1. Current approved work is documentation + incremental planning alignment.
2. Hand-editing app source artifacts can corrupt solution metadata.
3. Power Apps changes must be made in Power Apps Studio when implementation is explicitly approved.

---

## Safe Files for This Pass

Only documentation files are in scope:
- `README.md`
- `docs/BRICKYARD-OPERATIONS-HUB-PLAN.md`
- `docs/BUILD-PHASES.md`
- `docs/APP-SCREEN-PLAN.md`
- `docs/SHAREPOINT-DATA-MODEL.md`
- `docs/POWER-AUTOMATE-FLOWS.md`
- `docs/DO-NOT-EDIT-FILES.md`

---

## Enforcement

Before finalizing this pass, verify:
1. No files under `FieldOps2/` changed.
2. No app YAML/source/solution files were modified.
3. No PR is created unless explicitly requested.
