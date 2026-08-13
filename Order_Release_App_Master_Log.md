# Order Release App — Master Log

**Last Updated:** 2026-08-13

---

### Session 2 — Tag & Hold Feature (End-to-End)
**Date:** 2026-08-13

**Summary:** Pivoted from PO-centric workflow to field-crew-centric "Tag & Hold" feature. Built full stack: ASP.NET Core API reading live GE data, standalone HTML page, integrated into Field App via iframe, Power Automate email flow. Plumber flow: Active Projects → Job → Tag & Hold tile → Select Phase → See unreleased items → Add to cart → Submit → Email to Aaron.

**Key Discovery:**
- `Task-no` field on `po-line` table = Phase number (GE displays it as "Phase" in UI)
- `task` table has phase name lookup (e.g., Phase 6 = FINISH PLUMBING, Phase 15 = MECH. ROOM)

**API Endpoints Built (ASP.NET Core 8, `D:\Projects\OrderReleaseApp\api\`):**
- `GET /api/job/{jobNo}/phases` — All phases with unreleased items for a job, with phase names from `task` table
- `GET /api/job/{jobNo}/phase/{phase}/items` — All unreleased PO line items for a job+phase across all open POs
- `GET /api/po/search?q={query}` — PO search by number (original endpoints, still available)
- `GET /api/po/{poNo}` — Full PO detail with line items
- `GET /api/vendor/{vendorCode}` and `GET /api/vendor/search?q={query}` — Vendor lookup
- API runs on `http://0.0.0.0:5050` (HTTP) and `https://0.0.0.0:5051` (HTTPS, self-signed cert)
- CORS configured for horner.app, dev site HTTP/HTTPS

**Tag & Hold Page (`C:\inetpub\wwwroot\horner_app_dev\tagandhold.html`):**
- Standalone HTML page, accepts `?job=XXX&embedded=1` URL params
- Phase cards show phase name (bold), phase number, item count
- Item list shows description, vendor, ordered/received/available quantities
- Cart pattern (same as order sheets) with Add/Remove
- Submit groups items by vendor and POSTs to Power Automate flow
- Embedded mode hides its own header when loaded inside Field App iframe

**Field App Integration (dev build on port 8443):**
- TAG & HOLD tile added to job folder grid
- Opens tagandhold.html in iframe with job code passed via URL
- Breadcrumb: Home > Commercial > Active Projects > JOB > Tag & Hold
- Back button works via formViews array

**Power Automate Flow:**
- Flow 28 — Tag & Hold Release: HTTP trigger → loops vendors → loops items → builds HTML email with vendor-grouped tables → sends to eschieble@pinnacle-tec.com (test)
- Flow URL wired into tagandhold.html

**Dev Environment Setup:**
- IIS site "HornerAppDev" on HP-APP: HTTP port 8080, HTTPS port 8443
- Web root: `C:\inetpub\wwwroot\horner_app_dev\`
- Self-signed cert "Horner App Dev" (thumbprint 4B83FCBC..., 5-year expiry)
- Entra redirect URI added: `https://10.1.1.12:8443`
- Production build (372) copied to dev site, patched with Tag & Hold tile
- Production site untouched

**Production Deployment (same session):**
- ASP.NET Core Hosting Bundle installed on HP-APP
- API published to `C:\inetpub\wwwroot\horner_app\api\` as IIS sub-application under "Horner Field App" site
- App pool "OrderReleaseApi" running as `Horner\sql.readonly`, No Managed Code
- Controller routes changed from `api/[controller]` to `[controller]` (IIS provides `/api` prefix)
- API accessible at `https://horner.app/api/...` (same origin as Field App, trusted Starfield cert)
- tagandhold.html deployed to production web root, API_BASE set to `/api` (relative)
- Field App patched (build 372 → 373): TAG & HOLD tile, iframe view, breadcrumb, formViews
- Flow 28 URL wired into production tagandhold.html

**Open Items:**
1. **Local ledger/shadow inventory** — Track releases locally so quantities update before GE catches up via billing
2. **iPad testing** — Verify touch targets, scrolling, cart UX on actual iPad
3. **submittedBy field** — Add logged-in user name to the release email payload
4. **Flow trigger auth** — Currently "Anyone", consistent with other flows
5. ~~Production deployment~~ — DONE (build 373)
6. ~~API persistent hosting~~ — DONE (IIS sub-application under Horner Field App site)

**Next session:** iPad testing, local ledger design (shadow inventory for release tracking)

---


## Project Overview

**Purpose:** Replace Aaron's manual PO-highlight-and-retype workflow with a web app that pulls PO data directly from Global Edge's SQL backend, lets Aaron select line items and quantities for release, and emails the supplier automatically.

**Current Workflow (pain point):**
1. Field crew opens PO PDF, highlights items/quantities they need delivered
2. Emails highlighted PDF to Aaron (Purchasing)
3. Aaron manually retypes every line item into an email to the supplier
4. Time-consuming, error-prone, scales poorly with large orders

**Target Workflow:**
1. Field crew requests delivery (method TBD — could integrate with Field App later)
2. Aaron opens Order Release App, searches PO by number
3. App pulls live PO line items from Global Edge SQL
4. Aaron enters release quantities per line item
5. Hits "Release" — app emails supplier with formatted order via Aaron's Outlook (Graph API)

---

## Architecture

**Stack:**
- **Database:** Global Edge ERP — SQL Server instance `HP-SQL\GE`, database `Service` (read-only access)
- **Backend:** ASP.NET Core 8 Web API (hosted on HP-APP, IIS, Windows Server 2025)
- **Frontend:** React (PWA-enabled for native app feel)
- **Email:** Microsoft Graph API (sends as Aaron via Entra ID / MSAL auth)
- **Auth:** MSAL / Entra ID (same tenant as Field App: ef466c74-7a13-4920-854c-210669ea3c84)
- **Hosting:** HP-APP (10.1.1.12) — separate IIS site or virtual directory alongside Field App

**Key Decisions:**
- Domain account `Horner\sql.readonly` for GE database access — IIS app pool will run under this identity for production
- Windows Integrated auth for SQL connection (not SQL auth)
- Graph API for email (sends from Aaron's Outlook, lands in his Sent folder, preserves his signature)
- PWA manifest so Aaron can pin to taskbar — full app experience, no browser chrome
- ASP.NET Core + React aligns with Field App's planned backend migration path

---

## Database Schema (Global Edge — HP-SQL\GE / Service)

**857 tables total. Key tables for Order Release:**

### dbo.po (PO Header)
- `Po-no` varchar(24) — PO number (e.g., "1199233")
- `Vendor-code` varchar(12) — FK to vendor table
- `Name` varchar(80) — Vendor name (denormalized)
- `Job-no` varchar(16) — Job number (e.g., "SEL-25102")
- `Status-code` varchar(2) — `p`/`P` = open (20,992), `C` = closed (110,971), `x` = cancelled (2,026), `r` = received (796)
- `Po-date` datetime
- `Ship-name`, `Ship-address__1/2`, `Ship-city`, `Ship-st`, `Ship-zip` — delivery address
- `Buyer-code` varchar(100)
- `Memo__1/2` varchar(4000)

### dbo.[po-line] (PO Line Items)
- `Po-no` varchar(24) — FK to po
- `Line-no` int
- `Item-no` varchar(32)
- `Description__1` varchar(124), `Description__2` varchar(124)
- `Uom-code` varchar(8)
- `Qty-orig-ord` int — quantity originally ordered
- `Qty-received` int — quantity already received
- `Qty-to-rcve` int — quantity still to receive (key field for releases)
- `fob-vendor` numeric — unit price
- `Status-code` varchar(2) — `n` = new/open
- `memo` varchar(8000)

### dbo.vendor
- `Vendor-code` varchar(12)
- `Name` varchar(66)
- `E-Mail__1` through `E-Mail__5` varchar(610) — NOTE: some vendors have blank emails (e.g., Ferguson)
- `Telephone` varchar(28)
- `Contact` varchar(40)

### dbo.job
- `Job-no` varchar(16)
- `Description` varchar(60)
- `Address`, `City`, `St`, `Zip-code`

---

## Development Environment

**Location:** `D:\Projects\OrderReleaseApp\` on HP-APP

**Directory Structure:**
```
D:\Projects\OrderReleaseApp\
├── api\                    ← ASP.NET Core 8 Web API (scaffolded, builds clean)
│   ├── Controllers\
│   ├── Program.cs
│   ├── appsettings.json
│   └── OrderReleaseApi.csproj (Microsoft.Data.SqlClient 7.0.2 installed)
└── mcp-server\             ← Custom MCP server for Claude Desktop (PowerShell + dotnet tools)
    ├── index.js
    └── package.json
```

**Tools on HP-APP:**
- .NET 8 SDK 8.0.423 — installed at `C:\Program Files\dotnet\`
- Node.js — installed at `C:\Program Files\nodejs\`
- IIS — running (Field App on port 443)
- Claude Desktop with MCP connectors:
  - Filesystem MCP (scoped to `C:\inetpub\wwwroot\horner_app\` and `D:\Projects\OrderReleaseApp\`)
  - Custom PowerShell MCP server (`D:\Projects\OrderReleaseApp\mcp-server\`) — provides `powershell` and `dotnet` tools
  - Claude-in-Chrome

**MCP Server Notes:**
- PowerShell tool: bare `dotnet` command doesn't resolve in spawned process despite being in PATH; `dotnet` tool wraps Start-Process with file redirect as workaround
- Config location: `%APPDATA%\Claude\claude_desktop_config.json`
- `mcpServers` block added manually to config (not available through Settings UI)

---

## Key People

- **Eric Schieble** — Developer, IT Admin
- **Aaron** — Purchasing, primary end user
- **Patrick** — Company principal, approvals

---

## Open Questions

1. ~~Verify read-only SQL user exists~~ → `Horner\sql.readonly` is a domain account; Windows Integrated auth works; will configure IIS app pool identity for production
2. ~~GE SQL connection details~~ → `HP-SQL\GE`, database `Service`, Windows auth
3. ~~What tables hold PO data~~ → `po`, `po-line`, `vendor`, `job`, `item` — schema mapped
4. Supplier email addresses — some vendors (e.g., Ferguson) have blank email fields in GE. Need strategy: manual entry? Separate lookup table?
5. Email format preference — what does Aaron's current supplier email look like?
6. Scope of initial release — Aaron only, or other users too?
7. Entra app registration — new registration or extend existing Field App registration?
8. Field crew request integration — future phase tie-in with Field App?
9. `Qty-to-rcve` vs release quantity — does Aaron always release the full remaining qty, or partial? (App should support partial)

---

## Sessions

### Session 1 — 2026-08-06 — Project Kickoff, Environment Setup & DB Discovery

**Decisions:**
- Selected architecture: ASP.NET Core 8 Web API + React frontend + Graph API email
- Hosting on HP-APP alongside Field App (IIS)
- PWA for native app feel
- Auth via existing Entra tenant (MSAL)
- Windows Integrated auth for GE SQL (domain account `Horner\sql.readonly`)

**Infrastructure completed:**
- Created project directory `D:\Projects\OrderReleaseApp\`
- Built custom MCP server (Node.js) providing PowerShell and dotnet CLI tools for Claude Desktop
- Added MCP server to `claude_desktop_config.json`
- Installed .NET 8 SDK 8.0.423 on HP-APP
- Scaffolded ASP.NET Core Web API project (`dotnet new webapi`)
- Installed Microsoft.Data.SqlClient 7.0.2 NuGet package
- Verified SQL connection to `HP-SQL\GE` / `Service` database (Windows Integrated auth)
- Mapped PO-related database schema (po, po-line, vendor, job, item)
- Pulled sample PO data — confirmed real data flowing (e.g., PO 1199233, 16 line items, Ferguson)

**Next steps:**
- Build API endpoints: PO search, PO detail with line items, vendor lookup
- Build React frontend: PO search, line item grid with release quantity input, Release button
- Wire up Graph API for email send
- Configure IIS site for the app

---
