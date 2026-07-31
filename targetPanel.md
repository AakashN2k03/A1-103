# Design Mode — Target Panel (Inch-by-Inch Guide)

A plain-English walkthrough of **everything on the Target side** of Design Mode: the
right-hand panel, the file/sheet tree, the mapping grid, and every drawer, modal, and
field a user can touch.

> **What the Target panel is for (in one line):** the Source side describes your *raw
> incoming* data; the Target side describes the *output* you want to produce (the Oracle
> FBDI/HCM file or API payload) and — most importantly — **how each output column is
> filled from the source**. That "how" is called a **mapping**. Nothing here moves data;
> it only defines the blueprint.

Main files:
[TargetPanel.jsx](react/frontend/src/components/DesignMode/Target/Components/panel/TargetPanel.jsx),
[MappingComponent.jsx](react/frontend/src/components/DesignMode/Target/Components/mapping/MappingComponent.jsx)

> **Heads-up on layout:** the Target panel is the right-hand section that sits next to the
> Source panel on the same screen. Everything below lives inside it.

---

## 1. The panel header

At the top of the panel sits the title **"Target Files"** and an **upload bar** — the same
[FileUpload](react/frontend/src/components/DesignMode/Utils/Common/FileUpload.jsx) widget
used on the Source side:

| Control | What it does |
|---|---|
| **File-type dropdown** | Pick the kind of output template (Excel, CSV, FBDI, Schema, etc.). Decides which file extensions are accepted and how the file is read. |
| **Upload / Configure button** | Normally says **Upload** and opens your file picker. If the type is **Schema**, it says **Configure** and opens the Schema drawer (columns come from a DB query instead of a file). |

After you pick a file it's sent to the server, converted to a standard structure, and read
back as sheets and columns — each sheet gets a unique suffix so uploads never clash. An
"Uploading and processing file…" spinner shows during this.

---

## 2. The file tree

Below the upload bar is the list of target files as a three-level tree: **File → Sheet →
Mapping grid**. Empty state reads *"No files uploaded."*

### File row (top level)

| Element | Meaning |
|---|---|
| **▶ / ▼ arrow** | Expand/collapse the file to show its sheets. |
| **File icon** | Shows the file type at a glance (Excel, CSV, schema, etc.). |
| **File name (blue link)** | Click to open the **Publish File Configuration** drawer (section 8) — cloud integration, publish type, process method, etc. |
| **💾 Save icon** | Saves *all* sheets/columns of this file **and** its mappings to the server. It also downloads two JSON files: `<file>_target_metadata.json` and `<file>_mapping.json`. |
| **🗑 Delete icon** | Deletes the whole target file (confirmation popup first). |

### Sheet row (middle level)

Expand a file and each sheet is a lighter bar with several controls on the right:

| Element | Meaning |
|---|---|
| **▶ / ▼ arrow** | Expand to reveal the mapping grid for the sheet. |
| **Sheet name (blue link)** | Click to open the **Publish Sheet Configuration** drawer (section 7). |
| **Mapping-variant radios** (**Map1 / map2 / map3 …**) | *Only appear when a sheet has 2+ mapping variants.* Switch which mapping "version" the grid below is showing/editing. See **Replication** (section 6). |
| **＋ replicate button** | Opens the **Replicate Mapping Configuration** drawer (section 6) to create extra rule-driven mapping variants. |
| **Auto Map button** | Auto-matches this sheet's target columns to source fields by name. If mappings already exist, a confirmation popup warns it will overwrite them. |
| **🗑 Delete icon** | Deletes just this sheet (confirmation popup). |

---

## 3. The mapping grid (the heart of the Target panel)

Expand a sheet and you get a table — one row per **target column**. Built by
[MappingComponent.jsx](react/frontend/src/components/DesignMode/Target/Components/mapping/MappingComponent.jsx).
This is where you say *where each output column's value comes from*.

Columns of the grid, left to right:

| Column | What it does (plain English) |
|---|---|
| **Header** (with a **＋** button) | The target column's name, shown as a blue link. Click it to open the **Target Attribute** drawer (section 4). The **＋** at the top-left adds a brand-new column via the **Add Column** popup (section 9). |
| **Mand** | Checkbox — is this output column mandatory? |
| **View** | Checkbox — is this column visible in the grid? Locked on when the column is both mandatory and currently visible. |
| **Null** | Checkbox — "leave this column empty." Ticking it wipes any mapping, default, and condition for the column in one go. It's shown ticked automatically whenever the column has no mapping, no default, and no condition. |
| **Mapping (1-1)** | The core control: a **searchable dropdown of every available source field**. Pick one to say "fill this output column straight from that source column." Has a search box, a match counter, and remembers a previously-saved value even if it's not in the current list. Picking **"Select source field"** clears it. |
| **Default** | A text box for a fixed fallback value to use when there's no mapped source value. |
| **Cond** | A toggle. Flipping it opens the **Condition / Transformation** drawer (section 5) where you build more complex logic (lookups, IF/THEN, math, etc.) instead of a plain 1-to-1 mapping. |

So each target column is filled by **one** of: a direct **Mapping**, a **Default**, or a
**Condition/transformation** — or left **Null**.

---

## 4. The Target Attribute drawer

Opens when you click a **target column name**. It's the *same* drawer component as the
Source side ([ColumnLevelDrawer.jsx](react/frontend/src/components/DesignMode/Utils/Common/ColumnLevelDrawer.jsx)),
but in "target" mode. Title: **"Target Attribute : <sheet> / <column>"**. It defines what
the output column *is* and any validation it must satisfy.

**Same fields as the Source Attribute drawer**, with a few target-specific differences:

| Field / group | Notes for Target |
|---|---|
| **Header** | Editable display name. |
| **Name** | Read-only technical name. |
| **Description** | Free-text note. |
| **Keys** | **Mandatory** and **Unique** checkboxes. *(No "Primary ID" on the target side — that's Source-only.)* |
| **Options** | **Mask**, **View**, **Scrub** checkboxes. *(No "Virtual"/Virtual Expression — that's Source-only.)* |
| **Position** | *Target-only field.* The column's output position/order number. |
| **Constraints** | Referential-integrity dropdown — but here it lists **other target columns** (not source). |
| **Lookup / Lookup Column** | Same as Source: attach a lookup and choose Code/Meaning/Description. |
| **Grid / Grid Input / Grid Output** | Same as Source: attach a validation grid. |
| **Data Type** | String, Number, BigInt, Float, Positive/Negative Int/Decimal, Date. |
| **Show Aggregate** | Numeric types only. |
| **String Validations** | Min Length, Max Length, Pattern (list / **Custom** with the ✎ Type + ✨ AI Generate tabs and voice input), Padding. |
| **Date Validations** | Date Format. |
| **Delete field** | Red checkbox to remove the column on Save. |
| **Cancel / Save** | Save is blocked if a custom regex is invalid. |

*(For the full field-by-field breakdown of these shared controls, see the Source Panel
guide, section 3.)*

---

## 5. The Condition / Transformation drawer

Opens from the **Cond** toggle in the mapping grid. This is where a target column gets
*logic* instead of a plain mapping. Built by
[ConditionMappingDrawer.jsx](react/frontend/src/components/DesignMode/Target/Components/condition/ConditionMappingDrawer.jsx).
Header shows **"<sheet> / <column>"**.

| Element | Meaning |
|---|---|
| **Current Configuration:** | A read-only line showing which transformation(s) are currently active, or **None**. |
| **Condition Type dropdown** | Picks *one* transformation to apply. Choosing a new one clears the others (only one type is saved at a time). Options below. |
| **Cancel / Save** | Save is disabled until the chosen transformation is valid. |

The **Condition Type** options (label → what it does):

| Option | Plain English |
|---|---|
| **None** | Clears all transformation config for this column. |
| **ID** | Auto-generate sequential IDs (section 5a). |
| **Merge and Split** | Join fields together, or extract a substring (section 5b). |
| **Lookup** | Translate a value via a lookup table (section 5c). |
| **Grid** | Run source values through a rules grid to get an output (section 5d). |
| **Misc** | Balance (Credit/Debit), Current Date, or Percentage (section 5e). |
| **Switch** | IF / THEN / ELSE (CASE) logic (section 5f). |
| **Arithmetic Expression** | Build a math formula from columns (section 5g). |

Whichever you pick, its own panel appears below. None of these sub-panels have their own
Save button — they feed the drawer's single **Save**.

### 5a. ID
Auto-generates sequential IDs (e.g. `INV-1, INV-2 …`).
| Field | Meaning |
|---|---|
| **Scope** | Where the counter resets: **File**, **Sheet**, or **Column**. Required. |
| **Prefix (Optional)** | Text placed in front of each ID. *(Appears once a scope is chosen.)* |
| **Starting Sequence Number** | Where numbering begins. Default 1, whole numbers ≥ 1 only. |

### 5b. Merge and Split
Combine text from fields, or pull a piece out of one field. Starts with an **Operation**
dropdown: **None**, **Concat**, or **Substring**.

*If Concat:* **Prefix**, **Source** (dropdown — each pick adds a field to the chain),
**Separator** (inserted between fields), **Suffix**, and an **Added Fields** preview
(prefix = orange chip, fields = blue chips with a remove ✕, suffix = green chip).

*If Substring:* **Token** (which token to grab, whole number ≥ 1), **Separator** (how to
split), **Source** (single field only), **Start Position**, **No. of Characters**.

### 5c. Lookup
Translate a source value through a lookup table (e.g. code → name).
| Field | Meaning |
|---|---|
| **Lookup Type** | Which lookup table to use. |
| **Value** button | Opens a read-only **Lookup Configuration** table to browse the lookup's values. |
| **Lookup Field** (radios **code / meaning / description** + a field dropdown) | Which part of the lookup drives the match. |
| **Return** | What the lookup gives back: **Code / Value / Meaning / Description**. |
| **Default** (radios **Column / Text**) | Fallback when nothing matches — either another column or a typed value. |

### 5d. Grid
Feed source fields into a pre-built rules grid and read back a computed output.
| Field | Meaning |
|---|---|
| **Grid Name** | Which grid/rules table to use. |
| **Value** button | Opens a read-only **DataGrid Configuration** view of the grid. *(Appears after a grid is chosen.)* |
| **Grid Input** rows | One row per grid input: a checkbox + the grid input's name + a **source-field dropdown** to feed it. |
| **Output Column** | Which grid output column to return (auto-selects if there's only one). |

### 5e. Misc
A grab-bag of special transforms. First a main dropdown: **Balance Options**, **Current
Date**, or **Percentage**.
- **Balance Options** → a **Balance Type** (Credit / Debit) + a numeric **source column**.
- **Current Date** → stamps today's date. *(No extra control is shown; it silently uses a default date format.)*
- **Percentage** → a numeric **source** + a **percentage value** (0–100, shown with a `%` sign).

### 5f. Switch (IF / THEN / ELSE)
Build CASE logic. One or more **Condition** blocks plus an **ELSE** fallback.

Per **Condition N** block:
| Field | Meaning |
|---|---|
| **IF** | Source column to test. |
| **Operator** | `=`, `!=`, `>`, `>=`, `<`, `<=`, `IS NULL`, `IS NOT NULL`. |
| **Value** (**Column / Text**) | What to compare against — hidden for `IS NULL`/`IS NOT NULL`. |
| **THEN** (**Column / Text**, plus **Prefix** and **Suffix**) | The output when the condition matches. |
| **Add Condition** button | Adds another condition block. Each block has a remove (✕). |

The **ELSE** block (appears once a complete condition exists) sets the fallback output —
same **Column / Text** + **Prefix** / **Suffix** controls.

### 5g. Arithmetic Expression
Build a math formula like `A + B - C`.
| Element | Meaning |
|---|---|
| **Select Column** | Add a source column to the formula. |
| **Select Operator** | Join with `+`, `-`, `*`, `/`, or `%`. |
| **Added Fields** preview | Shows the formula as chips; each column chip has a remove ✕. A hint line walks you through the column → operator → column sequence. |

---

## 6. The Replicate Mapping Configuration drawer (mapping variants)

Opens from the **＋ replicate** button on a sheet row. Lets you create **up to 10**
mapping variants of a sheet, each gated by an IF-condition, so incoming rows are routed to
different mappings (e.g. "IF Country = 'US' use this mapping"). Built by
[MappingReplicationDrawer.jsx](react/frontend/src/components/DesignMode/Target/Components/replication/MappingReplicationDrawer.jsx).
Title: **"Replicate Mapping Configuration : <file> / <sheet>"**.

| Field | Meaning |
|---|---|
| **Number of Replications** | How many variant rows to create (1–10). Adding rows adds conditions; reducing trims them. |
| Per row: **Column** | The source column the condition tests. |
| Per row: **Operator** | `=`, `!=`, `>`, `<`, `>=`, `<=`; plus **startsWith / endsWith / contains** when the column is text. |
| Per row: **Value Type** (**Column / Text**) | Compare against another column or a typed literal. |
| Per row: **Value** | The comparison value (dropdown or text box, per Value Type). |
| Per row: **Clear** / **Remove** | Reset that condition, or delete the row. |
| **Cancel / Save** | Save is disabled until every row is either fully empty or fully filled. |

Once saved, the **Map1 / map2 / …** radio buttons appear on the sheet row so you can edit
each variant's grid.

---

## 7. The Publish Sheet Configuration drawer

Opens when you click a **sheet name**. Controls how this sheet is written to the published
output file. Built by
[TargetSheetConfigurationDrawer.jsx](react/frontend/src/components/DesignMode/Target/Components/sheetConfig/TargetSheetConfigurationDrawer.jsx).
Title: **"Publish Sheet Configurations : <file>/<sheet>"**.

| Field | Plain English |
|---|---|
| **Output File** *(required)* | Which output file name this sheet writes into (from a server list). |
| **Include Header** (checkbox) | Whether to write a header row (**Yes/No**). For HCM it defaults to Yes. Ticking it reveals **Header Separator** below. |
| **Header Separator** | *Only when Include Header is on.* The character separating header names; auto-filled from the file separator. |
| **File Separator** *(required)* | The character separating data values in the output. |
| **End of Line** | Line-ending style for the output (or **None**). |
| **Data Validation** | Attach a validation **hook** (or **None**). Non-CVR hooks open the Data Validation Configuration modal (the same hook modal described in the Source guide); a 👁 eye button re-opens it. CVR hooks store directly. |
| **Recon Header** | Which column acts as the reconciliation key/header for later matching (or none if none available). |
| **Cancel / Save** | Save validates the required fields first. |

---

## 8. The Publish File Configuration drawer

Opens when you click a **file name**. Sets file-wide publishing properties. Built by
[FilePropertiesDrawer.jsx](react/frontend/src/components/DesignMode/Target/Components/fileProperties/FilePropertiesDrawer.jsx).
Title: **"Publish File Configuration : <file>"**.

| Field | Plain English |
|---|---|
| **Integration** *(required)* | Which Oracle cloud module/integration this file publishes to. |
| **Publish Type** *(required)* | The output packaging/type (e.g. zip). |
| **Publish File Name** *(required)* | The published file's name (filtered by the chosen Integration). |
| **Process Method** | How the data is loaded — **FBDI**, **API**, or **Auto** — shown as three checkboxes that behave like a single choice. Each is enabled only if the chosen integration supports it (FBDI→Oracle, API→API, Auto→both). |
| **Email** (checkbox) | File-level "email required" flag. |
| **API Field Mapping** | *Only shown when Process Method is API or Auto.* A status box (**Not configured** / **X of Y fields mapped**, with a **Complete** badge when done) plus an ✏️ edit button that opens the **API Field Mapping** modal (section 8a). |
| **Preserve JSON** (checkbox) | *Admin user (id 1) only.* Keep existing mappings when re-uploading instead of regenerating. |
| **Cancel / Save** | Save validates the required fields first. |

### 8a. The API Field Mapping modal
Opens from the ✏️ button above. Maps each field the cloud module's API expects to one of
your target columns. Built by
[ApiFieldMappingModal.jsx](react/frontend/src/components/DesignMode/Utils/Common/ApiFieldMappingModal.jsx).
Title: **"API Field Mapping"**.

| Element | Meaning |
|---|---|
| **Search box** | Filter the API fields by name. |
| **AutoMap** button | Auto-match API fields to target columns by name (won't overwrite existing picks). |
| **X / Y mapped** counter | How many API fields are mapped. |
| **Mapping table** (**API Field** → **Target Column**) | One row per API field; the right side is a dropdown of your target columns. |
| **Cancel / Save** | Save stores the mapping (unmapped fields saved as empty). |

---

## 9. The Add Column popup

Opens from the **＋** button at the top-left of a mapping grid. Same component as the
Source side. Type a **Column Name** and Save to add a new target column (a unique internal
name/position is generated automatically). Duplicate names are rejected; Save is disabled
until you type a name.

---

## 10. Confirmation & warning dialogs

| Dialog | When it shows | What it says |
|---|---|---|
| **Overwrite AutoMap?** | You click **Auto Map** on a sheet that already has mappings. | "This sheet already has mappings. Re-running Auto Map will overwrite them. Continue?" — **OK / Cancel**. |
| **Unsaved Source File(s)** | You try to save a target file while its source file(s) aren't saved yet. | Lists the unsaved source file(s) and asks you to save them first — a single **Close** button. |
| **Delete confirmation** | You delete a file or sheet. | Standard confirm popup before anything is removed. |

---

## Quick recap — the whole Target flow

1. **Upload a target template** (or **Configure** a schema), pick its file type.
2. The file appears as **File → Sheets → mapping grid**.
3. Expand a sheet → for each target column, either **map** it to a source field, type a
   **Default**, flip **Cond** to build a transformation, or tick **Null** to leave it empty.
4. Use **Auto Map** to fill obvious 1-to-1 matches in one click.
5. Click a **column name** → fine-tune its type and validations in the **Target Attribute** drawer.
6. Click a **sheet name** → set output file name, header/separator/EOL, hook, and recon header.
7. Click a **file name** → set the cloud integration, publish type/name, process method
   (FBDI/API/Auto), and — for API — the **API Field Mapping**.
8. Optionally use **Replicate** to create rule-based mapping **variants** (Map1/map2/…).
9. Hit the **💾 Save** icon → persists metadata + mappings and downloads the
   `*_target_metadata.json` and `*_mapping.json` files.

> **Reminder:** the target can only map from source files/sheets you've **ticked** on the
> Source side, and you must **save the source first** — otherwise the Unsaved Source
> warning blocks the target save.
