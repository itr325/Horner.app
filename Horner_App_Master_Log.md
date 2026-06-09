# Horner Field App — Master Project Log
_Last updated: 2026-06-09 (Session 32)_
_Consolidates Handoffs 1–12 plus sessions 7–19. Append new sessions below "Session History."_

---

## ⚡ Current State

| Field | Value |
|---|---|
| **Current build** | `index_174.html` |
| **GitHub Pages URL** | `https://itr325.github.io/Horner.app/` |
| **SharePoint site** | `https://hornerplumbing.sharepoint.com` |
| **Active Projects path** | `/Active Projects` in the Documents library |

---

## 🔴 Still Open / To-Do

### Production Blockers
- [x] **`SHOW_DIAG`** — flipped to `false` in index_120 ✅ ~~flip back to `false` at line 88 before final production push. Was re-enabled in index_60 for field testing.

### Features / Work Items (rough priority order)
- [x] **Order Sheet material switch bug** — ✅ Fixed index_147. MatToggle warning gated on phase. Phase change only clears items on Cancel.
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

### Session Protocol

**START of every session:**
1. `curl` fetch the master log (cache-bust with `?$(date +%s)`) and read it fully
2. Check `**Next session:**` line at bottom of most recent session entry — that's what we're working on
3. Update memory if anything has changed

**END of every session:**
1. Append session summary to master log (newest first under Session History)
2. Update "Current State" table and "Still Open" list at top of log
3. **Ask Eric: "What are we working on next session?"**
4. Add `**Next session:** [answer]` as the last line of the session entry
5. Push updated log to GitHub
6. Update memory

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

### Session 32 — 2026-06-09 | index_168 → index_174

**Topic:** Per-item Add to Cart button + cart email sort/merge

**Closed:**
- Added **"Add to Cart" button** next to every QTY input on order sheets (standard `OrderSheet` only). Button is navy when qty entered, grey when empty. Tapping with no qty shows warning toast. Tapping with no phase selected shows phase warning. Field clears after adding.
- Existing footer "Add All to Cart" button retained.
- Layout: clean 3-column grid — ITEM (flex) | QTY centered (90px) | Add to Cart centered (100px).
- Cart email now **sorts items largest-to-smallest by size** within each phase (parses fractional sizes: `1-1/2"`, `3/4"`, `2"`, etc.). Items with no detectable size sort to bottom.
- Cart email now **merges duplicate items across forms** within the same phase — qty summed into single line.
- Form name sub-headers removed from email; each phase is one flat table.

**Next session:** New Home Page redesign.

---

### Session 31 — 2026-06-08 | index_167 → index_168

**Topic:** Timecard email formatting refinements + production cutover

**Changes:**
- Fixed timecard email Phase/Hours column layout: removed right-align on Hours, removed fixed 300px Phase width, removed width:100% from table so columns size naturally
- Set all four columns (Job, Date, Phase, Hours) to uniform 90px width
- Fixed DateCell overlay trick temporarily replaced with plain input for desktop Edge debugging — reverted back to overlay (app is iPad/iPhone only)
- Confirmed Flow 22 wired correctly — mailto replaced with flowPost
- Restored To: timecards@hornerplumbing.com after testing via eschieble@pinnacle-tec.com
- Bumped to index_168

**Still open:** Timecard custom job entry, Admin: Close Job, Logins, Photos .jpg, Back buttons, Horner Blue color swap.

---
### Session 30 — 2026-06-05 | index_166 → index_167

**Topic:** Flow 22 — Timecard Email via Power Automate / Office 365

**Changes:**
- Built Flow 22 (Send Timecard Email): identical structure to Flow 21 — HTTP trigger → Compose → Office 365 Outlook Send email (V2) from notifications@hornerplumbing.com
- Added `SEND_TIMECARD_URL` constant; replaced `mailto:` in `TimeCard.sendEmail` with `flowPost(SEND_TIMECARD_URL, ...)`
- Bug fix: original patch hit wrong `sendEmail` (a different form at line 7563); corrected to target TimeCard's `sendEmail` at line ~15497
- HTML email body built in-app: header table (Employee / Week Ending / Total Hours), then Regular Time and Labor sections each with Job / Date / Phase / Hours columns. Phase and Hours are split into separate columns; each phase on its own line with Total row at bottom separated by a horizontal rule.
- Plain-text body kept for in-app preview (unchanged)
- Form resets on successful send
- `DateCell` overlay trick temporarily replaced with plain date input to debug Edge desktop issue — reverted back to overlay after confirming it only needs to work on iPad/iPhone
- Bumped to index_167

**Flow 22 details:**
- Trigger: HTTP POST with JSON schema `{to, cc, subject, body}`
- Compose: `triggerBody()?['body']`
- Send email (V2): From notifications@hornerplumbing.com, To/CC/Subject/Body from trigger body, Is HTML = Yes
- To: `timecards@hornerplumbing.com`, CC: project PM email

**Still open:** Timecard custom job entry, Admin: Close Job, Logins, Photos .jpg, Back buttons, Horner Blue color swap.


---

### Session 29 — 2026-06-04 | index_156 → index_164

**Topic:** Flow 21 — Cart Email via Power Automate / Office 365

**Changes:**
- Built Flow 21 (Send Cart Email): HTTP trigger → Compose (wrap body in HTML div) → Office 365 Outlook Send email (V2) from notifications@hornerplumbing.com
- Added `SEND_CART_URL` constant; replaced `mailto:` in `CartSummaryView.sendEmail` with `flowPost(SEND_CART_URL, ...)`
- Switched from plain-text email body to HTML built in the app: header table (Job#/Date/Foreman/Phone), phase dividers (`<hr>`), per-form `<table>` with QTY/P/N/ITEM columns. Plain-text body kept for in-app preview.
- Tuned email spacing: hr margin → 0, phase div margin → 0, form name top margin → 6px
- Cart now clears (`onClearCart()`) automatically on successful send
- Fixed global cart FAB: was rendering in project list toolbar — moved to fixed bottom-right, hidden on `view === "projects"`, visible only inside job views
- To/CC temporarily redirected to eschieble@pinnacle-tec.com for testing

**Still open:** Restore To → `CONTACTS.purchasing` when Aaron approves. Flow 21 Compose step updated to pass `triggerBody()?['body']` directly (app sends HTML now).

---

### Session 28 — 2026-06-04 | index_147 → index_148

**Topic:** MatToggle material switch bug fixes (continued)

**Code changes:**
- `index_148`: MatToggle onChange — if no phase selected and user switches material with a qty set, show alert "Please select a phase before adding to cart." and block the action entirely
- `index_148`: MatToggle onChange — if phase IS selected and user switches material with qty set, confirm prompt appears; qty only clears if user clicks Cancel (was clearing unconditionally)

**No new Power Automate flows.**

---

### Session 29 — 2026-06-04 | index_156 → index_164

**Topic:** Flow 21 — Cart Email via Power Automate / Office 365

**Changes:**
- Built Flow 21 (Send Cart Email): HTTP trigger → Compose (wrap body in HTML div) → Office 365 Outlook Send email (V2) from notifications@hornerplumbing.com
- Added `SEND_CART_URL` constant; replaced `mailto:` in `CartSummaryView.sendEmail` with `flowPost(SEND_CART_URL, ...)`
- Switched from plain-text email body to HTML built in the app: header table (Job#/Date/Foreman/Phone), phase dividers (`<hr>`), per-form `<table>` with QTY/P/N/ITEM columns. Plain-text body kept for in-app preview.
- Tuned email spacing: hr margin → 0, phase div margin → 0, form name top margin → 6px
- Cart now clears (`onClearCart()`) automatically on successful send
- Fixed global cart FAB: was rendering in project list toolbar — moved to fixed bottom-right, hidden on `view === "projects"`, visible only inside job views
- To/CC temporarily redirected to eschieble@pinnacle-tec.com for testing

**Still open:** Restore To → `CONTACTS.purchasing` when Aaron approves. Flow 21 Compose step updated to pass `triggerBody()?['body']` directly (app sends HTML now).

---

### Session 28 — 2026-06-04 | index_146 → index_147

**Topic:** Order sheet bug fixes

**Code changes:**
- `index_147`: Fixed MatToggle orange warning — now only shows when a phase is already selected (`warning: needsMat && !!phase`)
- `index_147`: Fixed OrderSheet phase change — items now only clear if foreman clicks Cancel on the "Add to cart?" prompt (was clearing unconditionally)
- `index_147`: Same phase change fix applied to BlankOrderSheet

**No new Power Automate flows.**

---

### Session 29 — 2026-06-04 | index_156 → index_164

**Topic:** Flow 21 — Cart Email via Power Automate / Office 365

**Changes:**
- Built Flow 21 (Send Cart Email): HTTP trigger → Compose (wrap body in HTML div) → Office 365 Outlook Send email (V2) from notifications@hornerplumbing.com
- Added `SEND_CART_URL` constant; replaced `mailto:` in `CartSummaryView.sendEmail` with `flowPost(SEND_CART_URL, ...)`
- Switched from plain-text email body to HTML built in the app: header table (Job#/Date/Foreman/Phone), phase dividers (`<hr>`), per-form `<table>` with QTY/P/N/ITEM columns. Plain-text body kept for in-app preview.
- Tuned email spacing: hr margin → 0, phase div margin → 0, form name top margin → 6px
- Cart now clears (`onClearCart()`) automatically on successful send
- Fixed global cart FAB: was rendering in project list toolbar — moved to fixed bottom-right, hidden on `view === "projects"`, visible only inside job views
- To/CC temporarily redirected to eschieble@pinnacle-tec.com for testing

**Still open:** Restore To → `CONTACTS.purchasing` when Aaron approves. Flow 21 Compose step updated to pass `triggerBody()?['body']` directly (app sends HTML now).

---

### Session 28 — 2026-06-04 | index_146 → index_148

**Topic:** Order sheet bug fixes + cart debugging

**Code changes:**
- `index_147`: MatToggle orange warning now only shows when phase is already selected (`warning: needsMat && !!phase`)
- `index_147`: Phase change on OrderSheet — items only clear if user clicks Cancel on the "Add to cart?" prompt (was clearing unconditionally)
- `index_147`: Same phase change fix applied to BlankOrderSheet
- `index_148`: MatToggle onChange — if no phase selected and user switches material with qty set, alert "Please select a phase before adding to cart." and block entirely
- `index_148`: MatToggle onChange — if phase IS selected, qty only clears on Cancel (not on OK)

**Cart debugging:**
- Flow 19 (Save Cart) was failing with DirectoryNotFoundException — Build_Target_Path Compose was building `/Active Projects/(CODE) Name/cart.json` and passing that as the folder path to GetFolderByServerRelativeUrl, which then looked for a folder named cart.json. Fix: removed `/cart.json` from the Compose expression — folder path only, filename stays in the Files/Add action.
- Flow 19 body field changed to `binary(triggerBody()?['data'])` for correct binary encoding
- After fix: Flow 19 save and Flow 20 load both confirmed working. Cart persists across sessions.

**No new Power Automate flows.**

---

### Session 29 — 2026-06-04 | index_156 → index_164

**Topic:** Flow 21 — Cart Email via Power Automate / Office 365

**Changes:**
- Built Flow 21 (Send Cart Email): HTTP trigger → Compose (wrap body in HTML div) → Office 365 Outlook Send email (V2) from notifications@hornerplumbing.com
- Added `SEND_CART_URL` constant; replaced `mailto:` in `CartSummaryView.sendEmail` with `flowPost(SEND_CART_URL, ...)`
- Switched from plain-text email body to HTML built in the app: header table (Job#/Date/Foreman/Phone), phase dividers (`<hr>`), per-form `<table>` with QTY/P/N/ITEM columns. Plain-text body kept for in-app preview.
- Tuned email spacing: hr margin → 0, phase div margin → 0, form name top margin → 6px
- Cart now clears (`onClearCart()`) automatically on successful send
- Fixed global cart FAB: was rendering in project list toolbar — moved to fixed bottom-right, hidden on `view === "projects"`, visible only inside job views
- To/CC temporarily redirected to eschieble@pinnacle-tec.com for testing

**Still open:** Restore To → `CONTACTS.purchasing` when Aaron approves. Flow 21 Compose step updated to pass `triggerBody()?['body']` directly (app sends HTML now).

---

### Session 28 — 2026-06-04 | index_146 → index_156

**Topic:** Order sheet bug fixes, cart wiring, email formatting

**Code changes:**
- `index_147`: MatToggle orange warning gated on phase (`warning: needsMat && !!phase`)
- `index_147`: Phase change on OrderSheet/BlankOrderSheet — items only clear on Cancel
- `index_148`: MatToggle onChange — if no phase selected, alert blocks action entirely; qty only clears on Cancel when phase IS set
- `index_149`: Project list cart button calls `loadCart(p)` + `setProject(p)` before opening cart
- `index_150`: Added global cart icon next to Refresh in project list header; added cart button between Submit/Reset on all order sheet footers
- `index_151`: Moved cart button from footer to green FAB (bottom right); FAB now opens global cart
- `index_152`: FAB background changed to white with border; badge restored to red/white
- `index_153`: Email phase header spacing fixed (no extra blank lines around phase name); P/N column added to QTY/ITEM row
- `index_154`: Fixed-width space padding for columns (reverted next build)
- `index_155`: Back to tabs for QTY/P/N/ITEM columns
- `index_156`: Email preview switched to monospace font so tabs align correctly in preview

**Flow fixes:**
- Flow 19 (Save Cart): Build_Target_Path was appending `/cart.json` to folder path — removed it; folder only, filename stays in Files/Add action. Body changed to `binary(triggerBody()?['data'])`.
- Flow 19/20 confirmed working — cart persists across sessions.

**Up next:**
- Flow 21: Send cart email via Power Automate / Graph API (HTML formatted email, not mailto)

---

### Session 27 — 2026-06-04 | index_127 → index_146

**Topic:** Shopping Cart feature + misc fixes

**Code changes:**

- `index_127`: Added `valvetag` to `formViews` (back button padding fix)
- `index_128`: Fixed photo Skip path — re-encodes through canvas to guarantee real JPEG bytes (was uploading raw HEIC with .jpg extension)
- `index_129`: Shopping Cart feature built end-to-end:
  - `CartSummaryView` component — grouped by Phase → Form Type, foreman/phone fields, remove items, Clear Cart, Submit Cart
  - All OrderSheet + BlankOrderSheet forms get Add to Cart button + foreman pre-populated from project.foreman
  - Cart stored per-job as `cart.json` in job root folder on SharePoint (Flows 19 & 20)
  - Cart icon in job card header with badge
  - Email format: Job#/Date/Foreman/Phone header, then ━━━ Phase dividers, form name + ━━━ underline, QTY/ITEM columns
- `index_130`: Wired SAVE_CART_URL and LOAD_CART_URL (Flows 19 & 20)
- `index_131`: Cart icon opens cart + closes on Send; renamed "Open in Mail" → "Send" everywhere
- `index_132–135`: Cart icon UI iterations — folder icon (SVG, grey/silver on NAVY bg) in project list; cart icon always visible alongside folder; pre-fetch cart counts for all projects on startup so project list shows badges
- `index_136`: Pre-fetch cart counts after projects load (background calls, one per project)
- `index_137`: Both folder and cart icons always visible on project list
- `index_138`: SVG folder icon (grey/silver) on NAVY background square; cart badge overlay
- `index_139–140`: Cart icon + badge always visible; badge only on cart not folder
- `index_141`: Cart email body reformatted to match Word doc template (Job#/Date/Foreman/Phone header + phase dividers + form name + ━━━ underline + QTY/ITEM)
- `index_142`: Added ━━━ divider line under order sheet name in cart email
- `index_143`: Cart icon on project list opens cart directly (stopPropagation)
- `index_144`: Cart and TOP FABs always visible on all order sheets (removed `ordered.length > 0` condition)
- `index_145–146`: Phase change warning (prompt to add to cart before clearing), material change warning (same on MatToggle switch); cart button moved to right side of project card after PM/Foreman badges, opens cart only

**New flows:**
- Flow 19 - Save Cart: POST `{code, name, data}` → writes cart.json to job root folder
- Flow 20 - Load Cart: GET cart.json from job root folder → returns `{data}`

**Flow 21 planned (not built):** Send formatted HTML cart email via Microsoft Graph from notifications@hornerplumbing.com — to replace mailto when ready.

**Open items carried forward:**
- Flow 21 — HTML cart email via Graph (build when ready)
- Horner Blue (#0156A4) color swap — replace all NAVY (#1B3A6B) in a dedicated session
- Timecard custom job entry
- Admin: Close Job
- Logins & role-based permissions

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

### Session 26 — 2026-06-01 | index_124 → index_126

**Topic:** Flow 18 fixes + PM dropdown migration + Position/email card updates

**Code changes:**
- `index_124`: Added "Project Manager" to `EMP_POSITIONS` dropdown in Add/Remove Employee panel
- `index_125`: Replaced hardcoded `PMS` object with `getPMs()` / `getPmEmail()` helpers driven by `EMPLOYEES` filtered by `position === "Project Manager"`. PM now stored as full name string (same as Foreman). All PM dropdowns (New Job + Change PM) pull from EMPLOYEES. All form email lookups use getPmEmail(). PMS object deleted entirely.
- `index_126`: Email card label changed from "PM - Name" to "Project Manager - Name"

**Flow 18 fixes (no new flow):**
- Root cause of filter failure: "Include Nested Items" was true (default) — returning all files across all subfolders, causing Filter array to match wrong items
- Fix: Include Nested Items = false + Filter array condition changed from `contains` to `is equal to` with `split(triggerOutputs()?['body/{FullPath}'], '/')[1]` for exact job folder match
- Trigger condition added to only fire on SUBMITTALS and PLANS folders: `@or(contains(triggerOutputs()?['body/{FullPath}'], '/SUBMITTALS/'), contains(triggerOutputs()?['body/{FullPath}'], '/PLANS/'))` — note ALL CAPS folder names
- Email subject: `A file has been added or updated for {job name}`
- Email body: `{filename} has been added or updated for {job} in {folder path}`
- notifications@hornerplumbing.com shared mailbox confirmed as sender

**Next session:** Expand Flow 18 notifications to additional folders as needed.

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
| 18 | Flow 18 - Submittal Notifications | Emails PM + CC Foreman when files added/updated in SUBMITTALS or PLANS folders | — (trigger-based) |
| 19 | Flow 19 - Save Cart | POST `{code, name, data}` → writes cart.json to job root folder | `SAVE_CART_URL` |
| 20 | Flow 20 - Load Cart | GET cart.json from job root folder → returns `{data}` | `LOAD_CART_URL` |
| 21 | Flow 21 - Send Cart Email | POST `{to, cc, subject, body}` → sends HTML email via Office 365 Outlook from notifications@hornerplumbing.com | `SEND_CART_URL` |
| 22 | Flow 22 - Send Timecard Email | POST `{to, cc, subject, body}` → sends HTML timecard email via Office 365 Outlook from notifications@hornerplumbing.com | `SEND_TIMECARD_URL` |

All 22 URLs are plaintext in the public GitHub JS. **Shared-secret check still unimplemented — deferred to Logins.**

---

_To append a new session: add a new `### Session N` block at the top of Session History, update "Current State" and "Still Open" at the top, update the date in line 2._
