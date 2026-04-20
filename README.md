# MB Dealer Technician Compliance Tool

A self-contained HTML tool for analysing Mercedes-Benz dealer technician qualification compliance against the MB Dealer Standards requirements. No server, no installation — runs entirely in the browser.

> **Last updated:** April 2026 — `Compliance_Tool_v6.html`

---

## Quick Start

1. Open **`Compliance_Tool_v6.html`** in any modern browser (Chrome, Edge, Safari, Firefox)
2. Drop your **PDP Course Data `.xlsx`** file onto the load area, or click it to browse
3. The tool parses the spreadsheet instantly in-browser — no data leaves your machine
4. Use the three tabs (**Overview · By Site · ST Pipeline**) to explore compliance

**Defaults on load:**
- **Hide Apprentices** is ON — apprentice technicians are excluded from all counts
- **MB Retailers Only** filter is active — Overview shows only sites in the hardcoded MB official network list (car + van)

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

> **MT Waiver rule:** If every technician at a site is counted as ST or DT (nobody has MT as their highest level), the MT requirement is automatically waived. These sites display a blue ℹ icon in the Overview table and a blue info box in the By Site checklist — they are not penalised for the MT rule.

---

## The Three Tabs

### 1 · Overview

A fleet-level view across all loaded sites.

**Network filter** (top of tab):
- 🌐 All Sites — shows every site in the PDP data
- ⭐ MB Retailers Only — shows only sites in the hardcoded MB official network list (default)
- 🤝 Partners Only — shows only authorised repairer / partner sites

**Summary cards** (top row, clickable — filter the table below):
- **Total Sites** — all sites in the current network filter
- **Fully Compliant** — pass all 5 rules (%)
- **Pass 80%** — pass the overall qualified threshold (%)
- **Pass 60% ST** — pass the ST count threshold (%)
- **MT Met** — MT requirement met or waived (%)
- **DT Met** — scaled DT requirement met (%)

**Issue-count cards** (second row, clickable — filter to sites with exactly N issues):
- 1 Issue (green) · 2 Issues · 3 Issues · 4 Issues · 5 Issues (red)

**Site table** — one row per site, all columns sortable (click header):
- Site name · N techs · Overall % · Fully compliant · ST 60% · MT · DT (1+) · DT (scaled) · Issues count

Click any site name to jump to its By Site detail view.

---

### 2 · By Site

Detailed compliance view for a single dealer.

**Select a site** from the dropdown at the top. The **Hide Apprentices** toggle (in the load bar) applies globally across all tabs.

Displays:
- **Summary bar** — total techs, qualified count, overall %, DT count, ST %, MT-only count
- **Compliance gauge** — colour-coded bar (green ≥ 80%, amber ≥ 60%, red < 60%) with threshold markers
- **Brand breakdown cards** — separate MT / ST / DT counts for PC and Van
- **Five requirements checklist** — ✅ / ❌ / ℹ (waived) with explanatory detail for each rule
- **Technician table** — every technician with their job role, qualification tags, and tenure in their highest qualification

**Filter buttons** on the technician table: All · DT · ST · MT · Unqualified

---

### 3 · ST Pipeline

Identifies technicians who are enrolled in the ST pathway but have not yet qualified — priority targets for completing ST training.

**How it works (current course codes, verified April 2026):**

| Course prefix | Role |
|---------------|------|
| `T3035Q-…` | Enrolment trigger — confirms technician has started the ST pathway |
| `T0004F-…` | Pass course 1 |
| `T2121F-…` | Pass course 2 |
| `T2122F-…` | Pass course 3 |

All four are matched by **prefix** (e.g. `T3035Q-UK.PC-1234`), not exact code, to handle suffix variants in PDP data.

**Summary cards** (clickable — filter the table):
- **Enrolled** — total technicians with T3035Q found
- **0 passed** — enrolled but no pass courses yet
- **1 passed** — one of T0004F / T2121F / T2122F passed
- **2 passed** — two pass courses passed (closest to qualifying)

**Pipeline table columns:**
Site · Name · Job Role · T3035Q status (Enrolled / Passed pill) · T0004F ✓/✗ · T2121F ✓/✗ · T2122F ✓/✗

**Filter buttons:** All · 0 passed · 1 passed · 2 passed

---

## Global Controls

### Hide Apprentices Toggle

Located in the **load bar** (always visible). When checked (default ON), Apprentice Technicians are excluded from all counts, tables, and pipeline listings across every tab.

---

## MB Network Site Lists

The **MB Retailers Only** and **Partners Only** filters rely on two hardcoded arrays inside `Compliance_Tool_v6.html`:

### `MB_OFFICIAL_SITES`
~120 car retailer sites (`Mercedes-Benz of [City]`) plus ~68 van network sites including:
Arthur Spriggs & Sons, Bell Truck & Van (all branches), BLS Truck & Van, Ciceley Commercials, Euro Commercials (all branches), eStar Truck & Van, LSH Birmingham PDC, Marshall Truck and Van (all branches), MBNI Truck & Van, Mercedes-Benz Van Centres (all branches), Mertrux Truck & Van (all branches), Midlands Truck & Van, Northside Truck & Van, Rossetts Commercials, Rygor Group (all branches), SAGA Truck & Van (all branches), Sandown Commercials, Sytner Colindale Vans, Western Commercial (all branches).

### `MB_PARTNER_SITES`
Authorised repairers not in the main retail network (currently 2):
- `Europa Mercedes-Benz Authorised Repairer`
- `Regent Garage MB Authorised Repairer`

### Updating the site lists

Site names must match the **exact spelling used by PDP** — including spacing and punctuation. For example, `Marshall Truck and Van  - Southampton` has a **double space** before the dash.

To extract exact outlet names from a PDP export:
```python
import openpyxl
wb = openpyxl.load_workbook('PDP Course Data - DATE.xlsx', read_only=True)
ws = wb.active
names = sorted({row[1].value for row in ws.iter_rows(min_row=2) if row[1].value})
for n in names: print(n)
```

Then add/remove entries in the `MB_OFFICIAL_SITES` or `MB_PARTNER_SITES` arrays near the top of the `<script>` block.

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
| `Compliance_Tool_v6.html` | **Current version — use this one** |
| `Compliance_Tool_v5.html` | Previous version (ST Pipeline showed 0 — course code matching broken) |
| `Compliance_Tool_v4.html` | Earlier version |
| `Compliance_Tool_v3.html` | Earlier version |
| `Compliance_Tool_v2.html` | Earlier version |
| `Technician_Compliance_Tool.html` | Older build output |
| `README.md` | This file |

---

## Key Implementation Details (for future edits)

All logic lives in the single `<script>` block of `Compliance_Tool_v6.html`. Approximate section map:

| Section | Key functions / variables |
|---------|--------------------------|
| **Global state** | `hideAppr`, `ovNetworkFilter`, `ovSortCol`, `ovSortDir`, `pipelineFilter` |
| **Site lists** | `MB_OFFICIAL_SITES`, `MB_PARTNER_SITES` |
| **Data parsing** | `parseExcelRows` — reads xlsx rows into `window.allRows`; builds `prereqProgress` per technician |
| **Compliance logic** | `siteCompliance(rows)` — returns pass/fail flags including `mtWaived` |
| **Overview render** | `renderOverview()`, `statCard()`, `sortTh()`, `setOvSort()`, `setOvCardFilter()`, `setOvNetworkFilter()` |
| **By Site render** | `renderSite()`, `reqRow()`, `brandCard()` |
| **Pipeline render** | `renderPipeline()`, `setPipelineFilter()` |
| **Global toggle** | `toggleAppr()` — calls all three render functions |

**Course code matching** uses `indexOf` prefix matching because PDP codes carry a suffix (e.g. `T3035Q-UK.PC-1234`):
```js
s.indexOf('T3035Q') === 0   // enrolment trigger
s.indexOf('T0004F') === 0   // pass course 1
s.indexOf('T2121F') === 0   // pass course 2
s.indexOf('T2122F') === 0   // pass course 3
```

**Qualification prefix matching** for existing quals (e.g. `MBPCQST`, `MBVQDT`):
```js
QUAL_PREFIXES.forEach(p => { if (code.startsWith(p)) ... })
```

---

## Known Limitations

- **Internet required on first load** — SheetJS is loaded from CDN (`cdn.sheetjs.com`). Once cached, offline use works.
- **First sheet only** — the tool always reads the first worksheet in the workbook.
- **Column positions are fixed** — if MB changes the PDP export format, update the column index constants in `parseExcelRows`.
- **Site name matching is exact** — the network filter compares outlet names character-for-character. Any PDP formatting change for a site name will cause it to fall outside the MB Retailers filter until the array is updated.
- **ST Pipeline course codes** — the four codes (`T3035Q`, `T0004F`, `T2121F`, `T2122F`) were verified against the April 2026 PDP export. If MB introduce new pathway codes these will need adding to `parseExcelRows` and `renderPipeline`.

---

## Change History

| Version | Key changes |
|---------|-------------|
| v6 | Sortable Overview league table (all 9 columns); MT waiver rule; MB network filter (All / MB Retailers / Partners) with ~190 hardcoded car + van sites; ST Pipeline rebuilt with T3035Q/T0004F/T2121F/T2122F prefix matching; Overview cards redesigned (% large + count, compliant framing); bar chart replaced with 5 clickable issue-count cards; global apprentice toggle moved to load bar (default ON); pipeline summary cards clickable; default network filter = MB Retailers Only |
| v5 | Pipeline present but showed 0 matches — bare equality course code matching never fired |
| v2–v4 | Iterative early versions |
