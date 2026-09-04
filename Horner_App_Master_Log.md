# Horner Field App — Master Project Log
**Last updated:** 2026-09-04

> **Note:** This is the single master log for all Horner Field App development, including the Tag & Hold / Order Release features. The separate `Order_Release_App_Master_Log.md` has been retired and merged here as of Session 85.

---

## ⚡ Current State

| Field | Value |
|---|---|
| **Current build** | `index_387` |
| **Primary URL** | `https://horner.app` (IIS on HP-APP) |
| **Fallback URL** | `https://itr325.github.io/Horner.app/` (GitHub Pages — redirect only) |
| **Web root** | `C:\inetpub\wwwroot\horner_app\` |
| **App file** | `index.html` (single file, overwritten each build — BUILD_ID constant tracks version) |
| **Dev site** | `https://10.1.1.12:8443` — IIS site "HornerAppDev", web root `C:\inetpub\wwwroot\horner_app_dev\` |
| **API** | `https://horner.app/api/` — ASP.NET Core 8, IIS sub-app, app pool "OrderReleaseApi" (`Horner\sql.readonly`) |
| **API source** | `D:\Projects\OrderReleaseApp\api\` |
| **Tag & Hold page** | `C:\inetpub\wwwroot\horner_app\tagandhold.html` |
| **Master log** | `C:\inetpub\wwwroot\horner_app\Horner_App_Master_Log.md` (primary) — also pushed to GitHub |
| **SharePoint site** | `https://hornerplumbing.sharepoint.com` |
| **Active Projects path** | `/Active Projects` in the Documents library |

---

## ⚠️ Deploy Rules (Critical — learned the hard way)

- **ALL changes go to dev first** (`C:\inetpub\wwwroot\horner_app_dev\`) — never edit production `index.html` directly
- **Never use PowerShell `Set-Content` on index.html** — it mangles emoji/UTF-8. Use `[System.IO.File]::WriteAllText()` with explicit UTF-8 encoding, or push to GitHub and pull down via `Invoke-WebRequest`
- **Filesystem MCP `write_file`** is safe for small files (tagandhold.html, master log) but not for index.html (1.6MB+)
- **Dev → Production deploy path:** Edit dev file → test on `https://10.1.1.12:8443` → push to GitHub → `Invoke-WebRequest` from GitHub to production IIS
- **INCREMENT BUILD_ID ON EVERY DEPLOY** — no exceptions, even one-line fixes. This is what makes rollback possible.

### Backup Strategy
- **Versioned backups on server:** `C:\inetpub\wwwroot\horner_app_backup\` — copy `index.html` as `index_NNN.html` and `tagandhold.html` as `tagandhold_NNN.html` before every session
- **GitHub:** `index.html` and `tagandhold.html` always reflect last known good state — push at end of every session
- **Rollback:** Copy from `horner_app_backup\` or pull from GitHub via `Invoke-WebRequest`
- **Session start:** Always back up current files to `horner_app_backup\` BEFORE making any changes

---

## 🔴 Open Items / To-Do

### Tag & Hold — Email Formatting (in progress)
- [x] API: Fix item sort order (`pl.[Po-no], pl.[Line-no]` — was alphabetical)
- [x] API: Add `GET /api/job/{jobNo}/info` endpoint (returns jobName, address, city, state, zip)
- [x] `tagandhold.html`: Fetch job info on load, include in submit payload
- [x] `tagandhold.html`: Read foreman name + phone from URL params, include in payload
- [x] Flow 28: Rebuilt email body — prose format with PO#, job name, item list, jobsite block, foreman, confirm line
- [ ] **Test end-to-end on dev** — verify email arrives correctly formatted
- [ ] **Push to production** once tested
- [ ] **submittedBy field** — Add logged-in user's name to release email so Aaron knows who submitted (optional, low priority)
- [ ] **iPad testing** — Verify touch targets, scrolling, cart UX on actual iPad

### Field App Features (priority order)
- [ ] **ERP Audit** — Aaron reviewing Horner_Order_Sheet_Audit.xlsx; once returned: build new order sheets, fill gaps, auto-parse descriptions. Goal: eliminate Blank Order Sheet. Pending: Aaron to clarify MegaPress usage (gas-only vs. multi-use)
- [ ] **Project assignment feature** — PAUSED pending Patrick's decision on role/permission structure
- [ ] **Order sheet email formatting** — Flow 21 cart email
- [ ] **Timecard — custom job entry**
- [ ] **Residential Order Sheets**
- [ ] **Service tiles** (both still "Coming Soon")
- [ ] **Admin Panel — Close Job**
- [ ] **Horner Blue (#0156A4) color swap** — replace navy (`#1B3A6B`) throughout; save for dedicated session
- [ ] **Flow 4 (TBT rollover for new year)** — low priority
- [ ] **Employee ID cleanup** — 4 PM-added employees have timestamp IDs from `Date.now()`; fix via Edit Employee form

### App Migration (target late Aug 2026)
- [ ] Disable GitHub Pages
- [ ] Make repo private
- [ ] Remove GitHub Pages redirect URI from Entra app registration

### Backend Migration (future dedicated session)
- [ ] SQL Server Express + ASP.NET Core Web API — full backend replacing Power Automate flows
- [ ] Multi-tenant design from day one (packaging for other plumbing subcontractors)

---

## 🏗 Architecture & Process

### App Structure
- **Single-file HTML app.** React 18 (UMD, CDN). JSX pre-compiled to `React.createElement` — no runtime Babel, no build process.
- **Hosted:** `https://horner.app` (primary, IIS on HP-APP). GitHub Pages serves a meta-refresh redirect only.
- **App file:** `index.html` — single file, overwritten each deploy. `BUILD_ID` constant inside tracks version (currently `index_377`).
- **pdf-lib bundled inline** (line 62, ~524K chars). Never CDN. Use surgical `view` ranges only. Blob-excluding greps: `grep -n '<term>' index.html | awk -F: '$1!=62' | cut -c1-200`
- **PDF.js** loaded from CDN in `<head>`.
- **No service worker.**
- **Tag & Hold** — `tagandhold.html` at web root; accepts `?job=XXX&embedded=1&foreman=NAME&foremanPhone=PHONE`; integrated into Field App via iframe.
- **ASP.NET Core 8 API** at `https://horner.app/api/` — IIS sub-application. Source at `D:\Projects\OrderReleaseApp\api\`. App pool "OrderReleaseApi" running as `Horner\sql.readonly`.

### Deploy Process
- **Dev first, always.** Changes go to `C:\inetpub\wwwroot\horner_app_dev\`, test on `https://10.1.1.12:8443`.
- **Production deploy:** Push to GitHub → PowerShell `Invoke-WebRequest` from raw GitHub URL to `C:\inetpub\wwwroot\horner_app\index.html`.
- **Never `Set-Content` on index.html** — use `[System.IO.File]::WriteAllText()` or Invoke-WebRequest from GitHub.
- **BUILD_ID** (`const BUILD_ID = "index_NNN"`) must be incremented on every deploy.
- **Dev site API:** `C:\inetpub\wwwroot\horner_app_dev\api\` — IIS sub-app under "HornerAppDev", same app pool.
- **tagandhold.html and API changes** can be written directly via Filesystem MCP (small files, no emoji risk).

### Session Protocol
**START of every session:**
1. Read `C:\inetpub\wwwroot\horner_app\Horner_App_Master_Log.md` via Filesystem connector
2. Check open items and **Next session** notes
3. Back up current `index.html` to `C:\inetpub\wwwroot\horner_app_backup\index_NNN.html` and `tagandhold.html` to `tagandhold_NNN.html` BEFORE any changes
4. Update memory if anything has changed

**END of every session:**
1. Prepend new session summary to top of log (newest first)
2. Write updated log to server via Filesystem connector
3. Push log, `index.html`, and `tagandhold.html` to GitHub via API

### Key Infrastructure
- **IIS server:** HP-APP, 10.1.1.12, Windows Server 2025
- **Web root:** `C:\inetpub\wwwroot\horner_app\`
- **Dev web root:** `C:\inetpub\wwwroot\horner_app_dev\`
- **FortiGate:** 98.103.132.245, VIP → 10.1.1.12:443
- **SSL cert:** Starfield TLS (GoDaddy CA), valid to 1/10/2027
- **Entra App Registration:** Client ID `282e84c6-5e0b-4a7d-a58b-a773215c30b0`, Tenant ID `ef466c74-7a13-4920-854c-210669ea3c84`
- **Entra security groups:** Field (`b4f1695d`), Office (`cd7ef8cc`), PM (`b63d98a5`), Admin (`484ad2f7`)
- **MSAL:** `cacheLocation: "localStorage"` — do NOT revert to sessionStorage (iOS evicts on screen lock)
- **Filesystem MCP:** Scoped to `C:\inetpub\wwwroot\horner_app\` — dev site requires PowerShell
- **GitHub repo:** `itr325/Horner.app` — master log + index.html backup; PAT in `C:\inetpub\wwwroot\horner_app\github.token`
- **GE ERP:** SQL Server at `HP-SQL\GE`, database `Service`; domain account `Horner\sql.readonly`
- **Custom MCP server:** `D:\Projects\OrderReleaseApp\mcp-server\index.js` — Node.js, provides `powershell` and `dotnet` tools to Claude Desktop on HP-APP; config at `%APPDATA%\Claude\claude_desktop_config.json`
- **Backup folder:** `C:\inetpub\wwwroot\horner_app_backup\` — versioned copies of index.html and tagandhold.html
- **web.config:** `C:\inetpub\wwwroot\horner_app\web.config` — IIS config (MIME types, rewrite rules)
- **SVG assets:** `H.svg`, `horner-plumbing-logo.svg` in web root — Horner "H" logo files used in the app header
- **journal.txt:** `itr325/Horner.app` GitHub repo — automated session journal from May 2026 (pre-master log era, builds 99-113); kept as historical artifact

### GE ERP — Key Tables
| Table | Key Fields |
|---|---|
| `dbo.po` | `Po-no`, `Vendor-code`, `Job-no`, `Status-code` (p/P=open, C=closed, x=cancelled, r=received) |
| `dbo.[po-line]` | `Po-no`, `Line-no`, `Description__1/2`, `Qty-orig-ord`, `Qty-received`, `Qty-to-rcve`, `Task-no` (= Phase number), `Status-code` |
| `dbo.task` | Phase name lookup (e.g., Phase 6 = FINISH PLUMBING) |
| `dbo.vendor` | `Vendor-code`, `Name`, `E-Mail__1` through `E-Mail__5` |
| `dbo.job` | `Job-no`, `Description`, `Address`, `City`, `St`, `Zip-code`, `Forman` (numeric ID) |

### Key Architecture Decisions (do not revert)
- `SHOW_DIAG=false` — do not re-enable in production
- MSAL `cacheLocation: "localStorage"` — iOS fix
- Touch events (not pointer events) in `PdfMarkupView` — iOS multitouch fix
- PM driven by `EMPLOYEES` filtered by `position === "Project Manager"` via `getPMs()`/`getPmEmail()` helpers
- Foreman/Project Lead stored as plain full-name string
- All employee dropdowns sort by `firstName`
- Employee CSV parser uses simple `split(",")` — NOT regex
- `AdminEmployeesView` re-fetches from SharePoint on every open
- All `.json` files hidden from folder file listings
- Cart persists via `shop_cart.json` in SharePoint
- IIS controller routes use `[controller]` (not `api/[controller]`) — IIS mounts app at `/api`

### Power Automate Rules (gotchas)
- Always specify plain paste vs. Expression mode for every field
- SharePoint `{FullPath}` returns paths WITHOUT leading slash — prepend `/`
- Use `GetFileByServerRelativeUrl` (not `GetFileByServerRelativePath` — rejects paths with parentheses)
- Folder naming: `(CODE) Name` — concat: `concat('/Active Projects/(', code, ') ', name, '/...')`
- `base64ToBinary()` required for Office 365 email attachments
- Trigger "Who can trigger": must be "Anyone" (not "Anyone in my tenant") — app gets 401 otherwise
- Flow URL security deferred to Logins — `flowPost()` wrapper in place. Do NOT add secret conditions to flows.

---

## 📌 Power Automate Flow Reference

| # | Flow Name | Purpose | URL constant |
|---|---|---|---|
| 1 | Flow 1 - Create New Project Folder | Creates job folder + subfolders | `CREATE_JOB_URL` |
| 2 | Flow 2 - Syncing Projects Folders | Returns `[{code, name, pm, foreman, app}]` | `PROJECTS_FETCH_URL` |
| 3 | Flow 3 - Update PM | Updates PM column | `UPDATE_PM_URL` |
| 4 | Flow 4 - Change Foreman | Updates Foreman column | `UPDATE_FOREMAN_URL` |
| 5 | Flow 5 - Fetch Folder Contents | Returns `{folders, files}` for any path | `FETCH_FOLDER_CONTENTS_URL` |
| 6 | Flow 6 - Camera Upload | Uploads photos/videos | `UPLOAD_FILE_URL` |
| 7 | Flow 7 - PDF Editor - Get PDF File | Proxies SharePoint file as base64 | `DOWNLOAD_FILE_URL` |
| 8 | Flow 8 - PDF Markup Save | Uploads annotated PDF as `_vN` | `SAVE_MARKUP_URL` |
| 9 | Flow 9 - Save Valve Tags | Writes valve_tag_list.json | `SAVE_VALVE_TAGS_URL` |
| 10 | Flow 10 - Load Valve Tags | Reads valve_tag_list.json | `LOAD_VALVE_TAGS_URL` |
| 11 | Flow 11 - Save Signed TBT PDF | Saves signed TBT, deletes original | `SAVE_TBT_URL` |
| 12 | Flow 12 - Toolbox Talks Rollover | Creates next-year TBT folder | `ROLLOVER_URL` |
| 13 | Flow 13 - Codes & Charts Sync | Returns library items | `CODES_CHARTS_FETCH_URL` |
| 14 | Flow 14 - Admin Docs Sync | Returns library items | `ADMIN_DOCS_FETCH_URL` |
| 15 | Flow 15 - Load Employees | GET employees.csv | `LOAD_EMPLOYEES_URL` |
| 16 | Flow 16 - Save Employees | POST employees.csv | `SAVE_EMPLOYEES_URL` |
| 17 | Flow 17 - Save Photo Markup | Overwrites original photo | `SAVE_PHOTO_MARKUP_URL` |
| 18 | Flow 18 - Submittal Notifications | Emails PM + CC Foreman on file changes | — (trigger-based) |
| 19 | Flow 19 - Save Cart | Writes cart.json | `SAVE_CART_URL` |
| 20 | Flow 20 - Load Cart | Reads cart.json | `LOAD_CART_URL` |
| 21 | Flow 21 - Send Cart Email | Sends HTML order sheet cart email | `SEND_CART_URL` |
| 22 | Flow 22 - Send Timecard Email | Sends HTML timecard email | `SEND_TIMECARD_URL` |
| 23 | Flow 23 - Send Feedback Email | Sends feedback email | `SEND_FEEDBACK_URL` |
| 24 | Flow 24 - Get File Content | Returns raw file content | `GET_FILE_CONTENT_URL` |
| 25 | Flow 25 - Get Residential Jobs | Returns open residential job numbers | `GET_RES_JOBS_URL` |
| 28 | Flow 28 - Tag & Hold Release | HTTP trigger → prose email per vendor → Aaron | wired into `tagandhold.html` |

---

## 📋 Session History

### Session 86 — Tile Tweaks
**Date:** 2026-08-15
**Build:** index_380 → index_381

**Summary:** Converted all remaining list-view folder overlays to tile grid layout. Added direct-open shortcuts for single-form folders.

**Tile grid conversions:**
- **FORMS** — 3 tiles: Daily Field Report (📋), RFI Form (📄), RFC Form (📝)
- **RETURNS** — 5 tiles: BLK-GAL-BRA, CPVC, PEX, Finish, MISC (all 🔄, color-coded backgrounds)
- **T&M** — 1 tile: T&M Log (⏱️)
- **SAFETY** — 2 tiles: Site Safety Inspection (🧺), Toolbox Talks (📖 — opens subfolder)
- **VALVE TAG** — 1 tile: Valve Tag List (🏷️)

**Direct-open shortcuts (top-level job folder grid):**
- **T&M tile** — now opens T&M Log form directly, skips folder view
- **VALVE TAG tile** — now opens Valve Tag List form directly, skips folder view

**Subfolder hiding:**
- Toolbox Talks subfolder hidden from SAFETY folder listing (tile handles navigation)

**Technical:**
- Generic heading + list renderer preserved as fallback for any future folders without custom tile blocks
- All tile blocks use consistent styling: white bg, 1.5px #DDE3ED border, 12px radius, 46x46 icon circle, 10px bold label
- JS validation passed on all edits

**Next session:** Field crew testing feedback. Continue with ERP Audit when Aaron returns spreadsheet.

---

### Session 85 — Log Merge + Email Formatting + Production Incident + Recovery
**Date:** 2026-08-14 → 2026-08-15
**Build:** index_377 → index_378

**Summary:** Merged logs, rebuilt Tag & Hold email formatting end-to-end, caused and recovered from production corruption incident, established backup strategy.

**Completed:**
- Merged Order Release App log into single Horner Field App master log
- API: Fixed item sort order to `pl.[Po-no], pl.[Line-no]`
- API: Added `GET /api/job/{jobNo}/info` endpoint
- API: Deployed to both production and dev IIS sub-apps
- Dev API: Created separate `OrderReleaseApiDev` app pool, set `Horner\sql.readonly` credentials
- `tagandhold.html`: Job info fetch on load; foreman/phone from URL params; embedded mode fixed (hides title, keeps Cart button); vendor name in submit payload
- `index.html`: Foreman name + phone passed as URL params to tagandhold iframe
- Flow 28: Rebuilt email — vendor name header, prose intro with PO# + job name, monospace item list with qty + description, jobsite block, foreman/phone, confirm line, HR separator per vendor
- Email tested and confirmed working end-to-end on dev then pushed to production
- Backup strategy established: `C:\inetpub\wwwroot\horner_app_backup\`, GitHub mirrors, BUILD_ID on every change

**Production incident:**
- Cause: PowerShell `Set-Content` on 1.6MB index.html mangled UTF-8 emoji
- Recovery: Restored from server backup (3:07PM backup from backup server rebuild)
- Lesson: NEVER `Set-Content` on index.html; always use `[System.IO.File]::WriteAllText()` or Invoke-WebRequest from GitHub

**Deploy rules updated:**
- Dev first, always — test before pushing to production
- INCREMENT BUILD_ID on every change, no exceptions
- Back up before every session

**Next session:** UI tweaks — convert Forms list view (Daily Field Report, RFI Form, RFC Form) to tile grid layout matching rest of app. Also review other list views for same treatment.

### Session 85 — Log Merge + Email Formatting (partial) + Production Incident
**Date:** 2026-08-14

**Summary:** Merged Order_Release_App_Master_Log.md into this log. Made Tag & Hold email formatting changes (API, tagandhold.html, Flow 28). Caused and recovered from a production `index.html` corruption incident.

**Log merge:**
- Order Release App master log retired — all context absorbed here
- Current State table updated (was stale at index_329)
- Deploy workflow clarified and documented

**Tag & Hold email formatting (partially complete):**
- API: Fixed item sort order to `pl.[Po-no], pl.[Line-no]` (was alphabetical)
- API: Added `GET /api/job/{jobNo}/info` endpoint — returns jobName, address, city, state, zip from GE
- API published to production and dev IIS sub-apps
- `tagandhold.html`: Fetches job info on load; reads foreman/foremanPhone from URL params; includes both in submit payload
- `index.html` (dev): Updated iframe src to pass `&foreman=NAME&foremanPhone=PHONE` URL params
- Flow 28: Rebuilt email body — prose format per vendor: PO# + job name + delivery blank, item list, jobsite block, foreman/phone, confirm line, HR separator
- **NOT YET TESTED** — production incident interrupted before test

**Production incident:**
- Cause: Used PowerShell `Set-Content` to write 1.6MB `index.html` — mangled UTF-8 emoji throughout file
- Recovery: User uploaded pre-corruption `index.html` (build 377) → pushed to GitHub via Python → pulled to server via `Invoke-WebRequest`
- Result: Production restored to build 377, emoji intact, no functionality lost
- **Lesson learned:** NEVER use `Set-Content` or `Set-Content -Encoding UTF8` on index.html. Use `[System.IO.File]::WriteAllText()` or Invoke-WebRequest from GitHub. ALL changes to index.html go to DEV first.

**Dev environment:**
- Dev site now has API sub-app at `C:\inetpub\wwwroot\horner_app_dev\api\`
- Dev index.html and tagandhold.html updated with foreman URL param changes

**Next session:** Test Tag & Hold email end-to-end on dev, then push to production

---

### Session 84 — Tag & Hold Integration
**Date:** 2026-08-13

**Summary:** Added Tag & Hold tile to the Field App. API deployed to IIS as sub-app. Promoted to production as build 373 (later updated to 377 with tweaks).

**Changes:**
- TAG & HOLD tile added to job folder grid — opens `/tagandhold.html?job={code}&embedded=1` in iframe
- iframe positioned below app header (top: 90px), fills remaining viewport
- Breadcrumb: Home > Commercial > Active Projects > JOB > Tag & Hold
- `tag-and-hold` added to `formViews` array so Back button works
- tagandhold.html hides its own header in embedded mode
- Release All button added (gold, next to phase title) — adds all items at full qty
- Phase card grid widened (140px → 170px min) — fixes overflow on long phase names
- API published to `C:\inetpub\wwwroot\horner_app\api\` as IIS sub-application
- App pool "OrderReleaseApi" as `Horner\sql.readonly`, No Managed Code
- Flow 28 wired into tagandhold.html

**Dev environment stood up:**
- IIS site "HornerAppDev": HTTP 8080, HTTPS 8443
- Web root: `C:\inetpub\wwwroot\horner_app_dev\`
- Self-signed cert "Horner App Dev" (thumbprint 4B83FCBC..., 5-year expiry)
- Entra redirect URI `https://10.1.1.12:8443` added

---

### Session 81 — MSAL localStorage Fix
**Date:** 2026-07-17
**Build:** index_363

**Summary:** Fix persistent re-authentication bug — changed MSAL token cache from `sessionStorage` to `localStorage`.

**Fix:**
```js
cache: { cacheLocation: "localStorage", storeAuthStateInCookie: false }
```

---

### Session 80 — Copper Order Sheets + ERP Audit
**Date:** 2026-07-16
**Builds:** index_361 → index_362

**Summary:** Two new order sheets from ERP inventory dump. Scroll-to-top fix on all navigation. ERP audit spreadsheet built and sent to Aaron.

**Changes:**
- **Copper Solder Fittings** — 163 items, 11 size sections (1/4" through 4")
- **ProPress Fittings** — 114 items, 9 size sections (1/2" through 4")
- Both sheets: copper color (#B87333), generated via Python from ERP dump
- Scroll-to-top: `window.scrollTo(0, 0)` in `go()` and Back button handler

**ERP Audit:**
- 7,597 items analyzed; 1,570 field-relevant items not on any order sheet (34 categories)
- 96 app items with part numbers not in ERP — Aaron to verify
- Horner_Order_Sheet_Audit.xlsx sent to Aaron — awaiting return
- MegaPress deferred pending Aaron clarification (gas-only vs. multi-use)

---

### Session 79 — Pull-to-Refresh, Back Button, Inactivity Reload
**Date:** 2026-07-16
**Builds:** index_356 → index_360

**Summary:** Pull-to-refresh (global), styled Back button, Reload buttons removed, larger logo, inactivity auto-reload.

**Changes:**
- Pull-to-refresh: touch listeners, fires full reload on >72px pull from top. No indicator.
- Back button: frosted white pill, 14px chevron SVG, "Back" text
- Removed all Reload App / Refresh Projects buttons (kept empty-state refresh only)
- Header logo height: 26 → 38px
- Inactivity auto-reload: Page Visibility API, 10-minute threshold, resets on touch/click

---

### Session 72 — MSAL Auth (incomplete)
**Date:** 2026-06-29
**Build:** index_327 → index_329

**Summary:** Attempted to gate Admin Panel behind Entra ID login. index_328 broken (eager MSAL init crash). index_329 fixed with lazy init. Now resolved — current build is 377.

---

### Session 71 — IIS Migration
**Date:** 2026-06-29
**Build:** index_327

**Summary:** Full IIS migration. DNS, SSL cert (Starfield, full chain fix), web.config, Filesystem MCP connector, GitHub webhook + hourly sync. horner.app live.

---

### Session 70 — Form Fixes
**Date:** 2026-06-29
**Build:** index_323 → index_327

**Summary:** Removed form tile dots. Fixed blank renders for 7 views.

---

### Session 69 — UI Cleanup + Project Card Fixes
**Date:** 2026-06-26
**Build:** index_321 → index_323

**Summary:** Folder icon removed from project cards. Dynamic folder titles. Project card auto-grow. Project Lead badge wraps on narrow screens.

---

### Session 68 — Form Row Cleanup
**Date:** 2026-06-26
**Build:** index_320 → index_321

**Summary:** Removed clipboard icons, "Fillable form" subtitles, static headings across all form list views and Shop Orders.

---

### Session 67 — Order Sheet Additions
**Date:** 2026-06-26
**Build:** index_316 → index_320

**Summary:** Pipe length buttons (CPVC, PVC, SCH80). PEX Ball Valves section. Case Quantity sheet removed. Size button style standardized. PVC Foam Core pipe lengths. Copper Order Sheet (Type M, L, Level-Wound). Copper color size buttons.

---

### Session 66 — Residential Timecard + Calendar
**Date:** 2026-06-26
**Build:** index_305 → index_316

**Summary:** Residential timecard job# search fixes (4-char threshold, blur/refocus). Custom CalendarPicker replacing native date input.

---

### Session 65 — Cart Email + Submit Fix
**Date:** 2026-06-25
**Build:** index_301 → index_304

**Summary:** Cart email line items sorted by material sheet order. Fixed broken Submit Cart button.

---

### Order Release App — Session 1 (now merged)
**Date:** 2026-08-06

**Summary:** Project kickoff. Architecture decisions, environment setup, GE database discovery.

**Infrastructure completed:**
- `D:\Projects\OrderReleaseApp\` created
- Custom MCP server (Node.js) for PowerShell + dotnet tools
- .NET 8 SDK 8.0.423 installed
- ASP.NET Core Web API scaffolded
- Microsoft.Data.SqlClient 7.0.2 installed
- SQL connection to `HP-SQL\GE` / `Service` verified
- GE schema mapped (po, po-line, vendor, job, task tables)

---

### Session 86 — Timecard Send Confirmation Overlay
**Date:** 2026-09-01
**Build:** index_384 → index_388

**Summary:** Added full-screen overlay feedback to both Commercial and Residential timecard flows. Triggered by a foreman submitting a timecard and not knowing whether it went through — the old 3.5-second toast was too easy to miss, and if the network dropped, the form data was already gone.

**Changes (builds 385–387):**
- **TimecardSendOverlay component** — shared by both timecard views. Three states:
  - **Sending:** Spinner + "Sending Timecard… Please wait" (blocks interaction)
  - **Sent:** Green checkmark + "Timecard Sent! / Email delivered to payroll" + OK button (auto-dismisses 3s)
  - **Failed:** Red X + "Timecard Failed to Send / Your timecard data is still here. Tap Try Again to resend." + Try Again button
- **Form data preserved until success** — `setPreview(null)` and `reset()` moved into `.then()` so on failure the EmailPreview is still there with all entered data; foreman taps Try Again → Send to retry
- **`response.ok` check** — `.then()` now checks HTTP status; non-2xx throws into `.catch()` so flow errors show the red X, not the green checkmark
- **CSS:** `@keyframes tcSpin` for spinner animation
- **Flow 22 (Power Automate):** Added Response action (200, `{"status":"sent"}`) at end of flow so the HTTP trigger holds the connection open until the email actually sends. App spinner now reflects real flow duration.

- **`inputMode: "decimal"` on commercial timecard hours inputs** — iPad now shows numeric keypad instead of full keyboard when entering hours (residential already had this)

**Tested:** WiFi-off failure → red X + data preserved → WiFi on → Try Again → Send → green checkmark. Flow disabled → error path confirmed.

**Next session:** TBD

---

### Session 87 — 2026-09-03
**Builds:** 389 → 390 → 391 → 392 → 393 → 394 → 395 (production: 395)

**Additional Lead (AL) feature — full pipeline:**
- Added `additionalLead` field to Create New Job form (below Project Lead, optional)
- Purple AL badge on project list cards and detail header (conditional — only shows when assigned)
- Added `additionalLead` to project normalizer so it persists across syncs
- "Change Additional Lead" section added to Admin panel (Change PM / PL / AL)
- SharePoint: `AL` and `AL_Email` columns added to Documents library
- Power Automate flows updated:
  - Create Job flow: writes `additionalLead` to AL column, trigger schema updated
  - Fetch Projects flow: Select action maps `additionalLead` from AL column
  - New Update AL flow created (workflow `41728d4256584f5ea85ff2809efea317`)
- `UPDATE_AL_URL` constant added; tile and breadcrumb labels updated to "Change PM / PL / AL"

**Pipe lengths — QTY → Ft label:**
- All 7 pipe sections (BLK pipe, Type M/L copper, CPVC, Foam Core, SCH 40, SCH 80 CPVC) now show "Ft" instead of "QTY" via `qtyUnit: "Ft"` property
- Column header changed from `QTY (unit)` to just the unit name
- Pipe length selector (10'/20') unchanged

**Sticky jump bar fix:**
- Section tab bar on all order sheets now sticks below breadcrumb (was hidden behind it on scroll)
- Added `CRUMB_H` constant and `getCrumbH()` function for dynamic breadcrumb height measurement
- Handles multi-line breadcrumbs (e.g. BLK-GAL-BRA with long path)
- Both regular and case-qty jump bars + scroll offsets updated
- Added `className="crumb-bar"` to breadcrumb for DOM measurement

**File share button (📤):**
- Added to all FileRow instances — downloads file via flow, calls `navigator.share()` for native iOS share sheet (Save to Files, Mail, Messages, AirDrop)
- Also added to PDF viewer slide-out toolbar — uses already-loaded PDF bytes, no re-download
- Desktop fallback: browser download

**Valve Tag List — PDF submit + SharePoint save:**
- Submit now requires Address and Project Lead (can still save without them)
- Generates PDF client-side using pdf-lib (already bundled)
- Sends HTML email with PDF attachment via new Power Automate flow (workflow `96188f931b834045aab1fbe36bce40b5`)
- Saves PDF to project's VALVE TAG folder in SharePoint
- Replaced old `mailto:` approach with flow-based email
- `SEND_VALVE_TAG_URL` constant added

**Project tile reorder:**
- Project root: PLANS → ORDER SHEETS → TAG & HOLD → TIME CARDS appear first, rest alphabetical
- Tag & Hold tile injected after ORDER SHEETS via flatMap (not a SharePoint folder)
- TAG & HOLD added to FOLDER_STYLES with 📦 icon and #2E86C1 color

**Time Cards tile cleanup:**
- Commercial Time Card tile now first, History second
- Site Safety Inspection tile removed (redundant — exists in SAFETY folder)
- Header changed from "Time Cards / Site Safety Inspection" to "Time Cards"

**How To presentation:**
- Built 10-slide PowerPoint + PDF for Patrick meeting covering Tag & Hold, Timecards, Order Sheets, Adding a New Job, badge reference

**Next session:** Finish AL badge testing (verify Power Automate fetch flow returns additionalLead). Tile rearranging if more needed. ERP Audit when Aaron returns spreadsheet.

---

### Session 86 — 2026-09-04 (Build 396)

**Valve Tag List — fillable tag numbers + add-row buttons:**
- TAG # column changed from auto-numbered (i+1) to editable input field
- New `upTagNum(i, v)` function: typing a number auto-fills all rows below with incrementing values (type 5 in row 1 → row 2 gets 6, row 3 gets 7, etc.)
- `mkRow()` updated with `tagNum: ""` field
- All references guarded with `(r.tagNum || "").trim()` to handle legacy saved data from SharePoint that lacks the field (was causing blank-screen crash)
- Email HTML and PDF generation updated to use `r.tagNum` instead of sequential index
- `hasData` and `canSubmit` checks updated to include `tagNum`
- Input uses `type="number"`, `inputMode="decimal"`, `pattern="[0-9]*"` — iPhone shows numeric pad; iPad always shows full keyboard (iPadOS limitation, not a bug)
- Replaced single "Add 20 More Rows" button with three buttons: + 5 Rows, + 10 Rows, + 20 Rows in a flex row with gap and total count label
- `addRows()` parameterized to `addRows(count)`

**Next session:** ERP Audit when Aaron returns spreadsheet. App Migration remaining steps when Eric says go. AL badge testing.
