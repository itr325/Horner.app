# Visual Navigator SSO Setup — Memory File

**Project:** Visual Navigator SSO Setup
**Last Updated:** 2026-08-07

---

## Project Identity

- **Name:** Visual Navigator SSO Setup
- **Repository Folder:** `MCF/`
- **Initialized:** 2026-08-07

---

## Architecture & Stack

- **SSO Provider:** Microsoft Entra (Azure AD)
- **Target Application:** Singlepoint (by Syncronology)
- **Protocol:** SAML 2.0
- **Hostname:** midcityfoundry.syncronology.com

---

## Confirmed SAML URLs

| Field | Value |
|-------|-------|
| Entity ID (Identifier) | https://midcityfoundry.syncronology.com/identifier |
| Reply URL (ACS) | https://midcityfoundry.syncronology.com/saml/login |
| Logout URL | https://midcityfoundry.syncronology.com/login.aspx?mode=logout |
| SAML SP ID | https://midcityfoundry.syncronology.com/sp |

---

## Key Decisions

| Date       | Decision | Rationale |
|------------|----------|-----------|
| 2026-08-07 | Project created with MCF folder structure | Central hub for logs and memory |
| 2026-08-07 | Confirmed hostname: midcityfoundry.syncronology.com | Replaces generic `hostname` in Singlepoint doc |
| 2026-08-07 | Entra application name: SinglePoint | User confirmed |

---

## Entra Side — Configuration Checklist

1. Log into Microsoft Entra admin center
2. Navigate to Enterprise applications
3. Create new Non-gallery application
4. Navigate to Single sign-on → select SAML
5. Edit Basic SAML Configuration with confirmed URLs
6. Save configuration
7. Configure Attributes & Claims (Login → user.mail, Department → user.department)
8. Download Federation Metadata XML
9. Assign users/groups to the application
10. Add group-scoped claims (Allowed / Invisible / Disallowed)

## Singlepoint Side — Configuration Checklist

1. Set SAML SP ID under Administration → Security → Identification
2. Create new Identity Provider under Single Sign-On → Identity Providers
3. Upload Federation Metadata XML to IDP
4. Configure User Fields Mapping (Login → NameIdentifier, Email → Email)
5. Optionally enable auto-create users and map additional fields
6. Map module permissions under Permissions tab
7. Configure web.appsettings.config keys (SSODefaultDepartmentId, ssoDefaultCompanyId, etc.)
8. Optionally configure Entra group synchronization
9. Test SSO login from Singlepoint login page

---

## web.appsettings.config Keys (Template)

```xml
<add key="SSODefaultDepartmentId" value="TBD" />
<add key="SSOUserCreatedSystemProcess" value="TBD" />
<add key="ssoDefaultCompanyId" value="TBD" />
<add key="SSOUpdateClaimsOnAuthorize" value="false" />
<add key="SSOUserAuthorizedSystemProcess" value="TBD" />
<add key="SSOSynchronizeEntraGroups" value="true" />
<add key="SSOEntraGroupClaim" value="SPGroups" />
<add key="SSOEntraGroupMapping" value="TBD" />
```

---

## Open Questions

1. ~~What is the application name to use in Entra?~~ → **SinglePoint**
2. What DepartmentId and CompanyId values for web.appsettings.config?
3. Which Entra groups need to be mapped to Singlepoint groups?
4. Should auto-user-creation be enabled?
5. What system process GUIDs to use for SSOUserCreatedSystemProcess / SSOUserAuthorizedSystemProcess?

---

## Reference Links

- Source doc: Configuring_Single_Sign-On__SSO__using_Microsoft_Entra.pdf

---

*Update this file whenever project context changes.*
