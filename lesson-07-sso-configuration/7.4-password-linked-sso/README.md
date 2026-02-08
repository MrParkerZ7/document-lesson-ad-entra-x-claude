# 7.4 Password-Based and Linked SSO

## Overview

When applications don't support federated authentication protocols like SAML or OIDC, Microsoft Entra ID offers alternative SSO methods. Password-based SSO stores and replays credentials automatically, while Linked SSO redirects users to external identity providers or portals.

## Learning Objectives

- Understand when to use password-based SSO
- Configure password-based SSO for applications
- Set up credential management options
- Configure Linked SSO for external identity providers
- Understand requirements and limitations

---

## Password-Based SSO

### When to Use Password-Based SSO

Password-based SSO is appropriate when:

| Scenario | Recommendation |
|----------|----------------|
| App doesn't support SAML/OIDC | Use password-based |
| Legacy web application | Use password-based |
| Quick SSO integration needed | Use password-based |
| Vendor won't enable federation | Use password-based |
| Temporary solution during migration | Use password-based |

### How Password-Based SSO Works

```
1. User accesses My Apps portal
          |
2. Clicks on application tile
          |
3. Browser extension activates
          |
4. Extension retrieves stored credentials from Entra ID
          |
5. Extension auto-fills login form
          |
6. Extension submits the form
          |
7. User logged into application
```

### Requirements

| Requirement | Description |
|-------------|-------------|
| Browser Extension | My Apps Secure Sign-in Extension |
| Supported Browsers | Chrome, Edge, Firefox |
| Mobile | My Apps mobile app for iOS/Android |
| Credential Storage | Encrypted in Entra ID |

---

## Configuring Password-Based SSO

### Portal Navigation

```
Microsoft Entra ID → Enterprise applications → [App]
→ Single sign-on → Password-based
```

### Configuration Steps

```yaml
Step 1 - Select SSO Method:
  Navigate to: Enterprise applications → [App] → Single sign-on
  Select: Password-based

Step 2 - Configure Sign-on URL:
  Sign-on URL: https://app.example.com/login
  This is the page with the login form

Step 3 - Choose Credential Mode:
  Option A: Users enter own credentials
  Option B: Admin assigns shared credentials
  Option C: Users share credentials with others
```

### Credential Configuration Options

| Option | Description | Use Case |
|--------|-------------|----------|
| Individual | Each user enters own credentials | Personal accounts |
| Admin-assigned | Admin sets credentials for users | Shared service accounts |
| Group-shared | Credentials shared within a group | Team-shared accounts |

### Individual Credentials

```yaml
Configuration:
  Allow users to provide their own credentials: Yes

User Experience:
  1. User accesses app first time
  2. Prompted to enter credentials
  3. Credentials stored securely
  4. Auto-filled on subsequent access
```

### Admin-Assigned Credentials

```yaml
Configuration:
  Allow users to provide their own credentials: No

Admin Actions:
  1. Go to Users and groups
  2. Assign user/group
  3. Click "Update Credentials"
  4. Enter username and password
  5. Credentials assigned to user
```

### PowerShell: Assign Credentials

```powershell
# Connect to Microsoft Graph
Connect-MgGraph -Scopes "Application.ReadWrite.All"

# Get the service principal
$sp = Get-MgServicePrincipal -Filter "displayName eq 'Legacy App'"

# Note: Direct credential assignment via PowerShell is limited
# Use the portal for password-based SSO credential management
```

---

## Field Detection

### Automatic Detection

The browser extension automatically detects login form fields:

```yaml
Auto-detection looks for:
  - Input fields with type="text" or type="email"
  - Input fields with type="password"
  - Form submission buttons

Works best when:
  - Standard HTML login forms
  - Common field naming conventions
  - Single login form per page
```

### Manual Field Configuration

When auto-detection fails:

```
Enterprise applications → [App] → Single sign-on
→ Configure [app name] Password Single Sign-on Settings
```

```yaml
Manual Configuration:
  Login URL: https://app.example.com/login

  Username field:
    Selector: #username
    Or CSS: input[name="user"]
    Or XPath: //input[@id='login']

  Password field:
    Selector: #password
    Or CSS: input[type="password"]

  Submit button:
    Selector: #loginBtn
```

### Capture Mode

For complex login pages:

```yaml
Steps:
  1. Enable capture mode in extension settings
  2. Navigate to login page
  3. Enter credentials manually
  4. Extension captures field mappings
  5. Mappings saved for future use
```

---

## Browser Extension

### Installing the Extension

| Browser | Extension Name | Installation |
|---------|---------------|--------------|
| Chrome | My Apps Secure Sign-in | Chrome Web Store |
| Edge | My Apps Secure Sign-in | Edge Add-ons |
| Firefox | My Apps Secure Sign-in | Firefox Add-ons |

### Extension Features

```yaml
Features:
  - Automatic credential fill
  - Password vault access
  - Quick app launch
  - Sync across devices

Settings:
  - Auto-submit after fill
  - Credential update prompts
  - Secure storage preferences
```

### Troubleshooting Extension Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Extension not filling | Field detection failed | Configure manual detection |
| Wrong fields filled | Multiple forms on page | Specify exact selectors |
| Not auto-submitting | Button detection failed | Configure submit selector |
| Extension not appearing | Not installed/enabled | Reinstall extension |

---

## Linked SSO

### What is Linked SSO?

Linked SSO redirects users to an external URL. It doesn't perform authentication in Entra ID but provides a tile in My Apps for easy access.

### When to Use Linked SSO

```yaml
Use Linked SSO when:
  - App uses external identity provider
  - App has its own login portal
  - Just need a quick-launch tile
  - Redirecting to partner systems
  - Accessing SaaS with existing accounts
```

### Configuring Linked SSO

```
Enterprise applications → [App] → Single sign-on → Linked
```

```yaml
Configuration:
  Sign-on URL: https://external-idp.com/login

  Examples:
    - Partner portal: https://partner.vendor.com
    - External IdP: https://idp.thirdparty.com/saml/login
    - Custom app: https://legacy.internal.com/app
```

### Linked SSO Flow

```
1. User clicks app tile in My Apps
          |
2. Browser redirects to configured URL
          |
3. External system handles authentication
          |
4. User accesses the application
```

---

## Comparison: Password vs Linked SSO

| Aspect | Password-Based SSO | Linked SSO |
|--------|-------------------|------------|
| Authentication | Entra ID stores/fills credentials | External system |
| Security | Credentials encrypted in Entra ID | N/A - just a link |
| User experience | Auto-login with extension | Redirect to login page |
| Credential management | Centralized | Decentralized |
| MFA integration | At Entra ID level only | At destination |
| Audit logging | Limited credential access logs | Redirect only |
| Requires extension | Yes | No |

---

## Security Considerations

### Password-Based SSO Security

```yaml
Risks:
  - Credentials stored centrally
  - Extension required on devices
  - Shared credentials harder to manage
  - Limited to browser access

Mitigations:
  - Enable MFA for My Apps access
  - Use Conditional Access policies
  - Audit credential access
  - Prefer individual credentials
  - Regular password rotation
```

### Best Practices

| Practice | Description |
|----------|-------------|
| Prefer federation | Use SAML/OIDC when possible |
| Individual credentials | Avoid shared accounts |
| MFA enforcement | Require MFA for My Apps |
| Access reviews | Regular access certification |
| Audit logs | Monitor credential access |
| Temporary use | Plan migration to federation |

---

## Limitations

### Password-Based SSO Limitations

```yaml
Technical:
  - Requires browser extension
  - May not work with complex login flows
  - JavaScript-heavy pages may fail
  - Multi-page logins problematic

Operational:
  - Password changes require manual update
  - No automatic provisioning
  - Limited audit capabilities
  - Harder to revoke access
```

### Linked SSO Limitations

```yaml
Limitations:
  - No SSO - just a redirect
  - No credential management
  - No audit trail for access
  - External authentication required
  - No Conditional Access enforcement
```

---

## Migration Path

### From Password-Based to Federation

```yaml
Step 1: Identify Apps Using Password SSO
  - Review Enterprise applications
  - Filter by SSO method

Step 2: Check Federation Support
  - Contact vendor for SAML/OIDC support
  - Review documentation
  - Test in sandbox

Step 3: Configure Federation
  - Set up SAML or OIDC
  - Migrate users
  - Test thoroughly

Step 4: Retire Password SSO
  - Switch SSO method
  - Notify users
  - Remove stored credentials
```

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- Password-based SSO flow
- Browser extension interaction
- Linked SSO redirect flow

---

## Key Takeaways

- Password-based SSO stores and auto-fills credentials for non-federated apps
- Requires My Apps browser extension or mobile app
- Three credential modes: individual, admin-assigned, group-shared
- Field detection can be automatic or manually configured
- Linked SSO simply redirects to external URLs
- Prefer SAML/OIDC when available - use password SSO as fallback
- Apply MFA and Conditional Access for additional security

---

## Navigation

[7.3 OIDC SSO](../7.3-oidc-sso/README.md) | [7.5 My Apps Portal](../7.5-my-apps-portal/README.md)
