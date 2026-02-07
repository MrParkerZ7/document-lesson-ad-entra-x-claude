# Lesson 02: Tenant Setup and Configuration

## Overview

This lesson covers how to set up and configure a Microsoft Entra ID tenant for your organization. A tenant is your organization's dedicated instance of Entra ID.

## Learning Objectives

By the end of this lesson, you will:
- Understand how to create an Entra ID tenant
- Configure tenant-level settings
- Set up custom domains
- Configure branding for your organization

---

## 1. Creating an Entra ID Tenant

### Prerequisites
- Microsoft account or organizational account
- Azure subscription (for some features)
- Global Administrator role (for existing tenant)

### Option 1: Through Azure Portal

1. Navigate to [Azure Portal](https://portal.azure.com)
2. Search for "Microsoft Entra ID"
3. Click "Manage Tenants" → "Create"
4. Choose tenant type:
   - **Microsoft Entra ID** - For employees and internal apps
   - **Microsoft Entra ID (B2C)** - For customer-facing applications

### Option 2: Through Microsoft 365 Admin Center

1. Sign up for Microsoft 365 Business
2. A tenant is automatically created during signup
3. Access via [admin.microsoft.com](https://admin.microsoft.com)

### Tenant Configuration Form

```
Organization Name: [Your Company Name]
Initial Domain: [yourcompany].onmicrosoft.com
Country/Region: [Select your region]
```

> **Note**: The initial `.onmicrosoft.com` domain cannot be changed after creation.

---

## 2. Tenant Properties Configuration

### Accessing Tenant Properties

```
Azure Portal → Microsoft Entra ID → Overview → Properties
```

### Key Settings

| Setting | Description | Recommendation |
|---------|-------------|----------------|
| Name | Display name for your tenant | Company name |
| Country or region | Primary location | Cannot be changed |
| Notification language | Admin notifications | Primary language |
| Tenant ID | Unique identifier (GUID) | Keep for reference |
| Primary domain | Default domain suffix | Custom domain preferred |

### Technical Contact Information

Configure technical contacts for:
- Security notifications
- Service health alerts
- Billing communications

---

## 3. Custom Domain Setup

### Why Use Custom Domains?

- Professional appearance (user@yourcompany.com vs user@yourcompany.onmicrosoft.com)
- Brand consistency
- User-friendly email addresses
- Required for some integrations

### Step-by-Step Domain Setup

#### Step 1: Add the Domain

```
Microsoft Entra ID → Custom domain names → Add custom domain
```

Enter your domain name: `yourcompany.com`

#### Step 2: Verify Domain Ownership

Add a DNS record to prove ownership:

**Option A: TXT Record (Recommended)**
```
Type: TXT
Host: @
Value: MS=ms12345678
TTL: 3600
```

**Option B: MX Record**
```
Type: MX
Host: @
Value: ms12345678.msv1.invalid
Priority: 32767
TTL: 3600
```

#### Step 3: Wait for DNS Propagation

- Typically 15-60 minutes
- Can take up to 72 hours
- Click "Verify" once propagated

#### Step 4: Set as Primary (Optional)

```
Select domain → Make primary
```

### DNS Records Summary

| Purpose | Record Type | Host | Value |
|---------|-------------|------|-------|
| Domain verification | TXT | @ | MS=xxxxxxxx |
| Exchange Online | MX | @ | yourcompany-com.mail.protection.outlook.com |
| Autodiscover | CNAME | autodiscover | autodiscover.outlook.com |
| SPF | TXT | @ | v=spf1 include:spf.protection.outlook.com -all |

---

## 4. Company Branding

### Why Configure Branding?

- Consistent user experience
- Professional appearance
- Reduces phishing susceptibility
- Improves user confidence

### Branding Elements

#### Sign-in Page Customization

```
Microsoft Entra ID → Company branding → Configure
```

| Element | Specifications | Description |
|---------|---------------|-------------|
| Banner logo | 280×60 px, <10 KB | Displayed on sign-in page |
| Square logo | 240×240 px | Used in various places |
| Square logo (dark) | 240×240 px | For dark backgrounds |
| Background image | 1920×1080 px, <300 KB | Sign-in page background |
| Page background color | Hex color code | Fallback if image fails |
| Sign-in page text | Max 1024 characters | Custom message |

### Branding Configuration Example

```json
{
  "bannerLogo": "company-logo-horizontal.png",
  "squareLogo": "company-logo-square.png",
  "backgroundColor": "#1a1a2e",
  "signInPageText": "Welcome to Contoso Corp. Please sign in with your corporate credentials.",
  "usernameHintText": "user@contoso.com"
}
```

### Localized Branding

Configure different branding for different languages:

1. Create default branding (browser language)
2. Add language-specific branding:
   - Select language from dropdown
   - Configure elements for that language

---

## 5. Tenant Settings Configuration

### User Settings

```
Microsoft Entra ID → User settings
```

| Setting | Options | Recommendation |
|---------|---------|----------------|
| App registrations | Users can register apps: Yes/No | No (for security) |
| Administration portal | Restrict access: Yes/No | Yes (admins only) |
| LinkedIn connections | Allow connection: Yes/No | Based on policy |
| Guest user access | Various levels | Most restrictive |

### External Collaboration Settings

```
Microsoft Entra ID → External Identities → External collaboration settings
```

| Setting | Description |
|---------|-------------|
| Guest user access | What guests can see in directory |
| Guest invite settings | Who can invite guests |
| Collaboration restrictions | Domain allow/deny lists |
| Leave settings | Can guests leave organization |

### Recommended Configuration for Organizations

```yaml
Guest User Access:
  - Directory Access: Limited (own profile only)
  - Can see other users: No
  - Can enumerate groups: No

Invitation Settings:
  - Admins and users in guest inviter role can invite: Yes
  - Members can invite: No
  - Guests can invite: No

Domain Restrictions:
  - Allow invitations to specified domains only
  - Allowed domains: [partner1.com, partner2.com]
```

---

## 6. Security Defaults

### What are Security Defaults?

Pre-configured security settings that provide baseline protection:

- Require MFA for all users
- Block legacy authentication
- Protect privileged activities
- Require MFA for Azure management

### Enable/Disable Security Defaults

```
Microsoft Entra ID → Properties → Manage Security defaults
```

> **Note**: Security defaults cannot be used with Conditional Access. Choose one approach.

### When to Use Security Defaults vs Conditional Access

| Scenario | Recommendation |
|----------|----------------|
| Small organization, basic needs | Security Defaults |
| Need custom policies | Conditional Access |
| Compliance requirements | Conditional Access |
| Legacy apps required | Conditional Access |
| P1/P2 license available | Conditional Access |

---

## 7. Administrative Units

### Purpose

Organize users and groups for delegated administration:
- Delegate admin rights to specific departments
- Scope admin access to specific users
- Support organizational structures

### Creating Administrative Units

```
Microsoft Entra ID → Administrative units → New administrative unit
```

### Example Structure

```
Contoso Corporation
├── AU: IT Department
│   ├── Members: IT Users
│   └── Admins: IT Helpdesk (Password reset only)
├── AU: Sales Department
│   ├── Members: Sales Users
│   └── Admins: Sales Managers (Group management)
└── AU: Regional Office - Europe
    ├── Members: EU Users
    └── Admins: EU IT Team (Full user management)
```

### Assigning Roles to Administrative Units

```powershell
# Using Microsoft Graph PowerShell
$adminUnit = Get-MgDirectoryAdministrativeUnit -Filter "displayName eq 'IT Department'"
$user = Get-MgUser -Filter "userPrincipalName eq 'itadmin@contoso.com'"

New-MgDirectoryAdministrativeUnitScopedRoleMember `
    -AdministrativeUnitId $adminUnit.Id `
    -RoleId "helpdesk-administrator-role-id" `
    -RoleMemberInfo @{ Id = $user.Id }
```

---

## 8. Tenant Best Practices

### Naming Conventions

```
Users:      firstname.lastname@domain.com
Groups:     GRP-[Type]-[Name] (e.g., GRP-SEC-IT-Admins)
Apps:       APP-[Environment]-[Name] (e.g., APP-PROD-CRM)
Policies:   POL-[Type]-[Name] (e.g., POL-CA-RequireMFA)
```

### Documentation Requirements

Maintain documentation for:
- [ ] Tenant configuration decisions
- [ ] Custom domain setup details
- [ ] Administrative unit structure
- [ ] Role assignments and justifications
- [ ] Security baseline configuration

### Regular Reviews

| Task | Frequency |
|------|-----------|
| Review tenant settings | Quarterly |
| Audit admin assignments | Monthly |
| Check domain verification | Yearly |
| Update branding | As needed |
| Review security defaults/CA policies | Monthly |

---

## 9. PowerShell Configuration Examples

### Connect to Microsoft Graph

```powershell
# Install module
Install-Module Microsoft.Graph -Scope CurrentUser

# Connect with required scopes
Connect-MgGraph -Scopes "Directory.ReadWrite.All", "Domain.ReadWrite.All"
```

### Get Tenant Information

```powershell
# Get organization details
Get-MgOrganization | Select-Object DisplayName, Id, VerifiedDomains

# Get all domains
Get-MgDomain | Select-Object Id, IsDefault, IsVerified, AuthenticationType
```

### Configure Tenant Settings

```powershell
# Update organization settings
Update-MgOrganization -OrganizationId $tenantId -SecurityComplianceNotificationMails @("security@contoso.com")
```

---

## Summary

Setting up your Entra ID tenant properly is crucial for:
- Establishing a secure foundation
- Maintaining consistent branding
- Enabling proper administration
- Supporting organizational structure

---

## Hands-On Exercise

1. Create or access your test tenant
2. Add a custom domain (use a test domain if available)
3. Configure company branding
4. Set up one administrative unit
5. Review and configure security defaults or prepare for Conditional Access

---

## Next Lesson

[Lesson 03: User and Group Management →](../lesson-03-user-management/README.md)

---

## Additional Resources

- [Add Custom Domain](https://learn.microsoft.com/en-us/entra/fundamentals/add-custom-domain)
- [Configure Company Branding](https://learn.microsoft.com/en-us/entra/fundamentals/how-to-customize-branding)
- [Administrative Units](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/administrative-units)
