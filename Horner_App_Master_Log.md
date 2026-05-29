# Horner Field App — Master Project Log
_Last updated: 2026-05-28 (Session 17)_
_Consolidates Handoffs 1–12 plus sessions 7–17. Append new sessions below "Session History."_

---

## ⚡ Current State

| Field | Value |
|---|---|
| **Current build** | `index_98.html` |
| **GitHub Pages URL** | `https://itr325.github.io/Horner.app/` |
| **SharePoint site** | `https://hornerplumbing.sharepoint.com` |
| **Active Projects path** | `/Active Projects` in the Documents library |

---

## 🔴 Still Open / To-Do

### Production Blockers
- [ ] **`SHOW_DIAG`** — flip back to `false` at line 88 before final production push. Was re-enabled in index_60 for field testing.

### Features / Work Items (rough priority order)
- [ ] **Timecard — custom job entry** — allow foreman to type a custom job name/code on the timecard instead of only selecting from the project list.
- [ ] **Admin Panel — Add/Remove Employee** — add/remove from a master employee list that populates timecards and foremen dropdowns across all forms. Need to define fields (name, position, phone, email, etc.). Long-term replaces hardcoded `EMPLOYEES` array.
- [ ] **Toolbox Talks** *(Flows 1–3 complete. Flow 4 — Upload Templates — still needed but not urgent.)* End-to-end tested and working. Toolbox Talks folder moved inside SAFETY folder (index_82 + Flow 1 + Flow 12 updated).
- [ ] **Admin Panel — Close Job** — move a job OUT of Active Projects to a specified archive location. Needs a Power Automate flow to move the SharePoint folder.
- [ ] **Photos — save extension as .jpg** — currently uploads preserve original extension (HEIC on iPad). Force `.jpg` output on capture.
- [ ] **App/Admin Panel — Logins & Permissions** — user authentication and role-based access control. **Blocks the flow URL security fix** — see note below.
- [x] **Remove imported/legacy folder dragging code** — DONE (index_86). `project.app`, "Imported" pill, and all FTP/drag logic stripped.
- [ ] **Submittals folder — file notifications + upload** — email foremen when new file(s) added/updated.
- [ ] **App — Back button on all pages** — universal back navigation on every view.
- [ ] **App — Open files in-app** — photos and docs should open in-app rather than redirecting to SharePoint.
- [x] **Codes & Charts** — `LibraryBrowser` component built; SharePoint flow wired end-to-end. ✅ index_95
- [x] **Admin Docs** — `LibraryBrowser` component built; SharePoint flow wired end-to-end. ✅ index_95
- [ ] **PDF Edit — snap dimensions** — snap/constrain lines while marking up plans.
- [ ] **Pre-Fab Request, Trapeze Pre-Fab Template, Standard Testing Record, RFC form, RFI form** — original batch, no rush.
- [ ] **Flow URL security — deferred to Logins** — all 14 flow URLs are plaintext in the public GitHub Pages JS. A shared secret in client code is security theater (secret is visible right next to the URLs in view-source). Real fix requires moving the app behind auth (Azure Static Web Apps + Entra ID) or a thin proxy — both of which are part of the Logins work anyway. **`flowPost()` wrapper already added in index_96** — when auth is built, swap the secret injection for a real token in `flowPost` and every call inherits it automatically. Do NOT add shared-secret conditions to flows in the meantime.
- [ ] **Test save-back on a `/Rotate` 90/180/270 page** — never verified on a real rotated PDF.
- [ ] **Test save-back from deeply nested folder** — all verification was single-level (`PLANS`).
- [ ] **Probe actual field iPads** — canvas ceiling, dpr, render ms. All performance data so far from a spare (probably older) iPad.
- [ ] **Tuning ~1.3s sharpen delay** — do not tune without real-device data first.
- [ ] **Employee admin panel + SharePoint sync** — `EMPLOYEES` array is hardcoded (47 people).
- [ ] **PDF markup change notifications** — email crew when PDF changes. Blocked on crew-list data source.
- [ ] **PM data cleanup** — misconfigured flow in session 3 overwrote PM column on multiple projects.
- [ ] **Backfill PM column** on legacy drag-and-drop folders showing "Unassigned."

---

## 🏗 Architecture & Process

### App Structure
- **Single-file HTML app.** React 18 (UMD, CDN). JSX pre-compiled to `React.createElement` — no runtime Babel, no build process. Hosted on GitHub Pages.
- **pdf-lib bundled inline** (line 62, ~524K chars). Never CDN — iOS Safari was unreliable fetching it from unpkg. This line matches almost every grep; **use surgical `view` ranges only** and blob-excluding greps: `grep -n '<term>' index_XX.html | awk -F: '$1!=62' | cut -c1-200`
- **PDF.js** loaded from CDN in `<head>` (acceptable — iPads always connected).
- **No service worker.** Stale builds come from HTTP cache only. "↻ Reload App" button defeats it with `?v=<timestamp>`.

### Key Constants (near top of script)
| Constant | Purpose |
|---|---|
| `BUILD_ID` | Matches filename (e.g. `"index_87"`); shown in bottom-left stamp and diag header |
| `SHOW_DIAG` | `true` = diag panel visible in PDF editor. Must be `false` for production |
| `PROJECTS_FETCH_URL` | Power Automate: Syncing Projects Folders |
| `CREATE_JOB_URL` | Power Automate: Create New Project Folder |
| `UPDATE_PM_URL` | Power Automate: Update PM |
| `UPDATE_FOREMAN_URL` | Power Automate: Change Foreman |
| `FETCH_FOLDER_CONTENTS_URL` | Power Automate: Fetch Folder Contents |
| `DOWNLOAD_FILE_URL` | Power Automate: Download File |
| `SAVE_MARKUP_URL` | Power Automate: Save Markup |
| `SAVE_VALVE_TAGS_URL` | Power Automate: SaveValveTags (line 227) |
| `LOAD_VALVE_TAGS_URL` | Power Automate: LoadValveTags (line 228) |
| `SAVE_TBT_URL` | Power Automate: Save Signed TBT PDF (Flow 11) |
| `ROLLOVER_URL` | Power Automate: Toolbox Talks Rollover (Flow 12) |
| `CODES_CHARTS_FETCH_URL` | Power Automate: Fetch Codes & Charts library items (Flow 13) |
| `ADMIN_DOCS_FETCH_URL` | Power Automate: Fetch Admin Docs library items (Flow 14) |
| `SHAREPOINT_HOST` | `https://hornerplumbing.sharepoint.com` |
| `FORM_FOLDERS` | Maps folder names → fillable forms. Update if folder names change |

### SharePoint Folder Naming
- Active Projects folders use format `(CODE) Name` — e.g. `(EJS-5678) Testing Foreman`
- **NOT** `CODE - Name`. All flow Compose expressions must use: `concat('/Active Projects/(', code, ') ', name, '/...')`
- `{FullPath}` returns paths **without a leading slash** — always prepend `/`
- Use `GetFileByServerRelativeUrl` (not `GetFileByServerRelativePath`) — the latter rejects paths with parentheses

### PDF Editor Critical Notes
- **Touch events, NOT pointer events** in `PdfMarkupView`. Pointer events fire a `pointercancel` race on iOS multitouch.
- **Listeners attached manually** via `handlersRef` pattern with `{ passive: false }`.
- **Annotation coordinates normalized (0–1) per page.**
- **Zoom/pan is imperative** (`applyTransform` directly on DOM node).
- **Detail canvas must NOT be inside the zoom wrapper.**
- **Save-back is additive.**

### Valve Tag Persistence Notes
- `valve_tag_list.json` written to `Active Projects/(JOB)/VALVE TAG/` by SaveValveTags flow.
- Filtered from app's file list so field crews never see it.
- VALVE TAG folder **must exist** in SharePoint for Save to work.
- Use `GetFileByServerRelativeUrl`, not `GetFileByServerRelativePath`.

### Deploy Process
1. Rename build to `index.html`
2. Push to GitHub
3. Reload App on device (↻ Reload App button or hard reload)

### Revision Convention
- Each change: `index_70 → 71 → ...`
- Bump `BUILD_ID` to match filename every session.
- Discuss before touching the file. Use surgical view ranges.

### Developer Notes
- **Eric** owns all coding AND Power Automate flows (Patrick handed off as of Session 9).
- **Go step by step on flows.** Always specify paste vs Expression mode for every field.
- GitHub Pages URL accessible via `web_fetch` but NOT via bash `curl`. Upload file to chat for bash access.
- Always fetch the current build fresh via upload at the start of each session.

---

## 📋 Session History (newest first)

---

### Session 17 — 2026-05-28 | index_97 → index_98

**Topic:** PDF zoom ceiling raised to 25x

**What changed:**
- `index_98`: `Math.min(25, ...)` — one line. Note: at very high zoom the canvas re-renders at the viewport region; test on a real iPad before deciding 25x is the final ceiling.

**No new Power Automate flows.**

---

### Session 16 — 2026-05-28 | index_96 → index_97

**Topic:** Daily Field Report iPhone SE fix + PDF zoom raised to 15x (superseded by Session 17)

**Closed:**
| Item | Resolution |
|---|---|
| Daily Field Report cut off on iPhone SE | Table grid changed from fixed px (`54px 110px 1fr 64px 28px`) to proportional (`44px minmax(0,1fr) minmax(0,2fr) 52px 24px`). Trade and Description columns now flex instead of Trade holding a hard 110px. Added `minWidth: 0` and `boxSizing: border-box` to both text inputs. |
| PDF zoom ceiling | Raised from 6x → 15x (then to 25x in Session 17). |

**What changed:**
- `index_97`: `DailyLog` header + row `gridTemplateColumns` updated. Trade/desc inputs get `minWidth: 0`. Zoom `Math.min` updated.

**No new Power Automate flows.**

---

### Session 15 — 2026-05-28 | index_95 → index_96

**Topic:** Flow URL security review + `flowPost` wrapper

**Closed:**
| Item | Resolution |
|---|---|
| Plaintext flow URL security | Reviewed. A shared secret in client JS is security theater — the secret is visible right next to the URLs in view-source. Real fix requires moving the app behind auth. **Deferred to Logins work item.** |
| `flowPost()` wrapper | Added (index_96). Centralizes all Power Automate calls. Currently injects `APP_SECRET` but the value in client JS provides no real security. When Logins is built, replace the secret injection with a real token here — every call inherits it automatically. **Do NOT add secret conditions to flows.** |

**What changed:**
- `index_96`: Added `APP_SECRET` constant and `flowPost(url, body)` helper. All 14 `fetch(...)` calls replaced with `flowPost(...)`.

**No new Power Automate flows.**

---

### Session 14 — 2026-05-28 | index_87 → index_95

**Topic:** Codes & Charts + Admin Docs library browsers

**Closed:**
| Item | Resolution |
|---|---|
| Codes & Charts tile | Fully wired — `LibraryBrowser` component fetches from SharePoint via Power Automate, breadcrumb navigation, back button wired to `libPath` state. |
| Admin Docs tile | Same — shared `LibraryBrowser` component, separate flow and library path. |

**What changed:**
- `index_88–95`: Added `LibraryBrowser` React component shared by both library views. Breadcrumb nav with back button goes up one level (not straight home). Home screen tiles navigate to `"codesCharts"` and `"adminDocs"` views. Flow returns all items — app filters to current level by decoding URL and matching parent path segment.

**New Power Automate Flows (2):**

Both flows are identical in structure — 4 steps:
1. HTTP Trigger (Anyone)
2. Get files (properties only) — respective library, Folder = `triggerBody()?['path']` (Expression mode)
3. Select — maps `name`, `url`, `modified`, `isFolder`
4. Response — Body: `{"items": @{body('Select')}}`

> Note: `isFolder` returns as string `"True"`/`"False"` — app handles this.

| Flow | Library | SharePoint Path | URL Constant |
|---|---|---|---|
| Flow 13 — Fetch Codes & Charts | Codes & Charts | `/Codes%20%20Charts` | `CODES_CHARTS_FETCH_URL` (`34eda8b84cfe406c8d3898591ddd7425`) |
| Flow 14 — Fetch Admin Docs | Admin Docs | `/Admin%20Docs` | `ADMIN_DOCS_FETCH_URL` (`f63a5174735b41ce8f09502c5ebec443`) |

---

### Session 13 — 2026-05-27 | index_86 → index_87

**Topic:** TBT post-sign bug fix + logo centering

**Closed:**
| Item | Resolution |
|---|---|
| Signed TBT disappearing from folder view | Fixed in `ToolboxTalksList`. Flow 11 deletes the original after saving the signed version — the UI wasn't handling the signed-only state. `signedMap` now stores the file object (not `true`). After first pass, any signed file with no matching original is pushed into `originalFiles`. `displayName`, `isSigned`, and `fileToOpen` all updated to handle filenames that already contain `_SIGNED_`. |
| Logo off-center on mobile | Fixed. Header flexbox had gear button (36px) on the right with no counterpart on the left, pushing the logo left of true center. Added an invisible 36px `flexShrink: 0` spacer div on the left to balance it. |

**What changed:**
- `index_87`: `ToolboxTalksList` — signed-only file handling. Header — left spacer div added.

**No new Power Automate flows.**

---

### Session 12 — 2026-05-27 | index_81 → index_86

**Topic:** Home screen, UI cleanup, imported job removal, navigation fixes

**Closed:**
| Item | Resolution |
|---|---|
| Toolbox Talks folder path | Moved into SAFETY folder in app (`isToolboxTalksYearView` updated). Flow 1 + Flow 12 updated by Eric. |
| PM default on Create New Job | Fixed — defaults to `— Unassigned —`, resets correctly after job creation. Added `— Unassigned —` option to dropdown. |
| Foreman on job cards | Added orange badge to both Active Projects list and job header card. |
| New home screen | `view = "home"` as initial view. 3 tiles: Active Projects, Codes & Charts, Admin Docs. Codes & Charts and Admin Docs show "coming soon" toast. |
| Hamburger sidebar removed | Replaced with logo tap to return home from any page. |
| Horner Plumbing logo | Embedded inline (base64), centered in white header on every page. |
| Admin breadcrumb | Fixed to `Home › Admin` (was `Home › Projects › Admin`). |
| Back button | Added to Crumb bar on all pages. Correct back logic per view. Hidden on home screen. |
| Reload App in Active Projects | Moved outside `projects.length > 0` gate — always visible. |
| Back button folder navigation | Fixed — `go("folders")` now passes `null` for folder to reset correctly. |
| Empty state always rendering | Fixed — `projects.length === 0` empty state was a sibling to `view === "projects"` block; gated on `view === "projects" &&`. |
| Imported / FTP drag code removed | Stripped `project.app`, `p.app`, "Imported" pill from all project cards and job header. `rootSyntheticFolders` simplified. |

**What changed:**
- `index_82`: Toolbox Talks path, PM default, Foreman badges on cards/header
- `index_83`: Home screen + logo + 3 tiles, sidebar removed, breadcrumb Home root
- `index_84`: Back button, Reload App always visible, Admin breadcrumb, PM `— Unassigned —` option
- `index_85`: Back navigation folder fix (reset folder to null on back)
- `index_86`: Removed all Imported/FTP/`project.app` code; fixed header card paren bug

**No new Power Automate flows.** Flow 1 and Flow 12 updated by Eric (Toolbox Talks path into SAFETY).

---

### Session 11 — 2026-05-25 | index_80 → index_81

**Topic:** Weekly Safety Inspection form

**Closed:**
| Item | Resolution |
|---|---|
| Weekly Safety Inspection form | Built and wired. `SiteWeeklySafety` component with 11 sections, 42 checklist items in 2-column layout matching original PDF. |
| Form routing | Appears as a tile in both **SAFETY** and **TIME CARDS** folders on app-created projects. |
| Foreman field | Dropdown filtered to `position === "Foreman"`, pre-populated from `project.foreman`. |
| Submit | Emails to `CONTACTS.safety` (safety@hornerplumbing.com), CC's project PM. Subject: `{CODE} Weekly Safety Inspection - MM/DD/YY`. |

**What changed:**
- `index_81`: Added `SAFETY_SECTIONS` constant and `SiteWeeklySafety` component (~165 lines) before `function App()`.
- `getFormView("Site Safety Inspection")` → `"safety"`.
- `"safety"` added to `formViews` array.
- `FORM_FOLDERS["SAFETY"]` already had `"Site Safety Inspection"`. Added it to `FORM_FOLDERS["TIME CARDS"]` as well.
- Render wired: `view === "safety" && activeForm && React.createElement(SiteWeeklySafety, ...)`.

**SiteWeeklySafety checklist sections:**
- Left col (col:0): PPE (6 items), Housekeeping (4), Ladders (5), Electrical Protection (4), Fire Prevention (4)
- Right col (col:1): Hand & Power Tools (3), Floor & Wall Openings (5), Miscellaneous (4), Stairs/Hand Rails (2), Scaffolds (3), Excavation (2)

**No new Power Automate flows needed** — form submits via mailto.

---

### Session 10 — 2026-05-25 | index_79 → index_80

**Topic:** Toolbox Talks — December Rollover flow (Flow 12) + Create New Job TBT folder fix

**Closed:**
| Item | Resolution |
|---|---|
| Flow 3 — Toolbox Talks Rollover | Built and working. Creates next-year TBT folder in every active job and copies all 52 templates into each. Admin panel button wired. |
| Create New Job flow missing Toolbox Talks folder | Fixed — added 4 steps to Create New Project Folder flow: Create Toolbox Talks folder, Create current year subfolder, List template files, Apply to each: Copy template file. |
| Templates library path | Finalized as `Templates/Toolbox Talks/PDFs` (renamed from `Toolbox Talks - Templates/2026`). Static folder — no year subfolder needed on template side. |
| Flow 3 foreach source bug | Inner Apply to each was looping over `List_Active_Jobs` instead of `List_Template_Files`. Fixed. |
| Flow 3 path bug | Copy_File URI was using `{FilenameWithExtension}` for job folder. Fixed to `{FullPath}` with leading `/`. |
| Body fields not in Expression mode | Create Toolbox Talks Folder and Create TBT Year Folder Body fields were plain paste. Fixed to Expression mode. |

**What changed:**
- `index_80`: Added `ROLLOVER_URL` constant. Added **Toolbox Talks Rollover** card to Admin panel with confirmation dialog and success/failure toasts.

**Flow 12 — Toolbox Talks Rollover structure:**
```
Trigger (HTTP)
Compose: Next_Year → string(add(int(formatDateTime(utcNow(), 'yyyy')), 1))
List Active Jobs (Get files, library: Active Projects, no folder filter, nested: false)
Apply to each (jobs) [foreach: outputs('List_Active_Jobs')?['body/value']]
  ├── Create Year Folder (HTTP POST _api/web/folders)
  │     Body: concat('{"__metadata":{"type":"SP.Folder"},"ServerRelativeUrl":"/', items('Apply_to_each')?['{FullPath}'], '/Toolbox Talks/', outputs('Next_Year'), '"}')
  ├── List Template Files (Get files, library: Templates, folder: Toolbox Talks/PDFs, nested: false)
  └── Apply to each 2 (templates) [foreach: body('List_Template_Files')?['value']]
        └── Copy File (HTTP POST)
              Uri: concat('_api/web/GetFileByServerRelativeUrl(''/Templates/Toolbox Talks/PDFs/', items('Apply_to_each_2')?['{FilenameWithExtension}'], ''')/CopyTo(strNewUrl=''/', items('Apply_to_each')?['{FullPath}'], '/Toolbox Talks/', outputs('Next_Year'), '/', items('Apply_to_each_2')?['{FilenameWithExtension}'], ''')')
```

---

### Session 9 — Flows 2–4: Specs

**Flow 2 — New Job Creation** ✅ BUILT (Session 10) — see Session 10 for full step details.

**Flow 3 — December Rollover** ✅ BUILT (Session 10) — see Session 10 for full step details.

**Flow 4 — Upload Templates** (Admin panel button, HTTP trigger) — NOT YET BUILT (low priority):
- Input: `{ year, filename, contentBase64 }`
- Write to Templates site, library: `Toolbox Talks`, folder: `/{year}/`, overwrite: yes

---

### Session 8 — 2026-05-20 | index_71 → index_74

**Closed:**
| Item | Resolution |
|---|---|
| Foreman dropdown on Create New Job | Done — index_71. Dropdown from EMPLOYEES filtered by Foreman, written to SharePoint on job creation. |
| Change Foreman in Admin Panel | Done — index_72. Mirrors Change PM card exactly. |
| Update Foreman Power Automate flow | Done — index_73. New flow (#10) built: HTTP trigger → Get folder metadata → Update file properties (Foreman column). |
| Foreman not showing in job list | Fixed — index_74. Project normalizer was dropping `foreman` from fetch response. Added `foreman: p.foreman || ""` to `.map()`. Also added `foreman` to Syncing Project Folders Select step in Power Automate. |

**What changed:**
- `index_71`: Foreman `<select>` dropdown added to Create New Job form. `foreman` field added to CREATE_JOB_URL POST body.
- `index_72`: `UPDATE_FOREMAN_URL` constant added. **Change Foreman** admin card added.
- `index_73`: `UPDATE_FOREMAN_URL` wired with real URL. Change Foreman flow built (flow #10).
- `index_74`: Bug fix — project normalizer `.map()` now includes `foreman: p.foreman || ""`.

**Architecture note:** Foreman stored as plain name string (e.g. `"Joseph Zaczek"`), NOT the key→object pattern PM uses.

---

### Session 7 — 2026-05-19 | index_62 → index_70

**Closed:** Valve Tag List form — full form built, two Power Automate flows wired, SharePoint persistence working end-to-end.

**Key gotchas:** `{FullPath}` has no leading slash. `GetFileByServerRelativePath` rejects `()` paths — use `GetFileByServerRelativeUrl`. Both Response actions firing simultaneously → set mutually exclusive run-after. VALVE TAG folder missing from Create New Project Folder flow → added.

---

### Session 6 — 2026-05-19 | index_60 → index_62

**Closed:** One-finger pan mode toggle — ✋ button. Pan is default tool on load. `tool === "pan"` drives all logic.

---

### Session 5 — 2026-05-19 | index_57 → index_60

**Closed:** Field iPad shakeout, rotation audit, nested folder save-back, flow error responses, latest version badge (green "LATEST" pill).

**iPad diag numbers:** `dpr 2, zoom 1.00x, buf 1436×1025 (1.47 MP), css 718×513, fit 0.237, area 746×1048, canvas ceiling: max 8192×8192 (67.1 MP)`

---

### Session 4 — 2026-05-14 | index_55 → index_57

**Shipped:** Save-back to SharePoint with server-side revision control. End-to-end verified on-device.

---

### Session 3 — 2026-05-14 | index_44 → index_55

**Shipped:** PDF resolution refresh on zoom — region/viewport rendering. Confirmed working on-device.

---

### Session 2 — 2026-05-14 | index_44 (baseline)

**Status:** Issue #2 not solved. Four attempts failed. index_44 = V39 + issue #1 only.

---

### Session 1 (overnight 5/12 → 5/13) | index_34 → index_39

**Shipped:** Photo-capture markup tool. PDF annotation editor (`PdfMarkupView`) built from scratch. `Download File` Power Automate flow. Key architectural decisions locked in.

---

### Daytime session 5/12 | index_31 → index_34

**Shipped:** Create New Project Folder flow template, synthetic form folders, refresh with cache invalidation, `Upload File` flow, Site Photos & Video capture.

**⚠️ "TOOLBOX TALKS" is 2 words** (was "TOOL BOX TALKS" — 3 words).

---

### Daytime session 5/12 | index_28 → index_31

**Shipped:** Dynamic folder navigation — SharePoint is now canonical source for folder/file structure.

---

### Daytime sessions 5/8–5/11 | index_21 → index_28

**Shipped:** T&M Log v4, employee list rebuild (47 people), folder cleanup, nested sub-folder navigation.

---

### Sessions 1–3 — 2026-05-07/08 | index_21

Change PM admin card, PM handling, initial SharePoint integration.

---

## 📌 Power Automate Flow Reference

| # | Flow | Purpose | URL constant |
|---|---|---|---|
| 1 | Create New Project Folder | Creates job folder + subfolders (incl. VALVE TAG + Toolbox Talks). Sets PM, App, Foreman columns | `CREATE_JOB_URL` |
| 2 | Syncing Projects Folders | Returns `[{code, name, pm, foreman, app}]` | `PROJECTS_FETCH_URL` |
| 3 | Update PM | Updates PM column on existing folder | `UPDATE_PM_URL` |
| 4 | Change Foreman | Updates Foreman column on existing folder | `UPDATE_FOREMAN_URL` |
| 5 | Fetch Folder Contents | Returns `{folders, files}` for any path | `FETCH_FOLDER_CONTENTS_URL` |
| 6 | Upload File | Uploads photos/videos to date-subfolder | `UPLOAD_FILE_URL` |
| 7 | Download File | Proxies SharePoint file as base64 | `DOWNLOAD_FILE_URL` |
| 8 | Save Markup | Uploads annotated PDF as `_vN` | `SAVE_MARKUP_URL` |
| 9 | SaveValveTags | Writes valve_tag_list.json to VALVE TAG folder | `SAVE_VALVE_TAGS_URL` |
| 10 | LoadValveTags | Reads valve_tag_list.json; 404 if not found | `LOAD_VALVE_TAGS_URL` |
| 11 | Save Signed TBT PDF | Saves signed TBT, deletes original | `SAVE_TBT_URL` |
| 12 | Toolbox Talks Rollover | Creates next-year TBT folder in every active job, copies 52 templates into each | `ROLLOVER_URL` |
| 13 | Fetch Codes & Charts | Returns `{items}` for Codes & Charts library at given path | `CODES_CHARTS_FETCH_URL` |
| 14 | Fetch Admin Docs | Returns `{items}` for Admin Docs library at given path | `ADMIN_DOCS_FETCH_URL` |

All 14 URLs are plaintext in the public GitHub JS. **Shared-secret check still unimplemented.**

---

_To append a new session: add a new `### Session N` block at the top of Session History, update "Current State" and "Still Open" at the top._
