# 6.9 Gallery Apps & Application Proxy

## Overview

The Microsoft Entra ID application gallery provides pre-integrated applications with simplified SSO setup. Application Proxy enables secure remote access to on-premises applications without VPN. This lesson covers both features.

## Learning Objectives

- Browse and add applications from the gallery
- Configure SAML SSO for gallery applications
- Understand Application Proxy architecture
- Set up Application Proxy connectors
- Publish on-premises applications

---

## Application Gallery

### What is the Gallery?

The Microsoft Entra ID application gallery contains thousands of pre-integrated applications with:

- Pre-configured SSO settings
- Step-by-step tutorials
- Automatic updates
- Microsoft and vendor support

### Accessing the Gallery

```
Microsoft Entra ID → Enterprise applications
→ New application → Browse Azure AD Gallery
```

### Popular Gallery Applications

| Application | SSO Type | Features |
|-------------|----------|----------|
| Salesforce | SAML | SSO, Provisioning |
| ServiceNow | SAML | SSO, Provisioning |
| Workday | SAML | SSO, HR Provisioning |
| Zoom | SAML/OIDC | SSO |
| Slack | SAML | SSO, Provisioning |
| AWS | SAML | SSO, Multi-role |
| Google Workspace | SAML | SSO |
| Dropbox | SAML | SSO, Provisioning |
| Box | SAML | SSO, Provisioning |
| DocuSign | SAML | SSO |

---

## Adding a Gallery Application

### Step-by-Step Process

```yaml
1. Search and Select:
   - Browse or search for application
   - Click on the application tile
   - Review application details

2. Create:
   - Provide display name
   - Click Create

3. Configure SSO:
   - Go to Single sign-on blade
   - Select SAML or OIDC
   - Configure as per tutorial

4. Assign Users:
   - Add users/groups
   - Configure roles if available

5. Test:
   - Use Test button in SSO config
   - Verify sign-in works
```

### Example: Adding Salesforce

```yaml
Step 1: Search "Salesforce" in gallery → Create

Step 2: Single sign-on → SAML

Step 3: Basic SAML Configuration:
  Identifier: https://yourdomain.my.salesforce.com
  Reply URL: https://yourdomain.my.salesforce.com?so=...
  Sign on URL: https://yourdomain.my.salesforce.com

Step 4: User Attributes (default usually works):
  emailaddress → user.userprincipalname
  unique_name → user.userprincipalname

Step 5: Download Certificate (Base64)
  Upload to Salesforce SSO settings

Step 6: Copy Login URL, Logout URL
  Configure in Salesforce

Step 7: Test SSO from Entra ID
```

---

## SAML Configuration

### Basic SAML Configuration

| Field | Description | Source |
|-------|-------------|--------|
| Identifier (Entity ID) | Unique app identifier | From app vendor |
| Reply URL (ACS) | Where SAML response is sent | From app vendor |
| Sign on URL | App login page | From app vendor |
| Relay State | Post-login redirect (optional) | From app vendor |
| Logout URL | Single logout endpoint | From app vendor |

### User Attributes & Claims

Standard claims:
```yaml
Required Claim (Name ID):
  Source: user.userprincipalname
  Format: Email Address

Common Additional Claims:
  emailaddress: user.mail
  givenname: user.givenname
  surname: user.surname
  displayname: user.displayname
  groups: user.groups [All]
```

### Adding Custom Claims

```
Single sign-on → User Attributes & Claims → Add new claim
```

```yaml
Name: department
Namespace: (leave blank for standard)
Source: Attribute
Source attribute: user.department
```

### SAML Signing Certificate

```yaml
Options:
  - Sign SAML response
  - Sign SAML assertion
  - Sign SAML response and assertion

Download formats:
  - Federation Metadata XML (recommended)
  - Certificate (Base64)
  - Certificate (Raw)

Expiration:
  - Certificates auto-generated
  - Monitor for expiration
  - Rotate before expiry
```

---

## Application Proxy

### What is Application Proxy?

Application Proxy enables secure remote access to on-premises web applications:

- No VPN required
- Works through standard HTTPS
- Supports SSO with on-premises apps
- Integrates with Conditional Access

### Architecture

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   External   │────▶│   Entra ID   │────▶│  Connector   │
│    User      │     │   Cloud      │     │  (On-prem)   │
└──────────────┘     └──────────────┘     └──────┬───────┘
                                                  │
                            Internal network      │
                     ─────────────────────────────┼─────
                                                  │
                                          ┌───────▼───────┐
                                          │  Internal App │
                                          │  (On-prem)    │
                                          └───────────────┘
```

### Key Benefits

```yaml
Security:
  - No inbound connections through firewall
  - All traffic through HTTPS
  - Pre-authentication with Entra ID
  - Conditional Access integration
  - DDoS protection

Simplicity:
  - No VPN infrastructure
  - Works with any browser
  - Mobile device support
  - No client software needed
```

---

## Setting Up Application Proxy

### Prerequisites

```yaml
Licensing:
  - Microsoft Entra ID P1 or P2
  - Or Microsoft 365 Business Premium

Infrastructure:
  - Windows Server for connector
  - Outbound HTTPS (443) to Azure
  - .NET Framework 4.7.1+
```

### Step 1: Install Connector

```
Microsoft Entra ID → Application proxy → Download connector service
```

Connector requirements:
```yaml
Server:
  - Windows Server 2016+
  - .NET Framework 4.7.1+
  - TLS 1.2 enabled

Network:
  - Outbound 443 to *.msappproxy.net
  - No inbound ports required
  - Can't use SSL inspection

Best Practices:
  - Install 2+ connectors for HA
  - Use connector groups
  - Place near internal apps
```

### Step 2: Register Connector

```yaml
1. Run installer on Windows Server
2. Sign in with Global Admin
3. Connector registers automatically
4. Verify in Entra portal
```

### Step 3: Create Connector Group (Optional)

```
Application proxy → Connector groups → Add
```

Use for:
- Geographic distribution
- Application isolation
- Different networks

---

## Publishing an Application

### Adding App Proxy Application

```
Enterprise applications → New application
→ Create your own application
→ Configure Application Proxy
```

### Configuration

```yaml
Basic:
  Name: Internal HR Portal
  Internal URL: https://hr.internal.contoso.com
  External URL: https://hr-contoso.msappproxy.net

  Or custom domain:
  External URL: https://hr.contoso.com

Pre-Authentication:
  - Microsoft Entra ID (recommended)
  - Passthrough (no Entra auth)

Connector Group: Default or custom

Additional Settings:
  - Backend SSL certificate validation
  - Translate URLs in headers
  - Translate URLs in body
```

### Custom Domains

```yaml
Default: appname-tenant.msappproxy.net

Custom Domain Setup:
  1. Add custom domain to Entra ID
  2. Verify DNS ownership
  3. Upload SSL certificate
  4. Configure in app proxy settings
  5. Update DNS CNAME record
```

---

## SSO with Application Proxy

### SSO Methods for On-Premises Apps

| Method | Description | Use Case |
|--------|-------------|----------|
| Integrated Windows (IWA) | Kerberos delegation | Windows-authenticated apps |
| Header-based | Pass user info in headers | Legacy apps |
| Password vaulting | Store and replay credentials | Apps without SSO |
| SAML | SAML tokens to on-prem | SAML-capable apps |
| OIDC | Modern tokens | Modern on-prem apps |

### Kerberos Constrained Delegation (KCD)

For Windows Integrated Authentication:

```yaml
Prerequisites:
  - Connector server domain-joined
  - App pool using domain account
  - SPNs configured correctly

Configuration:
  1. Create service account for app
  2. Register SPNs on service account
  3. Configure connector for delegation
  4. Enable IWA in app proxy settings

SPN Example:
  setspn -S http/webapp.contoso.com CONTOSO\webapp-svc
```

### Header-Based SSO

```yaml
Pass Entra ID user info to app:

Headers passed:
  - X-MS-PROXY-SSOEXT-USERNAME: user@contoso.com
  - X-MS-PROXY-SSOEXT-UPN: user@contoso.com
  - X-MS-PROXY-SSOEXT-DISPLAYNAME: John Doe

Custom headers possible with claims mapping
```

---

## PowerShell Management

### List App Proxy Applications

```powershell
Connect-MgGraph -Scopes "Application.Read.All"

# Get apps with proxy configured
Get-MgApplication -All | Where-Object {
    $_.OnPremisesPublishing -ne $null
} | Select-Object DisplayName,
    @{N='InternalUrl';E={$_.OnPremisesPublishing.InternalUrl}},
    @{N='ExternalUrl';E={$_.OnPremisesPublishing.ExternalUrl}}
```

### List Connectors

```powershell
Get-MgOnPremisesPublishingProfileConnector -OnPremisesPublishingProfileId "applicationProxy"
```

### List Connector Groups

```powershell
Get-MgOnPremisesPublishingProfileConnectorGroup -OnPremisesPublishingProfileId "applicationProxy"
```

---

## Best Practices

### Gallery Apps

```yaml
Do:
  - Use gallery apps when available
  - Follow vendor tutorials
  - Test SSO before rollout
  - Configure provisioning if supported
  - Monitor certificate expiration

Don't:
  - Skip testing phase
  - Ignore attribute mapping
  - Forget to assign users
```

### Application Proxy

```yaml
Do:
  - Deploy 2+ connectors for HA
  - Use connector groups for isolation
  - Enable Conditional Access
  - Use custom domains when possible
  - Monitor connector health

Don't:
  - Use single connector in production
  - Skip SSL certificate validation
  - Publish sensitive apps without CA
  - Ignore connector updates
```

---

## Troubleshooting

### Gallery App SSO Issues

| Issue | Check |
|-------|-------|
| SSO fails | Verify Entity ID matches |
| User not found | Check Name ID attribute |
| Claims missing | Verify claim mappings |
| Certificate error | Check certificate in app |

### Application Proxy Issues

| Issue | Check |
|-------|-------|
| Page not loading | Connector status and health |
| Authentication fails | Pre-authentication settings |
| Slow performance | Connector location, network |
| 500 errors | Backend app health |

### Connector Troubleshooting

```yaml
Check connector health:
  Application proxy → Connectors → Status

Common issues:
  - Connector offline: Check service running
  - Network blocked: Verify outbound 443
  - Certificate issues: Re-register connector
```

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- Gallery app integration flow
- Application Proxy architecture
- Connector deployment

---

## Key Takeaways

- Use gallery apps for simplified SSO setup
- Application Proxy provides VPN-less access to on-premises apps
- Deploy multiple connectors for high availability
- Combine with Conditional Access for security
- Use custom domains for professional external URLs
- Monitor connector health regularly

---

## Navigation

← [6.8 Enterprise Applications](../6.8-enterprise-applications/README.md) | [Lesson 07: SSO Configuration →](../../lesson-07-sso-configuration/README.md)
