# 7.2 SAML-Based SSO

## Overview

Security Assertion Markup Language (SAML) is an XML-based standard for exchanging authentication data between an identity provider and a service provider. This lesson covers SAML concepts, configuration, and common implementation scenarios.

## Learning Objectives

- Understand SAML terminology and concepts
- Configure SAML-based SSO in Entra ID
- Set up user attributes and claims
- Manage SAML signing certificates
- Troubleshoot common SAML issues

---

## SAML Concepts

### Key Terminology

| Term | Description |
|------|-------------|
| Identity Provider (IdP) | Authenticates users (Entra ID) |
| Service Provider (SP) | Application requesting authentication |
| Assertion | XML document containing user claims |
| Entity ID | Unique identifier for IdP or SP |
| ACS URL | Assertion Consumer Service URL |
| Metadata | XML file with IdP/SP configuration |

### SAML Roles

```yaml
Identity Provider (Entra ID):
  - Authenticates users
  - Issues SAML assertions
  - Maintains user directory
  - Enforces authentication policies

Service Provider (Application):
  - Requests authentication
  - Trusts IdP assertions
  - Grants access based on claims
  - Manages local user sessions
```

---

## SAML Authentication Flow

### SP-Initiated SSO (Most Common)

```
1. User accesses application (SP)
          ↓
2. SP generates SAML Request
          ↓
3. User redirected to IdP with SAML Request
          ↓
4. IdP authenticates user (if no session)
          ↓
5. IdP generates SAML Response with assertion
          ↓
6. User redirected back to SP's ACS URL
          ↓
7. SP validates assertion
          ↓
8. SP grants access
```

### IdP-Initiated SSO

```
1. User accesses IdP (My Apps portal)
          ↓
2. User clicks on application tile
          ↓
3. IdP generates SAML Response
          ↓
4. User redirected to SP's ACS URL
          ↓
5. SP validates assertion
          ↓
6. SP grants access
```

---

## Configuring SAML SSO

### Portal Navigation

```
Microsoft Entra ID → Enterprise applications → [App]
→ Single sign-on → SAML
```

### Basic SAML Configuration

| Field | Description | Example |
|-------|-------------|---------|
| Identifier (Entity ID) | Unique app identifier | `https://app.contoso.com` |
| Reply URL (ACS URL) | Where assertion is sent | `https://app.contoso.com/sso/saml` |
| Sign on URL | App login page (optional) | `https://app.contoso.com/login` |
| Relay State | Post-login redirect (optional) | `/dashboard` |
| Logout URL | Single logout endpoint | `https://app.contoso.com/logout` |

### Configuration Example

```yaml
Basic SAML Configuration:
  Identifier (Entity ID): https://salesforce.com
  Reply URL: https://contoso.my.salesforce.com?so=00D...
  Sign on URL: https://contoso.my.salesforce.com
  Logout URL: https://contoso.my.salesforce.com/logout

Note: Values provided by the application vendor
```

---

## User Attributes & Claims

### Default Claims

The following claims are typically included:

| Claim | URI | Source |
|-------|-----|--------|
| Name Identifier | `nameidentifier` | user.userprincipalname |
| Email | `emailaddress` | user.mail |
| Given Name | `givenname` | user.givenname |
| Surname | `surname` | user.surname |
| Display Name | `name` | user.displayname |

### Name Identifier (NameID)

The NameID uniquely identifies the user to the SP:

```yaml
Configuration:
  Name identifier format: Email address
  Name identifier value: user.userprincipalname

Formats available:
  - Email address: user@contoso.com
  - Persistent: Random GUID per user
  - Transient: Random per session
  - Unspecified: Application decides
```

### Adding Custom Claims

```
User Attributes & Claims → Add new claim
```

```yaml
Name: department
Namespace: (leave blank for simple name)
Source: Attribute
Source attribute: user.department

Name: employeeId
Source: Attribute
Source attribute: user.employeeid

Name: groups
Source: Attribute
Source attribute: user.groups [All groups]
```

### Claim Transformations

```yaml
Transform claim value:
  - ToLower: Convert to lowercase
  - ToUpper: Convert to uppercase
  - RegexReplace: Pattern replacement
  - Substring: Extract part of value

Example - Extract domain:
  Source: user.userprincipalname
  Transform: RegexReplace
  Pattern: .*@(.*)
  Replacement: $1
  Result: contoso.com
```

---

## SAML Signing Certificate

### Certificate Purpose

The signing certificate cryptographically signs SAML responses to prove they come from the legitimate IdP.

### Certificate Options

| Option | Description |
|--------|-------------|
| Sign SAML response | Signs the outer response |
| Sign SAML assertion | Signs the inner assertion |
| Sign both | Most secure, signs both |

### Certificate Lifecycle

```yaml
Creation:
  - Auto-generated on SAML setup
  - 3-year validity by default

Download formats:
  - Certificate (Base64): Most common
  - Certificate (Raw): Binary format
  - Federation Metadata XML: Auto-config

Rotation:
  - Create new before expiry
  - Update in application
  - Set as active
  - Delete old certificate
```

### Downloading Certificate

```
Single sign-on → SAML Signing Certificate
→ Download Certificate (Base64)
```

### Certificate Expiry Notification

```
SAML Signing Certificate → Notification Email Addresses
→ Add email addresses for alerts
```

### PowerShell: Check Certificate Expiry

```powershell
Connect-MgGraph -Scopes "Application.Read.All"

$sp = Get-MgServicePrincipal -Filter "displayName eq 'Salesforce'"

$sp.KeyCredentials | ForEach-Object {
    [PSCustomObject]@{
        Type = $_.Type
        EndDateTime = $_.EndDateTime
        DaysRemaining = ($_.EndDateTime - (Get-Date)).Days
    }
}
```

---

## Federation Metadata

### What is Metadata?

An XML document containing IdP configuration that SPs can import automatically.

### Entra ID Metadata URL

```
https://login.microsoftonline.com/{tenant-id}/federationmetadata/2007-06/federationmetadata.xml
```

Or use the App Federation Metadata URL from the SSO configuration page.

### Metadata Contents

```xml
<EntityDescriptor entityID="https://sts.windows.net/{tenant-id}/">
  <IDPSSODescriptor>
    <KeyDescriptor use="signing">
      <!-- Signing certificate -->
    </KeyDescriptor>
    <SingleLogoutService Location="..." />
    <SingleSignOnService Location="..." />
  </IDPSSODescriptor>
</EntityDescriptor>
```

---

## Common SAML Configurations

### Salesforce

```yaml
Identifier: https://saml.salesforce.com
Reply URL: https://yourcompany.my.salesforce.com?so=00D...
Sign on URL: https://yourcompany.my.salesforce.com

Claims:
  - Federation ID: user.mail
  - User.ProfileId: (custom mapping)
```

### ServiceNow

```yaml
Identifier: https://yourcompany.service-now.com
Reply URL: https://yourcompany.service-now.com/navpage.do
Sign on URL: https://yourcompany.service-now.com

Claims:
  - email: user.mail
  - name: user.displayname
```

### AWS Console

```yaml
Identifier: urn:amazon:webservices
Reply URL: https://signin.aws.amazon.com/saml

Claims:
  - Role: Custom (AWS role ARN)
  - RoleSessionName: user.userprincipalname
  - SessionDuration: 3600
```

---

## Testing SAML SSO

### Built-in Test

```
Single sign-on → Test this application
```

Options:
- Sign in as current user
- Sign in as someone else

### Using SAML Tracer

Install browser extension (SAML-tracer for Firefox/Chrome):

1. Enable SAML-tracer
2. Initiate SSO flow
3. Capture SAML Request/Response
4. Analyze assertions and claims

### Validating SAML Response

Check for:
- Valid signature
- Correct Issuer (IdP Entity ID)
- Correct Audience (SP Entity ID)
- Valid timestamps (NotBefore, NotOnOrAfter)
- Expected claims present

---

## Troubleshooting

### Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| Invalid signature | Certificate mismatch | Re-upload certificate to SP |
| Invalid audience | Wrong Entity ID | Verify Identifier setting |
| User not found | No matching user in SP | Configure provisioning or NameID |
| Time skew | Clock difference > 5 min | Sync NTP |
| Missing claims | Claim not configured | Add claim mapping |

### Sign-in Log Analysis

```
Microsoft Entra ID → Sign-in logs → Filter by app
```

Review:
- Authentication Details tab
- SAML assertion content
- Error codes and messages

### Error: AADSTS50105 - User Not Assigned

```yaml
Cause: User/group not assigned to application

Solution:
  1. Go to Enterprise Applications → [App]
  2. Users and groups → Add user/group
  3. Assign user or group

Or:
  Properties → Assignment required → No
```

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- SAML authentication flow
- Certificate configuration
- Claims mapping

---

## Key Takeaways

- SAML enables SSO between IdP (Entra ID) and SP (application)
- Configure Entity ID and ACS URL from application vendor
- NameID uniquely identifies users to the application
- Add custom claims for application-specific attributes
- Manage certificate lifecycle to avoid SSO failures
- Use SAML tracer tools for debugging

---

## Navigation

← [7.1 SSO Concepts](../7.1-sso-concepts/README.md) | [7.3 OIDC SSO →](../7.3-oidc-sso/README.md)
