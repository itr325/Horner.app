### Session 73
**Date:** 2026-06-30
**Build:** index_329 → index_334 (all confirmed deployed and working as of session end)

**Summary:** Started by fixing a critical auth fail-open bug, then discovered MSAL CDN itself was 404ing (bundled inline to fix), then verified the full Entra SSO + RBAC flow end-to-end including both the allow and deny paths. Admin Panel access is now genuinely gated by "Horner App Admin" group membership, confirmed working in production.

**1. Fixed auth fail-open bug (index_330):**
- Eric tested index_329 on horner.app — tapping the gear icon opened the Admin Panel with **zero authentication**. Root cause: `msalRequireLogin()` called `onSuccess(null)` whenever `msalInit()` returned null instead of calling `onError`, silently letting anyone in on any MSAL load hiccup.
- Fix: `msalRequireLogin` now calls `onError(...)` when MSAL can't init; gear icon shows `showToast("Admin login unavailable, try again.", false)` instead of opening the panel.

**2. Discovered and fixed root cause of MSAL load failures (index_332):**
- After deploying 330, gear icon still failed with "MSAL not loaded" console warning. Eric checked the Network tab: `alcdn.msauth.net/.../msal-browser.min.js` was returning **404**, not blocked/slow.
- Root cause (confirmed via web search): Microsoft has been decommissioning CDN hosting for MSAL.js v2 builds as part of the v3 migration — same failure class as the pdf-lib CDN issue from earlier in the project.
- Fix: downloaded the real `@azure/msal-browser` v2.39.0 package from npm (`npm pack`), verified it (version-stamped header, correct UMD wrapper, valid JS syntax via `node -e 'new Function()'`), and bundled it inline — same pattern as pdf-lib. No more dependency on the external CDN.
- Eric confirmed: "Got the login" — popup now actually opens and authenticates.

**3. Verified the `groups` token claim end-to-end (index_333_diag → confirmed, then reverted):**
- Configured Token configuration in the Entra App Registration: Add groups claim → Security groups → Group ID.
- Built a temporary diagnostic build (index_333_diag, based on clean index_332) that console.logged `account.idTokenClaims` and the `groups` claim specifically on login.
- Eric tested it: `groups` claim came through correctly — `['9806a8c3-4471-4696-831f-9c7923f1346d', '484ad2f7-ab3b-46fc-95f0-88b926cb9f75']`, the second ID matching "Horner App Admin" exactly. Confirms the full AD group → Entra sync → token claim → app pipeline works.
- Noted: COOP ("Cross-Origin-Opener-Policy policy would block the window.closed call") console errors appeared during login — researched and confirmed this is a known, benign MSAL v2/Chrome interaction (modern browsers block the legacy popup-close detection MSAL v2 uses; functionality is unaffected). Not something to fix — login completes successfully despite the console noise.

**4. Built and verified the real RBAC gate (index_334):**
- Built fresh from clean index_332 (NOT on top of the diagnostic build — no console.log token dump in this build).
- Added `ADMIN_GROUP_ID = "484ad2f7-ab3b-46fc-95f0-88b926cb9f75"` constant and `msalIsAdmin(account)` helper that checks `account.idTokenClaims.groups` for that ID.
- Gear icon onClick: after successful MSAL login, calls `msalIsAdmin()` before opening the Admin Panel. Non-members see `"Your account does not have Admin Panel access."` toast instead of getting in (fail-closed, consistent with the index_330 pattern).
- **Tested BOTH paths in production:**
  - Allow path: Eric (member of Horner App Admin) logs in → gets into Admin Panel. ✅
  - Deny path: Eric temporarily removed himself from the Horner App Admin group in AD, used a private/incognito window to force a fresh (non-cached) login, confirmed he saw the "does not have Admin Panel access" toast and was blocked. Re-added himself to the group afterward and confirmed access restored. ✅
- This is the first time the auth gate has been verified end-to-end with a real deny-path test, not just "login works."

**Deploy workflow — resolved this session, now permanent:**
- Discussed at length why Claude can push to GitHub (file already on sandbox disk, streams via `curl`/`git`) but can't reliably write the full 1.2MB index.html directly to the IIS server (`Filesystem:write_file` requires full content as a literal string param — confirmed this session that even `view`-ing the full file truncates ~17,000 lines from the middle, so round-tripping it through Claude's context isn't viable).
- Explored and **discarded**: a custom `/claude-deploy` authenticated push endpoint on horner.app (too much new attack surface for an internet-facing endpoint that overwrites the live app); a new Linux server (doesn't solve the core problem — same reachability/content-passing constraints apply unless it's just GitHub again).
- **Permanent workflow:** Claude pushes every new build to GitHub (`itr325/Horner.app`) as `index_NNN.html` + `index.html` + updated master log. **Eric manually downloads `index.html` from GitHub and copies it into `C:\inetpub\wwwroot\horner_app\`, overwriting the live file.** No more patch_NNN.ps1 scripts, no more PowerShell run by Eric for routine deploys.
- `sync.ps1` / GitHub webhook auto-deploy: confirmed removed by Eric earlier, not in use, not being rebuilt.
- Master log: GitHub copy is source of truth going forward (Claude pushes here reliably); server copy at `C:\inetpub\wwwroot\horner_app\Horner_App_Master_Log.md` updated manually by Eric when convenient, alongside the index.html copy.

**Current state (end of session):**
- horner.app live build: **index_334**, confirmed deployed and working (RBAC tested both allow and deny paths).
- GitHub: index_334 pushed (versioned + live copy).
- MSAL bundled inline (v2.39.0) — no more external CDN dependency for auth.
- Admin Panel access genuinely restricted to "Horner App Admin" group members.

**Still open / not yet done:**
- Broader multi-role RBAC (PM, Foreman, etc. with different permission levels) — explicitly NOT needed right now per Eric ("just one role: can access Admin Panel vs everyone else" — this is what's built). Multi-role is still backlog if/when needed.
- Logout/session expiry behavior not yet tested (how long does the cached MSAL session last before requiring re-login — Eric deferred this check this session).
- Server-side master log copy needs manual sync from Eric next time he's doing a deploy.

**Next session:** Confirm server-side master log copy is up to date (Eric to sync manually). Otherwise, return to the open feature backlog: Order sheet email formatting, Timecard custom job entry, Residential Order Sheets, Service tiles, Admin Close Job. RBAC/Entra SSO work is now considered functionally complete for the single-admin-role use case.

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
| **Current build (GitHub)** | `index_334` |
| **Current build (live IIS)** | `index_334` (confirmed deployed and working — RBAC tested both allow and deny paths) |
| **Primary URL** | `https://horner.app` (IIS on HP-APP) |
| **Fallback URL** | `https://itr325.github.io/Horner.app/` (GitHub Pages) |
| **Web root** | `C:\inetpub\wwwroot\horner_app\` |
| **Master log** | `C:\inetpub\wwwroot\horner_app\Horner_App_Master_Log.md` (server copy — may lag GitHub; GitHub is source of truth, see Deploy Process) |
| **SharePoint site** | `https://hornerplumbing.sharepoint.com` |
| **Active Projects path** | `/Active Projects` in the Documents library |
| **Admin Panel access** | Gated by "Horner App Admin" Entra group (Object ID `484ad2f7-ab3b-46fc-95f0-88b926cb9f75`) — confirmed working |

---

## 🔴 Still Open / To-Do

### ⚠️ Immediate — Start of Next Session
- [ ] Confirm server's master log copy has been manually synced from GitHub by Eric
- [ ] MSAL/Entra auth is now considered functionally complete for the single-admin-role use case — no immediate action needed unless new issues surface

### Features / Work Items (rough priority order)
- [x] **MSAL / Entra auth** — DONE: bundled inline (CDN was 404ing), fail-closed on load failure, RBAC gate via "Horner App Admin" group, tested both allow and deny paths in production
- [ ] **Multi-role RBAC** (PM, Foreman, etc. with different permission levels) — backlog, not currently needed (single admin-role gate is sufficient per Eric)
- [ ] **Session/logout behavior** — not yet tested how long the cached MSAL session lasts before requiring re-login
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
- Each change: `index_327 → 328 → 329 → 330 → ... → 334 → ...`
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
