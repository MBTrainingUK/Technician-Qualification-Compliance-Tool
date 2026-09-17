# MB Dealer Technician Compliance Tool

A self-contained HTML tool for analysing Mercedes-Benz dealer technician qualification compliance against the MB Dealer Standards, and for working out **what training still needs booking**. No server, no installation — runs entirely in the browser.

> **Current version:** `Compliance_Tool_v11.html` — September 2026

---

## Quick Start

1. Open **`Compliance_Tool_v11.html`** in any modern browser (Chrome, Edge, Safari, Firefox)
2. Drop your **PDP Course Data `.xlsx`** file onto the load area, or click it to browse
3. The tool parses the spreadsheet in-browser — no data leaves your machine
4. Use the five tabs (**Overview · Pipelines · Course Demand · By Site · Network Population**)

**Links:**
- <https://mbtraininguk.github.io/Technician-Qualification-Compliance-Tool/> — the repo root, the entry point worth circulating: it always forwards to the current version
- <https://mbtraininguk.github.io/Technician-Qualification-Compliance-Tool/Compliance_Tool_v10.html> — the link already circulated; kept working, and forwards to the current version too

**Defaults on load:**
- Opens on the **Overview**, filtered to **non-compliant sites**, worst first
- **Exclude apprentices** is ON
- **MB Retailers Only** network filter is active
- Passive delegates are excluded (see [Passive delegates](#passive-delegates))
- **Light theme**, unless you switched to dark before — the choice is remembered

---

## The v10 interface

v10 is a presentation rebuild of v8. **Nothing was removed functionally** — every figure, filter, drill-down, sort and export behaves identically, and that was verified by running both versions against the same export and diffing the results line for line.

What changed:

| | v8 | v10 |
|---|---|---|
| **Theme** | Dark only | **Light by default, dark toggle** in the app bar, remembered per browser |
| **Before loading** | Full interface with empty panels | A single centred drop panel, nothing else |
| **After loading** | Load box stayed at full size forever | Collapses to a chip in the app bar; click it to load a different export |
| **Qualification codes** | Permanent bar across the top | In the **ⓘ** help popover |
| **Scope controls** | Two rows of buttons inside every tab | One **sticky filter bar** below the tabs |
| **Apprentice toggle** | Inside the load box | In the filter bar |
| **Tab explanations** | A paragraph at the top of each tab | Behind an **ⓘ** next to the heading |
| **Pipeline state key** | Always-on row of five items | Inside the same disclosure |
| **Methodology footer** | Long line at the bottom of every page | In the help popover |
| **Tabs** | Emoji-prefixed buttons | Plain underlined text |
| **Pipeline summary** | Five solid colour blocks | Light cards with a coloured edge and figure |
| **Numbers** | Proportional figures | Tabular figures, so columns align |

**Colour is now a token set.** v8 carried 34 distinct colours in its stylesheet and 27 more written inline by the JavaScript, with no shared vocabulary. v10 defines one set of CSS custom properties — surface, border, text, accent, and semantic pass/warn/fail — and the dark theme simply swaps their values. No colour is hardcoded anywhere, which is what makes the toggle safe.

**Why light by default:** Power BI, Tableau, Looker and Metabase are all light-first because their output ends up in decks, emails and print. The Course Demand PDFs already rendered light, so the screen and the paper now match. The PDF stays light whatever the screen is set to.

### Where things moved

| Looking for | Now |
|---|---|
| Qualification code key | **ⓘ** in the app bar |
| How technicians are counted | **ⓘ** in the app bar |
| Enrolled vs booked definition | **ⓘ** in the app bar, and the Pipelines **ⓘ** |
| The five requirements | **ⓘ** in the app bar |
| Which file is loaded, and how many rows | The chip in the app bar, or **ⓘ** → Data |
| Network / Car / Van / apprentice filters | The filter bar under the tabs |
| Pipeline symbol key | Pipelines tab, **ⓘ** next to the heading |
| Course demand method | Course Demand tab, **ⓘ** |

By Site has no filter bar — it shows one dealer, chosen from its own dropdown.

---

## What v8 changed, and why

v8 was rebuilt against Lee Coleman's brief of 1 August 2026 (`Technical Training Administration Support - Analysis Project.eml`) and the follow-up meeting notes (`Transcript.rtf`).

| Change | Reason |
|--------|--------|
| Overview leads with **non-compliant sites**, not "% compliant" | Meeting: flip "24% fully compliant" to "76 non-compliant sites" |
| **Interactive donut charts** replace the stat cards — compliant and non-compliant shown together, click to filter | Meeting: "replace compliance boxes with pie charts showing both at once; click to filter" |
| "Sites by number of issues" row **removed** | Meeting: remove unless Lee or Lou request it — Lee's focus is the 80% figure |
| ST Pipeline becomes **Pipelines**, with MT, ST, DT and a combined view | Meeting: rename to ST/MT pipeline, add DT, show all three pathways in one view |
| New **enrolled vs booked** distinction, four states | Meeting: "booked is the meaningful status, not enrolled on pathway" |
| **"Enrolled but not yet booked"** summary filter across all sites, sorted by site | Meeting: explicitly requested |
| **Passive delegate filter** | Meeting: filter out passive delegates (Greg's flag) |
| New **Course Demand** tab | Lee's email: "How many courses do we need to schedule?" |
| Car/Van split combines with the network filter | Meeting: split into MB-only car and MB-only van views |
| Car vs Van course variants no longer conflated | Bug in v6/v7 — see [Fixed in v8](#fixed-in-v8) |

---

## Data It Reads

The standard MB PDP Course Data export. v8 reads eight columns (zero-indexed):

| Column | Field | Used for |
|--------|-------|----------|
| B (1) | Outlet name | Site grouping |
| C (2) | Learningform | Telling e-Training apart from instructor-led — this is what makes "booked" detectable |
| D (3) | Course no | Qualifications and pathway progress |
| F (5) | Person status | Active / passive filter |
| G, H (6, 7) | Last name, First name | Display |
| I (8) | Username | Person identity |
| J (9) | Primary job | Technician role filter |
| K (10) | Start date | Course date → booked vs enrolled |
| L (11) | Completed on | Date a qualification was awarded |
| M (12) | Facility name | Where a booked course sits (tooltip) |
| N (13) | Registration status | `Passed` / `Enrolled` |

> **If MB change the export format**, update the column index constants in `parseExcelRows`.

### Held qualifications

Detected by award-code prefix, on `Passed` rows only:

| Code prefix | Qualification |
|-------------|--------------|
| `MBPCQMT` | PC Maintenance Technician |
| `MBPCQST` | PC Systems Technician |
| `MBPCQDT` | PC Diagnostic Technician |
| `MBVQMT` | Van Maintenance Technician |
| `MBVQST` | Van Systems Technician |
| `MBVQDT` | Van Diagnostic Technician |

The award codes are deliberately kept as the source of truth rather than the programme codes: in the September 2026 data, 1,978 people hold a passed MT award against only 484 with a passed `T2112Q`, and every `T2112Q` holder is inside the award set. Programme codes are used for *pipeline* status instead.

---

## Enrolled vs booked

The distinction the network actually cares about, and the thing v7 could not express. Every pathway course resolves to one of five states:

| Symbol | State | Rule |
|--------|-------|------|
| ✓ | **Passed** | Registration status = `Passed` |
| ◑ (blue) | **Booked** | `Enrolled` on an Instructor-led / Virtual Classroom course with a date in the future |
| ◑ (amber) | **Sat, awaiting result** | Same, but the date has passed and the result is not yet in |
| ○ | **Enrolled, not booked** | `Enrolled` with no course date — e-Training, or pathway enrolment only |
| ✗ | **Not enrolled** | No record |

Why the rule works: in the 14 September 2026 export, every `Enrolled` row splits cleanly by learning form — 2,528 instructor-led/virtual rows carry a future course date, 148,786 e-Training rows carry none. The latter is exactly the "delegates sit on enrolled indefinitely" case from the meeting.

---

## Qualification pathways

Sourced from `Technician Qualification Pathways.pdf` (Qualification and Programme Brochure — GTLS CHANNEL) and confirmed present in the PDP data. Codes carry a brand suffix: `.PC` = Car, `.VN` = Van, bare `-UK` = counts for both, `.TR` = Truck (ignored).

| Pathway | Programme | Face-to-face courses tracked |
|---------|-----------|------------------------------|
| **MT** — Maintenance Technician | `T2112Q` | `T2115F` Electrical Systems Basic (2d) · `T2116F` Maintenance and Service Basic (2d) |
| **ST** — Systems Technician Basic | `T3035Q` | `T0004F` Diagnosis Strategy Pt 1 (4d) · `T2121F` Electrical Measurement & Scope (2d) · `T2122F` On-board Electrical & Networking (2d) |
| **DT** — Diagnostic Technician | `T2482Q` | `T0005F` Diagnosis Strategy Pt 2 (4d) · `T2777F` C-DT Initial Test (1d) · `T2778B` Electrical Components (4d) · `T2779B` Drivetrain (4d) · `T2780B` Suspension (4d) · `T2781F` Test Preparation (2d) · `T2782F` Final Test (1d) |

The five codes Lee named in his brief (T2115, T2116, T0004, T2121F, T2122F) are exactly the MT + ST face-to-face set, and are marked ★ in the Course Demand tab.

Digital pre-requisites (T2397D, T0103E, T2160D, T2164D, T2166D–T2169D, T2182D, T2195D, T2159D) are deliberately **not** tracked: they are self-serve e-Learning and never the scheduling constraint.

---

## Compliance Rules

Each site is assessed against five requirements. Unchanged from v6/v7.

| # | Rule | Pass threshold |
|---|------|---------------|
| 1 | **Overall qualified** — technicians holding any MB qual | ≥ 80% |
| 2 | **Systems Technician (ST) count** — surplus DTs also count here | ≥ 60% of total techs |
| 3 | **Maintenance Technician (MT) only** — at least one tech whose highest role is MT | ≥ 1 |
| 4 | **Diagnostic Technician (DT) present** | ≥ 1 |
| 5 | **Scaled DT count** — 1 DT per 1–15 techs, +1 per additional 15 | `1 + floor((n-1)/15)` |

> **Surplus DT rule:** a DT beyond the scaled requirement is reallocated to the ST 60% threshold, so sites that invested in DTs are not penalised.

> **MT Waiver rule:** if every technician is ST or DT, the MT requirement is waived and shown with a blue ℹ.

Each technician counts in exactly **one** role: **DT > ST > MT > none**.

---

## Scope

One shared scope control sits at the top of **Overview, Pipelines, Course Demand and Network Population**. Whatever you pick holds across all four — pick Van on the Pipelines tab and the Overview, demand figures and population counts follow.

- **Network** — 🌐 All Sites · ⭐ MB Retailers Only (default) · 🤝 Partners Only
- **Car/Van** — All Sites · 🚗 Car Only · 🚐 Van Only

The two combine, giving the MB-only car and MB-only van views the meeting asked for.

> **Changed in v8:** Network Population used to keep its own separate Car/Van filter and ignored the network filter entirely, so its headcounts covered the whole network regardless of what the Overview was showing. It now uses the shared scope like every other tab. The Car vs Van split section still shows both columns whatever the Car/Van setting — that is the point of the split — but it does honour the network filter.

By Site has no scope bar; it shows one dealer, chosen from its own dropdown.

---

## The Five Tabs

### 1 · Overview

Opens on what is failing.

- **Hero line** — "N sites are not meeting the minimum franchise standard", with counts below 80%, below 60% ST, and the number of technicians at failing sites
- **Three donut charts** — Overall Compliance, 80% Qualification, 60% Systems Technician. Each shows compliant and non-compliant together; click a segment or a legend entry to filter the table
- **Scope bar** — see [Scope](#scope) below
- **Site table** — sortable on all nine columns, defaulting to non-compliant, most issues first

### 2 · Pipelines

Who is working towards a qualification, and whether it is booked.

- **Pathway toggle** — MT · ST · DT · All Three
- **Filters** — All · *N* course(s) completed · Any booking · Not started · 🎯 Needed to clear the standard

The completed-count filters show how far through a pathway someone actually is — courses **passed**, not booked. They are generated per pathway, one for each exact count from 1 up to one below the number of courses: MT has two courses so gets **1 course completed**; ST has three so gets **1 course completed** and **2 courses completed**; DT has seven so gets six. They are omitted in the All Three view, where a count spanning three pathways means little. Switching pathway resets a count filter that no longer applies.

**Any booking** covers the diary side separately — anyone with at least one dated course.

"Enrolled, not booked" remains as the orange summary card above the filter row — the meeting asked for that figure — and clicking the card still filters to it.
- **Table** — site, name, job role, brand, qualification held, programme enrolment, and a state dot per course. Hover a booked dot for the date and facility.

A technician appears in a pathway if they show engagement on it, **or** if their site needs them to close a gap. Without that gap test the DT pipeline would simply list every Systems Technician in the network — the standard asks for 1 DT per 15 technicians, not for everyone to become one.

### 3 · Course Demand

Lee's fourth question: how many courses need scheduling.

Two sets of figures per course:

- **To clear the standard** — counts only the technicians each site actually needs to qualify (enough to reach 80% qualified, 60% ST and the scaled DT requirement), taking those closest to finishing first
- **Full pathway** — everyone in the pipeline, for the longer view

`Courses to schedule = ceil(outstanding / class size)`, where **outstanding** means the technician needs the course and has not got it booked.

> ⚠️ **Class size defaults to 12 and is a placeholder.** It is an editable input at the top of the tab, not a buried constant — confirm the real MB figure and every number on the tab updates.

Scope toggle: non-compliant sites only (default) or all sites in scope.

**Every figure is clickable.** Clicking a Need it / Booked / Outstanding number expands a panel under that course row listing the delegates behind it, grouped by site with a headcount per site, each delegate's job role and their state on that course. Booked entries show the course date and training facility. Click a site name to jump to its By Site view; click the figure again (or × close) to collapse. The Schedule columns are derived and are not clickable.

### Excel export

Each drill-down panel has a **Download Excel** button, for handing a course list to whoever is doing the scheduling. The workbook has two sheets:

- **Delegates** — one row per delegate in site order: *Site · Delegate · Job Role · Status · Course Date · Venue*. The site repeats on every row and an autofilter is applied, so it can be cut by site or status immediately.
- **About** — the course, what the list is, the delegate and site counts, the scope it was produced under, and the date generated. A file sitting in someone's inbox next month still explains itself.

Filename is `<COURSE>-<what>-<date>.xlsx`, e.g. `T2115F-outstanding-2026-09-16.xlsx`.

> Built with SheetJS, which is already loaded to read the PDP file — so the export adds no dependency. v8's PDF export used a second CDN library; replacing it with Excel removed that, and the tool now has exactly one external dependency.
>
> Freeze panes are not written by the SheetJS community build, so the header row does not stay pinned when scrolling. The autofilter dropdowns work normally.

---

### 4 · By Site

Per-dealer detail, as in v7, plus a new **Next Step** column in the technician table showing the pathway that technician needs next and the state of each of its courses.

### 5 · Network Population

Carried over from v7 unchanged.

---

## Passive delegates

The meeting asked for passive delegates to be filtered out — Greg's flag marks leavers never properly removed from GTLS. Column F (`Person status`) carries it, and v8 excludes any row whose status is not `Active`.

> **In the 14 September 2026 export this filter does nothing: all 598,392 rows read `Active`.** The Overview says so explicitly — "Passive excluded: 0 — this export contains no passive records" — rather than implying it did something. It starts working the moment a flagged export arrives.
>
> Worth asking Greg for an export that carries the flag so the filter can actually be proven.

If the column is blank (an older export), nothing is filtered rather than everything.

---

## Fixed in v8

1. **Car/Van course variants were conflated.** v6/v7 matched `T0004F` and friends by bare prefix, so a Car technician's ST pipeline could be ticked green by a Van course. v8 parses the `-UK.PC` / `-UK.VN` suffix and tracks each brand separately. A dual-brand technician takes the stronger of the two states, so they are never shown as behind twice.
2. **The ST pipeline paired a PC programme with Van courses** — `T3035Q` against the Van-named T0004F/T2121F/T2122F. Resolved by the variant fix above.
3. **`index.html` pointed at v6** while v7 was the newest file on disk. It now points at v8.

---

## Technician Roles

Recognised as technicians:

- Maintenance Technician Car / Van / Car & Van
- Systems Technician Car / Van / Car & Van
- Technician Car / Van / Car & Van
- Diagnostic Technician Car / Van / Car & Van
- Apprentice Technician · Mobile Technician · S24h Technician · Fleet Workshop Technician

A technician's brand comes from the job title; for roles that name neither Car nor Van (S24h, Mobile, Apprentice) it falls back to the site type.

---

## MB Network Site Lists

The network filters rely on hardcoded arrays inside the HTML: `MB_OFFICIAL_SITES` (car retailers plus the van network), `MB_PARTNER_SITES` (authorised repairers), and `VAN_NETWORK_SITES`.

Site names must match the **exact spelling used by PDP**, including spacing and punctuation — `Marshall Truck and Van  - Southampton` has a double space before the dash.

To extract exact outlet names from a PDP export:

```python
import openpyxl
wb = openpyxl.load_workbook('PDP Course Data - DATE.xlsx', read_only=True)
ws = wb.active
names = sorted({row[1].value for row in ws.iter_rows(min_row=2) if row[1].value})
for n in names: print(n)
```

---

## Privacy & Data Handling

**All processing happens locally in your browser.** The spreadsheet is never uploaded and no dealer or delegate data ever leaves your machine — including exports, which are generated in the browser.

Two libraries are loaded from CDN, and nothing else leaves the page:

One library is loaded from CDN and nothing else leaves the page: [SheetJS](https://sheetjs.com/), used both to read the PDP export and to write the delegate spreadsheets.

Closing the tab discards all data.

---

## Files in This Folder

| File | Description |
|------|-------------|
| `Compliance_Tool_v11.html` | **Current version — use this one** |
| `Compliance_Tool_v10.html` | **Not a build** — a redirect to the current version, because this filename is the link that was circulated. Point future releases at v12 by editing it |
| `Compliance_Tool_v8.html` | Dark-only interface |
| `Compliance_Tool_v7.html` | Added the Network Population tab |
| `Compliance_Tool_v6.html` | The version `index.html` used to point at |
| `Old versions/` | v2–v5, the original build, and the real v10 tool |
| `index.html` | Redirects to the current version |
| `README.md` | This file |

---

## Key Implementation Details

All logic lives in the single `<script>` block. Approximate section map:

| Section | Key functions / variables |
|---------|--------------------------|
| **Shell (v10)** | `applyTheme()`, `toggleTheme()`, `toggleHelp()`, `infoBtn()`, `infoPanel()`, `toggleInfo()`, `renderFilterBar()`, `resetScope()` |
| **Global state** | `hideAppr`, `ovNetworkFilter`, `ovTypeFilter` (shared scope), `ovCardFilter`, `ovSortCol`, `pipelinePathway`, `pipelineFilter`, `demandClassSize`, `demandScope` |
| **Site lists** | `MB_OFFICIAL_SITES`, `MB_PARTNER_SITES`, `VAN_NETWORK_SITES` |
| **Pathways** | `PATHWAYS`, `COURSE_LOOKUP`, `courseBase()`, `courseBrand()`, `rowState()`, `pwEnsure()`, `pwSet()`, `pwView()`, `techBrand()`, `pwDot()` |
| **Data parsing** | `parseExcelRows()` — builds `SITE_DATA`, per-person `quals` and `pw` pathway state; sets `PASSIVE_EXCLUDED` |
| **Compliance logic** | `siteCompliance()` (returns pass/fail flags plus `qualified` and `effSTs`), `siteGap()` |
| **Scope** | `scopeBar()`, `setScopeNetwork()`, `setScopeType()`, `rerenderAllScoped()` |
| **Overview** | `renderOverview()`, `donut()`, `setOvCardFilter()`, `setOvTypeFilter()`, `setOvNetworkFilter()`, `setOvSort()` |
| **Pipelines** | `pipelineScope()`, `levelBelow()`, `progressScore()`, `collectPipeline()`, `renderPipeline()` |
| **Course Demand** | `renderDemand()`, `setDemandClassSize()`, `setDemandScope()`, `demandCell()`, `toggleDemandDrill()`, `drillData()`, `demandDrillRow()`, `DEMAND_DATA` |
| **Excel export** | `exportDrillXlsx()`, `drillData()`, `scopeSentence()` |
| **By Site** | `renderSite()`, `buildTable()`, `nextPathway()`, `reqRow()`, `brandCard()` |

---

## Known Limitations

- **Internet required on first load** — SheetJS comes from `cdn.sheetjs.com`. Once cached, offline use works. From v11 SheetJS is only needed for the Excel export and as the fallback reader, but it is still fetched on load.
- **The fast reader needs `DecompressionStream`** — Chrome/Edge 80+, Safari 16.4+, Firefox 113+. Anything older falls back to SheetJS automatically and simply loads at the old speed.
- **First sheet only** — the tool always reads the first worksheet.
- **Column positions are fixed** — update `parseExcelRows` if MB change the export.
- **Site name matching is exact** — any PDP spelling change drops a site out of the network filter until the array is updated.
- **Class size is a placeholder of 12** — see [Course Demand](#3--course-demand).
- **The passive filter is unproven** — no export seen so far contains a passive record.
- **Pathway codes verified September 2026** — if MB introduce new programme or course codes, add them to `PATHWAYS`.

---

## Change History

| Version | Key changes |
|---------|-------------|
| **v11** | **Loads in about 1.5s instead of 15s.** A purpose-built reader parses the PDP export directly — reading the zip, decompressing with the browser's native `DecompressionStream` and scanning the sheet XML straight into rows — replacing `XLSX.read`, which took 12.5s and 1.8GB of heap on the September export. SheetJS remains loaded for the delegate Excel export and as an automatic fallback if the fast reader cannot handle a file. **Also fixes a date off-by-one:** `toISODate` read SheetJS's dates via `toISOString()`, which on any machine ahead of UTC (the UK in summer, CET all year) reported every date a day early — including the `Course Date` column of the delegate export. Dates are now the true calendar date, so any date shown in By Site or exported moves one day later than v10 reported. No compliance figure changes: verified across all 598,393 rows and all 298 scored sites against the same export |
| **v10** | Interface rebuild. Delegate lists export to **Excel** instead of PDF, dropping the jsPDF dependency; apprentice filter became a segmented control and the selected filter segment is now solid accent rather than just bold.  Light theme by default with a dark toggle; every colour moved to design tokens; two-state shell so the loader collapses to a chip once data is in; sticky filter bar replacing the per-tab button rows; qualification codes, counting rules and methodology moved into a help popover; per-tab prose behind ⓘ disclosures; plain text tabs; tabular figures. No change to any calculation — verified by diffing both versions against the same export |
| v8 | Rebuilt to Lee's brief and the meeting notes. Overview leads with non-compliant sites; interactive donut charts replace stat cards; issue-count row removed; Pipelines tab covers MT, ST and DT with an enrolled-vs-booked distinction and an "enrolled but not yet booked" summary; new Course Demand tab answering how many courses to schedule, with every figure clickable to list the sites and delegates behind it and export that list; per-pathway completed-count filters; one shared scope bar across all multi-site tabs; passive delegate filter on column F; Next Step column in By Site; Car/Van course variant conflation fixed; `index.html` repointed |
| v7 | Network Population tab |
| v6 | Sortable Overview league table; MT waiver rule; MB network filter; ST Pipeline rebuilt with prefix matching; issue-count cards; global apprentice toggle |
| v5 | Pipeline present but showed 0 — bare equality course code matching never fired |
| v2–v4 | Iterative early versions |
