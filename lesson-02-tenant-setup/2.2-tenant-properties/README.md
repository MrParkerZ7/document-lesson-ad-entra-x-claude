# 2.2 Tenant Properties Configuration

## Overview

After creating a tenant, you need to configure its properties. This sub-lesson covers the essential tenant-level settings and how to configure them.

## Learning Objectives

- Access and understand tenant properties
- Configure key tenant settings
- Set up technical contacts
- Understand which settings are permanent

---

## Accessing Tenant Properties

```
Azure Portal → Microsoft Entra ID → Overview → Properties
```

Or directly:
```
https://entra.microsoft.com/#view/Microsoft_AAD_IAM/TenantProperties.ReactView
```

---

## Key Properties

| Property | Description | Editable |
|----------|-------------|----------|
| **Name** | Display name for your tenant | ✅ Yes |
| **Tenant ID** | Unique identifier (GUID) | ❌ No |
| **Country/Region** | Primary location | ❌ No |
| **Primary domain** | Default domain suffix | ✅ Yes (after adding custom) |
| **Tenant type** | Entra ID or B2C | ❌ No |

---

## Configurable Settings

### Organization Name

The display name shown in:
- Azure Portal
- Admin centers
- User consent prompts
- Email notifications

```powershell
# Update organization name via PowerShell
Update-MgOrganization -OrganizationId $tenantId -DisplayName "Contoso Corporation"
```

### Technical Contact

Configure contacts for:
- Security notifications
- Service health alerts
- Billing communications
- Compliance notifications

```
Microsoft Entra ID → Properties → Technical contact
```

### Notification Settings

| Setting | Purpose |
|---------|---------|
| Global privacy contact | GDPR/privacy inquiries |
| Privacy statement URL | Link to your privacy policy |
| Technical contact | Technical notifications |

---

## Tenant Identifiers

### Tenant ID (GUID)

```
Format: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
Example: 72f988bf-86f1-41af-91ab-2d7cd011db47
```

Used for:
- API calls
- Application configuration
- Cross-tenant references
- Support requests

### Primary Domain

Initially: `yourcompany.onmicrosoft.com`

After adding custom domain: Can be changed to `yourcompany.com`

---

## PowerShell Configuration

```powershell
# Connect to Microsoft Graph
Connect-MgGraph -Scopes "Organization.ReadWrite.All"

# Get organization details
$org = Get-MgOrganization

# View properties
$org | Select-Object DisplayName, Id, VerifiedDomains,
                     TechnicalNotificationMails,
                     PrivacyProfile

# Update settings
Update-MgOrganization -OrganizationId $org.Id `
    -TechnicalNotificationMails @("it-admin@contoso.com") `
    -PrivacyProfile @{
        ContactEmail = "privacy@contoso.com"
        StatementUrl = "https://contoso.com/privacy"
    }
```

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual overview of tenant properties.

---

## Key Takeaways

1. Tenant ID and Country/Region cannot be changed
2. Organization name is used throughout Microsoft services
3. Configure technical contacts for important notifications
4. Primary domain can be changed after adding custom domains

---

## Next Sub-Lesson

[2.3 Custom Domain Setup →](../2.3-custom-domains/README.md)
