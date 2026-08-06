### Session 82 / App Migration — GitHub Pages Redirect
**Date:** 2026-08-06

**Summary:** Began retiring GitHub Pages as a distribution surface for the Horner Field App ("App Migration" project).

**Changes:**
- Replaced `index.html` on GitHub with a lightweight meta-refresh redirect to `https://horner.app`
- Old index_327 build preserved as `index_327.html` — no data lost
- New SHA: `a035355bffd2fec0c88dbaef59b406b44bc779ae`

**App Migration — Remaining Steps (target mid-to-late August 2026):**
1. Disable GitHub Pages (Settings → Pages → Source → None)
2. Make repo private (also mitigates `sig=` token exposure in versioned HTML files)
3. Remove `https://itr325.github.io/Horner.app/` redirect URI from Entra app registration (`282e84c6-...`)

### Session 81 (continued) / My Files Feature
**Date:** 2026-07-24
**Builds:** index_363 → index_372

**Summary:** Built the My Files personal FTP tile feature end-to-end — tile, file browser, upload, and file viewing (PDF, photo, other).

**New Power Automate Flows:**
- **Flow 26 — Get Personal FTP Contents** (`GET_PERSONAL_FTP_URL`): Mirrors Flow 5 but targets Personal FTPs library. Body: `{ path }` → returns `{ folders, files }` with `ServerRelativeUrl` for each file. Trigger auth must be "Anyone" (not "Anyone in my tenant") or gets 401.
- **Flow 27 — Upload to Personal FTPs** (`UPLOAD_PERSONAL_FTP_URL`): Body: `{ filename, contentBase64, folder }`. SharePoint Create File step uses Expression mode for all three fields. Trigger auth must be "Anyone".

**New Constants:**
- `GET_PERSONAL_FTP_URL` — Flow 26
- `UPLOAD_PERSONAL_FTP_URL` — Flow 27

**Feature: My Files Tile**
- Appears on home screen after Help tile, only if user has a matching folder in SharePoint "Personal FTPs" library
- Folder matched by first letter of email prefix (e.g. `eschieble@...` → `ESchieble`)
- Shows file/folder browser with Upload button (full width)
- Upload: picks any file from iOS native picker (includes camera option natively)
- File opening: PDFs → `openLibraryPdf()` (PDF markup viewer, read-only); Images → download via Flow 17 + open in photo viewer (read-only, no save-back); Other files → direct SharePoint URL in new tab

**Build history:**
- **index_364:** Initial My Files. Flow 26 URL missing sp/sv/sig params — tile didn't appear
- **index_365:** Fixed Flow 26 URL. Tile appeared. Upload used wrong flow (Flow 6 hardcodes Active Projects)
- **index_366:** Removed camera button (redundant with iOS native picker), Upload full width
- **index_367:** Added Flow 27, fixed upload params (`contentBase64` not `base64`)
- **index_368:** Fixed file opening — non-PDF/image open via direct SharePoint URL instead of Flow 17 (502)
- **index_369:** Fixed PDF opening — `openLibraryPdf(f)` instead of fake `pdfViewer` view
- **index_370:** Images tried `openLibraryPdf` — shows "Loading PDF", wrong
- **index_371:** Images tried `window.open` SharePoint URL — Eric rejected, wants in-app viewer
- **index_372 [CURRENT IIS]:** Images download via Flow 17 + open in photo viewer read-only. All file types working.

**Key learnings:**
- Power Automate new-style trigger (no sp/sv/sig) returns 401 unless trigger auth = "Anyone"
- Photo viewer requires a `File` object — must download via Flow 17 first, can't open by URL
- `openLibraryPdf(f)` = PDF viewer read-only (no folder for save-back); `openPdfMarkup(f)` = PDF viewer with save-back to current project folder
- Flow 17 (DOWNLOAD_FILE_URL) works for Personal FTPs paths

**Pinned for next session:**
- SharePoint delegated auth — pass user's MSAL access token to SharePoint REST API directly, so Modified By shows actual user instead of "Horner Plumbing" service account. Requires adding SharePoint scope to Entra app registration (`282e84c6...`). Eric has decision authority.

**Next session:** SharePoint delegated auth (Modified By attribution).

---

### Session 80
**Date:** 2026-07-16
**Builds:** index_361 → index_362

**Summary:** Two new order sheets (Copper Solder Fittings, ProPress Fittings) built from ERP inventory dump. Scroll-to-top fix on all navigation.

**Changes:**
- **index_361:** Added two new order sheets generated from ERP_Inventory_Dump.xlsx:
  - **Copper Solder Fittings** — 163 items across 11 size sections (1/4" through 4"). Includes 90s, 45s, street elbows, tees, reducing tees, couplings (w/stop, no stop, reducing), adapters (CxM, CxF, FTGxM, FTGxF), pressure caps, test caps, flush bushings, FTG reducers. Each item has its ERP item number as the UPC field.
  - **ProPress Fittings** — 114 items across 9 size sections (1/2" through 4"). Same fitting categories plus PRESSxMPT adapters, PRESSxFIP adapters, PRESSxFPT drop ears, and ball valves. Larger sizes (2-1/2"+) use bronze (BRZ) fittings.
  - Both sheets use copper color (#B87333), appear in Order Sheets list, use the shared OrderSheet component with section-based layout.
  - Data generated via Python script parsing the ERP dump, cleaning labels (removing WROT/COPPER/BRZ/model number prefixes), grouping by primary fitting size.
- **index_362:** Scroll-to-top on all navigation. Added `window.scrollTo(0, 0)` in both the `go()` function and the Back button handler. Every page transition now starts at the top — fixes issue where order sheets opened mid-page requiring scroll up to select project/phase.

**ERP Inventory Analysis:**
- Full ERP dump analyzed: 7,597 items across 1,026 Sort Name categories
- Identified potential new order sheets beyond copper: Cast Iron (25 items), Gas Fittings (80+), SharkBite (18), Hangers (150+), Cleanouts (50+), Fernco/Mission (29), Valves (90+)
- MegaPress fittings (53 items) found under FITTINGS-GAS — deferred, Aaron to clarify if used only for gas or also hydronic/mechanical

**Architecture Discussion:**
- Discussed moving from hardcoded data to SQL database + API backend
- Eric comfortable with SQL Server Express + ASP.NET Core Web API on HP-APP
- End goal: native app (not just web app), so backend API is inevitable
- Further discussed: packaging as a product for other sub-contractors (not SaaS — shipped product, customer owns deployment)
- Decision: build for Horner first but design multi-tenant from day one
- For now: continue with inline data, migrate to DB in a future dedicated session

**Key decisions:**
- Copper fittings split into two separate sheets (Solder vs ProPress) per Aaron's input
- MegaPress deferred — need clarification on gas-only vs multi-use
- If it's in the ERP system, it goes on the order sheet (no curation)
- Scroll-to-top applies globally to all navigation, not just order sheets

**index_362 deployed to IIS via PowerShell Invoke-WebRequest**

**ERP Audit Spreadsheet:**
- Built comprehensive audit spreadsheet (Horner_Order_Sheet_Audit.xlsx) sent to Aaron for review
- Cross-referenced all 7,432 ERP items against 1,105 unique UPCs in the app
- 1,009 ERP items matched to app, 6,423 not on any order sheet
- Filtered to 1,570 field-relevant items across 34 categories (excluded 5,018 finish/showroom/non-field)
- Spreadsheet has dropdown data validation with all 14 current order sheet names + ability to type new sheet names (errorStyle="warning")
- 96 app items have part numbers not found in ERP — Aaron to verify/correct
- Key missing categories: PVC Fittings (170), Valves (159), Gas/MegaPress (106), PEX Fittings (105), Hangers (92), Black Fittings (88), Black Nipples (83), Firestop (59), Tools (57), Galv Fittings (54), PVC Pipe (52), PEX Pipe (49), Brass Fittings (48)
- PVC pressure fittings (Schedule 40 pressure) identified as distinct from DWV fittings currently on PVC sheet
- Goal: eliminate the Blank Order Sheet by covering all field items on proper order sheets
- Sent to Aaron for markup — once returned, Claude will parse descriptions to auto-generate section data

**Next session:** Process Aaron's completed audit spreadsheet to build new order sheets and fill gaps in existing ones. Or begin database/API scoping.

---

### Session 79
**Date:** 2026-07-16
**Builds:** index_356 → index_360

**Summary:** Pull-to-refresh (global), styled Back button, Reload/Refresh buttons removed, larger header logo, inactivity auto-reload.

**Deploy workflow this session:** All builds done in Python on clean base files with `assert count == 1` guards, pushed to GitHub, Eric manually copies to IIS. No edit_file on IIS — burned by cascading patch failures earlier in session (index_351–355 all broken). This workflow is now the standard.

**Changes:**
- **index_356:** Pull-to-refresh on every page — touch listeners on `document`, fires `window.location.href = pathname + ?v= + Date.now()` when user pulls down >72px from scroll top. No indicator. Built on clean index_349 base.
- **index_357:** Styled Back button — frosted white pill (`rgba(255,255,255,0.18)` bg, `1px solid rgba(255,255,255,0.35)` border, borderRadius 6, fontSize 12, fontWeight 700) with 14px chevron SVG (strokeWidth 2.8) + "Back" text.
- **index_358:** Removed all Reload App buttons (home screen ×2, projects header, folders view, subPath view) and Refresh Projects List button from projects header. Kept empty-state "Refresh Projects List" button (only shown when zero projects found).
- **index_359:** Header logo height increased from 26 to 38 — fills the blue bar better.
- **index_360:** Inactivity auto-reload — Page Visibility API fires on resume; if last touch/click was >10 minutes ago, reloads the app. Resets timer on every touchstart/click.

**Key decisions:**
- Pull-to-refresh does a full reload (lands on home screen) — intentional, matches old Reload App behavior, field crew uses it to grab latest build
- No pull-to-refresh indicator — silent, just reloads on release
- IIS no-cache headers + ?v= param = always fresh build on reload
- Inactivity threshold: 10 minutes

**Next session:** Go through entire inventory list — identify what order sheets need to be created and what items need part numbers.

---

### Session 72
**Date:** 2026-06-29
**Build:** index_327 → index_329 (index_329 NOT yet deployed — patch_329.ps1 on server, needs to be run)

**What we did — MSAL Auth (incomplete):**
- **Goal:** Gate the Admin Panel (gear icon) behind Microsoft Entra ID login via MSAL.js
- **App Registration:** Already created in prior session — Client ID: `282e84c6-5e0b-4a7d-a58b-a773215c30b0`, Tenant ID: `ef466c74-7a13-4920-854c-210669ea3c84`
- **index_328 (broken):** Patch script added MSAL CDN script tag + eager init (`new msal.PublicClientApplication()` at top level). Crashed on load with `Uncaught ReferenceError: msal is not defined` — CDN script hadn't loaded yet when the inline script ran.
- **index_329 (ready, not deployed):** Fixed with lazy init pattern — `msalInstance` starts as `null`, initialized on first gear tap via `msalInit()`. Falls through gracefully if MSAL CDN unavailable. Built at `/home/claude/index_329.html` in Claude sandbox. `patch_329.ps1` written to server.
- **Deploy workflow problem discovered:** `Filesystem:write_file` cannot write large files (1.2MB) — content must be passed as inline string in tool call JSON, which exceeds limits. Claude also cannot execute PowerShell scripts — can only read/write files. Patch scripts still require Eric to run them manually.
- **Current server state:** `index.html` is index_328 (broken, stuck on loading splash). `patch_329.ps1` is on server ready to fix it.

**⚠️ FIRST THING NEXT SESSION:**
Run this on HP-APP to deploy the fix:
```powershell
powershell -ExecutionPolicy Bypass -File "C:\inetpub\wwwroot\horner_app\patch_329.ps1"
```
Then verify https://horner.app loads and gear icon prompts for Microsoft login.

**Deploy workflow going forward:**
- Claude CANNOT write large files directly via Filesystem connector (1.2MB exceeds inline limit)
- Claude CANNOT execute scripts — only read/write files
- **Correct pattern:** Claude writes patch_NNN.ps1 to server → Eric runs it → done
- Need to figure out a better long-term solution (maybe a small local script that pulls from Claude sandbox?)

**Next session:** Run patch_329.ps1, verify MSAL login works on gear icon, then continue RBAC / AD groups work.

---

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
| **Current build** | `index_329` (patch_329.ps1 on server — run it!) |
| **Primary URL** | `https://horner.app` (IIS on HP-APP) |
| **Fallback URL** | `https://itr325.github.io/Horner.app/` (GitHub Pages) |
| **Web root** | `C:\inetpub\wwwroot\horner_app\` |
| **Master log** | `C:\inetpub\wwwroot\horner_app\Horner_App_Master_Log.md` |
| **SharePoint site** | `https://hornerplumbing.sharepoint.com` |
| **Active Projects path** | `/Active Projects` in the Documents library |

---

## 🔴 Still Open / To-Do

### ⚠️ Immediate — Start of Next Session
- [ ] Run `powershell -ExecutionPolicy Bypass -File "C:\inetpub\wwwroot\horner_app\patch_329.ps1"` on HP-APP
- [ ] Verify https://horner.app loads and gear icon prompts Microsoft login

### Features / Work Items (rough priority order)
- [ ] **MSAL / Entra auth** — patch_329 fixes crash; then verify login flow works
- [ ] **RBAC with Entra AD groups** — after MSAL confirmed working
- [ ] **Order sheet email formatting**
- [ ] **Timecard — custom job entry**
- [ ] **Residential Order Sheets**
- [ ] **Service tiles** (both still "Coming Soon")
- [ ] **Admin Panel — Close Job**
- [ ] **Horner Blue (#0156A4) color swap**
- [ ] **Flow 4 (TBT rollover for new year)** — low priority
- [ ] **Employee name dropdown audit**
- [ ] **Employee ID cleanup**

---

## 🏗 Architecture & Process

### App Structure
- **Single-file HTML app.** React 18 (UMD, CDN). JSX pre-compiled to `React.createElement` — no runtime Babel, no build process.
- **Hosted:** `https://horner.app` (primary, IIS on HP-APP) + `https://itr325.github.io/Horner.app/` (fallback)
- **pdf-lib bundled inline** (line 62, ~524K chars). Never CDN. **use surgical `view` ranges only** and blob-excluding greps: `grep -n '<term>' index_XX.html | awk -F: '$1!=62' | cut -c1-200`
- **PDF.js** loaded from CDN in `<head>`.
- **No service worker.**

### Deploy Process
- Claude CANNOT write large files (1.2MB) directly via Filesystem connector — content must be inline string, exceeds limits
- Claude CANNOT execute scripts — can only read/write files
- **Current pattern:** Claude writes patch_NNN.ps1 to server → Eric runs it manually
- GitHub webhook + hourly sync still active as fallback

### Session Protocol
**START of every session:**
1. Read `C:\inetpub\wwwroot\horner_app\Horner_App_Master_Log.md` via Filesystem connector
2. Check `**Next session:**` / `⚠️ Immediate` items
3. Update memory if anything has changed

**END of every session:**
1. Append session summary to master log (newest first)
2. Write updated log back to server via Filesystem connector

### Key Infrastructure
- **IIS server:** HP-APP, 10.1.1.12, Windows Server 2025
- **Web root:** `C:\inetpub\wwwroot\horner_app\`
- **FortiGate:** 98.103.132.245, VIP HP-APP → 10.1.1.12:443
- **SSL cert:** Starfield TLS (GoDaddy CA), valid to 1/10/2027
- **Entra App Registration:** Client ID `282e84c6-5e0b-4a7d-a58b-a773215c30b0`, Tenant ID `ef466c74-7a13-4920-854c-210669ea3c84`
- **Filesystem MCP:** Connected to `C:\inetpub\wwwroot\horner_app\` with full access

### Revision Convention
- Each change: `index_327 → 328 → 329 → ...`
- Bump `BUILD_ID` to match every session.

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
