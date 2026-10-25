# Task Manager — signage consultancy daily task tracker

Single self-contained HTML app: `signage-task-tracker.html` (vanilla JS + CSS, no framework, no build step). Open it in a browser to run it.

## Company context
- Signage consultancy with two teams only: **Design** (schematic/concept design, wayfinding, 3D models) and **Technical** (detail design, sign placement, BIM modelling, BOQ, message schedules). No site team.
- Roles: **Admin** (full access, manages users/PINs, activity log) and **Manager** (edits schedules and tasks with a PIN). Signed-out viewers are view-only. Demo PINs: Admin 0000, managers 1234 (demo only).

## Pages
1. **Dashboard**, top to bottom:
   - Overview: blue Today card and KPI cards (Tasks, Completed with % bar, In progress, Leave days), plus the Task status mix bar.
   - Search records and the All records table, joined in one card. The search filters every module.
   - Working days by project and the Submissions log, side by side:
     - Working days by project shows Project (no. / name / PM), Design, Tech. and Total person-days. A person-day is split equally across the projects worked that day, and a half-day leave counts as 0.5. The table is sorted by the nearest submission (due) date and the Project header has a funnel filter.
     - The Submissions log is grouped by date, with today highlighted. It has funnel filters on Date, Project no., Name, Manager and Status (Pending / Submitted).
2. **Task Manager** (monthly board):
   - One module per day, with a Name column coloured by team.
   - Up to 3 tasks per person per day, each with project no. → name auto-filled, task and status.
   - LEAVE (full or half-day) and a SUBMISSIONS row: project no. → name, item, Manager (auto from the project's PM) and Sent to.
   - A Download Excel button exports the month in the board layout plus Task list and Submissions sheets, using a hand-written XLSX zip generator.
3. **Team & Projects** drawer: staff, managers list, and projects with no., name, PM, due date and status.

## Rules / preferences
- Light theme only, Autodesk-style: light grey sidebar, blue `#0696d7` accents. No dark mode.
- Project status: Active (red light), On Hold (amber), Completed (green).
- Task status: Not Started, In Progress, Internal Review, Completed, On Hold. There is no "Submitted" task status.
- No work type, hours or workload charts.
- Project number comes first in editors and auto-fills the name. Invalid numbers are blocked.

## Data
- Stored in browser `localStorage`:
  - `signage-tracker-v5` holds {staff, projects, reports, managers, users, log}.
  - `signage-tracker-saved-v5` holds saved filters.
  - `signage-tracker-prefs-v1` holds prefs.
  - `signage-tracker-auth-v2` holds the signed-in user.
- Migrations run at load.
- Data is per browser. The suggested next step is a shared database so the whole team sees the same data.

## PWA (installable app)
- `manifest.webmanifest`, `sw.js`, and the icons in `icons/` sit next to the HTML. The page links them and registers the service worker only over http(s).
- Service worker: pages are network-first, falling back to a cached copy when offline. Icons, manifest and Google Fonts are cache-first. Bump `VERSION` in `sw.js` when changing cached static files.
- The sidebar has an "Install app" button, shown when the browser offers install, plus an iOS hint for "Add to Home Screen". Offline and online toasts tell the user when the connection changes.
- Data is localStorage, so the app works fully offline.

## Deployment
- Live at **https://tasks.nizamrazak.tech** on the Hostinger VPS (id 2092970, 187.77.136.7, Ubuntu). The DNS A and AAAA records for `tasks` point at the VPS.
- The host runs Caddy as a systemd service with `/etc/caddy/Caddyfile`. That file has a `tasks.nizamrazak.tech` block that does `reverse_proxy 127.0.0.1:8085` with automatic Let's Encrypt HTTPS. The backup of the original file is `/etc/caddy/Caddyfile.bak-before-tasks`. The same file also serves keradrive.nizamrazak.tech; don't break it.
- `docker-compose.yml` is the hPanel Docker project "task-manager". On every start it downloads the `main` branch tarball, copies the HTML to `/srv/index.html` along with the manifest, sw.js and icons, and runs Caddy with `web/Caddyfile` on 127.0.0.1:8085.
- To update the live site, push to `main` and then restart the "task-manager" project, either in hPanel Docker Manager or with the API `vps_docker_restart`.
