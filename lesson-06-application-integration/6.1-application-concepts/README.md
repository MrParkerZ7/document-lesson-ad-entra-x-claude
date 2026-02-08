# 6.1 Application Concepts

## Overview

Understanding the difference between App Registrations and Enterprise Applications is fundamental to working with applications in Microsoft Entra ID. This lesson explains the relationship between these two objects and when to use each.

## Learning Objectives

- Understand what App Registrations and Enterprise Applications are
- Learn the relationship between Application Objects and Service Principals
- Identify which configuration belongs where
- Understand multi-tenant application scenarios

---

## Application Object vs Service Principal

### The Two Sides of an Application

When you register an application in Entra ID, two objects are created:

| Object | Also Known As | Purpose |
|--------|---------------|---------|
| Application Object | App Registration | Global definition (template) |
| Service Principal | Enterprise Application | Tenant-specific instance |

### Key Differences

| Aspect | App Registration | Enterprise Application |
|--------|-----------------|----------------------|
| Scope | Home tenant only | Each tenant that uses the app |
| Count | One per app | One per tenant using the app |
| Created by | Developer/Admin | Automatically on first consent |
| Configures | Identity settings | Access settings |

---

## What Each Object Manages

### App Registration (Application Object)

Configured by developers or administrators in the home tenant:

```yaml
Identity Configuration:
  - Application (Client) ID
  - Directory (Tenant) ID
  - Object ID

Authentication:
  - Redirect URIs
  - Platform configurations
  - Implicit grant settings
  - Public client flows

Credentials:
  - Client secrets
  - Certificates

API Settings:
  - API permissions requested
  - Exposed APIs (scopes)
  - App roles

Branding:
  - Name and logo
  - Publisher information
  - Terms of service
```

### Enterprise Application (Service Principal)

Configured by administrators in each tenant:

```yaml
Access Control:
  - User and group assignments
  - Assignment requirement

Single Sign-On:
  - SSO method (SAML, OIDC, Password)
  - SSO configuration settings

Provisioning:
  - User provisioning to the app
  - Attribute mappings
  - Scoping filters

Properties:
  - Enabled/Disabled status
  - Visible to users
  - Notes

Security:
  - Conditional Access policies
  - Token lifetime
```

---

## Application Registration Types

### Single-Tenant Application

Application exists only in your tenant:

```
Your Tenant
├── App Registration (Application Object)
│   └── Defines app identity and capabilities
│
└── Enterprise Application (Service Principal)
    └── Controls access in your tenant
```

Use case: Internal line-of-business applications

### Multi-Tenant Application

Application can be used by other tenants:

```
Your Tenant (Home Tenant)
├── App Registration (Application Object)
│   └── Global definition
│
└── Enterprise Application (Service Principal)
    └── Access in your tenant

Other Tenant 1
└── Enterprise Application (Service Principal)
    └── Created when users consent
    └── Local access control

Other Tenant 2
└── Enterprise Application (Service Principal)
    └── Created when users consent
    └── Local access control
```

Use case: SaaS applications, partner integrations

---

## How Objects Get Created

### Scenario 1: Register New Application

```
Developer creates App Registration
        │
        ▼
Application Object created in home tenant
        │
        ▼
Service Principal automatically created
in home tenant (Enterprise Application)
```

### Scenario 2: Multi-Tenant App First Access

```
User in external tenant accesses app
        │
        ▼
User prompted for consent
        │
        ▼
Service Principal created in their tenant
(Enterprise Application)
        │
        ▼
App Registration remains in home tenant only
```

### Scenario 3: Gallery Application

```
Admin adds app from gallery
        │
        ▼
Service Principal created from Microsoft's
App Registration (managed by Microsoft)
        │
        ▼
No App Registration in your tenant
```

---

## Finding Your Applications

### App Registrations

```
Microsoft Entra ID → App registrations
```

View:
- **Owned applications**: Apps you created
- **All applications**: All apps in your tenant

### Enterprise Applications

```
Microsoft Entra ID → Enterprise applications
```

Filters:
- **Enterprise Applications**: Your business apps
- **Microsoft Applications**: O365, Azure services
- **Managed Identities**: System/user-assigned identities
- **All applications**: Everything

---

## Identifiers

Each application has multiple identifiers:

| Identifier | Description | Example |
|------------|-------------|---------|
| Application (Client) ID | Unique app identifier | `a1b2c3d4-e5f6-...` |
| Object ID | Internal Azure resource ID | `f1e2d3c4-b5a6-...` |
| Directory (Tenant) ID | Your tenant identifier | `t1e2n3a4-n5t6-...` |
| Service Principal Object ID | Enterprise app's internal ID | Different from app's Object ID |

**Important**: The Object ID of the App Registration differs from the Object ID of the Enterprise Application (Service Principal).

---

## Practical Example

### Scenario: Building a CRM Application

**Step 1: Create App Registration**
```yaml
Name: Contoso CRM
Account Type: Single tenant
Redirect URI: https://crm.contoso.com/auth

Result:
  Application ID: abc123...
  Object ID: def456...
```

**Step 2: App Registration Automatically Creates Enterprise App**
```yaml
Enterprise Application created:
  Name: Contoso CRM
  Object ID: ghi789... (different!)
  Assignment required: No (default)
```

**Step 3: Configure Each Object**

In App Registration:
- Add redirect URIs
- Configure permissions (User.Read, etc.)
- Create client secret

In Enterprise Application:
- Enable assignment requirement
- Add users/groups
- Apply Conditional Access

---

## PowerShell Examples

### Get App Registration

```powershell
Connect-MgGraph -Scopes "Application.Read.All"

# By display name
Get-MgApplication -Filter "displayName eq 'Contoso CRM'"

# By client ID
Get-MgApplication -Filter "appId eq 'abc123-...'"
```

### Get Enterprise Application (Service Principal)

```powershell
Connect-MgGraph -Scopes "Application.Read.All"

# By display name
Get-MgServicePrincipal -Filter "displayName eq 'Contoso CRM'"

# By app ID
Get-MgServicePrincipal -Filter "appId eq 'abc123-...'"
```

### List All Applications

```powershell
# App Registrations
Get-MgApplication -All | Select-Object DisplayName, AppId

# Enterprise Applications
Get-MgServicePrincipal -All | Select-Object DisplayName, AppId
```

---

## Common Confusion Points

### Q: Why are there two objects?

**A**: Separation of concerns:
- App Registration: "What can this app do?" (developer)
- Enterprise Application: "Who can use this app?" (admin)

### Q: Which one do I configure for SSO?

**A**: Enterprise Application. SSO settings are tenant-specific.

### Q: Where do I add API permissions?

**A**: App Registration. Permissions are part of the app's identity.

### Q: Why can't I find my gallery app in App Registrations?

**A**: Gallery apps use Microsoft's App Registration. You only get a Service Principal (Enterprise Application).

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- App Registration vs Enterprise Application
- Multi-tenant application flow
- Object relationship

---

## Key Takeaways

- App Registration defines the application's identity and capabilities
- Enterprise Application controls access within a specific tenant
- Both objects are created when you register an app
- Multi-tenant apps create Service Principals in each tenant
- Gallery apps only create Enterprise Applications (no local App Registration)
- Each object has a different Object ID

---

## Navigation

[Lesson 06 Overview](../README.md) | [6.2 Registering Applications →](../6.2-registering-applications/README.md)
