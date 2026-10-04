# Qiddiya-Car-Park-Drawings-Dashboard — file guide

This dashboard is split into 5 files. **You only ever need to replace `data.json`** (and
occasionally `config.json`) — `index.html` and `logo.png` shouldn't need to change.

| File | What it is | When to replace it |
|---|---|---|
| `index.html` | All design, layout, charts, filters, calculations, and interactive logic. | Never, for a routine data update. Only if you want a visual/behavior change. |
| `data.json` | The actual drawing data — every submission record, plus the "Total Expected" counts per level/discipline/package. | **Every time you have a new Excel export.** This is the file that changes most often. |
| `config.json` | Dashboard title, subtitle, and logo path/alt text. | Only when you want to rename the dashboard or swap the logo. |
| `logo.png` | The SIAC Construction logo shown in the header. | Only if you want to change the logo image. |

Nothing is hard-coded inside `index.html` — it loads `config.json` and `data.json` at runtime
via `fetch()`, so updating the dashboard is just a matter of replacing those files with new
versions that have the same filename and same shape.

## Updating the data (routine)

1. Send me the new Excel file (`Log_.xlsx` or whatever the latest export is called).
2. I'll regenerate `data.json` from it, in the same structure as below.
3. Replace the old `data.json` in your GitHub Pages repo with the new one — same filename,
   same location (repo root, or wherever `index.html` lives). Commit and push.
4. Done. `index.html` doesn't change, so there's no risk of breaking the design/layout/charts.

### `data.json` structure

```json
{
  "meta": { "sourceWorkbook": "...", "generatedAt": "...", "packages": ["..."], "notes": [ "..." ] },
  "submissions": [
    {
      "no": "QRUT171-USC-...-CB20003-PDF",
      "title": "CAR PARK - CA4 LEVEL: CB4 EXCAVATION...",
      "discipline": "CIV",
      "type": "Shop Drawing",
      "status": "B - Approved with Comments",
      "rev": "00",
      "revDate": "2026-02-17",
      "modified": "2026-02-17",
      "level": "CB4",
      "package": "CP - Car Park",
      "overdue": false
    },
    ...
  ],
  "totalExpected": {
    "CB4||STR||Shop Drawing||CP - Car Park": 536,
    "CB3||STR||Shop Drawing||CP - Car Park": 290,
    ...
  }
}
```

- **`submissions`** is read entirely from the **Submission** sheet of the Excel workbook.
  This is the single source of truth for every KPI card, chart, and table row.
- **`totalExpected`** is read from the **LIST-Drawings** sheet, keyed as
  `"level||discipline||type||package"` → count (the last part is the zone, written with the
  same label used in `submissions[].package`). It's used only as the comparison baseline in
  the "Status by Discipline" and "Submitted vs. Total Expected by Level/Zone" charts.
- `overdue` is pre-computed per row (Pending status, more than 14 days since its revision
  date) — the dashboard doesn't recompute this from a "today" date in the browser, since a
  static GitHub Pages site has no reliable server-side clock to anchor that against
  consistently. If you want a different overdue rule, tell me and I'll adjust how I generate
  this field.
- Any manual data corrections (e.g. the Surveying/Mapping total-expected fix) are applied
  when I generate `data.json`, and noted in `meta.notes` for traceability.

## Package filter (Area / Zone)

The first filter, **Package**, is built from the Excel *Area / Zone* column — currently
`CP - Car Park`, `AN - Ancillary` and `MU - Museum`. Every record in `data.json` carries this
value in its `package` field, and `totalExpected` keys end with the same value.

- Selecting a package updates the KPI cards, all four charts and the table. The "Total Expected"
  baselines in the two comparison charts follow the selected package too.
- It works together with the other filters (the options in each filter narrow to what's still
  available), and **Reset filters** sets it back to *All*.
- The table has a **Package** column, and Excel / CSV / PDF exports include it.
- The top banner ("Submitted so far") is project-wide and does not change with the filters.
- If a `data.json` has no `package` values, the Package filter hides itself automatically.

New zones added to the Excel file appear in the filter automatically once `data.json` is
regenerated.

## Submission Trend: Monthly / Weekly

The trend chart has a **Monthly | Weekly** toggle in its header. Weekly view groups submissions
by the Monday that starts each week (using *Date Modified*) and shows weeks with no submissions
as 0. Both views respond to the Package / Discipline / Type / Status / Overdue filters. This is
built into `index.html` — no data or config change is needed to use it.

## Updating the title, subtitle, or logo (occasional)

Edit `config.json` directly — no need to ask me, and no need to touch `index.html`:

```json
{
  "projectName": "Qiddiya-Car-Park-Drawings-Dashboard",
  "title": "Drawings Status Dashboard",
  "subtitle": "Qiddiya — Full Project Scope (QRUT171) · Submission Sheet Records vs Full Project Scope · As of 30 Sep 2026",
  "logo": "logo.png",
  "logoAlt": "SIAC Construction logo"
}
```

- `title` sets both the browser tab title and the big heading at the top.
- `subtitle` is the line underneath it.
- `logo` / `logoAlt` point at the logo file — replace `logo.png` with a new image (same
  filename) to swap the logo, or change this path to point at a different file.
- `projectName` is recorded for reference (e.g. as the suggested repo name) — it isn't
  rendered on the page itself.

## Deploying to GitHub Pages

This repo/folder is intended to be named **Qiddiya-Car-Park-Drawings-Dashboard**.

Commit all 4 site files (`index.html`, `data.json`, `config.json`, `logo.png`) to the same
folder in your Pages repo (repo root, or `/docs`, whichever your Pages source is set to) and
push. GitHub Pages serves them as static files — no build step required.

**Important:** this dashboard loads `data.json`/`config.json` via `fetch()`, which only works
when the page is served over `http(s)` — GitHub Pages does this automatically. If you ever
want to preview it locally before pushing, don't just double-click `index.html`; serve the
folder instead, e.g.:

```bash
cd path/to/these-files
python3 -m http.server 8000
# then open http://localhost:8000/index.html
```

Opening `index.html` directly from disk (`file://...`) will show a "Could not load the
dashboard" error, because browsers block `fetch()` of local files for security reasons. This
only affects local previewing — it's not an issue on GitHub Pages itself.
