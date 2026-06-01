# Horner Field App — Master Project Log
_Last updated: 2026-05-31 (Session 26)_
_Consolidates Handoffs 1–12 plus sessions 7–19. Append new sessions below "Session History."_

---

## ⚡ Current State

| Field | Value |
|---|---|
| **Current build** | `index_123.html` |
| **GitHub Pages URL** | `https://itr325.github.io/Horner.app/` |
| **SharePoint site** | `https://hornerplumbing.sharepoint.com` |
| **Active Projects path** | `/Active Projects` in the Documents library |

---

## 🔴 Still Open / To-Do

### Production Blockers
- [x] **`SHOW_DIAG`** — flipped to `false` in index_120 ✅ ~~flip back to `false` at line 88 before final production push. Was re-enabled in index_60 for field testing.

### Features / Work Items (rough priority order)
- [ ] **Timecard — custom job entry** — allow foreman to type a custom job name/code on the timecard instead of only selecting from the project list.
- [x] **Admin Panel — Add/Remove Employee** — ✅ index_116. Full add/edit/remove UI. employees.csv stored in Templates library via 2 new Power Automate flows. firstName sort on all dropdowns.
- [x] **Admin Panel — tile landing** — ✅ index_114. Admin panel now has a 2×2 tile grid (New Job, Change PM/Foreman, Toolbox Talks Rollover, Add/Remove Employees). Each tile opens its own sub-view with back button + breadcrumb.
- [ ] **Admin Panel — Close Job** — move a job OUT of Active Projects to a specified archive location. Needs a Power Automate flow to move the SharePoint folder.
- [ ] **Photos — save extension as .jpg** — currently uploads preserve original extension (HEIC on iPad). Force `.jpg` output on capture.
- [ ] **App/Admin Panel — Logins & Permissions** — user authentication and role-based access control. **Blocks the flow URL security fix** — see note below.
- [x] **App — Open files in-app** — ✅ index_111–113. PDFs ✓, Images ✓ (lightbox), Docs ✓ (blob → iOS Open In sheet).
- [x] **Measure tool** — ✅ index_99–112. CAD-style dimensioning, calibration, drag-to-reposition, undo/cancel.
- [x] **Codes & Charts** — `LibraryBrowser` component built; SharePoint flow wired end-to-end. ✅ index_95
- [x] **Admin Docs** — `LibraryBrowser` component built; SharePoint flow wired end-to-end. ✅ index_95
- [ ] **Toolbox Talks** *(Flows 1–3 complete. Flow 4 — Upload Templates — still needed but not urgent.)*
- [x] **Submittals folder — file notifications** — ✅ Flow 18 working. Emails PM + CC Foreman on file add/modify. Verbiage tweaks deferred.
- [ ] **App — Back button on all pages** — universal back navigation on every view.
- [ ] **PDF Edit — snap dimensions** — snap/constrain lines while marking up plans.
- [ ] **Pre-Fab Request, Trapeze Pre-Fab Template, Standard Testing Record, RFC form, RFI form** — original batch, no rush.
- [ ] **Flow URL security — deferred to Logins** — all 14 flow URLs are plaintext in the public GitHub Pages JS. Real fix requires moving the app behind auth. **`flowPost()` wrapper already added in index_96** — when auth is built, swap the secret injection for a real token in `flowPost`. Do NOT add shared-secret conditions to flows in the meantime.
- [ ] **Test save-back on a `/Rotate` 90/180/270 page** — never verified on a real rotated PDF.
- [ ] **Probe actual field iPads** — canvas ceiling, dpr, render ms.
- [ ] **Employee admin panel + SharePoint sync** — `EMPLOYEES` array is hardcoded (47 people).
- [ ] **PM data cleanup** — misconfigured flow in session 3 overwrote PM column on multiple projects.

---

## 🏗 Architecture & Process

### App Structure
- **Single-file HTML app.** React 18 (UMD, CDN). JSX pre-compiled to `React.createElement` — no runtime Babel, no build process. Hosted on GitHub Pages.
- **pdf-lib bundled inline** (line 62, ~524K chars). Never CDN — iOS Safari was unreliable fetching it from unpkg. This line matches almost every grep; **use surgical `view` ranges only** and blob-excluding greps: `grep -n '<term>' index_XX.html | awk -F: '$1!=62' | cut -c1-200`
- **PDF.js** loaded from CDN in `<head>` (acceptable — iPads always connected).
- **No service worker.** Stale builds come from HTTP cache only.

### Key Constants (near top of script)
| Constant | Purpose |
|---|---|
| `BUILD_ID` | Matches filename (e.g. `"index_113"`); shown in bottom-left stamp and diag header |
| `SHOW_DIAG` | `true` = diag panel visible in PDF editor. Must be `false` for production |
| `PROJECTS_FETCH_URL` | Power Automate: Syncing Projects Folders |
| `CREATE_JOB_URL` | Power Automate: Create New Project Folder |
| `UPDATE_PM_URL` | Power Automate: Update PM |
| `UPDATE_FOREMAN_URL` | Power Automate: Change Foreman |
| `FETCH_FOLDER_CONTENTS_URL` | Power Automate: Fetch Folder Contents |
| `DOWNLOAD_FILE_URL` | Power Automate: Download File |
| `SAVE_MARKUP_URL` | Power Automate: Save Markup |
| `SAVE_VALVE_TAGS_URL` | Power Automate: SaveValveTags |
| `LOAD_VALVE_TAGS_URL` | Power Automate: LoadValveTags |
| `SAVE_TBT_URL` | Power Automate: Save Signed TBT PDF |
| `ROLLOVER_URL` | Power Automate: Toolbox Talks Rollover |
| `CODES_CHARTS_FETCH_URL` | Power Automate: Fetch Codes & Charts library items |
| `ADMIN_DOCS_FETCH_URL` | Power Automate: Fetch Admin Docs library items |
| `SHAREPOINT_HOST` | `https://hornerplumbing.sharepoint.com` |

### SharePoint / Flow Notes
- Active Projects folders use format `(CODE) Name` — e.g. `(EJS-5678) Testing Foreman`
- `{FullPath}` returns paths **without a leading slash** — always prepend `/`
- Use `GetFileByServerRelativeUrl` (not `GetFileByServerRelativePath`) — the latter rejects paths with parentheses
- Foreman stored as plain name string (not a key like PM)
- `valve_tag_list.json` filtered from app's file list — crews never see it
- VALVE TAG folder **must exist** in SharePoint for Save to work

### PDF Editor Critical Notes
- **Touch events, NOT pointer events** in `PdfMarkupView`. Pointer events fire a `pointercancel` race on iOS multitouch.
- **Listeners attached manually** via `handlersRef` pattern with `{ passive: false }`.
- **Annotation coordinates normalized (0–1) per page.**
- **Zoom/pan is imperative** (`applyTransform` directly on DOM node).
- **Detail canvas must NOT be inside the zoom wrapper.**
- **Save-back is additive.**
- **PDF zoom max is 25x.**

### File Opening (index_113)
- **PDFs** → `PdfMarkupView` markup editor
- **Images** (jpg/png/heic/gif/webp/bmp/tif) → full-screen lightbox via download flow → base64 data URL
- **Word/Excel/PPT/CSV** → `openDocBlob()`: download flow → Uint8Array → Blob + correct MIME → `<a download>` trigger → iOS "Open in..." sheet
- **LibraryBrowser** accepts `onOpenPdf` + `onOpenDoc` props; `openLibraryPdf` sets `markupPdfFolder=null` (no save-back for library docs)
- iframe approach was tried for docs and rejected — SharePoint sets `X-Frame-Options: SAMEORIGIN`

### Measure Tool (index_99–112)
- 📏 toolbar button; scale bar (dropdown + Calibrate + Undo + Cancel)
- 3-phase drag-to-place: drag pt1 → lift → drag pt2 → lift → dimension placed
- Drag-to-reposition: tap+drag existing dimension label/line changes offset
- CAD-style dimension lines: inward arrows + extension lines + label, scale with zoom
- DOM cursor overlay (z-index 500) + SVG rubber-band overlay (z-index 499)
- Measurements stored with `offset` field (CSS px perpendicular distance)
- Default scale: 1/4"=1'-0"; calibration flow available

### Developer Notes
- **Eric** owns all coding AND Power Automate flows (Patrick handed off as of Session 9).
- **Go step by step on flows.** Always specify paste vs Expression mode for every field.
- Fetch current build at session start: `curl -s https://itr325.github.io/Horner.app/index.html`
- Master log lives at: `https://raw.githubusercontent.com/itr325/Horner.app/main/Horner_App_Master_Log.md`

### Deploy Process
1. Rename build to `index.html`
2. Push to GitHub
3. Reload App on device (↻ Reload App button or hard reload)

### Revision Convention
- Each change: `index_70 → 71 → ...`
- Bump `BUILD_ID` to match filename every session.

---

## 📋 Session History (newest first)

---

### Session 26 — 2026-05-31 | no code changes (flow fix only)

**Topic:** Flow 18 - Submittal Notifications fix

**No code changes this session.**

**Flow 18 fixed and working:**
- Root cause: "Get files (properties only)" had **Include Nested Items = true (default)**, returning all files across all subfolders. Filter array `contains` was either matching wrong items or failing entirely.
- Fix 1: Set **Include Nested Items = false** → returns only top-level job folders from Active Projects
- Fix 2: Changed Filter array condition from `contains(triggerPath, folderName)` to **is equal to** with right side as `split(triggerOutputs()?['body/{FullPath}'], '/')[1]` — extracts job folder name directly from trigger path and exact-matches against `{FilenameWithExtension}`
- Result: Single email sent correctly, no double-trigger, correct PM + Foreman populated
- Email verbiage tweak deferred to future session

**No new Power Automate flows.**

---

### Session 25 — 2026-05-31 | index_120 (no code changes)

**Topic:** Power Automate flow renaming

**No code changes this session.**

**All 17 flows renamed** with `"Flow N - "` prefix convention. Final names:

| # | Name |
|---|---|
| 1 | Flow 1 - Create New Project Folder |
| 2 | Flow 2 - Syncing Projects Folders |
| 3 | Flow 3 - Update PM |
| 4 | Flow 4 - Change Foreman |
| 5 | Flow 5 - Fetch Folder Contents |
| 6 | Flow 6 - Camera Upload |
| 7 | Flow 7 - PDF Editor - Get PDF File |
| 8 | Flow 8 - PDF Markup Save |
| 9 | Flow 9 - Save Valve Tags |
| 10 | Flow 10 - Load Valve Tags |
| 11 | Flow 11 - Save Signed TBT PDF |
| 12 | Flow 12 - Toolbox Talks Rollover |
| 13 | Flow 13 - Codes &amp; Charts Sync |
| 14 | Flow 14 - Admin Docs Sync |
| 15 | Flow 15 - Load Employees |
| 16 | Flow 16 - Save Employees |
| 17 | Flow 17 - Save Photo Markup |

**Session protocol note added:** Every new session — fetch master log via curl to get caught up before starting work.


---

### Session 25 — 2026-05-31 | index_120 → index_123

**Topic:** SHOW_DIAG fix + Flow 18 Submittal Notifications + employee loading fix

**Closed:**
- `SHOW_DIAG` flipped to `false` in index_120 ✅
- `DEFAULT_EMPLOYEES` emptied — SharePoint CSV is now sole source of truth ✅
- Flow 15 CSV load fixed: `decodeURIComponent` added in app; Flow 15 uses `encodeUriComponent(base64ToString(...))` ✅
- Flow 3 + Flow 4 updated to write `PM_Email` / `Foreman_Email` to new hidden SharePoint columns ✅
- App sends `pmEmail` + `foremanEmail` in Flow 3 & 4 calls (index_121) ✅
- Scientific notation ID parse fixed: `parseInt` → `Number()` ✅
- Flow 18 - Submittal Notifications built: triggers on file create/modify, filters `{FullPath}` contains `/Submittals/`, emails PM + CC Foreman ✅

**Still open in Flow 18:**
- Filter array empty array error — job folder match failing (`{FullPath}` vs `{FilenameWithExtension}` mismatch)
- Double-trigger dedup needed

**New flows:** Flow 18 - Submittal Notifications
**New SP columns:** `PM_Email`, `Foreman_Email` (hidden) on Active Projects library

---

### Session 24 — 2026-05-31 | index_117 → index_119

**Topic:** Photo viewer complete + MarkupView zoom/pan

**Flow 17 — Save Photo Markup (new)**
- "Create or update file" action — guaranteed overwrite of original file on SharePoint
- POST body: `{ code, name, folderPath, filename, contentBase64 }`
- Flow 6 uses "Create file" (makes duplicates) — Flow 17 needed for overwrite

**index_118 changes:**
- `SAVE_PHOTO_MARKUP_URL` constant added (Flow 17)
- `openFile()` image branch replaced — existing photos now open in `MarkupView` instead of old lightbox
- `handlePhotoMarkupDone` + `photoMarkupFile`/`photoMarkupMeta` state — on Done, calls Flow 17 to overwrite original file
- Skip button hidden when opening existing photo (`onSkip={null}`)
- `generateUploadFilename` fixed — all images now always save as `.jpg` (was preserving `.heic` from iPad camera)
- 3 scenarios fully handled: view-only (Cancel), markup+save (Done=overwrite), re-edit already-marked-up photo (Done=overwrite again)

**index_119 changes:**
- `MarkupView` fully rewritten with pinch-to-zoom (max 10x) and pan
- Switched from pointer events → touch events (avoids iOS `pointercancel` race on multitouch)
- `canvasWrapperRef` + `viewRef` — zoom/pan applied imperatively via CSS transform
- `beginGesture` / `updateGesture` — adapted from PdfMarkupView pinch logic
- "⛶ Fit" button in toolbar resets zoom to fit-to-screen
- "← Back" replaces "✕ Cancel" in top bar and error screen
- `handleDone` guards `props.onSkip` calls with null check

**Next session:** Rename all Power Automate flows with "Flow N - " prefix convention.

---

### Session 23 — 2026-05-30 | index_117 (no new build — planning session)

**Topic:** Photo viewer redesign + Microsoft outage resilience

**No code changes this session — planning only.**

**Photo viewer plan (paused — resume next session):**
- Open existing SharePoint photos in `MarkupView` (same component used post-capture) instead of current lightbox
- 3 scenarios:
  1. User opens photo, closes without drawing → nothing uploads, file unchanged
  2. User opens photo, marks it up, saves → overwrite original file on SharePoint
  3. User opens already-marked-up photo, adds more edits, saves → overwrite again (canvas flattens old + new markups naturally)
- Needs **new Flow 17 "Save Photo Markup"** — "Create or update file" action with exact path + filename → guaranteed overwrite
- Also need to force `.jpg` on all image uploads in `generateUploadFilename` (currently preserves `.heic` from iPad camera)
- Skip button hidden when opening existing photo (no re-upload of original needed)

**Bigger issue raised — Microsoft outage resilience:**
- Power Automate was fully down today → entire app offline for field crews
- This is unacceptable for a mission-critical field app
- Options discussed:
  - **(a) Startup health check + banner** — ping flows on load, show clear "Server unavailable — read-only mode" message instead of silent failures. Quick win.
  - **(b) Offline read-only cache** — cache project list + recent files locally so crews can view data during outages. Moderate effort.
  - **(c) Backend migration** — replace Power Automate with Azure Functions or similar. Eliminates Microsoft-outage dependency. High effort.
- **Decision: address resilience strategy before resuming photo viewer work.**

**No new Power Automate flows.**

---

### Session 22 — 2026-05-29 | index_116 → index_117

**Topic:** Team Email card moved to home screen + mailto links

**Closed:**
| Item | Resolution |
|---|---|
| Team Email Routing on home screen | Card moved from Admin panel to home screen, below the 3 tile grid. Each email is a tappable mailto: link that opens native mail app. Admin panel retains a copy. |
| Build number discipline | Rule enforced: every push must bump BUILD_ID. |

**Notes:**
- Ran into JS syntax errors from improper arrow functions inside `.map()` in pre-compiled React. Fixed by using `function(c){ return ...; }` form instead of arrow functions in that context.
- Always validate JS with node before pushing: `node -e "new Function(script)"` on each script tag.

---

### Session 21 — 2026-05-29 | index_114 → index_116

**Topic:** Add/Remove/Edit Employees admin panel + SharePoint CSV persistence

**Closed:**
| Item | Resolution |
|---|---|
| Admin — Add/Remove Employee | Full feature built. Add form: First Name, Last Name, Position (optional), Department (required), Email, Phone (auto-formats to 111-111-1111). Edit inline on any row. Remove with confirm step. Search + department filter. |
| Employee CSV persistence | employees.csv stored in Templates library on SharePoint. Load flow (15) fetches on app startup — falls back to hardcoded DEFAULT_EMPLOYEES if file missing. Save flow (16) writes full CSV on every add/edit/remove. |
| firstName sort | All employee dropdowns across the entire app now sort by firstName. BILLING_EMPLOYEES converted to getBillingEmployees() function to stay live. |
| Build number discipline | Established rule: every push must bump BUILD_ID. |

**What changed (index_116):**
- `DEFAULT_EMPLOYEES` hardcoded fallback + runtime `EMPLOYEES` (loaded from SharePoint)
- `parseEmployeeCSV()` / `serializeEmployeeCSV()` helpers
- `loadEmployeesFromSharePoint()` called at app startup
- `LOAD_EMPLOYEES_URL` / `SAVE_EMPLOYEES_URL` constants wired
- `getBillingEmployees()` replaces static `BILLING_EMPLOYEES`
- `AdminEmployeesView` component — full add/edit/remove UI
- Phone auto-formatter (111-111-1111) on add and edit forms
- All employee dropdowns sort by `firstName`

**New flows:**
- Flow 15: Load Employees — GET employees.csv from Templates library → returns `{csvData}`
- Flow 16: Save Employees — POST `{csvData}` → writes employees.csv to Templates library

---

### Session 20 — 2026-05-29 | index_113 → index_114

**Topic:** Admin Panel tile restructure

**Closed:**
| Item | Resolution |
|---|---|
| Admin Panel — split into tiles | Replaced single long-scroll admin view with a 2×2 tile grid (styled to match home page). Four new sub-views: `adminNewJob`, `adminPmForeman`, `adminRollover`, `adminEmployees`. Each has back button → Admin and breadcrumb Home › Admin › …. Team Email Routing reference card preserved on admin landing. |
| Add/Remove Employees tile | Placeholder sub-view built showing live counts from `EMPLOYEES` array (47 employees / foreman count). Full feature deferred. |

**What changed (index_114):**
- Admin landing: `view === "admin"` now renders a 2×2 tile grid + Team Email Routing reference card
- `view === "adminNewJob"` → Create New Job form
- `view === "adminPmForeman"` → Change PM card + Change Foreman card (combined)
- `view === "adminRollover"` → Toolbox Talks Rollover card
- `view === "adminEmployees"` → placeholder card with live employee/foreman counts
- Breadcrumb and back-button routing updated for all four sub-views
- `useEffect` reset for `editPm`/`editForeman` now also fires on `adminPmForeman`
- `BUILD_ID` bumped to `index_114`

**No new Power Automate flows.**

---

### Session 19 — 2026-05-29 | index_110d → index_113

**Topic:** Zoom gesture fix + Open Files In-App (#6)

**Closed:**
| Item | Resolution |
|---|---|
| Zoom gesture quick-pinch | Added final `updateGesture` call in `onTouchEnd` using `changedTouches` + remaining touches — handles quick pinches with zero touchmove events. Partial improvement; remaining edge case at very high zoom accepted as-is. |
| Measure tool | Signed off — shipping as-is. |
| PDFs from Codes & Charts / Admin Docs opening in SharePoint | `onOpenPdf` was hardcoded `null` on both `LibraryBrowser` instances. Fixed — wired to new `openLibraryPdf` handler that opens without setting `markupPdfFolder` (no save-back for read-only library docs). |
| Images opening in SharePoint | Added in-app full-screen lightbox: download flow → base64 → data URL. `FileRow` updated with 🖼️ icon and "Tap to view" subtitle. Covers jpg/png/heic/gif/webp/bmp/tif. |
| Word/Excel docs opening in SharePoint | Tried iframe overlay first — blocked by SharePoint `X-Frame-Options: SAMEORIGIN`. Replaced with `openDocBlob()`: download flow → Uint8Array → Blob + correct MIME → `<a download>` trigger → iOS "Open in..." sheet. Works from both project folders and LibraryBrowser (via `onOpenDoc` prop). |

**What changed:**
- `index_110d`: `onTouchEnd` final gesture update using changedTouches
- `index_111`: Images in-app lightbox; `FileRow` image detection; `openLibraryPdf` handler wired on LibraryBrowser
- `index_111b`: Fixed `onOpenPdf: null` on Codes & Charts and Admin Docs
- `index_112`: iframe overlay for docs (later replaced)
- `index_113`: Replaced iframe with `openDocBlob()` blob download; `onOpenDoc` prop on LibraryBrowser

**No new Power Automate flows.**

---

### Session 18 — 2026-05-29 | index_99 → index_110d

**Topic:** Measure tool build-out (full session)

**Closed:** Complete ephemeral measurement tool:
- 3-phase drag-to-place, drag-to-reposition, CAD-style dimension lines
- Arrows/labels scale with zoom; DOM cursor overlay; SVG rubber-band
- Calibration flow; Undo/Cancel; default scale 1/4"=1'-0"
- PDF never blurs during measurement; detail canvas draws crosshairs sharply
- Zoom gesture: added `d0 = Math.max(40, dist)` minimum to prevent runaway zoom from close-finger starts

**No new Power Automate flows.**

---

### Session 17 — 2026-05-28 | index_97 → index_98

**Topic:** PDF zoom ceiling raised to 25x

**What changed:**
- `index_98`: `Math.min(25, ...)` — one line.

---

### Session 16 — 2026-05-28 | index_96 → index_97

**Topic:** Daily Field Report iPhone SE fix + PDF zoom raised to 15x

**Closed:**
| Item | Resolution |
|---|---|
| Daily Field Report cut off on iPhone SE | Table grid changed from fixed px to proportional columns. Trade/Description flex instead of fixed 110px. |
| PDF zoom ceiling | Raised from 6x → 15x (then to 25x in Session 17). |

---

### Session 15 — 2026-05-28 | index_95 → index_96

**Topic:** Flow URL security review + `flowPost` wrapper

**Closed:** `flowPost()` wrapper added (index_96). All 14 `fetch(...)` calls replaced. Security deferred to Logins.

---

### Session 14 — 2026-05-28 | index_87 → index_95

**Topic:** Codes & Charts + Admin Docs library browsers

**Closed:** `LibraryBrowser` React component built, shared by both views. Breadcrumb nav. Two new flows (13 & 14).

**New flows:** Flow 13 (Codes & Charts) + Flow 14 (Admin Docs) — each: HTTP Trigger → Get files (properties only) → Select (name/url/modified/isFolder) → Response `{"items":@{body('Select')}}`. `isFolder` returns as string `"True"`/`"False"`.

---

### Session 13 — 2026-05-27 | index_86 → index_87

**Topic:** TBT post-sign bug fix + logo centering

**Closed:** Signed TBT disappearing fixed (`signedMap` stores file object; signed-only files pushed into `originalFiles`). Logo off-center fixed (invisible spacer div on left).

---

### Session 12 — 2026-05-27 | index_81 → index_86

**Topic:** Home screen, UI cleanup, imported job removal, navigation fixes

**Closed:** New home screen with 3 tiles. Hamburger removed. Logo tap = home. Back button on all pages. PM default fixed. Foreman badges on cards. Imported/FTP code stripped.

---

### Session 11 — 2026-05-25 | index_80 → index_81

**Topic:** Weekly Safety Inspection form

**Closed:** `SiteWeeklySafety` component — 11 sections, 42 items, 2-column layout. Appears in SAFETY and TIME CARDS folders. Submits via mailto.

---

### Session 10 — 2026-05-25 | index_79 → index_80

**Topic:** Toolbox Talks Rollover flow (Flow 12) + Create New Job TBT folder fix

**Closed:** Flow 12 built and working. Create New Job flow updated to create Toolbox Talks folder + copy 52 templates.

---

### Session 9 — Flows 2–4 Specs

Flow 4 (Upload Templates) — NOT YET BUILT (low priority).

---

### Session 8 — 2026-05-20 | index_71 → index_74

**Topic:** Foreman dropdown + Change Foreman admin card + Flow 10

---

### Session 7 — 2026-05-19 | index_62 → index_70

**Topic:** Valve Tag List form — full form, two flows (9 & 10), end-to-end working.

---

### Sessions 1–6 — 2026-05-07 to 2026-05-19 | index_21 → index_62

PDF annotation editor, photo capture, SharePoint integration, pan tool, folder navigation, forms.

---

## 📌 Power Automate Flow Reference

| # | Flow Name | Purpose | URL constant |
|---|---|---|---|
| 1 | Flow 1 - Create New Project Folder | Creates job folder + subfolders (incl. VALVE TAG + Toolbox Talks). Sets PM, App, Foreman columns | `CREATE_JOB_URL` |
| 2 | Flow 2 - Syncing Projects Folders | Returns `[{code, name, pm, foreman, app}]` | `PROJECTS_FETCH_URL` |
| 3 | Flow 3 - Update PM | Updates PM column on existing folder | `UPDATE_PM_URL` |
| 4 | Flow 4 - Change Foreman | Updates Foreman column on existing folder | `UPDATE_FOREMAN_URL` |
| 5 | Flow 5 - Fetch Folder Contents | Returns `{folders, files}` for any path | `FETCH_FOLDER_CONTENTS_URL` |
| 6 | Flow 6 - Camera Upload | Uploads photos/videos to date-subfolder | `UPLOAD_FILE_URL` |
| 7 | Flow 7 - PDF Editor - Get PDF File | Proxies SharePoint file as base64 | `DOWNLOAD_FILE_URL` |
| 8 | Flow 8 - PDF Markup Save | Uploads annotated PDF as `_vN` | `SAVE_MARKUP_URL` |
| 9 | Flow 9 - Save Valve Tags | Writes valve_tag_list.json to VALVE TAG folder | `SAVE_VALVE_TAGS_URL` |
| 10 | Flow 10 - Load Valve Tags | Reads valve_tag_list.json; 404 if not found | `LOAD_VALVE_TAGS_URL` |
| 11 | Flow 11 - Save Signed TBT PDF | Saves signed TBT, deletes original | `SAVE_TBT_URL` |
| 12 | Flow 12 - Toolbox Talks Rollover | Creates next-year TBT folder in every active job, copies 52 templates into each | `ROLLOVER_URL` |
| 13 | Flow 13 - Codes & Charts Sync | Returns `{items}` for Codes & Charts library at given path | `CODES_CHARTS_FETCH_URL` |
| 14 | Flow 14 - Admin Docs Sync | Returns `{items}` for Admin Docs library at given path | `ADMIN_DOCS_FETCH_URL` |
| 15 | Flow 15 - Load Employees | GET employees.csv from Templates library → returns `{csvData}` | `LOAD_EMPLOYEES_URL` |
| 16 | Flow 16 - Save Employees | POST `{csvData}` → writes employees.csv to Templates library | `SAVE_EMPLOYEES_URL` |
| 17 | Flow 17 - Save Photo Markup | POST base64 image → overwrites original file on SharePoint | `SAVE_PHOTO_MARKUP_URL` |

All 17 URLs are plaintext in the public GitHub JS. **Shared-secret check still unimplemented — deferred to Logins.**

---

_To append a new session: add a new `### Session N` block at the top of Session History, update "Current State" and "Still Open" at the top, update the date in line 2._
