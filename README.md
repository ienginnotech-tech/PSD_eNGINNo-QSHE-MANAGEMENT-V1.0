# PSD eNGINNo — QSHE Management V1.0

A lightweight, static web app for running the QSHE (Quality / Safety / Health /
Environment) management framework day to day: tracking the SSPPC modules
(Structure, System, Process, People, Culture) — including the JCI **Facility
Management and Safety (FMS)** plans folded into Module 3 — Process — plus
quarterly OKRs, KPIs, a CAPA/incident log, per-building (BU) scoping, user
management, and CSV/print export.

**Single file, zero build step.** Everything (HTML + CSS + JS) lives in one
`index.html` so there is nothing to misconfigure when deploying — no
folder-path or case-sensitivity issues on GitHub Pages, no missing
`css/`/`js/` subfolders if you upload via drag-and-drop on github.com.

## Quick start

Just upload `index.html` to your repo (repo root, or enable GitHub Pages on
whichever branch/folder holds it) — that's the entire deployment. Or run it
locally:

```bash
python3 -m http.server 8080
# then open http://localhost:8080
```

or double-click `index.html` directly (works fine even on `file://` since
there are no external file dependencies to break).

**Default login:** `admin` / `qshe2026` — change this immediately from the
**ผู้ใช้งาน (Users)** page after first login.

## Troubleshooting: "the page loads but looks unstyled"

If you see plain black-and-white text with no colors/layout, the browser
loaded the HTML but not the stylesheet — this only happens with the old
multi-file structure (`css/style.css`, `js/*.js` as separate files) if those
subfolders didn't make it into the repo, which is a very common outcome of
GitHub's web "upload files" drag-and-drop (it can flatten folder structure).
**This version doesn't have that failure mode** — CSS and JS are inlined
directly in `index.html`, so if the page loads at all, it will be styled
correctly. If you're still seeing unstyled content, hard-refresh
(Ctrl/Cmd+Shift+R) to bypass a cached old version, and confirm the file
you deployed is this updated `index.html` (check the file size — it should
be roughly 75 KB).

## What's inside

```
index.html          Entry point — self-contained: CSS inlined in <style>,
                     all app logic inlined in <script>. Deploy this one file.
css/style.css        Source copy of the stylesheet, kept for readability while
                       editing — NOT loaded by index.html directly anymore.
js/data.js            Source copy: reference data (5 SSPPC modules, the
                       process/FMS checklist, first-run seed data).
js/storage.js         Source copy: localStorage read/write helpers.
js/auth.js             Source copy: login/session + building-access checks.
js/export.js           Source copy: CSV export + print-to-PDF report.
js/app.js               Source copy: all view rendering, routing, CRUD logic.
```

> The `css/` and `js/` files are kept in the repo as the readable, editable
> "source" — edit them, then copy the changed block back into the matching
> `<style>`/`<script>` section of `index.html` before redeploying. This keeps
> day-to-day editing pleasant while keeping deployment foolproof.

## Features

- **Login & registration** — a dark, blue/green "tech" auth screen with a
  self-service **ลงทะเบียนผู้ใช้ใหม่ (register)** flow. New sign-ups are
  created with `status: "pending"` and cannot log in until an admin reviews
  them from the **ผู้ใช้งาน (Users)** page, assigns a role + building(s), and
  approves.
- **Settings** (gear icon, top of sidebar) — everyone can pick a font (IBM
  Plex Sans Thai / Sarabun / Prompt), a theme (Light / Dark / Ocean
  blue-green), and a font size (Small / Medium / Large); choices persist in
  `localStorage` and apply instantly. Admins additionally see a **module
  visibility** section — unchecking a module (e.g. hide "System" if a
  building doesn't need it) removes it from the sidebar for every user.
- **Language toggle** (button next to Settings) — switches the app's
  navigation, page titles, dashboard chrome, and common buttons between Thai
  and English. Scope: this covers the app's structural UI ("chrome"), not
  user-entered data (OKR objectives, KPI names, CAPA titles, FMS reference
  content) — those stay as entered, same as any real i18n system wouldn't
  auto-translate a user's own text.
- **Buildings / BU** — admins can add any number of buildings or business
  units; every record (process status, OKR, KPI, CAPA) is scoped to one.
- **Users & permissions** — two roles (`admin`, `user`), plus a transient
  `pending` state for unapproved sign-ups; each active user is assigned
  specific buildings or "ทุกอาคาร" (all buildings). Non-admins only see and
  edit data for their assigned buildings.
- **SSPPC modules** — Structure / System / Process / People / Culture, each
  with a checklist of processes (status: ยังไม่เริ่ม / กำลังดำเนินการ /
  เสร็จสมบูรณ์ + a note field). Module 3 — Process includes the 7 JCI FMS
  plans (Safety, Security, Hazardous Materials & Waste, Fire Safety, Utility
  Systems, Emergency & Disaster Management, Construction & Renovation/ICRA).
- **OKR** — quarterly Objectives with multiple Key Results, each with a 0–100%
  progress value; the app rolls these up into an overall progress bar.
- **KPI** — target vs. actual per pillar, with automatic RAG (red / amber /
  green) status based on target direction (higher-is-better or
  lower-is-better).
- **CAPA / Incident log** — open items, due dates, overdue highlighting.
- **Export** — every dataset (process status, OKRs, KPIs, CAPAs, buildings)
  can be exported to CSV (opens cleanly in Excel, UTF-8 BOM included so Thai
  text displays correctly). An "Executive Summary" print view is also
  available for a quick PDF via the browser's print dialog.

## Data model (localStorage keys)

| Key                    | Shape                                                            |
|-------------------------|-------------------------------------------------------------------|
| `qshe_users`            | `{id, username, password, name, role, status, buildingIds[]}[]` (`status`: `active` \| `pending`) |
| `qshe_buildings`        | `{id, code, name, bu}[]`                                          |
| `qshe_processStatus`    | `{id, buildingId, processId, status, note, updatedAt, updatedBy}[]` |
| `qshe_okrs`              | `{id, buildingId, quarter, pillar, objective, owner, keyResults[]}[]` |
| `qshe_kpis`              | `{id, buildingId, pillar, name, target, actual, unit, period, direction}[]` |
| `qshe_capas`             | `{id, buildingId, title, source, owner, openedDate, dueDate, status}[]` |
| `qshe_session`           | current logged-in user (cleared on logout)                       |
| `qshe_uiPrefs`           | `{font, fontSize, theme}` — Settings choices, applied on every page load |
| `qshe_moduleVisibility`  | `{structure, system, process, people, culture}` booleans, admin-controlled |
| `qshe_lang`              | `"th"` \| `"en"` — current UI language |

The static reference list of modules/processes/FMS plans lives in code
(`js/data.js` → `PROCESS_REFERENCE`), not in storage, so upgrading the
checklist is a code change + git commit, which keeps an audit trail.

## ⚠️ Security note — read before deploying beyond a demo

This app's login and all of its data live entirely in the browser's
`localStorage`:

- Anyone with access to the browser's dev tools can read the user list
  (including passwords) and every record.
- There is no server-side validation, no encryption at rest, and no real
  session expiry.
- Data does **not** sync between devices or users — each browser has its own
  independent copy. Use this for a single-user prototype, an offline demo, or
  an internal tool on a trusted kiosk machine, not for multi-user production
  use with sensitive data.

To take this to production, the recommended path is:
1. Replace `js/storage.js`'s localStorage calls with `fetch()` calls to a
   real backend (e.g. a small Node/Express + Postgres API, or a BaaS like
   Supabase/Firebase).
2. Move authentication to that backend with hashed passwords and real
   sessions/JWTs (`js/auth.js` is written so this swap only touches that one
   file).
3. Add server-side authorization checks mirroring `Auth.canAccessBuilding`.

## Extending

- **Add a process/FMS plan**: add an entry to `PROCESS_REFERENCE` in
  `js/data.js` (give it a unique `id`, a `module`, a `name`, and an `owner`;
  add `category: "fms"` + `fmsCode` if it's an FMS plan).
- **Add a module**: add to the `MODULES` array in `js/data.js`, then add a
  matching entry to `NAV_ITEMS` in `js/app.js`.
- **Charting**: the dashboard currently uses plain CSS progress bars to avoid
  a dependency; drop in Chart.js (`<script src="https://cdn.jsdelivr.net/npm/chart.js">`)
  and add a `<canvas>` inside `renderDashboard()` in `js/app.js` if you want
  trend charts.

## License

Internal tool — add your organization's license of choice here before
publishing the repository.


## Master Plan module (added)

Sidebar group **Master Plan** (after the SSPPC modules, can be hidden by admin in Settings):
1. **QSHE Action Plan 2026-2027** – add / edit / delete plans, status, month timeline (Oct 2026 – Dec 2027).
2. **Inspection Risk Summary** – charts (score by category vs 70/85% thresholds, risk-level mix, unit comparison), top risk items, table view, CSV and print.
3. **Risk Assessment Checklist** – add / edit / delete units (หน่วยงาน) and score the 56 template items (1-5 or N/A). Auto-saved.

Scoring follows the Excel: risk 1=Critical, 2=High, 3=Medium, 4-5=Low; category/overall >=85% Pass, >=70% Fair, else Improve. N/A and unscored items are excluded from the percentage. Master Plan data is organisation-level (not per building).

Editing: change files in `css/` and `js/`, then run `python3 build.py` to regenerate the single-file `index.html`.

Photos: each checklist item has 2 photo slots (1 = before / finding, 2 = after correction). Images are resized in the browser and stored in IndexedDB (per browser, not in localStorage); thumbnails also appear in the Inspection Risk Summary's high-risk table.
