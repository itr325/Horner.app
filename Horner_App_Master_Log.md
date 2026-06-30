### Session 73
**Date:** 2026-06-30
**Build:** index_329 → index_330

**What we did — Fixed critical auth fail-open bug, settled on GitHub-only deploy workflow:**
- **Bug found:** Eric tested index_329 on horner.app — tapping the gear icon opened the Admin Panel with **zero authentication**. Root cause: `msalRequireLogin()` called `onSuccess(null)` whenever `msalInit()` returned null (MSAL CDN not loaded/blocked/slow) instead of calling `onError`. This meant any MSAL load hiccup silently let anyone into the Admin Panel — a fail-open, not a fail-closed.
- **Fix (index_330):** `msalRequireLogin` now calls `onError({errorCode: "msal_unavailable", ...})` when MSAL can't init. Gear icon onClick now shows an existing-pattern toast (`showToast("Admin login unavailable, try again.", false)`) instead of silently console-erroring and opening the panel anyway.
- **Validated:** JS syntax checked via `node -e 'new Function(src)'` — clean. Diffed index_329 vs index_330 (content-blob-excluding) — confirmed only the 3 intended lines changed (BUILD_ID, msalRequireLogin body, gear onClick handler).
- **NOT yet verified on-device:** Eric still needs to confirm on an iPad/iPhone that (a) gear icon still opens Admin Panel normally when MSAL loads fine, and (b) it now shows the error toast instead of opening the panel when MSAL fails to load.

**Deploy workflow — resolved, simplified:**
- Long discussion this session about why Claude can push to GitHub (via `curl`/`git` in sandbox — file already sits on sandbox disk, streams disk-to-disk) but historically struggled to write directly to the IIS web root (`Filesystem:write_file` requires full file content to be retyped as a literal string parameter — confirmed this session: attempting to `view` the full 1.2MB index_330.html truncated ~17,000 lines from the middle, proving large-file content cannot reliably round-trip through Claude's context this way).
- Explored and **discarded**: building a custom `/claude-deploy` authenticated push endpoint on horner.app (too much new attack surface for an internet-facing endpoint that can overwrite the live production app); building a new Linux server (doesn't solve the core problem — any new server has the same reachability/content-passing constraints unless it's just GitHub again).
- **DECISION — new deploy workflow:** Claude pushes every new build to GitHub (`itr325/Horner.app`) as both `index_NNN.html` (versioned) and `index.html` (live copy in repo). Eric manually downloads `index.html` from GitHub and copies it into `C:\inetpub\wwwroot\horner_app\`, overwriting the existing file. **No more patch_NNN.ps1 scripts, no more asking Eric to run PowerShell on the server.** Master log lives in both places — GitHub (source of truth Claude pushes to) and the server copy (Claude still reads it via Filesystem connector at session start, since that's still reachable for reads — only large-content *writes* are the problem).
- **Note:** `sync.ps1` / GitHub webhook auto-deploy was previously removed by Eric (was being built for a different sync approach he didn't want). Not in use. Eric does the IIS copy manually now — this is the new permanent norm, not a stopgap.

**Current server state (as of session end):**
- IIS (`horner.app`) is still running **index_329** (the broken fail-open build) — Eric has NOT yet copied index_330 over manually.
- GitHub now has index_330 pushed (both versioned and as index.html in repo).
- Server-side master log at `C:\inetpub\wwwroot\horner_app\Horner_App_Master_Log.md` is now behind this GitHub version — Eric copying index_330 over is a good time to also replace the server's master log copy with this one.

**⚠️ FIRST THING NEXT SESSION:**
- Confirm whether Eric has copied index_330's index.html onto IIS yet (check live BUILD_ID at horner.app).
- If yes: confirm on-device that the auth fail-closed fix actually works (gear icon blocks access + shows toast when MSAL can't load).
- If site still on index_329: flag that the Admin Panel is still wide open with no auth on the live server.

**Next session:** Verify index_330 deployed + auth fix confirmed on-device, then continue Entra SSO / RBAC work (AD groups, role-based UI gating).

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
- **GitHub webhook + auto-deploy:** Claude Code built webhook.ps1 (HMAC-SHA256 verified GitHub push receiver on https://horner.app/gh-webhook, port 443 shared with IIS via http.sys), sync.ps1 (downloads index.html from GitHub raw, deploys only if changed), HornerAppWebhook scheduled task (boot-start, SYSTEM, auto-restart), HornerAppSync hourly fallback task. **(Note: removed by Eric in Session 73 — see above. Not in use.)**
- **Filesystem MCP connector:** Added Filesystem connector in Claude Desktop pointed at `C:\inetpub\wwwroot\horner_app` with full access. Claude can now write directly to the server from this chat — no GitHub middleman needed for deployment!
- **New deployment workflow:** Build index.html here → Claude writes directly to `C:\inetpub\wwwroot\horner_app\index.html` → instantly live on horner.app. GitHub remains source control/backup. No more GitHub Pages cache delay. **(Superseded in Session 73 — large-file direct writes don't work reliably; see Session 73 deploy workflow notes.)**
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
Last updated: 2026-06-30

## ⚡ Current State

| Field | Value |
|---|---|
| **Current build (GitHub)** | `index_330` |
| **Current build (live IIS)** | `index_329` (Eric needs to manually copy index_330's index.html from GitHub onto IIS — see Immediate To-Do) |
| **Primary URL** | `https://horner.app` (IIS on HP-APP) |
| **Fallback URL** | `https://itr325.github.io/Horner.app/` (GitHub Pages) |
| **Web root** | `C:\inetpub\wwwroot\horner_app\` |
| **Master log** | `C:\inetpub\wwwroot\horner_app\Horner_App_Master_Log.md` (server copy — may lag GitHub; GitHub is now source of truth, see Deploy Process) |
| **SharePoint site** | `https://hornerplumbing.sharepoint.com` |
| **Active Projects path** | `/Active Projects` in the Documents library |

---

## 🔴 Still Open / To-Do

### ⚠️ Immediate — Start of Next Session
- [ ] Confirm whether Eric copied index_330's `index.html` from GitHub onto IIS yet
- [ ] If deployed: verify on-device the auth fail-closed fix works (gear icon shows error toast, doesn't open Admin Panel, when MSAL can't load)
- [ ] If NOT deployed: flag that live site is still running the broken fail-open auth (index_329)
- [ ] Replace server's master log copy with the latest GitHub version next time Eric is on the server

### Features / Work Items (rough priority order)
- [ ] **MSAL / Entra auth** — index_330 fixes fail-open bug; needs on-device verification
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

### Deploy Process — UPDATED Session 73
- **Confirmed root cause this session:** Claude CANNOT reliably write/copy large files (1.2MB+) directly onto the IIS server. `Filesystem:write_file` requires full content as a literal string param; `view`-ing the full file to retype it truncates ~17K lines from the middle. No tool bridges Claude's sandbox disk directly to the server disk (no "copy" primitive, only `move_file` which is server-internal-only).
- **Current permanent workflow:** Claude pushes every new build to GitHub (`itr325/Horner.app`) as `index_NNN.html` + updates `index.html` in the repo + updates the master log in the repo. **Eric manually downloads `index.html` from GitHub and copies it into `C:\inetpub\wwwroot\horner_app\`, overwriting the live file.** No patch scripts, no PowerShell run by Eric for routine deploys — that pattern is retired.
- Claude still reads the server-side master log via Filesystem connector at session start (reads of small-to-medium text work fine — only large binary-ish writes are the actual constraint).
- GitHub webhook / sync.ps1 auto-deploy: **removed by Eric, not in use.** Don't reference or rebuild without Eric asking.

### Session Protocol
**START of every session:**
1. Read `C:\inetpub\wwwroot\horner_app\Horner_App_Master_Log.md` via Filesystem connector (server copy)
2. ALSO check GitHub master log if server copy seems behind (GitHub is now source of truth for the log, since Claude pushes there reliably)
3. Check `**Next session:**` / `⚠️ Immediate` items
4. Update memory if anything has changed

**END of every session:**
1. Push new build (`index_NNN.html` + `index.html`) and updated master log to GitHub
2. Tell Eric the build number is ready on GitHub — Eric pulls/copies it to IIS himself

### Key Infrastructure
- **IIS server:** HP-APP, 10.1.1.12, Windows Server 2025
- **Web root:** `C:\inetpub\wwwroot\horner_app\`
- **FortiGate:** 98.103.132.245, VIP HP-APP → 10.1.1.12:443
- **SSL cert:** Starfield TLS (GoDaddy CA), valid to 1/10/2027
- **Entra App Registration:** Client ID `282e84c6-5e0b-4a7d-a58b-a773215c30b0`, Tenant ID `ef466c74-7a13-4920-854c-210669ea3c84`
- **Filesystem MCP:** Connected to `C:\inetpub\wwwroot\horner_app\` — reliable for reads and small file writes/patches; NOT reliable for full-file writes of the main 1.2MB index.html

### Revision Convention
- Each change: `index_327 → 328 → 329 → 330 → ...`
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
