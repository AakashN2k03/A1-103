# Schema Reference — Run/Upload Tables

Column-level documentation for the file run/upload tables, intended as context you can feed to an AI SQL assistant.

> **Note:** This is a *technical* schema reference (table and column names). It is **not** for the business-facing DMate Helper chat, whose knowledge articles (`app_docs/*.toon`) deliberately avoid table/column names.

---

## Table: `fpdm_cnf_file_run_upload`

**Purpose:** Parent record for each data file uploaded and run against a configured template during **Run / Upload mode**. One row is created per uploaded file (per version). It ties an uploaded file back to its template configuration and tracks the file's processing progress and publish state.

### Relationships

- **Parent:** `fpdm_cnf_files` — via `template_id` → `file_id` (`ON DELETE CASCADE`). Deleting the template config removes its uploads.
- **Children:**
  - `fpdm_cnf_file_run_upload_stats` — per-sheet record counts, via `upload_id`.
  - `fpdm_cnf_file_run_upload_ver` — stored file binary per version, via `upload_id`.
  - `fpdm_cnf_run_control` — per-table run / publish tracking, via `upload_id`.

### Columns

| Column | Type | Null / Default | Description |
|--------|------|----------------|-------------|
| `upload_id` | `INT`, auto-increment | PK, not null | Surrogate primary key. Unique identifier of a single uploaded-file run. Referenced by all child tables (`_stats`, `_ver`, `run_control`). |
| `template_id` | `VARCHAR(50)` | nullable, FK | The template/config this upload belongs to. Foreign key to `fpdm_cnf_files.file_id`. Identifies which template's mappings and validations the uploaded file was processed against. |
| `file_name` | `VARCHAR(255)` | nullable | Original name of the uploaded data file (e.g. the Excel/spreadsheet the user uploaded). |
| `version` | `VARCHAR(100)` | not null, default `'1.0'` | Version label of this upload. Increments when the same template's file is re-uploaded, allowing multiple upload versions to coexist. Mirrored in `fpdm_cnf_file_run_upload_ver.version`. |
| `publish_flag` | `VARCHAR(1)` | not null, default `'N'` | Publish state of this upload's data. **`'N'`** = uploaded/validated but not yet published to the target (default). **`'I'`** = data has been inserted/published into the target tables (set once all target tables for the file succeed). Downstream steps (publish-ready lookups, reconciliation) filter on `publish_flag = 'I'`. *(Note: the `'Y'`/`'R'` values seen elsewhere belong to `fpdm_cnf_run_control`, not this table.)* |
| `run_status` | `JSON` | nullable | Processing-progress snapshot written while the file is being run. Object with `progress_percent` (integer 0–100) and `status` (`"IN_PROGRESS"` until all sheets are processed, then `"COMPLETED"`). |
| `created_by` | `VARCHAR(100)` | nullable | Username of the person who performed the upload. |
| `created_on` | `DATETIME` | default `CURRENT_TIMESTAMP` | Timestamp when the upload row was created. |

### Notes

- `upload_id` is the join key to reach per-sheet counts (`fpdm_cnf_file_run_upload_stats`), the stored file bytes (`fpdm_cnf_file_run_upload_ver`), and run/publish tracking (`fpdm_cnf_run_control`).
- A "ready to publish" upload is one where `publish_flag = 'I'`; a not-yet-published upload is `publish_flag = 'N'`.
- `run_status` is progress metadata only — the authoritative record counts live in the `_stats` child table.

### DDL

```sql
CREATE TABLE fpdm_cnf_file_run_upload (
  upload_id    INT AUTO_INCREMENT PRIMARY KEY,
  template_id  VARCHAR(50),
  file_name    VARCHAR(255),
  version      VARCHAR(100) NOT NULL DEFAULT '1.0',
  publish_flag VARCHAR(1) NOT NULL DEFAULT 'N',
  run_status   JSON,
  created_by   VARCHAR(100),
  created_on   DATETIME DEFAULT CURRENT_TIMESTAMP,
  KEY (template_id),
  CONSTRAINT fk_file_run_upload_template_id
    FOREIGN KEY (template_id) REFERENCES fpdm_cnf_files (file_id)
    ON DELETE CASCADE ON UPDATE RESTRICT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_as_cs;
```

---

## Table: `fpdm_cnf_file_run_upload_stats`

**Purpose:** Per-sheet record-count statistics for an uploaded file run. Holds one row **per sheet** within an upload, giving a health check on how many records were read, passed/failed validation, are ready to publish, and were actually published. These counts drive the record-count summaries shown in Run/Upload and on the publish/monitoring views (usually `SUM`-aggregated across a file's sheets).

### Relationships

- **Parent:** `fpdm_cnf_file_run_upload` — via `upload_id` (`ON DELETE CASCADE`). Deleting the upload removes its stats rows.
- **Grain:** one row per (`upload_id`, `sheet_name`). A single upload with multiple sheets produces multiple rows here.

### Columns

| Column | Type | Null / Default | Description |
|--------|------|----------------|-------------|
| `id` | `INT`, auto-increment | PK, not null | Surrogate primary key of the stats row. |
| `upload_id` | `INT` | not null, FK | The upload run these counts belong to. Foreign key to `fpdm_cnf_file_run_upload.upload_id`. Join key back to the parent upload. |
| `sheet_name` | `VARCHAR(255)` | nullable | Name/alias of the sheet (dataset) within the uploaded file that this row counts. Excel-style multi-sheet files produce one row per sheet. |
| `rec_total` | `INT` | nullable | Total number of records read from the sheet. Aggregated as `total_records` (sum across sheets). |
| `rec_published` | `INT` | default `0` | Number of records actually published/inserted into the target tables. Updated after a successful publish. `not_published` is derived as `rec_total - rec_published`. |
| `rec_publish_ready` | `INT` | default `0` | Number of records that are ready to be published (validated and eligible, not yet published). Surfaced as `ready_to_publish`. |
| `rec_valid` | `INT` | nullable | Number of records that passed validation. Surfaced as `validated`. |
| `rec_invalid` | `INT` | nullable | Number of records that failed validation. Surfaced as `invalid`. |
| `last_updated_by` | `VARCHAR(100)` | nullable | Username that last wrote/updated these counts (e.g. whoever ran or published). |
| `last_updated_on` | `DATETIME` | default `CURRENT_TIMESTAMP` | Timestamp of the last update to this stats row. |

### Notes

- Grain is **per sheet**: to report totals for a whole upload, `SUM` each `rec_*` column grouped by `upload_id`.
- Canonical derived metrics (as used in the app):
  - `total_records` = `SUM(rec_total)`
  - `validated` = `rec_valid`
  - `invalid` = `rec_invalid`
  - `ready_to_publish` = `rec_publish_ready`
  - `published` = `rec_published`
  - `not_published` = `rec_total - rec_published`
- Lifecycle: `rec_total`, `rec_valid`, `rec_invalid`, `rec_publish_ready` are set during the upload/validation run; `rec_published` is filled in later, after data is published to the target.
- This is the **authoritative** source of record counts. The parent's `run_status` JSON is only progress metadata (percent / IN_PROGRESS / COMPLETED).

### DDL

```sql
CREATE TABLE fpdm_cnf_file_run_upload_stats (
  id                INT AUTO_INCREMENT PRIMARY KEY,
  upload_id         INT NOT NULL,
  sheet_name        VARCHAR(255),
  rec_total         INT,
  rec_published     INT DEFAULT 0,
  rec_publish_ready INT DEFAULT 0,
  rec_valid         INT,
  rec_invalid       INT,
  last_updated_by   VARCHAR(100),
  last_updated_on   DATETIME DEFAULT CURRENT_TIMESTAMP,
  KEY (upload_id),
  CONSTRAINT fk_file_run_upload_stats_upload_id
    FOREIGN KEY (upload_id) REFERENCES fpdm_cnf_file_run_upload (upload_id)
    ON DELETE CASCADE ON UPDATE RESTRICT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_as_cs;
```
