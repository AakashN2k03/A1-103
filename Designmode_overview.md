# Design Mode — Overview

## What it is

Design Mode is the **first stage and "Blueprint Layer"** of DMate / CloudDMate — a
data-migration tool that loads data into Oracle Cloud (FBDI / HCM). It is the
**configuration and metadata-definition layer** where users define *how* data should
be structured, validated, mapped, and transformed.

Crucially, **Design Mode does not process any data**. It only defines the rules that
later stages execute. Its output is three JSON blueprints (Source Metadata, Target
Metadata, Mapping) plus validation/hook config stored in the database.

### Where it sits in the flow

```
Design Mode  →  Run Mode (Upload)  →  Publish  →  Reconciliation & Audit
  (blueprint)     (real data)         (to Oracle)   (post-load compare)
```

Design Mode outputs are the **inputs** that every downstream mode executes against.

## What users do in Design Mode

- **Source configuration** — upload Excel / CSV / FBDI / Schema, or connect a DB
  source; identify sheets, extract columns, set masking, visibility, and primary keys.
- **Attribute & validation engine** — per-column rules via the *Column Level* drawer:
  mandatory/optional, data type (String, Number, Float, Int, Decimal, Date…), length,
  date format, lookup, grid, and constraint/regex (with an AI regex generator + voice
  dictation).
- **Mapping engine** — map source → target columns. Types: Direct, Default, Lookup,
  Merge, Aggregate, Case, Arithmetic, Null. Supports **Auto Mapping** and **Mapping
  Variants / Replication** (up to 10 variants per target column).
- **Transformation engine** — IF/ELSE conditions, Case rules, arithmetic, merge,
  aggregate, lookup, and ID generation (prefix + sequence at file/sheet/column scope).
- **Target configuration** — FBDI / custom output format, output file naming,
  reconciliation-header selection.
- **Metadata & mapping generator** — produces the 3 JSONs (downloadable + DB-stored).
- **Re-upload & Preserve JSON** — re-apply existing mappings on re-upload, or
  regenerate from scratch.

## Frontend (React)

Route `design-mode` is wired in [App.jsx](react/frontend/src/App.jsx#L388), gated by
`canAccess('Design Mode')`. It mounts `SourcePanel` (imported as `DesignModePOC`).

Main component tree lives under
[react/frontend/src/components/DesignMode/](react/frontend/src/components/DesignMode/):

| File | Responsibility |
|---|---|
| `Source/Components/SourcePanel.jsx` | **Main Design Mode page / orchestrator** (~2,600 lines) |
| `Source/Components/DBSource.jsx` | DB-source (query-based) configuration |
| `Target/Components/panel/TargetPanel.jsx` | Target structure + mapping orchestration (~3,800 lines) |
| `Target/Components/mapping/MappingComponent.jsx` | Per-column mapping UI |
| `Utils/Common/ColumnLevelDrawer.jsx` | Column-level validation drawer (~1,450 lines) |
| `AutoMappingEngine.tsx` | Auto source→target mapping engine |
| `api/designModeAPI.ts` | Upload + FBDI metadata extraction |

Transformation sub-drawers each live in their own `Target/Components/` folder:
`arithmetic/`, `case/`, `condition/`, `grid/`, `id/`, `lookup/`, `merge/` (merge +
aggregate), `replication/`, `sheetConfig/`, `fileProperties/`.

## Backend (FastAPI)

Design endpoints are split across three places (router prefix `/api/design`):

| File | Role |
|---|---|
| [routers/design_mode.py](react/backend/app/routers/design_mode.py) | xlsx/xlsm/FBDI parsing, `/process-file/*`, lookup validate, `ColumnSequencer` (unique physical column names) |
| [data_migration_app.py](react/backend/app/data_migration_app.py) | **Core persistence** — `save-template`, `post-mapping`, `get-mapping`, `source-query`, metadata publish/save |
| `routers/design_mode_new.py` | ⚠️ Present but **NOT wired in** — orphaned/legacy, not imported or mounted anywhere |

Related validation-config routers: `gridConfig.py`, `lovConfig.py`, `lookupConfig.py`,
`flex_value.py`.

## Data model

Schema in [react/backend/SQL/01b_ddl_create.sql](react/backend/SQL/01b_ddl_create.sql);
Pydantic models in `app/models/schemas.py`. Core tables:

- `fpdm_cnf_files` — uploaded source/target template files
- `fpdm_cnf_sheets` — sheets + column metadata JSON
- `fpdm_cnf_mapping` — mapping / transformation JSON
- `fpdm_cnf_src_query` — DB-source query definitions
- `fpdm_cnf_lookup_master` / `fpdm_cnf_lookup` / `fpdm_cnf_lov` — lookup / LOV data
- `fpdm_cnf_grid_master` / `fpdm_cnf_grid_column_detail` — grid (hierarchical) validation
- `fpdm_cnf_hook_validation` / `fpdm_cnf_hook_sheets` — custom hook SQL validations

## Relationship to other modes

- **Run Mode** (`run-mode`, `routers/run_mode.py`) — uploads real data, validates it
  against Design Mode's rules, edits, and versions. *AI Cleanse lives here, not in
  Design Mode.*
- **Publish** (`publish`, `routers/publish.py`) — transforms and pushes data to Oracle.
- **Reconciliation** (`components/Reconciliation/`) — post-publish comparison.
- **Setup Mode** (`setup-mode`) — users, roles, modules (RBAC).

