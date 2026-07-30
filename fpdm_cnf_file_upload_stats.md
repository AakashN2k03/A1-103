# Design Mode — Source Side: Complete Reference & Knowledge Base

> **What this document is.** An exhaustive, "inch-by-inch" description of the **Source** half of Design Mode in CloudDMate — the left panel and its three drawers (**File-level**, **Sheet-level**, **Column/Field-level**), plus every button, checkbox, dropdown, dialog, and the exact data each one produces.
>
> **Who it is for.** It is written to be fed to an AI assistant as a knowledge base so that a non-technical user can ask plain questions ("What does the *Mandatory* checkbox do?", "Why is the *Name* field greyed out?", "How do I make a column a primary key?", "What is a Hook?") and get a correct answer. Engineers can also use it — file paths and stored field names are included.
>
> **Companion doc:** [DESIGN_MODE_KNOWLEDGE_BASE.md](DESIGN_MODE_KNOWLEDGE_BASE.md) covers the whole module (source **and** target, transformations, backend, DB). This file zooms into the Source side only.

---

## Table of Contents

0. [How to read this document](#0-how-to-read-this-document)
1. [Plain-English overview](#1-plain-english-overview)
2. [Screen anatomy (the map)](#2-screen-anatomy-the-map)
3. [The header / toolbar](#3-the-header--toolbar)
4. [Uploading a source file](#4-uploading-a-source-file)
5. [The Source Files tree (file → sheet → field)](#5-the-source-files-tree-file--sheet--field)
6. [File‑level drawer — "Database Source Configuration"](#6-file-level-drawer--database-source-configuration)
7. [Sheet‑level drawer — "Source Sheet Configuration"](#7-sheet-level-drawer--source-sheet-configuration)
8. [Column/Field‑level drawer — "Source Attribute"](#8-columnfield-level-drawer--source-attribute)
9. [Add Column dialog](#9-add-column-dialog)
10. [Delete confirmation dialog](#10-delete-confirmation-dialog)
11. [Schema Configuration drawer (DB source)](#11-schema-configuration-drawer-db-source)
12. [What "Save" and "Delete" actually do](#12-what-save-and-delete-actually-do)
13. [The data the Source side produces](#13-the-data-the-source-side-produces)
14. [Master field reference table](#14-master-field-reference-table)
15. [Glossary (plain English)](#15-glossary-plain-english)
16. [FAQ — layman questions & answers](#16-faq--layman-questions--answers)

---

## 0. How to read this document

- **Bold field names** (e.g. **Mandatory**) are exactly what the user sees on screen.
- `code font` is an internal/stored name (what gets saved to the metadata JSON / database).
- "**Plain meaning**" = what it means for a normal user.
- "**Stores**" = what changes in the saved data when you use it.
- Source files: the whole Source panel is [SourcePanel.jsx](react/frontend/src/components/DesignMode/Source/Components/SourcePanel.jsx) (its React component is literally named `POC`). The three drawers are separate components (named in each section).

---

## 1. Plain-English overview

**Design Mode is where you teach the system what your data files look like** *before* any real data is loaded. The **Source** side answers one question: **"What does an incoming file look like?"**

You do three things on the Source side:

1. **Upload a sample file** (Excel, CSV, an Oracle FBDI template, a "business object" file) — or point at a **database query**. The system reads it and lists its **sheets** (tabs) and the **fields** (columns) inside each sheet.
2. **Describe each field** — its data type, whether it's required, whether it's a primary key, size limits, validation patterns, date formats, etc. This is done in the **Column‑level drawer**.
3. **Configure the file and each sheet** — number of header rows, separators, a data-validation "Hook", whether an email is required, database binding, etc.

When you press **Save** on a file, all of this becomes a **source template** stored in the system. Later, in **Run Mode**, real files are checked against this template; in **Target/mapping**, these source columns become the ingredients you map into the output file.

> **Key idea for the assistant:** Nothing on the Source side transforms or moves data. It only *describes* and *validates the shape of* source data. Transformations (lookups, CASE logic, arithmetic, etc.) happen on the **Target** side.

---

## 2. Screen anatomy (the map)

Design Mode is a two-panel screen. This document covers the **left (Source)** panel.

```
┌───────────────────────────── Design Template (header) ───────── [Refresh] ┐
│                                                                            │
│  ┌── SOURCE FILES (left, ~33%) ──────┐   ┌── TARGET + MAPPING (right) ──┐  │
│  │  [type ▼] [Upload]                │   │   (covered in the main KB)   │  │
│  │                                   │   │                              │  │
│  │  ▸ ☐ 📗 MyFile [ 1234 ]  💾  🗑    │   │                              │  │
│  │     ▸ ☐ Sheet1            🗑       │   │                              │  │
│  │        ┌─────────────┬──────┐     │   │                              │  │
│  │        │ ➕ Header    │ Mand │     │   │                              │  │
│  │        │ PersonNumber │  ☐   │     │   │                              │  │
│  │        │ FirstName    │  ☑   │     │   │                              │  │
│  │        └─────────────┴──────┘     │   │                              │  │
│  └───────────────────────────────────┘   └──────────────────────────────┘  │
└────────────────────────────────────────────────────────────────────────────┘
```

Three levels of hierarchy, each with its **own configuration drawer**:

| Level | You click… | Opens the drawer… | Section |
|------|------------|-------------------|---------|
| **File** | the file **name** (blue hyperlink) | **Database Source Configuration** (`DBSource`) | [§6](#6-file-level-drawer--database-source-configuration) |
| **Sheet** | the sheet **name** (blue hyperlink) | **Source Sheet Configuration** (`SourceSheetConfiguration`) | [§7](#7-sheet-level-drawer--source-sheet-configuration) |
| **Field/Column** | a field **name** (blue underline) | **Source Attribute** (`ColumnLevelDrawer`) | [§8](#8-columnfield-level-drawer--source-attribute) |

---

## 3. The header / toolbar

At the top of the whole Design Mode page:

- **Title:** "Design Template".
- **Refresh button** (top-right). **Plain meaning:** reloads everything from the server — re-fetches saved source templates, the file-type list, and the target file types. **Technical:** increments an internal `refreshKey`, which re-runs the data-loading effects (`fetchSourceMetadata`, `fetchSourceFileTypes`, `fetchTargetFileTypes`). Use it if you saved something elsewhere and want the panel to catch up, or if a list looks stale.

The Source panel's own header shows **"Source Files"** and the upload control (next section).

**Loading overlays** you may see:
- "Loading…" — fetching saved templates on open/refresh.
- "Uploading and processing file…" — during an upload/convert.
- "Saving source metadata…" — while a file's Save is in progress.

---

## 4. Uploading a source file

The upload control ([FileUpload.jsx](react/frontend/src/components/DesignMode/Utils/Common/FileUpload.jsx)) has two parts: a **file-type dropdown** and a **button**.

### 4.1 The file-type dropdown

Its options come from a configurable list (LOV `DESIGN_FILE_TYPES_SRC`). The type you pick controls **which file extensions are accepted** and **how the file is parsed**:

| Type (label) | Accepts | Plain meaning |
|---|---|---|
| **Excel** / **Excel Split** | `.xlsx`, `.xls` | A standard Excel workbook. "Split" variants are handled the same way at upload. |
| **CSV** | `.csv` | A comma-separated text file. |
| **FBDI** | `.xlsm` | An Oracle **File-Based Data Import** macro-enabled template (used for Oracle Cloud loads). Defaults to **4 header rows**. |
| **BussObj** (business object) | `.xlsx`, `.xls`, `.json` | A business-object export that also carries a **lookup type** per column (row 11 of the sheet). |
| **Schema** | (no file — uses a DB query) | Instead of uploading a file, you pick a **database query**; the system derives the columns from the query. Button reads **"Configure"** and opens the Schema drawer ([§11](#11-schema-configuration-drawer-db-source)). |
| **Fixed Width** | `.txt`, `.dat` | A fixed-width flat file. |

> If you pick a type but choose a file with the wrong extension (e.g. type = CSV but you pick an `.xlsx`), you get an error: *"You selected file type X, but 'file' is not a valid X file…"* This guard is `validateUploadFileType`.

### 4.2 The button

- Normally labelled **"Upload"** — opens the OS file picker.
- Labelled **"Configure"** when the type is **Schema** — opens the Schema drawer instead of a file picker.

### 4.3 What happens after you pick a file (step by step)

1. The extension is validated against the chosen type.
2. The raw file is sent to the backend (`/api/design/process-file/{ext}`), which returns a **normalised Excel workbook**.
3. The browser reads that workbook and interprets **fixed rows** as column metadata:

   | Row (in the sheet) | Becomes |
   |---|---|
   | Row 1 | column **name / header** |
   | Row 2 | **alias** |
   | Row 3 | **data type** (mapped to string / number / bigint / float / date) |
   | Row 4 | **size** |
   | Row 5 | **mandatory** |
   | Row 6 | **description** |
   | Row 7 | **data format** |
   | Row 8 | **view** (TRUE/FALSE) |
   | Row 9 | **unique** (TRUE/FALSE) |
   | Row 10 | **position** |
   | Row 11 | **lookup type** *(BussObj files only)* |

4. Each new sheet gets a **suffix** like `_1234` (a server-issued number) so multiple uploads never clash. The file's name also gets that suffix.
5. For **BussObj**, any lookup types found are validated; if a lookup has no values yet, the system triggers a background job (BIP) to populate it.
6. The file appears in the tree as **not yet saved** (it lives only in the browser until you press its Save button).

> **Layman note:** Uploading does **not** save anything permanently. It just loads the file's structure into the screen. You must press the file's **Save** (floppy) button to persist it. Unsaved files are "local only".

---

## 5. The Source Files tree (file → sheet → field)

Each uploaded/saved file is a collapsible row. Expand it to see its **sheets**; expand a sheet to see its **fields**.

### 5.1 File row — every element

From left to right:

1. **Chevron (▸/▾)** — expand/collapse the file to show its sheets.
2. **File-type icon** — visual hint of the source type: 📄 CSV, 🗄 (database) for Schema, `{ }` for BussObj, 📗 Excel otherwise.
3. **Checkbox (☐)** — **"include this file in mapping."**
   - **Plain meaning:** ticking it makes **all** of this file's columns available on the Target side to map from. Unticking removes them from the Target's source list.
   - Ticking a file auto-ticks all its sheets; unticking removes them.
   - **Technical:** drives `selectedSourceFilesForMap`/`selectedSourceSheetsForMap`, which filter `allSourceSheetsColumnsMap` (the list of columns the Target panel can see). If **nothing** is ticked, **everything** is exposed by default.
4. **File name** (blue, e.g. `MyFile [ 1234 ]`) — the `[ 1234 ]` is the auto suffix, shown prettily. **Clicking the name opens the File-level drawer** ([§6](#6-file-level-drawer--database-source-configuration)).
5. **Save button (💾 blue floppy)** — saves this file's **entire** metadata (all sheets, all columns, all your drawer edits) to the server as a source template. See [§12](#12-what-save-and-delete-actually-do).
6. **Delete button (🗑 red trash)** — deletes the whole file/template (asks for confirmation first).

### 5.2 Sheet row — every element

Under an expanded file, each sheet shows:

1. **Chevron** — expand/collapse to show fields.
2. **Checkbox** — include just this sheet's columns in mapping. If you tick every sheet of a file, the file's checkbox auto-ticks; untick them all and it unticks.
3. **Sheet name** (blue; shows the sheet's alias) — **clicking opens the Sheet-level drawer** ([§7](#7-sheet-level-drawer--source-sheet-configuration)).
4. **Delete button (🗑)** — deletes just this sheet (confirmation first).

### 5.3 Field row — every element

Under an expanded sheet is a small two-column table:

- **Header of the table:**
  - **➕ (plus) button** — **Add Column**: opens the Add Column dialog ([§9](#9-add-column-dialog)) to create a brand-new field on this sheet.
  - **"Header"** column title — lists field names.
  - **"Mand"** column title — the Mandatory checkboxes.
- **Each field row:**
  - **Field name** (blue underline, bold) — **clicking opens the Column-level drawer** ([§8](#8-columnfield-level-drawer--source-attribute)) for that field. A green ✅ briefly appears next to a field when its drawer is open.
  - **Mandatory checkbox** — a quick toggle for "is this field required?" (same value as the **Mandatory** checkbox inside the Column drawer). Ticking here marks the field mandatory without opening the drawer.

> **Which name is shown?** For a **freshly uploaded (unsaved)** file the tree shows the column's internal `name`; for a **saved** file it shows the `alias`. This is the `isLocalOnly` rule and it matches what the Column drawer's **Header** field shows.

---

## 6. File‑level drawer — "Database Source Configuration"

**Opens when:** you click a **file name** in the tree.
**Component:** [DBSource.jsx](react/frontend/src/components/DesignMode/Source/Components/DBSource.jsx).
**Title:** "Database Source Configuration" with the file name beneath it.

This drawer configures **file-wide** settings — things that apply to the whole file, not a single sheet or column.

| Control | Plain meaning | Details / stores |
|---|---|---|
| **Source Type** (dropdown + 🔄 refresh) | Where this file's data ultimately comes from (e.g. a database type). Options come from LOV `SOURCE_TYPE`. The 🔄 button reloads the list. | Stored as `DbSource.srcType`. |
| **Source Query** (dropdown) | If the chosen Source Type is a **query** type, pick the named SQL query that feeds this file. **Disabled** ("Not applicable for this source type") unless the selected type's meaning is `query`. Options come from `/api/design/source-query`. | Stored as `DbSource.queryId`. |
| **Email** (checkbox) | Marks that this file **requires an email** (e.g. notification/approval requirement in later processing). | Stored as `email_required` (file-level flag in the metadata JSON). |
| **Preserve JSON** (checkbox) | Keep a preserved copy of this file's JSON definition (used so a module can be rebuilt later). **Only visible to the super-user (user id = 1).** | Stored as `preserve` (only emitted on save for user id `'1'`). |
| **Save / Cancel** | Save writes these choices back into the file's in-memory metadata (they get persisted the next time you Save the file from the tree). | `onSave({ srcType, queryId, preserveJson, emailRequired })`. |

> **Note:** Saving in *this* drawer only updates the file's settings in the screen. The values are actually written to the server when you press the file's **💾 Save** button in the tree (which calls the save-template API).

---

## 7. Sheet‑level drawer — "Source Sheet Configuration"

**Opens when:** you click a **sheet name** in the tree.
**Component:** [SourceSheetConfiguration.jsx](react/frontend/src/components/DesignMode/Source/Components/SourceSheetConfiguration.jsx).
**Title:** "Source Sheet Configuration: *template / sheet*".

Configures how **one sheet** is read.

| Control | Plain meaning | Default / stores |
|---|---|---|
| **Number of Headers** | How many rows at the top of the sheet are header rows (labels), not data. | Default **1**; **4** for FBDI files. Stored as `NoOfHeaders`. |
| **Field Separator** | The character separating fields (mainly for delimited files). | Default `,`. Stored as `output_file_seperator`. |
| **Number of Trailers** | How many rows at the **bottom** are trailer/footer rows, not data. | Default **0**. Stored as `NoOfTrailers`. |
| **Data Validation** (dropdown) | Attach a **Hook** — a reusable validation/enrichment routine — to this sheet. "None" = no hook. Options come from `/api/get-hook`. | Stored as `Hook` (`{ hook_name, hook_items }`). |
| **👁 eye button** | Appears next to a chosen (non-CVR) hook; re-opens the Hook detail editor to view/edit its items. | — |
| **Save / Cancel** | Writes these settings into the sheet's metadata (persisted on the next file Save). | `onSaveConfig(localConfig)`. |

### 7.1 What a "Hook" is, and the Hook detail editor

- A **Hook** is a named, reusable **data-validation / enrichment routine** that runs against a sheet (e.g. cross-checks, look-ups, standardisation). You pick it by name.
- Choosing a hook (other than **CVR**) opens the **Hook detail editor** ([HookDetailModal.jsx](react/frontend/src/components/DesignMode/Utils/Common/HookDetailModal.jsx)) where each hook **item** can be given a SQL `SELECT` query and a resulting column/attribute. The editor derives a physical helper table name like `fpdm_hook_<item>_<suffix>` and offers the query's columns as choices.
- **CVR** is a special hook that needs **no item configuration** — selecting it just stores `Hook: { hook_name: "CVR" }`.
- Choosing "None" clears the hook (`hookName: '', hookConfig: null`).
- If you open the detail editor and close it **without** saving, your previous hook selection is restored (nothing is lost).

**Stored shape:** `Hook: { hook_name: "MyHook", hook_items: [ { item_name, updated_query, column, attribute }, … ] }`.

---

## 8. Column/Field‑level drawer — "Source Attribute"

**Opens when:** you click a **field name** in the tree.
**Component:** [ColumnLevelDrawer.jsx](react/frontend/src/components/DesignMode/Utils/Common/ColumnLevelDrawer.jsx) (internal name `SourceSRdrawer`).
**Title:** "Source Attribute : *sheet / column*".

This is the most detailed drawer. It defines **everything about one field**. (The same component is reused on the Target side; here `componentName = "source"`, so a few controls below are **source-only**.)

It slides in from the right and is grouped top-to-bottom as follows.

### 8.1 Identity — Header, Name, Description

| Field | Plain meaning | Editable? | Notes |
|---|---|---|---|
| **Header** | The **display name** of the field (what humans see). | ✏️ Editable | For an **unsaved** file this edits the internal `name`; for a **saved** file it edits the `alias`. (That's the `isLocalOnly` flip — see FAQ.) |
| **Name** | The **internal/technical name** of the field. | 🔒 Read-only (greyed) | It's the counterpart of Header. You can't edit it directly; it's derived/fixed. |
| **Description** | Free-text notes about the field. | ✏️ Editable | Stored as `description`. |

### 8.2 Keys — Mandatory, Unique, Primary ID

Checkboxes on the **"Keys"** row:

| Checkbox | Plain meaning | Rules |
|---|---|---|
| **Mandatory** | The field must have a value (cannot be null/empty). | Stored as `mandatory` (`true` / `"NOT NULL"`). If **Primary ID** is on, Mandatory is forced on and locked. |
| **Unique** | Every row must have a different value in this field. | Stored as `unique`. Forced on and locked when **Primary ID** is on. |
| **Primary ID** *(source only)* | This field is the sheet's **primary key** — the unique identifier used to **join** this sheet to others during mapping. | Stored as `primaryId`. Ticking it **auto-ticks Unique + Mandatory** and **un-ticks Primary ID on every other column in the sheet** (only one primary key per sheet). |

> **Why Primary ID matters:** on the Target side, when a target sheet pulls columns from **two different source sheets**, the system needs each source sheet's primary key to build the join. No primary key → the mapping reports an error. So mark exactly one Primary ID per source sheet that will be joined.

### 8.3 Options — Mask, View, Virtual, Scrub

Checkboxes on the **"Options"** row:

| Checkbox | Plain meaning | Notes |
|---|---|---|
| **Mask** | Hide/obscure the value (e.g. sensitive data shown as ****). | Stored as `mask`. |
| **View** | Whether the field is **visible** in the data grid (Run Mode). | Stored as `view`. Locked **on** when the field is both Mandatory and View (you can't hide a mandatory field you're already viewing). |
| **Virtual** *(source only)* | This field is **computed by an expression**, not read directly from the file. Ticking it reveals the **Virtual Expression** box. | Stored as `virtual` (+ `virtual_expression`). Unticking clears the expression. |
| **Scrub** | Mark the field for **data scrubbing/cleansing** (PII removal / cleanup). | Stored as `scrub`. |

### 8.4 Virtual Expression *(source only)*

Shown only when **Virtual** is ticked. A multi-line text box to describe the expression that computes this field. Stored as `virtual_expression`.

### 8.5 The "validation" box

A grey panel titled **"validation"** containing:

**Constraints (Referential Integrity)** — a **searchable dropdown**.
- **Plain meaning:** links this field to **another column in the same file** (e.g. this "Department Code" must exist in another sheet's "Department" list). It's a referential-integrity constraint.
- Only columns **from other sheets of the same file** appear (you cannot self-reference the same sheet). If none qualify, it says *"No constraints available… self-referencing the same sheet is not allowed."*
- Has a search box, "None" option, and a live match count.
- Stored as `referentialIntegrity` in `"sheet.column"` form.

**Lookup** — dropdown of lookup types (from the distinct lookup list).
- **Plain meaning:** bind this field to a **lookup/LOV** (a code list) so its value can be validated/translated.
- When a lookup type is chosen, a second dropdown **Lookup Column** appears with **None / Code / Meaning / Description** — which part of the lookup this field corresponds to. Stored as `lookup_type` and `lookup_column` (prefixed `lookup_…`).

**Grid** — dropdown of configured grids.
- **Plain meaning:** bind this field to a **grid** (a decode matrix). When a grid is chosen, **Grid Input** and **Grid Output** dropdowns appear (populated from the grid's input/output columns). Stored as `grid_name`, `grid_input`, `grid_output`.

**Data Type** — dropdown. Determines which validation section shows below.
- Options: **String, Number, BigInt, Float, Positive Int, Positive Decimal, Negative Int, Negative Decimal, Date**. Stored as `dataType`.

**Show Aggregate** — checkbox, appears **only for numeric types**.
- **Plain meaning:** allow this numeric field to be aggregated (summed/totalled) in views. Stored as `aggregate`.

### 8.6 String Validations (only when Data Type = String)

| Field | Plain meaning | Stores |
|---|---|---|
| **Min Length** | Minimum number of characters. | `validations.minLength` (default 1). |
| **Max Length** | Maximum number of characters. | `validations.maxLength` (defaults to the field's size). |
| **Pattern** | A validation **regular expression** the value must match. Dropdown: **None**, named patterns (from LOV `DESIGN_STRING_PATTERN`, e.g. "Email"), or **Custom**. | `validations.pattern` + `validations.patternType`. |
| **Custom pattern editor** (when Pattern = Custom) | Two tabs: **✎ Type** (type the regex yourself) and **✨ AI Generate** (describe it in words, e.g. "10-digit phone", and the system generates the regex via `/api/generate-regex`). Supports voice dictation. | — |
| **Padding** | Characters used to pad the value to length. | `validations.padding`. |

**Regex validation & the Save block:** a custom regex is checked instantly in the browser (JavaScript) and then confirmed **Python-compatible** on the backend (`/api/validate-pattern`).
- Invalid JS regex → red "Invalid regex pattern".
- Valid JS but not Python-compatible → orange "Incompatible with Python — pattern will fail at runtime".
- **Save is disabled until a custom pattern is confirmed Python-valid.**

### 8.7 Date Validations (only when Data Type = Date)

- **Date Format** — dropdown: **None**, `MM/dd/yyyy`, `dd/MM/yyyy`, `yyyy-MM-dd`, `MM-dd-yyyy`, `dd-MM-yyyy`, `yyyy/MM/dd`.
- **Plain meaning:** how dates in this field are formatted.
- Stored as `dateFormat`; on save it's converted to metadata tokens (`yyyy`→`%Y`, `MM`→`%m`, `dd`/`DD`→`%d`). "None" stores an empty string.

### 8.8 Delete field

- **Delete field** — a red-labelled checkbox at the bottom.
- **Plain meaning:** tick it and press **Save** to **remove this field** from the sheet.
- **Behaviour:** removes the column from the sheet everywhere in the screen. If the column already existed in a **saved** template it's also tracked in `DeletedColumns` so the backend drops it; if it was a newly-added unsaved column it's just discarded.

### 8.9 Footer & save rules

- **Cancel** — close without applying.
- **Save** — apply the edits to the field. **Disabled** while saving **or** when a custom regex isn't confirmed Python-valid.
- On save (`onSaveField`), the parent panel:
  - merges the field back into the sheet;
  - if you turned this into a **Primary ID**, removes primary from other columns;
  - if you changed the **data type** or **max length** of a column that was already saved, records it under `ModifiedColumns` (so the backend can alter the table);
  - keeps the tree label and the Mandatory checkbox in sync.

---

## 9. Add Column dialog

**Opens when:** you click the **➕** button in a sheet's field table.
**Component:** [ColumnAddition.jsx](react/frontend/src/components/DesignMode/Utils/Common/ColumnAddition.jsx).

A small centered dialog with a single **Column Name** text box.
- **Enter** = Save, **Escape** = Cancel.
- Creating a column adds a new field marked `isNew: true`, with sensible defaults (`dataType: "string"`, `size: 100`, not mandatory, not primary), and an auto-generated internal suffix (`_01a`, `_02a`, …) so names don't collide.
- Duplicate names are rejected with an error toast.
- The new field appears immediately in the tree and can be opened in the Column drawer to configure fully. It's persisted when you Save the file.

---

## 10. Delete confirmation dialog

**Opens when:** you click a file's or sheet's **🗑** button.
**Component:** [DeleteComponent.jsx](react/frontend/src/components/DesignMode/Utils/Common/DeleteComponent.jsx).

- Title **"Confirm Delete"**, asks *"Are you sure you want to delete X?"* with **Cancel** / **Delete** buttons (Delete shows a spinner while working).
- For a **saved source template** (file delete), it also fetches and shows a table of **Run Mode files that will be deleted** along with it (via `/api/get-file-uploads-by-template`) — a warning of downstream impact. If none exist / the call fails, it says so.
- **Cancel** aborts; **Delete** calls the confirm handler (delete file or delete sheet).

---

## 11. Schema Configuration drawer (DB source)

**Opens when:** the upload type is **Schema** and you press **Configure**.
**Component:** [SchemaDrawer.jsx](react/frontend/src/components/DesignMode/Utils/Common/SchemaDrawer.jsx).
**Title:** "Schema Configuration".

Used when the source is a **database query** rather than an uploaded file.

| Control | Plain meaning |
|---|---|
| **Schema Name** (dropdown) | Pick a named source query (from `/api/design/source-query`). If there's only one, it's auto-selected. |
| **Query** (viewer/editor) | Shows the SQL behind the selected schema. Read-only by default; an **Edit** toggle makes it editable. **Copy** button copies it; **Reset** reverts your edits. A "Modified" badge appears if you change it. Line/character counts are shown. |
| **Save** | If you edited the query, it's saved back via `PATCH /api/design/source-query` (`updated_query`). Then the drawer returns the `query_id`, and the system runs `/api/design/process-file/schema` to turn the query result into a **schema Excel**, which is parsed exactly like an uploaded file (same row-to-metadata mapping) and added to the tree. |
| **Cancel** | Close without generating a schema. |

---

## 12. What "Save" and "Delete" actually do

### Saving a file (💾 in the tree)

1. Gathers the file's metadata + all your in-memory drawer edits and builds the **save payload** (`buildSourcePayload`).
2. `POST /api/design/save-template` (with `user_id`, `module_id`, environment).
3. On success: the file is marked **saved**, the payload JSON is **downloaded** to your machine (a `*_source_metadata` file), and the "new/deleted/modified" change-tracking is cleared.
4. The backend inserts the file, its sheets and columns into the database and generates the physical tables; it also stores any sheet **Hook** config.

### Deleting

- **Saved template** → calls the delete API (`/api/metadata/file` for a file, `/api/design/sheet-del` for a sheet). A file delete warns about dependent Run Mode files.
- **Unsaved (local-only)** file/sheet → simply removed from the screen (nothing on the server to delete).

### The mandatory quick-toggle vs. the drawer

The **Mand** checkbox in the field table and the **Mandatory** checkbox in the Column drawer set the **same** value. Editing one keeps the other in sync.

---

## 13. The data the Source side produces

Everything you do on the Source side becomes one JSON object per file. Two forms matter:

### 13.1 In-memory shape (`json_dump`)

```jsonc
{
  "json_dump": {
    "file": {
      "template_name": "Worker_0858",
      "isLocalOnly": true,            // true until you press Save
      "DbSource": { "srcType": "", "queryId": null },
      "email_required": false,
      "originalFiletype": "excel"     // excel | csv | fbdi | bussobj | schema
    },
    "sheets": [{
      "name": "Worker_0858", "alias": "Worker",
      "columns": [{
        "name": "person_number", "alias": "PersonNumber",
        "dataType": "string", "size": 100,
        "mandatory": false, "primaryId": false, "unique": false,
        "view": true, "scrub": false, "mask": false,
        "output_position": 1, "source_position": 1,
        "referentialIntegrity": "", "lookup_type": "", "lookup_column": "",
        "grid_name": "", "grid_input": "", "grid_output": "",
        "virtual": false, "virtual_expression": "",
        "dataFormat": "",
        "validations": { /* see §8.6/§8.7 */ }
      }]
    }]
  }
}
```

### 13.2 Save payload (sent to `/api/design/save-template`)

```jsonc
{
  "file": {
    "id": "worker_0858", "type": "source", "module": "HCM",
    "template_name": "Worker_0858",
    "sheet_name": ["worker_0858"], "sheet_alias": ["Worker"],
    "DbSource": { "srcType": "", "queryId": null },
    "originalFiletype": "excel", "email_required": false,
    "uploadUser": "jdoe"
    // "preserve": true      // only for super-user (id '1')
  },
  "sheets": [{
    "name": "worker_0858", "alias": "Worker",
    "HeaderNames": ["PersonNumber"], "columnNames": ["person_number"],
    "NewColumns": [...], "DeletedColumns": [...], "ModifiedColumns": [...],  // change tracking (optional)
    "NoOfHeaders": 1, "output_file_seperator": ",", "NoOfTrailers": 0,
    "Hook": { "hook_name": "...", "hook_items": [...] },   // optional
    "columns": [{
      "name","alias","description","dataType","size",
      "mandatory","unique","primaryId","mask","view","scrub",
      "aggregate"/*numeric*/, "virtual_expression"/*if set*/,
      "referentialIntegrity","lookup_type","lookup_column",
      "grid_name","grid_input","grid_output",
      "output_position","source_position","include_in_output",
      "validations": {
        "pattern": null, "patternType": null,
        "minLength": 1, "maxLength": 100,       // string
        "minValue": null, "maxValue": null,     // numeric
        "dateFormat": "%Y/%m/%d",               // date (metadata tokens)
        "defaultValue": null, "allowedValues": [], "padding": ""
      }
    }],
    "totalColumns": 1
  }]
}
```

> `dateFormat` is written in metadata tokens (`%Y`, `%m`, `%d`), converted from the UI tokens (`yyyy`, `MM`, `dd`) at save time.

---

## 14. Master field reference table

Every per-column attribute, in one place:

| On-screen | Stored key | Type | Where edited | Plain meaning |
|---|---|---|---|---|
| Header | `name` (local) / `alias` (saved) | text | Column drawer | Display name |
| Name | `alias` (local) / `name` (saved) | text | (read-only) | Internal name |
| Description | `description` | text | Column drawer | Notes |
| Mandatory | `mandatory` | bool/`"NOT NULL"` | drawer + tree "Mand" | Value required |
| Unique | `unique` | bool | Column drawer | No duplicates |
| Primary ID | `primaryId` | bool | Column drawer (source) | Sheet's join key (one per sheet) |
| Mask | `mask` | bool | Column drawer | Obscure value |
| View | `view` | bool | Column drawer | Visible in grid |
| Virtual | `virtual` | bool | Column drawer (source) | Computed field |
| Virtual Expression | `virtual_expression` | text | Column drawer (source) | The formula |
| Scrub | `scrub` | bool | Column drawer | Cleanse/PII |
| Data Type | `dataType` | enum | Column drawer | string/number/bigint/float/±int/±decimal/date |
| Show Aggregate | `aggregate` | bool | Column drawer (numeric) | Allow totals |
| Position | `output_position` | number | Column drawer (target) | Output order |
| Constraints | `referentialIntegrity` | `"sheet.column"` | Column drawer | RI to another sheet's column |
| Lookup | `lookup_type` | text | Column drawer | Bind to a lookup/LOV |
| Lookup Column | `lookup_column` | code/meaning/description | Column drawer | Which lookup part |
| Grid / Input / Output | `grid_name` / `grid_input` / `grid_output` | text | Column drawer | Bind to a grid decode |
| Min Length | `validations.minLength` | number | Column drawer (string) | Min chars |
| Max Length | `validations.maxLength` | number | Column drawer (string) | Max chars |
| Pattern | `validations.pattern` (+`patternType`) | regex | Column drawer (string) | Regex validation |
| Padding | `validations.padding` | text | Column drawer (string) | Pad characters |
| Date Format | `validations.dateFormat` / `dateFormat` | token | Column drawer (date) | Date layout |
| Size | `size` | number | (from upload) | Field length |
| — | `NoOfHeaders`, `NoOfTrailers`, `output_file_seperator` | — | Sheet drawer | Header/trailer/sep |
| — | `Hook` | object | Sheet drawer | Sheet validation hook |
| — | `DbSource`, `email_required`, `preserve` | — | File drawer | File-wide settings |

---

## 15. Glossary (plain English)

- **Template / source file** — a saved definition of what an incoming file looks like.
- **Sheet** — a tab within a file (like an Excel worksheet). Each becomes a table.
- **Field / column / attribute** — one column within a sheet. (The drawer calls it "Attribute".)
- **Metadata** — the description of the data (names, types, rules) — *not* the data itself.
- **Local-only / unsaved** — a file loaded into the screen but not yet saved to the server.
- **Suffix (`_1234`)** — a number appended to sheet/file names so repeated uploads never clash.
- **Mandatory** — the field must have a value.
- **Unique** — no two rows may share the same value.
- **Primary ID / primary key** — the field that uniquely identifies a row and is used to join sheets.
- **Referential integrity / Constraint** — a rule that this field's value must exist in another sheet's column.
- **Lookup / LOV** — a code list used to validate or translate values.
- **Grid** — a decode matrix (inputs → output) used to translate values.
- **Hook** — a reusable validation/enrichment routine attached to a sheet.
- **Virtual field** — a field computed by an expression rather than read from the file.
- **Scrub** — cleanse/remove sensitive data.
- **Aggregate** — allow a numeric field to be totalled/summed.
- **FBDI** — Oracle's File-Based Data Import template (`.xlsm`, 4 header rows).
- **BussObj** — a business-object file type that carries a lookup type per column.
- **Schema source** — a source defined by a database query instead of an uploaded file.

---

## 16. FAQ — layman questions & answers

**Q: I uploaded a file but it disappeared after refresh. Why?**
A: Uploading only loads the file into the screen ("local-only"). You must press the **💾 Save** button on the file row to persist it. Refresh reloads only *saved* templates.

**Q: The "Name" field is greyed out and I can't type in it. Is that a bug?**
A: No. **Name** is the internal/technical name and is read-only. Edit the **Header** (display name) instead — for saved files that updates the alias, for new uploads it updates the internal name.

**Q: How do I make a column the primary key?**
A: Open the column (click its name), tick **Primary ID**. It automatically becomes Unique + Mandatory, and any other primary key in that sheet is cleared (only one per sheet).

**Q: Why did the mapping later complain about a missing primary key?**
A: When a target sheet combines columns from two source sheets, each source sheet needs a **Primary ID** so the join can be built. Go back to the source sheets and mark one Primary ID each.

**Q: What's the difference between the checkbox next to a file and the "Mandatory" checkbox?**
A: The **file/sheet checkbox** controls whether that file/sheet's columns are **available on the Target side for mapping**. The **Mandatory** checkbox controls whether a **field is required**. Different purposes.

**Q: I ticked a file's checkbox — what changed?**
A: You made all of that file's columns selectable as mapping sources on the right (Target) panel. If you tick nothing, everything is available by default.

**Q: What is a "Hook"?**
A: A named, reusable validation/enrichment routine you can attach to a sheet (in the Sheet drawer, under **Data Validation**). Most hooks open a detail editor to configure their items; the special **CVR** hook needs no configuration.

**Q: The Save button in the column drawer is greyed out.**
A: You've entered a **Custom** regex pattern that isn't valid Python. Fix the pattern (the drawer shows red/orange warnings) — Save unlocks once it's confirmed valid.

**Q: How do I add a column that isn't in the file?**
A: Expand the sheet and click the **➕** in the field table header, type a name, Save. Then click the new field to configure its type/rules.

**Q: How do I delete a field/sheet/file?**
A: Field → open it, tick **Delete field**, Save. Sheet/file → click the 🗑 next to it and confirm. Deleting a saved file also warns which Run Mode files will be removed.

**Q: What does "Virtual" mean?**
A: The field isn't read from the file directly — it's computed by an **expression** you provide (shown when you tick Virtual). Source-side only.

**Q: What is the "Schema" source type?**
A: Instead of uploading a file, you point at a **database query**. Press **Configure**, pick/edit the query, Save, and the system derives the columns from the query result.

**Q: Where does my saved data go?**
A: Into the database as a **source template** (files/sheets/columns), scoped to your module/org. A copy of the JSON payload is also downloaded to your computer when you save.

---

*Scope: Source side only. Read directly from `SourcePanel.jsx`, `DBSource.jsx`, `SourceSheetConfiguration.jsx`, `ColumnLevelDrawer.jsx`, `ColumnAddition.jsx`, `SchemaDrawer.jsx`, `FileUpload.jsx`, `DeleteComponent.jsx`, `sourceApi.js`, `sourceHelpers.jsx`, and `sharedHelpers.jsx` on the `dmate_dev` branch. For the target/mapping side, transformations, backend APIs and database tables, see [DESIGN_MODE_KNOWLEDGE_BASE.md](DESIGN_MODE_KNOWLEDGE_BASE.md).*
