# Lesson 02: Tenant Setup and Configuration

## Overview

A tenant is your organization's dedicated instance of Microsoft Entra ID. This lesson covers everything you need to set up and configure your tenant properly, from initial creation to advanced delegation with Administrative Units.

---

## Sub-Lessons

| # | Sub-Lesson | Description |
|---|------------|-------------|
| 2.1 | [Creating a Tenant](./2.1-creating-tenant/README.md) | Tenant creation methods and initial setup |
| 2.2 | [Tenant Properties](./2.2-tenant-properties/README.md) | Configure tenant settings and contacts |
| 2.3 | [Custom Domains](./2.3-custom-domains/README.md) | Add and verify custom domains |
| 2.4 | [Company Branding](./2.4-company-branding/README.md) | Customize sign-in experience |
| 2.5 | [Tenant Settings](./2.5-tenant-settings/README.md) | User, guest, and collaboration settings |
| 2.6 | [Security Defaults](./2.6-security-defaults/README.md) | Baseline security configuration |
| 2.7 | [Administrative Units](./2.7-administrative-units/README.md) | Delegated administration |

---

## Learning Objectives

By the end of this lesson, you will:

- ✅ Create and configure an Entra ID tenant
- ✅ Set up custom domains for professional identities
- ✅ Configure company branding for sign-in pages
- ✅ Apply appropriate tenant security settings
- ✅ Understand Security Defaults vs Conditional Access
- ✅ Implement delegated administration with AUs

---

## Prerequisites

- Microsoft account or work account
- Azure subscription (for some features)
- DNS access for custom domain verification

---

## Quick Reference

### Key URLs

| Resource | URL |
|----------|-----|
| Azure Portal | https://portal.azure.com |
| Entra Admin Center | https://entra.microsoft.com |
| M365 Admin Center | https://admin.microsoft.com |

### Tenant Identifiers

```
Tenant ID:     xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx (GUID)
Initial Domain: yourcompany.onmicrosoft.com
Custom Domain:  yourcompany.com (after verification)
```

---

## Diagrams

Each sub-lesson includes a `.drawio` diagram file for visual learning. Open with:
- [diagrams.net](https://app.diagrams.net/)
- VS Code Draw.io Integration extension
- Desktop Draw.io application

---

## Summary

Proper tenant setup establishes the foundation for:
- Secure identity management
- Professional user experience
- Effective administration
- Organizational scalability

---

## Navigation

| Previous | Next |
|----------|------|
| [Lesson 01: Introduction ←](../lesson-01-introduction/README.md) | [Lesson 03: User & Group Management →](../lesson-03-user-management/README.md) |

---

## Additional Resources

- [Create a Tenant](https://learn.microsoft.com/en-us/entra/fundamentals/create-new-tenant)
- [Add Custom Domain](https://learn.microsoft.com/en-us/entra/fundamentals/add-custom-domain)
- [Configure Company Branding](https://learn.microsoft.com/en-us/entra/fundamentals/how-to-customize-branding)
- [Administrative Units](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/administrative-units)
