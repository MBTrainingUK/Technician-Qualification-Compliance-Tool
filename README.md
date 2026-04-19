# MB Dealer Technician Compliance Tool

A self-contained HTML tool for analysing Mercedes-Benz dealer technician qualification compliance against the MB Dealer Standards requirements. No server, no installation — runs entirely in the browser.

---

## Quick Start

1. Open **`Compliance_Tool_v6.html`** in any modern browser (Chrome, Edge, Safari, Firefox)
2. Drop your **PDP Course Data `.xlsx`** file onto the load area, or click it to browse
3. The tool parses the spreadsheet instantly in-browser — no data leaves your machine
4. Use the three tabs (**Overview · By Site · ST Pipeline**) to explore compliance

---

## What the Tool Does

### Data It Reads

The tool expects the standard MB PDP Course Data export. It reads these columns (zero-indexed):

| Column | Field |
|--------|-------|
| B (index 1) | Outlet / Dealer name |
| D (index 3) | Course code |
| G (index 6) | Last name |
| H (index 7) | First name |
| I (index 8) | Username |
| J (index 9) | Primary job role |
| L (index 11) | Completion date |
| N (index 13) | Registration status (`Passed` / `Enrolled` / `In Progress`) |

Only rows where the job role matches a recognised technician role are counted (see [Technician Roles](#technician-roles)).

Qualifications are detected by course code prefix:

| Code prefix | Qualification |
|-------------|--------------|
| `MBPCQMT` | PC Maintenance Technician |
| `MBPCQST` | PC Systems Technician |
| `MBPCQDT` | PC Diagnostic Technician |
| `MBVQMT` | Van Maintenance Technician |
| `MBVQST` | Van Systems Technician |
| `MBVQDT` | Van Diagnostic Technician |

---

## Compliance Rules

Each site is assessed against five requirements:

| # | Rule | Pass threshold |
|---|------|---------------|
| 1 | **Overall qualified** — technicians holding any MB qual | ≥ 80% |
| 2 | **Systems Technician (ST) count** — techs counted as ST (surplus DTs also count here) | ≥ 60% of total techs |
| 3 | **Maintenance Technician (MT) only** — at least one tech whose highest role is MT | ≥ 1 |
| 4 | **Diagnostic Technician (DT) present** | ≥ 1 |
| 5 | **Scaled DT count** — 1 DT required per 1–15 techs, +1 per additional 15 | Formula: `1 + floor((n-1)/15)` |

> **Surplus DT rule:** A DT who is the only DT at a site counts first toward the DT requirement. Any additional DTs beyond the scaled requirement are reallocated to count toward the ST 60% threshold. This avoids penalising sites that have invested heavily in DTs.

Each technician is counted in exactly **one** role using this priority: **DT > ST > MT > none**.

---

## The Three Tabs

### 1 · Overview

A fleet-level view across all loaded sites.

**Summary cards** (clickable — filter the table below):
- Total sites · Fully compliant · Non-compliant · Fail 80% overall · Fail 60% ST · No MT-only tech · DT requirement issue

**Requirement failure bars** — horizontal bar chart showing how many sites fail each of the five rules.

**Site table** — one row per site with:
- Technician count · Overall % · Pass/fail tick for each of the 5 rules · Issues count

Click any site name to jump straight to its By Site detail view.

**Car / Van filter** — classify sites by whether their technicians hold Car or Van roles, and filter accordingly.

---

### 2 · By Site

Detailed compliance view for a single dealer.

**Select a site** from the dropdown at the top.

Displays:
- **Summary bar** — total techs, qualified count, overall %, DT count, ST %, MT-only count
- **Compliance gauge** — colour-coded bar (green ≥ 80%, amber ≥ 60%, red < 60%) with the 80% and 60% threshold markers
- **Brand breakdown cards** — separate MT / ST / DT counts for PC and Van
- **Five requirements checklist** — ✅ / ❌ with explanatory detail for each rule
- **Technician table** — every technician at the site with their job role, qualification tags, and how long they have held their highest qualification

**Filter buttons** on the technician table: All · DT · ST · MT · Unqualified

**Hide Apprentices toggle** — excludes Apprentice Technicians from all counts and tables when switched on (applies globally across all tabs).

---

### 3 · ST Pipeline

Identifies technicians who are working toward the Systems Technician qualification but have not yet achieved it — your priority targets for completing ST training.

**How it works:**

The ST qualification (MBPCQST / MBVQST) is awarded only after completing all three pathways. Each pathway has two stages: mandatory e-learning followed by an SVQ practical assessment:

| Pathway | Award code | Mandatory e-learning | SVQ assessment |
|---------|-----------|---------------------|----------------|
| **Electrics** | T3041Q | T2342D | T2132F |
| **Engine** | T3044Q | T2918D | T2809F (Diesel) / T2137F (Gasoline, PC only) |
| **Chassis** | T3046Q | T2128E / T2179D / T2773D | T2126F |
| **Basic prereq** | — | T3035Q | — |

The pipeline tracks all of these sub-courses — not just the final pathway award codes — so technicians with partial progress show up correctly.

**Stage scoring per pathway (0–6):**

| Score | Stage |
|-------|-------|
| 0 | No activity |
| 1 | Basic prereq (T3035Q) enrolled/passed |
| 2 | Mandatory course enrolled |
| 3 | Mandatory course passed |
| 4 | SVQ assessment enrolled |
| 5 | SVQ assessment passed |
| 6 | Full pathway award passed |

**Pathway dots (E · En · C):**
- 🟢 Green — SVQ passed or pathway complete
- 🟡 Amber — Mandatory done or SVQ enrolled (closest to qualifying)
- ⚫ Grey — Not started, prereq only, or mandatory enrolled

**Summary cards:** Total in pipeline · 2+ pathways complete · 1 pathway complete · At SVQ stage · At mandatory stage

**Filter buttons:** All · 2+ complete · 1 complete · SVQ stage · Mandatory stage · PC / Van

---

## Technician Roles

The following job roles are recognised as technicians:

- Maintenance Technician Car / Van / Car & Van
- Systems Technician Car / Van / Car & Van
- Technician Car / Van / Car & Van
- Diagnostic Technician Car / Van / Car & Van
- Apprentice Technician
- Mobile Technician
- S24h Technician
- Fleet Workshop Technician

---

## Privacy & Data Handling

**All processing happens locally in your browser.** The spreadsheet is never uploaded to any server. The tool uses the [SheetJS](https://sheetjs.com/) library (loaded from CDN on first open — requires internet) to parse the `.xlsx` file in memory. Closing the browser tab discards all data.

---

## Files in This Folder

| File | Description |
|------|-------------|
| `Compliance_Tool_v6.html` | **Current version** — use this one |
| `Compliance_Tool_v5.html` | Previous version (pipeline showed 0 — replaced by v6) |
| `Compliance_Tool_v3.html` | Earlier version |
| `Technician_Compliance_Tool.html` | Build output (same as latest version) |
| `README.md` | This file |

---

## Rebuilding / Modifying

The tool is generated from **`/tmp/build3.py`** (Python 3, no dependencies):

```bash
python3 /tmp/build3.py
```

Output path: the `Dealer standards/` folder as `Technician_Compliance_Tool.html`.

Key sections in `build3.py`:

| Lines (approx) | Section |
|----------------|---------|
| 1–160 | CSS styles |
| 160–370 | Overview rendering (`renderOverview`, `siteCompliance`) |
| 370–415 | Data constants — `TECH_JOBS_SET`, `QUAL_PREFIXES`, `SUBCOURSE_MAP` |
| 415–560 | Excel parsing — `parseExcelRows`, `getSubCourse`, `getQual` |
| 560–700 | ST Pipeline — `renderPipeline` |
| 700–870 | By Site rendering — `renderSite`, `brandCard` |
| 870–952 | HTML scaffold, `<head>`, SheetJS CDN link |

---

## Known Limitations

- **Internet required on first load** — SheetJS is loaded from CDN (`cdn.sheetjs.com`). Once the browser has cached it, offline use works.
- **First sheet only** — the tool always reads the first worksheet in the workbook.
- **Column positions are fixed** — if MB changes the PDP export format (column order), the column index constants in `parseExcelRows` will need updating.
- **Brand detection for Van** — a course is treated as Van-brand if its course code contains `.VN`; otherwise it is treated as PC. If this convention changes in a future PDP export, update `getSubCourse()` and `getQual()`.
