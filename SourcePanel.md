# Design Mode — Source Panel (Inch-by-Inch Guide)

A plain-English walkthrough of **everything on the Source side** of Design Mode: the
left panel, the file/sheet/column tree, and every drawer and field a user can touch.

> **What the Source panel is for (in one line):** it's where you upload (or connect to)
> the file that holds your *raw incoming data*, then describe each sheet and each column
> — its name, type, rules, and validations — so the rest of the tool knows how to read
> and check that data. Nothing here moves data; it only *describes* it.

Main file: [SourcePanel.jsx](react/frontend/src/components/DesignMode/Source/Components/SourcePanel.jsx)

---

## 1. The top bar

At the very top sits a header labelled **"Design Template"** with one button:

| Control | What it does (plain English) |
|---|---|
| **Refresh** button | Re-loads everything from the server — the file list, file types, and all saved metadata. Use it if something looks stale. |

---

## 2. The Source Files panel (left column)

This is the tall panel on the left titled **"Source Files"**. It has two parts: an
**upload bar** at the top and a **file tree** below it.

### 2a. The upload bar

Built by [FileUpload.jsx](react/frontend/src/components/DesignMode/Utils/Common/FileUpload.jsx).
Two controls sit side by side:

| Control | What it does |
|---|---|
| **File-type dropdown** | Pick what *kind* of source you're adding. Options come from the server and typically include **Excel, Excel Split, CSV, FBDI, Fixed Width, Schema, BussObj**. Your choice decides which file extensions the picker will accept and how the file is read. |
| **Upload / Configure button** | For normal file types it says **Upload** and opens your computer's file picker. If you chose **Schema**, it says **Configure** and opens the Schema drawer instead (schema pulls columns from a database query, not a file). |

**What happens after you pick a file:** the app checks the extension matches the chosen
type, sends the file to the server to be converted into a standard structure, then reads
back the sheets and columns. Each sheet gets a unique suffix (e.g. `_0858`) so two
uploads never clash. A row-count is noted, and for **BussObj** files any lookup types
found are validated against the server. A "Uploading and processing file…" spinner shows
during this.

### 2b. The file tree

Below the upload bar is the list of everything you've added. It's a three-level tree:
**File → Sheet → Columns**. If nothing's there yet it just says *"No files uploaded."*

#### File row (top level)

Each file is a grey bar. Left to right:

| Element | Meaning |
|---|---|
| **▶ / ▼ arrow** | Expand or collapse the file to show its sheets. |
| **File icon** | Tells you the type at a glance — CSV, database (schema), `{}` (BussObj), or Excel. |
| **Checkbox** | "Use this whole file for mapping." Ticking a file auto-ticks all its sheets. This is how you tell the *Target* side which source columns are available to map from. |
| **File name (blue link)** | Click it to open the **Database Source Configuration** drawer (section 5) — where you attach a DB source, mark email-required, etc. |
| **💾 Save icon** | Saves *all* sheets and columns of this file's metadata to the server, and downloads a copy of the saved JSON. |
| **🗑 Delete icon** | Deletes the whole file. If it was already saved to the server it's removed there too; you'll get a confirmation popup first. |

#### Sheet row (middle level)

Expand a file and each sheet appears as a lighter bar:

| Element | Meaning |
|---|---|
| **▶ / ▼ arrow** | Expand to show the sheet's columns. |
| **Checkbox** | "Use this sheet for mapping." Ticking every sheet auto-ticks the file; unticking them all unticks the file. |
| **Sheet name (blue link)** | Click to open the **Source Sheet Configuration** drawer (section 4). |
| **🗑 Delete icon** | Deletes just this sheet (with a confirmation popup). |

#### Column table (bottom level)

Expand a sheet and you get a small two-column table of its fields:

| Column header | What's in it |
|---|---|
| **Header** (with a **＋** button) | Every column's display name, shown as a blue link. Click a name to open the **Source Attribute** drawer (section 3) where all the real detail lives. The **＋** button at the top opens the **Add Column** popup (section 6). |
| **Mand** | A checkbox marking the column as **Mandatory** (data must be present). You can flip it right here without opening the drawer. |

If a sheet has no columns it says *"No fields found. Please check if the file contains data."*

---

## 3. The Source Attribute drawer (the big one)

Opens when you click a **column name**. This is the heart of the Source panel — where you
define what a single column *is* and every rule it must obey. Built by
[ColumnLevelDrawer.jsx](react/frontend/src/components/DesignMode/Utils/Common/ColumnLevelDrawer.jsx).
It slides in from the right. Title reads **"Source Attribute : <sheet> / <column>"**.

### Top section — identity

| Field | Plain English |
|---|---|
| **Header** | The human-friendly display name of the column (what people see). Editable. |
| **Name** | The internal/technical column name. **Read-only** — greyed out, can't be changed. |
| **Description** | A free-text note about what this column means. |

### Keys (checkboxes)

| Checkbox | Meaning |
|---|---|
| **Mandatory** | Data is required for this column. |
| **Unique** | Every value in this column must be different (no duplicates). |
| **Primary ID** | This column is the sheet's unique identifier (like a primary key). **Ticking this auto-ticks Mandatory and Unique and locks them** — a primary key can't be optional or non-unique. Only one column per sheet can be Primary ID; setting a new one clears the old. |

### Options (checkboxes)

| Checkbox | Meaning |
|---|---|
| **Mask** | Hide/obscure this column's values (for sensitive data). |
| **View** | Whether this column is visible in the grid. If a column is both Mandatory *and* currently visible, View is locked on (you can't hide a required field). |
| **Virtual** | Marks the column as *calculated* rather than coming straight from the file. Ticking it reveals the **Virtual Expression** box below. Unticking clears that expression. |
| **Scrub** | Flags the column for data cleansing/scrubbing. |

- **Virtual Expression** (only shows when **Virtual** is ticked): a multi-line box where
  you describe how the column's value should be computed.

### Validation box

A grey panel titled **"validation"** grouping the rule fields:

| Field | Plain English |
|---|---|
| **Constraints** | A searchable dropdown to link this column to *another column in the same file* (referential integrity — e.g. "this must match a value that exists over there"). It only lists columns from **other sheets of the same file** (you can't self-reference the same sheet). Has a live search box; picking **None** clears it. |
| **Lookup** | Pick a lookup type (a controlled list of valid codes) the value must belong to. Choosing **None** clears the related Lookup Column below. |
| **Lookup Column** | *Only appears once a Lookup is chosen.* Which part of the lookup to match/return — **Code**, **Meaning**, or **Description** (or None). |
| **Grid** | Attach a "grid" (a hierarchical/tabular validation set). Choosing one reveals the two fields below. |
| **Grid Input** | *Only when a Grid is chosen.* Which grid column feeds in (the input side). |
| **Grid Output** | *Only when a Grid is chosen.* Which grid column is read out. |
| **Data Type** | The kind of value this column holds. Options: **String, Number, BigInt, Float, Positive Int, Positive Decimal, Negative Int, Negative Decimal, Date.** Changing the type changes which validation section shows below. |

- **Show Aggregate** (only for numeric types — Number/BigInt/Float and the signed
  Int/Decimal variants): a checkbox that lets this column be summed/aggregated later.

### Type-specific validations

The bottom section changes based on the **Data Type** you picked:

**If String** — a "String Validations" box:
| Field | Plain English |
|---|---|
| **Min Length** | Fewest characters allowed. |
| **Max Length** | Most characters allowed (defaults to the column's size). |
| **Pattern** | A rule the text must match. Pick a ready-made pattern from the list, **None**, or **Custom**. |
| **Custom pattern editor** (only when *Custom*) | Two tabs: **✎ Type** to write a regex by hand (with a mic button for voice dictation, plus live validity checks — red if the regex is broken, an orange warning if it won't work in the server's Python engine), and **✨ AI Generate** where you type something like "email address" or "10-digit phone" and the app writes the regex for you. |
| **Padding** | Characters to pad the value with (e.g. leading zeros). String only. |

**If Date** — a "Date Validations" box:
| Field | Plain English |
|---|---|
| **Date Format** | The expected date layout (e.g. `yyyy/MM/dd`), chosen from a list, or **None**. |

**If a numeric type** — no extra length/format box, just the **Show Aggregate** checkbox mentioned above.

### Bottom

| Control | Meaning |
|---|---|
| **Delete field** (red checkbox) | Marks this column for deletion. On Save the column is removed from the sheet (and, if it already existed on the server, recorded so the backend deletes it too). |
| **Cancel** | Close without saving. |
| **Save** | Apply all the above to the column. *Disabled if you have an invalid custom regex* — you can't save a broken pattern. |

---

## 4. The Source Sheet Configuration drawer

Opens when you click a **sheet name**. This describes how the whole *sheet/file* is laid
out, and lets you attach a data-validation hook. Built by
[SourceSheetConfiguration.jsx](react/frontend/src/components/DesignMode/Source/Components/SourceSheetConfiguration.jsx).
Title: **"Source Sheet Configuration: <file> / <sheet>"**.

| Field | Plain English |
|---|---|
| **Number of Headers** | How many rows at the top of the sheet are header rows (not data). Defaults to 4 for FBDI files, 1 otherwise. |
| **Field Separator** | The character that separates values (default comma `,`). Matters for CSV/flat files. |
| **Number of Trailers** | How many rows at the *bottom* are footer/summary rows to ignore (default 0). |
| **Data Validation** | A dropdown of "hooks" — reusable validation routines. Pick **None**, or a named hook. Picking most hooks opens the **Data Validation Configuration** modal (section 7) to set it up. The **CVR** hook needs no setup, so it's stored directly. A 👁 **eye** button next to a chosen hook re-opens that modal to view/edit it. |
| **Cancel / Save** | Discard or store the sheet configuration. |

---

## 5. The Database Source Configuration drawer

Opens when you click a **file name**. Use it to tell the app the file's data actually
comes from a database query, plus a couple of file-level flags. Built by
[DBSource.jsx](react/frontend/src/components/DesignMode/Source/Components/DBSource.jsx).
Title: **"Database Source Configuration"**.

| Field | Plain English |
|---|---|
| **Source Type** | Where the data comes from (e.g. a query-type source). Has a 🔄 refresh button to reload the list. If there's only one option it's auto-picked. |
| **Source Query** | *Enabled only when the Source Type is a "query" type.* Picks which saved database query supplies the data. Otherwise it shows "Not applicable for this source type." |
| **Email** (checkbox) | A file-level flag saved as `email_required` — marks that this file needs an email step/notification. |
| **Preserve JSON** (checkbox) | *Only visible to the admin user (id 1).* When on, re-uploading the file keeps existing mappings instead of regenerating them from scratch. |
| **Cancel / Save** | Discard or store the DB-source settings for this file. |

---

## 6. The Add Column popup

Opens from the **＋** button above a sheet's column table. A tiny centred popup. Built by
[ColumnAddition.jsx](react/frontend/src/components/DesignMode/Utils/Common/ColumnAddition.jsx).

| Field | Plain English |
|---|---|
| **Column Name** | Type a name for a brand-new column. Press Enter or click **Save** to add it. The app auto-generates a unique internal name/position for it, defaults it to a String of size 100, and marks it "new" so the next Save sends it to the server. Duplicate names are rejected. |
| **Cancel / Save** | Discard or add the column. Save is disabled until you type a name. |

---

## 7. The Schema drawer (for "Schema" source type)

Opens when the source type is **Schema** and you click **Configure**. Instead of
uploading a file, you point at a saved database query and the columns come from it. Built
by [SchemaDrawer.jsx](react/frontend/src/components/DesignMode/Utils/Common/SchemaDrawer.jsx).
Title: **"Schema Configuration"**.

| Field | Plain English |
|---|---|
| **Schema Name** | Dropdown of saved schemas/queries. Auto-selects if there's only one. |
| **Query** | Shows the SQL behind the chosen schema. Read-only by default with a **Read-only** badge. |
| **Copy** button | Copies the query to your clipboard. |
| **Edit toggle** | Flip it on to edit the query text. A **Modified** badge and a **Reset** button appear if you change it; Reset restores the original. Edits are saved back to the server. |
| **Line/character count** | Small live counters under the editor. |
| **Cancel / Save** | On Save, the query runs on the server to produce the sheet/column structure, which then loads into the Source panel just like an uploaded file. |

---

## 8. The Data Validation Configuration modal (hook setup)

Opens from the **Data Validation** dropdown in the Sheet Configuration drawer (section 4).
This is a large two-pane modal for configuring a validation "hook" item-by-item. Built by
[HookDetailModal.jsx](react/frontend/src/components/DesignMode/Utils/Common/HookDetailModal.jsx).
Title: **"Data Validation Configuration"**.

**Left pane — the item list:**
- A **search box** to filter items.
- **Select all / Deselect all** toggle.
- A checklist of validation **items**; each shows its sequence number and a status
  (**Mapped / Unmapped / Excluded**). Untick an item to exclude it. Counters at the
  bottom show "X / Y mapped" and "X / Y selected."

**Right pane — settings for the selected item.** A one-line row of controls:
| Field | Plain English |
|---|---|
| **Data Source** *(required)* | Where this check's data comes from — **Oracle** (the cloud) or **Local** (your file). Defaults to Oracle. Choosing **Local** forces the query editor on so you must supply the query. |
| **Header** *(required)* | Which of the sheet's columns this validation item maps to. Turns red if left empty on a selected item. |
| **Attribute** *(required)* | Which field *from the query's own SELECT columns* to compare against. Auto-fills if there's only one. |
| **Data Comparison** | How to compare — e.g. **Include** or **For each**. Auto-set based on the Data Source (Oracle → Include, Local → For each). |
| **Tolerance** *(required)* | How much difference is acceptable, e.g. `5` or `5%`. Only accepts digits and an optional `%`. |

Below the row is a **Query editor**:
- **Modified** badge, **Reset**, **Copy**, and an **Edit** toggle (locked on for Local sources).
- A large SQL textarea (read-only until Edit is on; highlighted red if a Local item needs a query but it's blank).
- Line/character counters.

**Footer:** **Cancel**, and a **Save** that's disabled until *every selected item* has a
data source, header, attribute, tolerance, and (for Local items) a query. On Save, any
Local queries are validated on the server first — an invalid one blocks the save with an
error toast.

---

## Quick recap — the whole Source flow

1. **Pick a type and upload** a file (or **Configure** a schema / attach a **DB source**).
2. The file appears in the tree as **File → Sheets → Columns**.
3. Click a **column** → set its identity, keys, options, and validations in the **Source Attribute** drawer.
4. Click a **sheet** → set header/trailer/separator counts and an optional **Data Validation hook**.
5. Click a **file name** → attach a DB source and file-level flags.
6. Tick the **checkboxes** on files/sheets to expose their columns to the Target (mapping) side.
7. Hit the **💾 Save** icon on the file to persist everything to the server.
