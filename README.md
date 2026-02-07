# Microsoft Entra ID - Complete Lesson Guide

A comprehensive guide for implementing Microsoft Entra ID (formerly Azure Active Directory) in organizations. This course covers everything from basic concepts to advanced enterprise scenarios.

## Course Overview

This course is designed for **all skill levels** - from beginners learning identity management fundamentals to advanced administrators implementing complex enterprise scenarios.

**Focus**: Practical implementation of Microsoft Entra ID in organizational environments.

---

## Lessons

| # | Lesson | Description |
|---|--------|-------------|
| 01 | [Introduction](./lesson-01-introduction/README.md) | What is Entra ID, key concepts, editions, architecture |
| 02 | [Tenant Setup](./lesson-02-tenant-setup/README.md) | Creating tenant, custom domains, branding, configuration |
| 03 | [User & Group Management](./lesson-03-user-management/README.md) | Users, groups, dynamic groups, licensing |
| 04 | [Authentication](./lesson-04-authentication/README.md) | MFA, passwordless, SSPR, authentication methods |
| 05 | [Conditional Access](./lesson-05-conditional-access/README.md) | Zero Trust policies, conditions, controls |
| 06 | [Application Integration](./lesson-06-application-integration/README.md) | App registration, OAuth/OIDC, permissions |
| 07 | [SSO Configuration](./lesson-07-sso-configuration/README.md) | SAML, OIDC, password-based SSO |
| 08 | [Security & Governance](./lesson-08-security-governance/README.md) | Identity Protection, PIM, Access Reviews |
| 09 | [Hybrid Identity](./lesson-09-hybrid-identity/README.md) | Azure AD Connect, PHS, PTA, federation |
| 10 | [Monitoring & Troubleshooting](./lesson-10-monitoring-troubleshooting/README.md) | Logs, alerts, diagnostics, common issues |

---

## Learning Path

### Beginner Track
Start with lessons 1-3 to understand core concepts:
1. Introduction - Understand what Entra ID is
2. Tenant Setup - Learn tenant configuration
3. User & Group Management - Manage identities

### Intermediate Track
Progress through lessons 4-7 for practical skills:
4. Authentication - Configure MFA and authentication
5. Conditional Access - Implement access policies
6. Application Integration - Integrate applications
7. SSO Configuration - Set up single sign-on

### Advanced Track
Complete lessons 8-10 for enterprise scenarios:
8. Security & Governance - Advanced security features
9. Hybrid Identity - Connect on-premises AD
10. Monitoring & Troubleshooting - Maintain environment

---

## Prerequisites

- Basic understanding of identity concepts
- Access to Azure/Microsoft 365 tenant (for hands-on exercises)
- Administrative access for configuration tasks

### For Hands-On Labs

- Microsoft 365 Developer tenant (free) or
- Azure subscription with Entra ID Premium trial
- PowerShell 7+ with Microsoft.Graph module
- Modern web browser

---

## Key Topics Covered

### Identity Management
- User lifecycle management
- Group management and automation
- Dynamic group membership
- License management

### Authentication & Security
- Multi-Factor Authentication (MFA)
- Passwordless authentication
- Self-Service Password Reset
- Conditional Access policies
- Identity Protection

### Application Access
- Application registration
- OAuth 2.0 / OpenID Connect
- SAML-based SSO
- Enterprise applications
- API permissions

### Governance
- Privileged Identity Management (PIM)
- Access Reviews
- Entitlement Management
- Audit and compliance

### Hybrid Scenarios
- Azure AD Connect
- Password Hash Sync
- Pass-through Authentication
- Federation (AD FS)

---

## Quick Reference

### Important URLs

| Resource | URL |
|----------|-----|
| Entra Admin Center | https://entra.microsoft.com |
| Azure Portal | https://portal.azure.com |
| My Apps | https://myapps.microsoft.com |
| My Access | https://myaccess.microsoft.com |
| Security Info | https://aka.ms/mysecurityinfo |

### PowerShell Quick Start

```powershell
# Install Microsoft Graph module
Install-Module Microsoft.Graph -Scope CurrentUser

# Connect to Microsoft Graph
Connect-MgGraph -Scopes "User.Read.All", "Group.Read.All"

# Get current user
Get-MgContext
```

---

## Additional Resources

- [Microsoft Entra Documentation](https://learn.microsoft.com/en-us/entra/)
- [Microsoft Learn - Entra ID](https://learn.microsoft.com/en-us/training/browse/?products=entra)
- [Microsoft Graph API](https://learn.microsoft.com/en-us/graph/)
- [Entra ID Pricing](https://www.microsoft.com/en-us/security/business/microsoft-entra-pricing)

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2024 | Initial release |

---

## License

This educational content is provided for learning purposes.

---

*Created with assistance from Claude AI*
