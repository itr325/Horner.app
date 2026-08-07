# Visual Navigator SSO Setup — Master Log

**Project:** Visual Navigator SSO Setup
**Created:** 2026-08-07
**Status:** In Progress — Paused

---

## Purpose

Central log tracking all activity, decisions, milestones, and changes for the Visual Navigator SSO Setup project. Configuring Microsoft Entra SAML SSO for the Singlepoint application.

---

## Log Entries

### 2026-08-07 — Project Initialized
- Created project structure under `MCF/`
- Master log and memory file established
- Files uploaded to GitHub repository

### 2026-08-07 — SSO Configuration Doc Received
- Received Singlepoint SSO configuration PDF
- Protocol: SAML 2.0
- SSO Provider: Microsoft Entra
- Target Application: Singlepoint (Syncronology)
- Confirmed correct hostname: `midcityfoundry.syncronology.com`

### 2026-08-07 — Entra Configuration Completed
- [x] Created Non-gallery app named "SinglePoint" in Entra
- [x] Configured SAML SSO with correct URLs
- [x] Set Identifier (Entity ID): https://midcityfoundry.syncronology.com/sp
- [x] Set Reply URL: https://midcityfoundry.syncronology.com/saml/login
- [x] Set Logout URL: https://midcityfoundry.syncronology.com/login.aspx?mode=logout
- [x] Added Login claim → user.mail
- [x] Added Department claim → user.department
- [x] Downloaded Federation Metadata XML

### 2026-08-07 — Singlepoint Configuration (Partial)
- [x] Set SAML Service Provider Id in Identification settings
- [x] Created Identity Provider named "Microsoft Entra"
- [x] Issuer and Sign-In/Sign-Out URLs populated (manual — metadata upload failed)
- [ ] Certificate NOT uploaded — file format/extension rejected by Singlepoint
- [x] User Fields Mapping configured (Login → Login, Email → Email)
- [x] IDP saved without certificate

### 2026-08-07 — Issues Encountered
1. **Metadata upload failed** — "Wrong metadata file format" error in Singlepoint
2. **Certificate upload failed** — .cer, .crt extensions rejected ("Invalid file extension")
3. **Identifier mismatch** — Singlepoint was sending `horizons.syncronology.com/sp` instead of `midcityfoundry` — resolved by fixing a config elsewhere
4. **SSO login redirects back to login page** — likely caused by missing certificate; Singlepoint can't verify SAML response without it

### 2026-08-07 — Paused
- Blocked on certificate upload to Singlepoint IDP
- Next step: resolve certificate format/upload issue, then retest SSO

---

## Entra Tenant Info

- **Tenant/Directory ID:** da345ce3-34ca-4401-ad5d-0dcbb408aa71
- **Certificate Thumbprint:** 528172EFCAC8E2627C6B35CA5D3C3FA261DC35E1
- **Certificate Expiration:** 8/7/2029
- **Notification Email:** eschieble@midcityfoundry.com

---

## Remaining Steps

- [ ] Resolve certificate upload in Singlepoint IDP
- [ ] Retest SSO login
- [ ] Configure group permissions and claims (if needed)
- [ ] Update web.appsettings.config keys (if needed)
- [ ] Enable auto-user-creation (if needed)
- [ ] Final SSO validation

---

*Append new entries at the bottom of the Log Entries section.*
