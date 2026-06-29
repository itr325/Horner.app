### Session 70
**Date:** 2026-06-29
**Build:** index_323 → index_325

**What we did:**
- **Remove form tile dots** (index_324): Removed the small blue dot (●) indicator from the top-right corner of folder tiles that contain fillable forms. Also removed the matching legend line ("● Blue dot = fillable forms available") from the project detail view.
- **Fix blank form renders** (index_325): ValveTagList, DailyLog, TMLog, and SiteWeeklySafety components were all fully coded but their React.createElement calls were never wired into the main app return — tapping those form tiles navigated correctly but rendered a blank screen. Pre-existing bug, likely introduced during the form tile UI refactor in index_321. All four now render correctly.

**Next session:** IIS migration — move app off GitHub Pages to horner.app on company IIS server.

---

### Session 70
**Date:** 2026-06-29
**Build:** index_323 → index_324

**What we did:**
- **Remove form tile dots** (index_324): Removed the blue dot (●) indicator that appeared in the top-right corner of folder tiles that contain fillable forms. Also removed the matching legend line ("● Blue dot = fillable forms available") from the project detail view.

**Next session:** TBD

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
- **Form list headings** (index_321): Replaced the static "Fillable Forms" left-aligned small-caps heading with a dynamic centered title based on `pathKey`:
  - FORMS → "Forms"
  - ORDER SHEETS → "Order Forms"
  - RETURNS → "Returns"
  - TIME CARDS → "Time Cards / Site Safety Inspection"
  - T&M → "T & M"
  - SAFETY → "Safety"
  - VALVE TAG → "Valve Tag List"
- **Form row icon removed** (index_321): Removed the 📋 clipboard icon div from every form list row across all folders.
- **"Fillable form" subtitle removed** (index_321): Removed the "Fillable form · submits via Outlook" subtitle from every form row.
- **Shop Orders view** (index_321): Changed "Shop Forms" heading to "Shop Order Forms" (centered); removed 📋 icon and "Fillable form · emails to shoporder@hornerplumbing.com" subtitle from the Shop Order Sheet row.

**Next session:** TBD

---

### Session 67
**Date:** 2026-06-26
**Build:** index_316 → index_320

**What we did:**
- **Pipe length 10'/20' buttons** (index_317): Added `sizeOpts`/`upcBySize` pattern to CPVC Pipe, PVC SCH40 Solid Pipe, and SCH80 CPVC Pipe sections. CPVC has full 10' and 20' UPCs; PVC and SCH80 have 10' UPCs only (20' TBD from Aaron).
- **PEX Ball Valves** (index_317): New section added to PEX Order Sheet before CPVC Ball Valves — 5 sizes (1-1/2" through 1/2") with UPONOR UPCs.
- **Case Quantity Reference Sheet removed** (index_317): Full removal — menu entry, `const CASE_QTY` array, routing, `formViews` entry, and render call all deleted.
- **Size button style standardized** (index_318): All `sizeOpts` buttons (10'/20' pipe lengths, braided supply lines) now match BLK/GAL/BRA MatToggle style — `borderRadius: 6`, `padding: 6px 8px`, `fontSize: 13`, `minWidth: 36`. This is now the default for all future size buttons.
- **PVC Foam Core Pipe** (index_318): Added 10'/20' buttons for all 7 sizes (20' UPCs TBD).
- **Copper Order Sheet** (index_319): New order sheet added between BLK-GAL-BRA and CPVC in the menu. Three sections: Type M Copper Pipe (8 sizes, 10'/20' buttons), Type L Copper Pipe (9 sizes, 10'/20' buttons), Level-Wound Copper (2 items, single UPC). UPCs populated where available; blanks where Aaron still needs to provide.
- **Copper color size buttons** (index_320): Size buttons on Copper Order Sheet render in copper color (`#B87333`) instead of navy. Other sheets unaffected. Uses new `sizeColor` property on sections — can be applied to any future sheet.

**Pending UPCs (Aaron to provide):**
- PVC SCH40 20' lengths (all sizes)
- PVC Foam Core 20' lengths (all sizes)
- SCH80 CPVC 20' lengths (all sizes)
- Type M Copper: 10' for 1"–2-1/2", both lengths for 3" and 2-1/2"
- Type L Copper: 10' for all sizes except 1/2"
- Copper fittings (ProPress) — Aaron to provide when ready

**Next session:** Aesthetics cleanup on the order sheet forms

---

### Session 66
**Date:** 2026-06-26
**Build:** index_305 → index_316

**What we did:**
- **Residential timecard job# search fixes** (index_305/306):
  - Changed trigger threshold from 2 → 4 characters before dropdown appears
  - Restored blur() on dropdown show (keyboard dismisses when suggestions appear)
  - Added onFocus handler so tapping back into field restores keyboard and re-shows suggestions
  - Note: index_302 was pushed in error (wrong base file — cloned index_301 instead of checking actual latest); corrected to index_305
- **GitHub cleanup:** Deleted 197 old build files; repo now keeps only index_301–316 (rolling window of ~15)
- **Week Ending field height** (index_307–315): Long battle with iOS Safari date input rendering — tried height, padding, overflow, grid column ratio fixes. iOS date input ignores most CSS overrides.
- **Custom CalendarPicker component** (index_316):
  - Replaced native `<input type="date">` with a custom React calendar modal
  - Tapping the Week Ending field opens a full month calendar overlay
  - Month navigation with ‹ › arrows
  - Today highlighted with navy border; selected date filled navy
  - Tapping outside modal closes it
  - Displays as `Jun 26, 2026` format
  - Defaults to Friday of current week (existing logic preserved)
  - Reverted employee select height back to default inputStyle (removed height:44)
  - Restored equal 1fr 1fr grid columns
- **IIS / horner.build discussion:** Plan to move app off GitHub Pages to company IIS server — solves cache/stale build issues with no-cache headers. Eric has domain horner.build ready.
- **iOS App Store discussion:** WKWebView wrapper or PWA are viable paths; tabled for future after IIS migration.

**Next session:** Order sheet changes — Aaron has provided additional items to add. Clarify all requirements before touching any code.

---

### Session 65
**Date:** 2026-06-25
**Build:** index_301 → index_304

**What we did:**
- Implemented cart email line item sorting by material sheet order: PVC → CPVC → SCH80 → PEX → other
- Added `EMAIL_ITEM_SORT_KEY` IIFE lookup (built from PVC/CPVC/SCH80/PEX arrays at runtime, maps item name → sheet order index)
- Sort priority: sheet order first, then largest-to-smallest size within each sheet, then alphabetical tiebreaker
- Applied same sort to both `CartSummaryView` and `ShopCartSummaryView` `mergePhaseItems` functions
- Fixed broken Submit Cart button (index_303 had sort referencing `EMAIL_ITEM_SORT_KEY` before it was defined — lookup was never injected due to earlier patch abort; fixed in index_304)
- Confirmed pipe-before-fittings ordering is naturally handled by size sort (larger diameter pipe floats above fittings)

**Next session:** Add lengths to all pipe sizes across PVC, CPVC, SCH80, PEX order sheets.

---

# Horner Field App — Master Project Log
Last updated: 2026-06-24

### Session 62
**Date:** 2026-06-24
**Build:** index_297 → index_299

**What we did:**
- **index_298 (reverted):** Made incorrect changes to Shop Order Sheet email routing (changed purchasing@ to shoporder@) without fully understanding what change was actually needed. Reverted immediately to index_297 state as index_299.
- **index_299:** Identical to index_297 — revert of index_298.
- **Lesson:** Always clarify exactly what change is needed before touching email routing or any send logic.

**Next session:** Shop Order Sheet — ask Eric to explain exactly what needs to change before writing any code.

---

### Session 61
**Date:** 2026-06-24
**Build:** index_296 → index_297

**What we did:**
- **iOS keyboard dismiss on job search** (index_297) — in `handleSearch`, when suggestions appear (≥2 chars match), call `inputEl.blur()` immediately so the keyboard collapses before the list renders. Fixes landscape mode on iPhone where the keyboard was covering 2/3 of the screen and obscuring the job suggestion list.
- Note: `blur()` is not possible to suppress on the iOS input accessory bar (the white bar with up/down arrows and checkmark above the keyboard) — that is a native Safari UI element with no web API to remove it.

**Next session:** Aesthetics cleanup on the order sheet forms

---

### Session 60
**Date:** 2026-06-24
**Build:** index_295 → index_296

**What we did:**
- **Residential timecard — phase + day expansion** (index_296)
  - Added Phase 9 (Warranty) and Phase 17 (Service) to `RES_PHASES` array
  - Added Saturday and Sunday columns to `DAYS` array (was Mon–Fri, now Mon–Sun)
  - Updated `mkRow` to initialize `Sat` and `Sun` hour slots
  - Updated both row-input grid and footer totals grid from `repeat(5,1fr)` to `repeat(7,1fr)`
  - Email output picks up all changes automatically (loops over `DAYS` and `RES_PHASES` dynamically)

**Next session:** Aesthetics cleanup on the order sheet forms

---

### Session 59
**Date:** 2026-06-24
**Build:** index_293 → index_295

**What we did:**
- **Finish Order Sheet — size selection for 5 items** using existing `sizeOpts`/`upcBySize` pattern
- **Water Closet section** (index_294) — added size buttons + per-size UPCs to 4 items:
  - `PEX WC Supply 12"/20"`: sizeOpts 12"/20", upcBySize SIOUX 287-12 / SIOUX 287-20
  - `Braided WC Supply 9"/12"/16"`: sizeOpts 9"/12"/16", upcBySize BRASSCRAFT B1-9DL-F / B1-12DL-F / B1-16DL-F
  - `Braided 1/2"x3/8" (16"/20"/30")`: sizeOpts 16"/20"/30", upcBySize BRASSCRAFT B1-16A-F / B1-20A-F / PROFLO PFX146326
  - `Braided 3/8"x3/8" (16"/20")`: sizeOpts 16"/20", upcBySize PROFLO PFX146343 / PFX146344
- **Vacuum Breakers & Vents section** (index_295) — added size buttons + per-size UPCs to Maxitrol:
  - `Maxitrol Regulator (1/2", 3/4")`: sizeOpts 1/2"/3/4", upcBySize MAXI 325-3L / MAXI 325-5A-66
- All items: entering a qty shows size buttons, row highlights orange until size selected, correct UPC pulls per selection, cart/submit blocked until size chosen
- Tested and confirmed working on iPad ✅

**Next session:** Residential timecard — add 2 new phases (names TBD, Eric to provide) + Saturday and Sunday hour columns.

---


### Session 58
**Date:** 2026-06-23
**Build:** index_292 → index_293

**What we did:**
- **Residential & Service Home Pages** (index_292) — Residential and Service tiles on home now route to their own home pages (`homeResidential`, `homeService`) each with 2 tiles: Time Cards + Order Sheets
  - Residential Time Cards routes to existing `res-timecard` (fully working)
  - All other tiles show "Coming soon" alert
  - Breadcrumbs updated: res-timecard shows Home › Residential › Time Cards
- **Removed Team Email Routing card** (index_293) from Commercial home page — card still present in Admin panel

**Next session:** More intensive work — TBD (likely Residential Order Sheets or timecard custom job entry).

---

### Session 57
**Date:** 2026-06-23
**Build:** index_291 → index_292

**What we did:**
- **Residential Home Page** — Residential tile on home now routes to `homeResidential` (was going straight to `res-timecard`)
- `homeResidential` view: 2-tile grid — Time Cards (routes to `res-timecard`) + Order Sheets ("Coming soon" alert)
- **Service Home Page** — Service tile on home now routes to `homeService` (was showing "coming soon" alert)
- `homeService` view: 2-tile grid — Time Cards ("Coming soon") + Order Sheets ("Coming soon")
- Both home pages use same responsive tile pattern as Commercial (isMobile-aware sizing)
- Breadcrumbs updated: `res-timecard` now shows Home › Residential › Time Cards
- `homeResidential` breadcrumb: Home › Residential; `homeService`: Home › Service

**Next session:** Aesthetics cleanup on the order sheet forms — timecard custom job entry, or Service/Residential Order Sheets.

---

### Session 56
**Date:** 2026-06-23
**Build:** index_290 → index_291

**What we did:**
- **Responsive layout for iPhone** — app was iPad-optimized with several hardcoded fixed sizes that looked cramped/broken on small phone screens
- Added `isMobile` React state (tracks `window.innerWidth < 480`, updates on resize/orientation change) in App component
- **Logo:** Changed hardcoded `width="380"` to `width="100%"` with `max-width: 380px` — no longer overflows iPhone screen
- **Home tile grid:** `gridTemplateColumns` now `1fr 1fr` on phones, `1fr 1fr 1fr` on iPad
- **Home tile padding/icons:** Icon containers scale from `72×72` → `52×52`, padding `24px` → `16px` on phones
- **Tile label font:** 13px → 11px on phones
- **Commercial home grid:** Same 2/3-col responsive behavior
- **Main content:** `maxWidth` respects `isMobile`, padding tightens from `14px` → `10px`
- **CSS media query added** (`max-width: 479px`): input/select/textarea forced to `font-size: 16px` (prevents iOS Safari zoom-on-focus), breadcrumb font tightened
- JS validated with `new Function()` — no errors

**Next session:** Build Residential and Service Home Pages — each with 2 tiles: Time Cards and Order Sheets.

---



### Session 55
**Date:** 2026-06-23
**Build:** index_283 → index_290

**What we did:**
- **Completed Flow 25 (Get Residential Jobs) — end-to-end working:**
  - Excel Online (Business) connector confirmed unable to read ANY file in "Horner Shares" library regardless of file type
  - VBA macro updated: `ExportOpenJobsXlsx` replaced with `ExportOpenJobsCsv` — saves plain CSV via `wb.SaveAs` with full SharePoint URL, works on any machine/user
  - Flow 25 rebuilt: HTTP trigger → SharePoint "Get file content using path" (`/Horner Shares/Residential Shared/Shared Project Documents/res_jobs.csv`) → Response: `split(string(body('Get_file_content_using_path')), decodeUriComponent('%0D%0A'))` → Content-Type: application/json
  - Returns flat JSON array of job number strings ✅

- **Residential timecard UI fixes (index_284–290):**
  - **index_284:** Suggestions dropdown scrollable (maxHeight:220, overflowY:auto); removed 8-item slice cap
  - **index_285:** Week Ending defaults to Friday of current week; employee dropdown filtered to `department === "Residential"`
  - **index_286:** Reverted to all employees (only 1 Residential in CSV at that point)
  - **index_287:** Employee dropdown driven by React `employees` state passed as prop — updates live after Admin save
  - **index_288:** Re-applied Residential department filter
  - **index_289:** Fixed CSV parser bug — regex `filter((_, idx) => idx % 2 === 0)` was dropping every other column (position/department/email/phone). Replaced with simple `split(",")`
  - **index_290:** `AdminEmployeesView` re-fetches from SharePoint on every open via `useEffect` calling `loadEmployeesFromSharePoint`

- **Employee CSV issues found and fixed:**
  - Residential employees added last week were missing — saves had failed silently
  - Some IDs stored as scientific notation in Excel (timestamp IDs)
  - Eric edited `employees.csv` directly, fixed IDs, added Residential crew with `department: Residential`, re-uploaded to Templates library

**Current state of residential timecard:** Fully working end-to-end ✅

**Next session:** Build Residential Home Page — similar structure to Commercial (Active Projects tile grid, etc.)

---
### Session 54 — Addendum
**Date:** 2026-06-22

**Next session start checklist:**
1. Paste updated VBA macro into `Residential Projects.xlsm` (Alt+F11, replace module contents)
2. Click Refresh List button — confirm `res_jobs.csv` appears in `/Horner Shares/Residential Shared/Shared Project Documents/`
3. Rebuild Flow 25:
   - Delete current actions (Get file content + List rows + Select)
   - Add: **Get file content using path** → site: `https://hornerplumbing.sharepoint.com`, path: `/Horner Shares/Residential Shared/Shared Project Documents/res_jobs.csv`
   - Add: **Response** → body: (Expression) `split(string(body('Get_file_content_using_path')), decodeUriComponent('%0A'))` → Content-Type: `application/json`
   - This returns a flat JSON array of job number strings
4. Update app parsing if needed (currently handles flat string arrays ✅)
5. Test job search in residential timecard on iPad

---
### Session 54
**Date:** 2026-06-22
**Build:** index_282 → index_283

**What we did:**
- **Fixed employee dropdown** — was filtering to `department === "Residential"` only (showing 1 person). Changed to show all employees sorted by `firstName`. (index_283)
- **Debugged Flow 25 (Get Residential Jobs)** — long sequence of issues:
  - `Get file content using path`: wrong path (`/Cloud/Residential Shared/...`) → fixed to `/Horner Shares/Residential Shared/Shared Project Documents/Residential Projects.xlsm`
  - `List rows present in a table`: Document Library couldn't be selected from picker → typed `Horner Shares` as custom value; table name was `Project_DB` (underscore) → should be `Project-DB` (hyphen) but saving that caused a 404 because Excel Online connector can't resolve `.xlsm` files in non-standard libraries
  - Tried multiple workarounds: HTTP request to SharePoint, Graph API, code view edits — all failed due to `.xlsm` + non-standard library combination
  - **Final solution**: add CSV export to the VBA macro in `Residential Projects.xlsm`. When Refresh List is clicked, macro writes `res_jobs.csv` to the same SharePoint folder containing one open job number per line.
  - Added CSV export block to `RefreshAndPushToChildren()` in the macro, right before `Done:` label, using the already-loaded `data` array and `col` dictionary.
  - Flow 25 will be rewired next session to read `res_jobs.csv` (plain text) instead of the Excel table — no connector issues.

**Current state of Flow 25:**
- Still pointing at Excel `List rows` action (broken)
- Waiting for Eric to paste updated macro into the `.xlsm` file and run Refresh List to confirm `res_jobs.csv` appears in SharePoint
- Once CSV confirmed, Flow 25 needs to be rebuilt: Get file content (`res_jobs.csv`) → parse lines → Response JSON array

**Next session:** Confirm `res_jobs.csv` exists in SharePoint, rebuild Flow 25 to read CSV, wire up job search in the app, then test end-to-end residential timecard submit.

---
### Session 53
**Date:** 2026-06-22
**Build:** index_281 → index_282

**What we did:**
- **Fixed blank white screen on Residential tile**
  - Root cause: `ResidentialTimecardView` was defined *inside* the `TimeCard` function body (line 16710, right after `function TimeCard({` opened at 16704) — making it a locally-scoped nested function, invisible at global scope.
  - At render time, `React.createElement(ResidentialTimecardView, ...)` received `undefined` as the component → silent React failure → blank white screen.
  - Fix: moved `ResidentialTimecardView` to global scope, immediately before `TimeCard` (now line 16709; TimeCard at 16931).
  - JS validated with acorn — no syntax errors.

**Next session:** Verify residential timecard works end-to-end on device. Then: timecard custom job entry.

---
### Session 52
**Date:** 2026-06-22
**Build:** index_281 (no new build — debugging)

**What we did:**
- Discovered blank white screen when tapping Residential tile (index_281)
- Root cause not yet identified — likely a render crash in `ResidentialTimecardView` on mount
- Session ended before fix was applied

**Next session:** Debug blank screen in `ResidentialTimecardView`. Suspected causes:
1. `EmailPreview` component being called with wrong/missing props
2. `CONTACTS.timecards` undefined in this context
3. Something in the render tree crashing silently
- Strategy: add try/catch wrapper or strip component down to bare minimum and add back piece by piece

---
---
### Session 51
**Date:** 2026-06-18
**Build:** index_280 → index_281

**What we did:**
- **Flow 25 - Get Residential Jobs wired end-to-end:**
  - Built Flow 25: HTTP trigger → Get file content (SharePoint, `Residential_Projects.xlsm`) → List rows from `Project_DB` table → Filter array Status=Open → Select Job No. → Response JSON array
  - Added `GET_RES_JOBS_URL` constant (Flow 25 URL)
  - `ResidentialTimecardView` now fetches open job list on mount via `useEffect`
  - Response normalized to flat string array (handles both string and object-with-empty-key formats from the Select action)
  - Jobs sorted alphabetically; silent fail if flow errors — field still works as free-text input
  - Type 2+ characters in any job# field → matching open job numbers appear as suggestions

**Next session:** Aesthetics cleanup on the order sheet forms.

---
---
### Session 50
**Date:** 2026-06-18
**Build:** index_279 → index_280

**What we did:**
- **Residential Time Card — moved to correct location:**
  - Removed "Residential Time Card" from `FORM_FOLDERS["TIME CARDS"]` — it was incorrectly showing up inside every commercial job's TIME CARDS folder
  - Residential home tile now navigates to `"res-timecard"` view (was showing "Coming soon!" alert)
  - `ResidentialTimecardView` now renders as a standalone top-level view with breadcrumb: Home › Residential Time Card
  - No `project` prop (residential timecard has no commercial job context)

**Next session:** Build Flow 25 (Get Residential Jobs) — HTTP trigger → List rows from `Project_DB` table in Patrick's SharePoint Excel file → filter Status=Open → return Job No. array. Wire `GET_RES_JOBS_URL` constant into `ResidentialTimecardView` job# search.

---
---
### Session 49
**Date:** 2026-06-18
**Build:** index_278 → index_279

**What we did:**
- **Residential Time Card:** Built `ResidentialTimecardView` component and wired it into the app
  - Added "Residential Time Card" to `FORM_FOLDERS["TIME CARDS"]` (appears alongside Commercial Time Card)
  - `getFormView()` routes it to `"res-timecard"`; added to `formViews` array
  - Header card: Employee dropdown (filters by `department === "Residential"`, falls back to all) + Week Ending date picker (2-col grid)
  - Job rows: job# search field + Phase dropdown (Ph.3 Rough / Ph.6 Finish) on top line; Mon–Fri hour inputs + auto-calculated row total
  - Footer: daily column totals + weekly grand total
  - "+ Add Job Row" button (starts with 3 rows)
  - Submits via existing `SEND_TIMECARD_URL` flow to `timecards@hornerplumbing.com`
  - Job# search field ready for Flow 25 data (no live suggestions yet — empty array placeholder)
  - iOS-safe: no setState in touchstart

**Next session:** Build Flow 25 — fetch open residential job numbers from Patrick's SharePoint Excel file, wire into the ResidentialTimecardView job# search suggestions.

---
---
_Consolidates Handoffs 1–12 plus sessions 7–19. Append new sessions below "Session History."_

---

### Session 48
**Date:** 2026-06-17
**Build:** index_274 → index_278

**What we did:**
- **Cart indicators on ORDER SHEETS:**
  - **Active Projects list:** Cart icon now shows a small red dot (no count) if main cart OR shop cart has any items for that job.
  - **ORDER SHEETS folder tile** (job home): Red dot on top-left corner when either cart has items.
  - **Form tiles inside ORDER SHEETS:** Cart icon (🛒 in yellow square) always visible; orange count badge appears when that form has items in the main cart.
  - **Shop Orders tile:** Orange count badge showing shop cart item count (unchanged from before).
  - Went through several iterations: index_275 (badge only when items), index_276 (simplified conditional), index_277 (always show cart icon), index_278 (dot-only on project list + ORDER SHEETS tile, counts only on individual form tiles).

**Next session:** Aesthetics cleanup on the order sheet forms.

---
---

## ⚡ Current State

| Field | Value |
|---|---|
| **Current build** | `index_274.html` |
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
---

### Session 47
**Date:** 2026-06-16
**Build:** index_273 → index_274

**What we did:**
- **Scale picker modal (index_274):** Tapping measure tool now shows a persistent modal requiring scale selection before any dimension can be placed. Large tap-friendly buttons for all scale options. Last-used scale highlighted. Modal dismisses on selection. Scale label in toolbar turns yellow until confirmed. Resets on each new page load. Cal tool also counts as confirming scale.

**Next session:** Residential timecard, or further measure tool work.

---
---

### Session 46
**Date:** 2026-06-16
**Build:** index_239 → index_273

**What we did:**

**Pinch zoom jump fixed (index_242):** Removed "final pair" updateGesture call on finger lift that was using a lifted finger's position causing jumps. Also cleared panRef when pinch starts and re-baselined pan when pinch ends with one finger remaining.

**Measurement format (index_240):** Switched from inches-only to feet-inches display (e.g. 5'-10-1/4").

**Scale calibration fixed (index_239):** Removed `setScaleDropdownVal("custom")` from confirmCalib that was breaking non-calibrated pages. Existing measurement labels now recompute when scale changes.

**Cal tool reworked (index_245/246):** Replaced awkward two-tap calibration with a "Cal" button on the active line. Place line on known dimension, tap Cal, enter real-world length. distPx computed eagerly from activeMeasRef at button-tap time to avoid stale state.

**Measurements save as PDF vectors (index_256+):** Measurements now saved as crisp pdf-lib vector graphics (drawLine/drawText) instead of rasterized canvas overlay. Sizes scale proportionally to page size, capped at CAD-standard proportions. hasAnnotations updated to include measurements.

**PDF dimension text size fixed (index_257/258):** Font ~8pt, arrows ~7pt on D-size. Scales for C through E size sheets.

**Snap working (index_268-271):** 
- Discovered PDF.js 3.x compiles all path ops into constructPath (op 91) with sub-arrays
- Sub-op codes are raw OPS codes (moveTo=13, lineTo=14, closePath=18, rectangle=19)
- Added CTM stack tracking (save/restore/transform ops) to transform local coords to page space
- snapToVector rewritten to snap to nearest point ON nearest line segment (not just endpoints)
- 30,000+ segments extracted from typical Revit PDF

**Scale investigation:** Formula confirmed mathematically correct. Variation in measurements (e.g. two 8'-5" bays reading differently) traced to snap landing on different wall layers (face of stud vs face of drywall). Not a formula bug — inherent to drawing geometry.

**Display format (index_273):** Restored feet-inches at 1/8" precision.

**Next session:** Residential timecard, or continue measure tool refinements.

---
---

### Session 45
**Date:** 2026-06-15
**Build:** index_238 → index_239

**What we did:**
- **Measure tool — scale calibration fix:**
  - **Root bug:** `confirmCalib` was calling `setScaleDropdownVal("custom")` after calibration. `"custom"` is not in `SCALE_OPTIONS`, so `getPxPerFoot` returned `null` on any page without a `measureScalePerPage` entry — causing all measurements to show "NTS".
  - **Fix:** Removed `setScaleDropdownVal("custom")`. The dropdown already shows "Custom" via `value: measureScalePerPage[pageNum] ? "custom" : scaleDropdownVal` — no side effects needed.
  - **Bonus fix:** After calibration (`confirmCalib`) and after changing the scale dropdown, all previously committed measurements on that page now have their labels recomputed to match the new scale. Previously, only newly placed measurements got the correct label.

**Next session:** Aesthetics cleanup on the order sheet forms.

---
---

### Session 44
**Date:** 2026-06-15
**Build:** index_235 → index_238

**What we did:**
- **Shop Orders — separate cart & email destination:**
  - Added **Shop Orders** square tile at top of ORDER SHEETS form list (matches job folder tile style, shows cart badge count)
  - Removed "Shop Order Sheet" from the shared FORM_FOLDERS list so it no longer appears in the regular form list
  - Shop Orders tile opens a `shop-orders` sub-page listing only the Shop Order Sheet form
  - Shop Order Sheet now uses a completely separate cart (`shop_cart.json`) — independent from the main `cart.json` cart
  - Submitting the shop cart emails to `shoporder@hornerplumbing.com` (CC: PM) via new `ShopCartSummaryView` component
  - `ShopCartSummaryView` mirrors `CartSummaryView` but hardcodes `to: shoporder@hornerplumbing.com` and subject line says "Shop Order"
  - Modified **Flow 19 (Save Cart)** and **Flow 20 (Load Cart)** to accept optional `filename` parameter — defaults to `cart.json` if not provided, uses `shop_cart.json` for shop cart calls. No breaking change to existing cart flows.
  - `shop_cart.json` filtered from all file list views
- **Hide all .json files:** Changed all three file list filters (2x job folder views + LibraryBrowser) from specific filename exclusions to a blanket `!/\.json$/i.test(f.name)` filter — hides all JSON files in all views automatically

**Next session:** Fix measure tool — dimension scaling (scale calibration broken).

---


### Session 43
**Date:** 2026-06-12
**Build:** index_234

**What we did:**
- **Feedback form photo attachment — complete:**
  - Added `fbPhoto` state + hidden `<input type="file" accept="image/*">` to `FeedbackForm`
  - "📷 Attach Screenshot" button triggers native iOS photo picker
  - Selected image shows thumbnail preview with filename + ✕ remove button
  - Image base64-encoded in app (data URL prefix stripped); sent as `attachmentName` / `attachmentBase64` / `attachmentType` in Flow 23 payload
  - Updated Flow 23 (Send Feedback Email): Attachments → Content field changed to `base64ToBinary(triggerBody()?['attachmentBase64'])` — raw base64 string was causing unreadable attachment; `base64ToBinary()` fixes it
  - Attachment is optional — form submits normally with no attachment if none selected
  - Form resets including photo on successful submit

**Next session:** Fix measure tool — scale calibration broken + save dimensions broken after rework.

---

### Session 42
**Date:** 2026-06-12
**Build:** index_231 → index_233

**What we did:**
- **Timecard History — complete end-to-end:**
  - Updated Flow 22 (Send Timecard Email): now also saves an HTML copy to `TIME CARDS/TIMECARDS/` after sending. Filename format: `EmpName - MM-dd-yyyy.html`
  - Flow 1 already updated by Eric to create `TIMECARDS` subfolder inside `TIME CARDS` on new job creation
  - App payload to Flow 22 updated: added `code`, `name`, `fileContent`, `empName`, `weekEnd` fields (index_231)
  - In-app viewer already worked via existing `openTxtFile` / `txtViewer` — no extra code needed
- **TimeCard reset fix:** After sending, first 5 job rows now re-populate with project code (was resetting to blank) (index_232)
- **Timecard history filename:** Uses employee name + week ending date in MM-dd-yyyy format via `formatDateTime()` in Flow 22 expression (index_233). Email body/subject already used MM/DD/YY via app-side `fmtDate()`.

**Next session:** Add photo/file/screenshot upload to the Feedback form.

---

### Session 39
**Date:** 2026-06-11
**Build:** index_187 (no code changes)

**What we did:**
- No app changes this session
- Bulk job migration in progress: creating all active jobs via Admin panel, then dragging files into SharePoint folders
- Turned off **Flow 18 - Submittal Notifications** to prevent flooding PMs with emails during migration
- Confirmed Flow 18 already CCs Project Lead (Foreman field) — no changes needed
- Plan: turn Flow 18 back on after migration and spot-check with a test file drop

**Next session:** Review and address field crew feedback items.

---

### Session 38
**Date:** 2026-06-10
**Build:** index_186 → index_187

**What we did:**
- User uploaded actual `employees.csv` — confirmed the existing `id` column (values 1–47 for field crew) IS the Employee ID, not a timestamp. 4 PM-added employees have timestamp IDs that need cleanup via the Edit form.
- Added **Employee ID** field to the Add Employee form (numeric only, required, duplicate check)
- Added **Employee ID** field to the inline Edit Employee form (pre-populated from existing `id`, numeric only, required, duplicate check) — allows cleaning up timestamp IDs
- When saving via Add or Edit, the `id` field is now set to the user-entered number (not `Date.now()`)
- **Timecard email** now shows "Employee ID" row in the HTML header table and plain-text body — pulled from `emp.id` automatically on send
- Employee ID does NOT appear on the timecard form while filling out — email-only
- No new CSV column added; uses the existing `id` column as-is

**Next session:** Residential timecard.


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

All 24 URLs are plaintext in the public GitHub JS. **Shared-secret check still unimplemented — deferred to Logins.**

---

_To append a new session: add a new `### Session N` block at the top of Session History, update "Current State" and "Still Open" at the top, update the date in line 2._

---

### Session 33
**Date:** 2026-06-09
**Build:** index_175

**What we did:**
- Established new session protocol: START = fetch fresh log with cache-bust; END = ask Eric what's next, append Next session line
- Built new landing home page with three division tiles: Commercial, Residential, Service
- Added Admin tile (gear icon) below Commercial
- Embedded Horner Plumbing SVG logo centered above tiles
- Created custom SVG icons for Commercial (warehouse) and Residential (house); used Eric's uploaded Horner service van SVG for Service tile
- Residential and Service tiles show "Feature not yet enabled. Coming soon!" popup
- Commercial tile navigates to existing 3-tile screen (Active Projects, Codes & Charts, Admin Docs)
- Removed nav bar logo image (was duplicate of main logo)
- Changed nav bar background from white to Horner Blue (#0256A4) with white text
- Removed gear icon from nav bar top right (Admin now accessible via home tile)
- Replaced all back buttons with breadcrumb navigation throughout the app
- Breadcrumb chain: Home › Commercial › Active Projects › Job › Folder › SubPath
- Codes & Charts and Admin Docs now show libPath segments in top breadcrumb; removed duplicate internal LibraryBrowser breadcrumb
- Added favicon: white background with blue Horner H outline

**Next session:** Add employee IDs to timecards, then start on timecards for the Residential side.

---

### Session 34
**Date:** 2026-06-10
**Build:** index_176 → index_180

**What we did:**
- Moved Admin back to small gear icon (top-right of nav bar, 70% opacity) — removed Admin tile from home page (unauthenticated panel should be low-profile)
- Hid `cart.json` from folder file list (same filter as `valve_tag_list.json`) — both views updated
- Added **Feedback & Issues** tile to home page (chat bubble icon, 4th tile in grid)
- Built `FeedbackForm` as proper React component (initial attempt used IIFE which broke hooks — fixed)
- Form fields: Name (dropdown — PMs + Foremen sorted by first name), Section (12 visible folder tiles), Feedback/Issue radio, Message textarea, Submit button
- Submit sends HTML email via Flow 23: To eschieble@pinnacle-tec.com, CC phorner@hornerplumbing.com
- Built **Flow 23 - Send Feedback Email**: HTTP trigger → Send an email (V2), same pattern as Flows 21 & 22
- Wired `SEND_FEEDBACK_URL` constant — end-to-end tested and working
- Added Flow 23 to flow reference table

**Next session:** Employee IDs on timecards, then Residential timecard.

---

### Session 35
**Date:** 2026-06-10
**Build:** index_181

**What we did:**
- Diagnosed Flow 1 (Create New Project Folder) failing with BadGateway / 0x8007007B path syntax error
- Root cause: `#` character in job name (e.g. "Townhomes #1") breaks SharePoint REST API URL paths — `#` is a URL fragment identifier and truncates the path
- Added special character validation to the New Job submit handler — blocks `# % & * : < > ? / \ | " { }` in both Job Code and Job Name fields
- Shows `alert()` popup with clear message listing blocked characters; flow is never called if invalid
- Deleted 3 broken partial folders from SharePoint (SEL-25103, SEL-25103.5, SEL-25104) and re-created with clean names

**Next session:** Employee IDs on timecards, then Residential timecard.

---

### Session 36
**Date:** 2026-06-10
**Build:** index_182

**What we did:**
- Renamed all UI-facing "Foreman" labels to "Project Lead" throughout the app
- Affected: project cards, New Job form, Admin tile/breadcrumb, Change PM/Project Lead section, all confirm/toast messages, Timecard, TBT, Weekly Safety, cart email body
- All underlying data keys (foreman, foremanName, editForeman), position filter strings (position === "Foreman"), and SharePoint field names left unchanged — no flow or data breakage
- Fixed BUILD_ID regression — working file was copied from index_180 instead of index_181, causing stamp to show index_180. Corrected in final push.
- Both session 35 (special char validation) and session 36 (Project Lead rename) are live in index_182

**Next session:** Employee IDs on timecards, then Residential timecard.

---

### Session 37
**Date:** 2026-06-10
**Build:** index_185

**What we did:**
- Diagnosed Flow 2 (Syncing Projects) returning lowercase PM names — root cause was `@toLower()` wrapper in the Select action's `pm` mapping, left over from the old hardcoded PMS object era. Fixed by removing `toLower()` — now returns PM names exactly as stored in SharePoint.
- Expanded Project Lead dropdowns (New Job form + Change Project Lead admin panel) to include positions: Foreman, Journeymen, 5th Year, and 4th Year — per Patrick's guidance that Project Lead is a role, not a job title
- Fixed BUILD_ID regression (working file copied from stale index — corrected at push time)

**Note:** All changes from Sessions 35 + 36 (special char validation, Foreman→Project Lead rename) are also in index_185 via index_181–184.

**Next session:** Employee IDs on timecards, then Residential timecard.

---

### Session 40
**Date:** 2026-06-11
**Build:** index_223 (sessions spanned index_188–223)

**What we did:**
- **Measure tool complete rework** based on field crew feedback from PDF Expert comparison
- New interaction model: tap to place line (centered on tap, horizontal default) → drag either endpoint to position → Done to commit
- Replaced 3-phase drag gesture with tap-to-place + endpoint drag
- Dimension format: inches only with 1/4" precision (e.g. `70-1/4"`) — removed feet
- Vector snapping: parses Revit PDF content stream for line/path geometry, snaps endpoints within 18px tolerance
- Max zoom bumped from 25x to 100x
- Sharp line during drag: screen-space overlay canvas (`measOverlayCanvasRef`) sits outside zoom wrapper — draws measurement imperatively in screen coords on every touchmove frame. Detail canvas stays visible so PDF remains sharp behind it. Annotation canvas hidden during drag, restored on touchend.

**Key debugging discoveries:**
- iOS cancels touchmove delivery if `setState`/`rerender()` is called during touchstart — solved by removing all React state updates from touchstart
- `onTouchEnd` measure branch was intercepting pinch-end events (touches.length===0 + tool==="measure") before gesture handler could fire `scheduleDetail()` — fixed with `&& !gestureRef.current` guard
- Imperative canvas draw during touchmove (same pattern as crosshair cursor) is the only reliable approach — React re-renders during touch sequence break iOS touch tracking
- Detail canvas covers annotation canvas at zoom>1 — must hide annotation canvas AND keep detail canvas visible during drag for sharp PDF + sharp line simultaneously

**Reverted:**
- 3x page canvas oversample experiment — caused PDF to load zoomed in with no zoom-out; reverted. pdf.js CPU rendering can't match PDF Expert's native GPU vector rendering — accepted limitation of free solution.

**Next session:** Field crew feedback items (continued). Residential timecard still on backlog.

---

### Session 41
**Date:** 2026-06-11
**Build:** index_230 (sessions spanned index_224–230)

**What we did:**

**Order History — complete end-to-end:**
- Flow 21 (Send Cart Email) updated: now also saves an HTML copy of the order to `ORDER SHEETS/ORDERS/Order_YYYY-MM-DD_HH-mm.html` in SharePoint after sending the email
- App payload to Flow 21 updated: added `code`, `name`, `fileContent` (HTML body) fields alongside existing email fields — no change to email format
- Flow 1 (Create Job) updated: now creates `ORDERS` subfolder inside `ORDER SHEETS` for every new job
- Flow 24 - Get File Content: new flow (3 steps — HTTP trigger → Get file content using path → Response). Returns raw file content. URL: stored in GET_FILE_CONTENT_URL constant
- App: in-app HTML viewer modal — tap any .html file in ORDERS folder → Flow 24 fetches content → renders in iframe with full email layout. Clean, fast, looks identical to the cart email preview.
- `.html` and `.txt` files now handled by `openTxtFile()` in job view instead of browser open

**Known issues for next session:**
- Measure tool: scale calibration not working correctly (Patrick testing)
- Measure tool: saving dimensions no longer works after the rework
- Timecard history — same pattern as order history, not yet built

**Flows (24 total):**
- Flow 24 - Get File Content added this session

**Next session:** Fix measure tool — scale calibration + save dimensions. Then timecard history.






### Session 63
**Date:** 2026-06-24
**Build:** index_299 → index_300

**What we did:**
- Fixed Shop Order Sheet direct Submit button emailing `purchasing@` instead of `shoporder@hornerplumbing.com`
- Added `shoporder` key to `CONTACTS` object
- Added `toEmail` prop to `OrderSheet` component (defaults to `CONTACTS.purchasing` — all other order sheets unaffected)
- Passed `toEmail: CONTACTS.shoporder` only on the `shop` view call
- CC field (`pm?.email`) was not touched
- Confirmed live and working on iPad after GitHub Pages cache cleared

**Next session:** Format the email sent from the order sheets.
---

### Session 64
**Date:** 2026-06-24
**Build:** index_300 → index_301

**What we did:**
- Fixed cart icon appearing on non-order-sheet fillable form tiles (Daily Field Report, RFI Form, RFC Form, Commercial Time Card, Site Safety Inspection)
- Root cause: cart icon div was rendered unconditionally on all fillable form tiles in the `currentForms.map()` block
- Fix: wrapped cart icon element with `folder === "ORDER SHEETS" &&` so it only appears when inside the ORDER SHEETS folder

**Next session:** Format the email sent from the order sheets.



---

### Session 65
**Date:** 2026-06-27
**Build:** No code changes this session

**What we did — IIS Migration Attempt:**
- Goal: move app from GitHub Pages to horner.app on company IIS Server 2025
- Installed and configured OpenSSH Server on Windows Server 2025 on custom port 2243
- Created dedicated `horner-deploy` local user, added to OpenSSH Users group
- Generated ED25519 SSH key pair; public key installed in `C:\Users\horner-deploy\.ssh\authorized_keys`
- Configured FortiGate 60F: VIP (HP-APP) mapping 98.103.132.245 → 10.1.1.12, custom service SSH-2243, firewall policy HP-APP SSH
- Debugged extensively: wrong public IP (.242 vs .245), policy disabled, geo-blocking (Sweden/Estonia catching AWS IPs), NAT setting, policy order
- **Root blocker discovered:** Anthropic sandbox network only allows outbound HTTPS (443/80) to whitelisted domains — raw TCP on custom ports is blocked at the sandbox level, not the FortiGate. SSH from Claude is not possible in this environment.

**IIS server state (ready for next steps):**
- OpenSSH running on port 2243 ✅
- FortiGate VIP + policy configured ✅
- `horner-deploy` user + SSH key auth configured ✅
- SSH key private: stored at /tmp/horner_deploy_key (ephemeral — regenerate next session if needed)

**Next session options (Eric to decide):**
1. GitHub webhook/pull — server polls GitHub for changes and pulls index.html automatically (recommended — no new infrastructure, builds on existing GitHub workflow)
2. Other relay approach over HTTPS

**Next session:** Decide on IIS deployment method, then implement. After that: order sheet email formatting.


