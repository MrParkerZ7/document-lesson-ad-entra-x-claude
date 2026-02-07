# 2.1 Creating an Entra ID Tenant

## Overview

A tenant is your organization's dedicated instance of Microsoft Entra ID. This sub-lesson covers how to create and initially configure a new tenant.

## Learning Objectives

- Understand what a tenant is
- Know the prerequisites for tenant creation
- Create a tenant through different methods
- Complete initial configuration

---

## What is a Tenant?

A **tenant** represents your organization in Microsoft Entra ID:

| Aspect | Description |
|--------|-------------|
| **Identity** | Unique instance of Entra ID |
| **Isolation** | Completely separate from other tenants |
| **Domain** | Has a default `.onmicrosoft.com` domain |
| **Identifier** | Unique Tenant ID (GUID) |

---

## Prerequisites

Before creating a tenant:

| Requirement | Details |
|-------------|---------|
| Account | Microsoft account or existing work account |
| Permissions | None needed for new tenant; Global Admin for existing |
| Subscription | Azure subscription (optional, for some features) |

---

## Creation Methods

### Method 1: Azure Portal

```
1. Navigate to portal.azure.com
2. Search for "Microsoft Entra ID"
3. Click "Manage Tenants" → "Create"
4. Select tenant type
5. Complete configuration form
```

### Method 2: Microsoft 365 Admin Center

```
1. Sign up for Microsoft 365 Business
2. Tenant created automatically during signup
3. Access via admin.microsoft.com
```

### Method 3: Azure CLI

```bash
az login
az account tenant create --tenant-name "contoso" --display-name "Contoso Corp"
```

---

## Tenant Types

| Type | Purpose | Use Case |
|------|---------|----------|
| **Microsoft Entra ID** | Workforce identity | Employees, internal apps |
| **Microsoft Entra ID (B2C)** | Customer identity | Consumer-facing apps |

---

## Configuration Form

When creating a tenant, provide:

```yaml
Organization name: Contoso Corporation
Initial domain name: contoso    # Results in contoso.onmicrosoft.com
Country/Region: United States   # Cannot be changed later!
```

> **Important**: The initial `.onmicrosoft.com` domain and country/region cannot be changed after creation.

---

## Post-Creation Steps

After tenant creation:

1. **Note the Tenant ID** - Save for future reference
2. **Verify Global Admin** - Confirm your admin access
3. **Configure basic settings** - Set notification preferences
4. **Plan custom domain** - Prepare DNS records
5. **Review security defaults** - Enabled by default

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of tenant creation.

---

## Key Takeaways

1. A tenant is your organization's dedicated Entra ID instance
2. Choose the right tenant type (Entra ID vs B2C)
3. Country/region and initial domain are permanent
4. Save your Tenant ID for future reference

---

## Next Sub-Lesson

[2.2 Tenant Properties Configuration →](../2.2-tenant-properties/README.md)
