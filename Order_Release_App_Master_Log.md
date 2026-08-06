# Order Release App — Master Log

**Last Updated:** 2026-08-06

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
- **Database:** Global Edge ERP SQL Server (read-only access)
- **Backend:** ASP.NET Core Web API (hosted on HP-APP, IIS, Windows Server 2025)
- **Frontend:** React (PWA-enabled for native app feel)
- **Email:** Microsoft Graph API (sends as Aaron via Entra ID / MSAL auth)
- **Auth:** MSAL / Entra ID (same tenant as Field App: ef466c74-7a13-4920-854c-210669ea3c84)
- **Hosting:** HP-APP (10.1.1.12) — separate IIS site or virtual directory alongside Field App

**Key Decisions:**
- Read-only SQL user for GE database (Eric believes already created — needs verification)
- Graph API for email (sends from Aaron's Outlook, lands in his Sent folder, preserves his signature)
- PWA manifest so Aaron can pin to taskbar — full app experience, no browser chrome
- ASP.NET Core + React aligns with Field App's planned backend migration path

---

## Key People

- **Eric Schieble** — Developer, IT Admin
- **Aaron** — Purchasing, primary end user
- **Patrick** — Company principal, approvals

---

## Open Questions

1. Verify read-only SQL user exists and has access to PO tables in GE
2. GE SQL Server connection details (server name, database name, port)
3. What tables/views hold PO header and line item data in GE?
4. Supplier email addresses — stored in GE or maintained separately?
5. Email format preference — what does Aaron's current supplier email look like?
6. Scope of initial release — Aaron only, or other users too?
7. Entra app registration — new registration or extend existing Field App registration?
8. Field crew request integration — future phase tie-in with Field App?

---

## Sessions

### Session 1 — 2026-08-06 — Project Kickoff & Architecture

- Defined the problem: Aaron manually retyping PO line items into supplier emails
- Evaluated approaches: Excel/VBA, Access, VB.NET, Power Apps, ASP.NET Core + React
- Selected architecture: ASP.NET Core Web API + React frontend + Graph API email
- Hosting on HP-APP alongside Field App (IIS)
- PWA for native app feel
- Auth via existing Entra tenant (MSAL)
- Created master log on GitHub

---
