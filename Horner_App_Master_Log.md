### Session 71
**Date:** 2026-06-29
**Build:** index_327 (no new build — infrastructure session)

**What we did — IIS Migration Complete:**
- **DNS:** Registered `horner.app` domain; added A record pointing to 98.103.132.245 (FortiGate public IP)
- **IIS site:** Created "Horner Field App" site bound to `horner.app` on port 443 via IIS Manager
- **SSL cert:** Starfield TLS cert (GoDaddy CA, valid to 1/10/2027) bound to site. Full chain fix details: (1) Diagnosed chain — intermediate (Starfield TLS Intermediate CA DV - R1v1) was present but root (Starfield TLS Root CA - R1) was untrusted on the machine. (2) Downloaded root cert from https://certs.starfieldtech.com/repository/sf_tls_root-r1.crt.pem. (3) Verified thumbprint ED1BED9C312B7783B0E3EF9DAEE9C642ECB86937 (self-signed, valid to 2040) before installing. (4) Installed into Cert:\LocalMachine\Root. (5) Second issue: IIS was not serving the intermediate cert to external clients — rebuilt certificate binding with full chain so browsers can verify without having the intermediate cached. iOS Edge now connects cleanly with green padlock. ChainBuildSucceeded: True confirmed via PowerShell.
- **web.config:** Created with no-cache headers (Cache-Control, Pragma, Expires). Fixed duplicate .json mimeMap error (removed redundant entry — already defined at server level).
- **index.html deployed:** Current build (index_327) copied to `C:\inetpub\wwwroot\horner_app\index.html` and confirmed loading at https://horner.app ✅
- **GitHub webhook + auto-deploy:** Claude Code built webhook.ps1 (HMAC-SHA256 verified GitHub push receiver on https://horner.app/gh-webhook, port 443 shared with IIS via http.sys), sync.ps1 (downloads index.html from GitHub raw, deploys only if changed), HornerAppWebhook scheduled task (boot-start, SYSTEM, auto-restart), HornerAppSync hourly fallback task.
- **Filesystem MCP connector:** Added Filesystem connector in Claude Desktop pointed at `C:\inetpub\wwwroot\horner_app` with full access. Claude can now write directly to the server from this chat — no GitHub middleman needed for deployment!
- **New deployment workflow:** Build index.html here → Claude writes directly to `C:\inetpub\wwwroot\horner_app\index.html` → instantly live on horner.app. GitHub remains source control/backup. No more GitHub Pages cache delay.
- **Master log + memory:** Moved to `C:\inetpub\wwwroot\horner_app\Horner_App_Master_Log.md` on the server. Claude reads/writes it directly via Filesystem connector each session.
- **GitHub Pages:** Kept live at https://itr325.github.io/Horner.app/ as fallback (repo made public again after brief private period that broke field crew access).

**Infrastructure state:**
- horner.app → IIS on HP-APP (10.1.1.12) ✅
- SSL cert valid, full chain serving correctly ✅
- Claude has direct filesystem write access to web root ✅
- GitHub webhook fires on every push to main ✅
- Hourly fallback sync task running ✅

**Next session:** Azure AD / Entra authentication — MSAL.js SSO login, AD groups synced from on-prem DC, role-based access control. Eric manages Horner 365 tenant. Will create App Registration in Entra (Client ID + Tenant ID), create AD groups on-prem and sync to Entra, then implement MSAL login gate + role-based UI.

---

### Session 70
**Date:** 2026-06-29
**Build:** index_323 → index_327

**What we did:**
- **Remove form tile dots** (index_324): Removed the small blue dot (●) indicator from the top-right corner of folder tiles that contain fillable forms. Also removed the matching legend line ("● Blue dot = fillable forms available") from the project detail view.
- **Fix blank form renders** (index_325–327): ValveTagList, DailyLog, TMLog, SiteWeeklySafety, ResidentialTimecardView, homeResidential, and homeService were all missing their React.createElement calls from the main app return — blank screens on tap. Pre-existing bug, likely from the form tile UI refactor in index_321. All seven now render correctly. homeResidential shows Time Cards (working) + Order Sheets (Coming Soon); homeService shows both as Coming Soon.

**Next session:** IIS migration — move app off GitHub Pages to horner.app on company IIS server.

---

### Session 69
**Date:** 2026-06-26
**Build:** index_321 → index_323

**What we did:**
- **UI cleanup** (index_321): Removed folder icon from Active Projects project cards. Replaced static "Fillable Forms" heading with dynamic centered titles per folder (Forms, Order Forms, Returns, Time Cards / Site Safety Inspection, T & M, Safety, Valve Tag List). Removed 📋 clipboard icon and "Fillable form · submits via Outlook" subtitle from all form rows. Shop Orders view: heading → "Shop Order Forms" centered, icon + subtitle removed.
- **Project card auto-grow** (index_322): Cards grow in height to fit content. Project name wraps freely (removed nowrap/ellipsis). Right column top-aligned. Fixed index_322 push conflict via git rebase.
- **Project Lead wraps** (index_323): Removed flexShrink:0 from right column, added maxWidth:55% so Project Lead badge wraps on narrow phone screens instead of squishing the job code.
- **IIS migration discussion:** Eric built an IIS server and got SSL cert installed. Domain is horner.app. Migration planned for next session.

**Next session:** IIS migration — move app off GitHub Pages to horner.app on company IIS server.

---

### Session 68
**Date:** 2026-06-26
**Build:** index_320 → index_321

**What we did:**
- **Active Projects list** (index_321): Removed the navy folder SVG icon from the left of each project card in the Active Projects list.
- **Form list headings** (index_321): Replaced the static "Fillable Forms" left-aligned small-caps heading with a dynamic centered title based on `pathKey`.
- **Form row icon removed** (index_321): Removed the 📋 clipboard icon div from every form list row across all folders.
- **"Fillable form" subtitle removed** (index_321): Removed the "Fillable form · submits via Outlook" subtitle from every form row.
- **Shop Orders view** (index_321): Changed "Shop Forms" heading to "Shop Order Forms" (centered); removed 📋 icon and subtitle from the Shop Order Sheet row.

**Next session:** TBD

---

### Session 67
**Date:** 2026-06-26
**Build:** index_316 → index_320

**What we did:**
- **Pipe length 10'/20' buttons** (index_317): Added `sizeOpts`/`upcBySize` pattern to CPVC Pipe, PVC SCH40 Solid Pipe, and SCH80 CPVC Pipe sections.
- **PEX Ball Valves** (index_317): New section added to PEX Order Sheet before CPVC Ball Valves — 5 sizes (1-1/2" through 1/2") with UPONOR UPCs.
- **Case Quantity Reference Sheet removed** (index_317): Full removal.
- **Size button style standardized** (index_318): All `sizeOpts` buttons now match BLK/GAL/BRA MatToggle style.
- **PVC Foam Core Pipe** (index_318): Added 10'/20' buttons for all 7 sizes.
- **Copper Order Sheet** (index_319): New order sheet — Type M, Type L, Level-Wound sections.
- **Copper color size buttons** (index_320): Size buttons on Copper Order Sheet render in copper color (#B87333).

**Next session:** Aesthetics cleanup on the order sheet forms

---

### Session 66
**Date:** 2026-06-26
**Build:** index_305 → index_316

**What we did:**
- Residential timecard job# search fixes (4-char threshold, blur/refocus)
- Custom CalendarPicker component replacing native date input
- IIS / horner.build discussion

**Next session:** Order sheet changes

---

### Session 65
**Date:** 2026-06-25
**Build:** index_301 → index_304

**What we did:**
- Cart email line item sorting by material sheet order
- Fixed broken Submit Cart button

**Next session:** Add lengths to all pipe sizes across PVC, CPVC, SCH80, PEX order sheets.

---

# Horner Field App — Master Project Log
Last updated: 2026-06-29

## ⚡ Current State

| Field | Value |
|---|---|
| **Current build** | `index_327.html` |
| **Primary URL** | `https://horner.app` (IIS on HP-APP) |
| **Fallback URL** | `https://itr325.github.io/Horner.app/` (GitHub Pages) |
| **Web root** | `C:\inetpub\wwwroot\horner_app\` |
| **Master log** | `C:\inetpub\wwwroot\horner_app\Horner_App_Master_Log.md` |
| **SharePoint site** | `https://hornerplumbing.sharepoint.com` |
| **Active Projects path** | `/Active Projects` in the Documents library |

---

## 🔴 Still Open / To-Do

### Features / Work Items (rough priority order)
- [ ] **Order sheet email formatting** — next up
- [ ] **Timecard — custom job entry**
- [ ] **Residential Order Sheets**
- [ ] **Service tiles** (both still "Coming Soon")
- [ ] **Admin Panel — Close Job**
- [ ] **App/Admin Panel — Logins & Permissions**
- [ ] **Horner Blue (#0156A4) color swap** — save for dedicated session
- [ ] **Flow 4 (TBT rollover for new year)** — low priority
- [ ] **Employee name dropdown audit** — sort by firstName everywhere
- [ ] **Employee ID cleanup** — 4 PM-added employees have timestamp IDs

---

## 🏗 Architecture & Process

### App Structure
- **Single-file HTML app.** React 18 (UMD, CDN). JSX pre-compiled to `React.createElement` — no runtime Babel, no build process.
- **Hosted:** `https://horner.app` (primary, IIS on HP-APP) + `https://itr325.github.io/Horner.app/` (fallback)
- **pdf-lib bundled inline** (line 62, ~524K chars). Never CDN. **use surgical `view` ranges only** and blob-excluding greps: `grep -n '<term>' index_XX.html | awk -F: '$1!=62' | cut -c1-200`
- **PDF.js** loaded from CDN in `<head>`.
- **No service worker.**

### Deploy Process (NEW as of Session 71)
1. Build new `index_NNN.html` in Claude sandbox
2. Validate JS: `node -e "new Function(src)"`
3. Claude writes directly to `C:\inetpub\wwwroot\horner_app\index.html` via Filesystem connector
4. Also push to GitHub for source control: `git add && git commit && git push`
5. Live on horner.app instantly — no cache delay!

### Session Protocol
**START of every session:**
1. Read `C:\inetpub\wwwroot\horner_app\Horner_App_Master_Log.md` via Filesystem connector
2. Check `**Next session:**` line — that's what we're working on
3. Update memory if anything has changed

**END of every session:**
1. Append session summary to master log (newest first)
2. Write updated log back to server via Filesystem connector
3. Also push to GitHub
4. Update memory

### Key Infrastructure
- **IIS server:** HP-APP, 10.1.1.12, Windows Server 2025
- **Web root:** `C:\inetpub\wwwroot\horner_app\`
- **FortiGate:** 98.103.132.245, VIP HP-APP → 10.1.1.12:443
- **SSL cert:** Starfield TLS (GoDaddy CA), valid to 1/10/2027
- **Webhook:** https://horner.app/gh-webhook (HMAC-SHA256, runs sync.ps1)
- **Sync script:** `C:\inetpub\horner_app\sync.ps1` (hourly fallback)
- **Filesystem MCP:** Connected to `C:\inetpub\wwwroot\horner_app\` with full access

### Revision Convention
- Each change: `index_327 → 328 → ...`
- Bump `BUILD_ID` to match filename every session.

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
| 21 | Flow 21 - Send Cart Email | Sends HTML cart email | `SEND_CART_URL` |
| 22 | Flow 22 - Send Timecard Email | Sends HTML timecard email | `SEND_TIMECARD_URL` |
| 23 | Flow 23 - Send Feedback Email | Sends feedback email | `SEND_FEEDBACK_URL` |
| 24 | Flow 24 - Get File Content | Returns raw file content | `GET_FILE_CONTENT_URL` |
| 25 | Flow 25 - Get Residential Jobs | Returns open residential job numbers | `GET_RES_JOBS_URL` |
