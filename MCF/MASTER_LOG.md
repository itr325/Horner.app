# Visual Navigator SSO Setup — Master Log

**Project:** Visual Navigator SSO Setup
**Created:** 2026-08-07
**Status:** In Progress

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
- Document uses generic `hostname.syncronology.com` — replaced with actual URL

### Configuration URLs (Confirmed)
- **Entity ID:** https://midcityfoundry.syncronology.com/identifier
- **Reply URL (ACS):** https://midcityfoundry.syncronology.com/saml/login
- **Logout URL:** https://midcityfoundry.syncronology.com/login.aspx?mode=logout
- **SAML SP ID:** https://midcityfoundry.syncronology.com/sp

---

## Upcoming

- [ ] Step 1: Register non-gallery app in Microsoft Entra
- [ ] Step 2: Configure SAML SSO with correct URLs
- [ ] Step 3: Set up Attributes & Claims
- [ ] Step 4: Download Federation Metadata XML
- [ ] Step 5: Configure Singlepoint IDP with metadata
- [ ] Step 6: Set up User Fields Mapping
- [ ] Step 7: Configure group permissions and claims
- [ ] Step 8: Update web.appsettings.config keys
- [ ] Step 9: Test SSO login

---

*Append new entries at the bottom of the Log Entries section.*
