# 6.3 Authentication Settings

## Overview

After registering an application, you need to configure its authentication settings. This lesson covers platform configurations, redirect URIs, token settings, and implicit grant options for different application types.

## Learning Objectives

- Configure platform-specific authentication settings
- Set up redirect URIs for different application types
- Understand implicit grant and hybrid flow options
- Configure front-channel and back-channel logout
- Enable public client flows when needed

---

## Accessing Authentication Settings

### Portal Navigation

```
Microsoft Entra ID → App registrations → [Your App] → Authentication
```

---

## Platform Configurations

### Available Platforms

| Platform | Use Case | Auth Method |
|----------|----------|-------------|
| Web | Server-side web apps | Authorization Code |
| Single-page application | Client-side JavaScript | Auth Code + PKCE |
| iOS/macOS | Native iOS/Mac apps | Auth Code + PKCE |
| Android | Native Android apps | Auth Code + PKCE |
| Mobile and desktop | Windows, Linux, console | Device Code, Auth Code |

### Adding a Platform

```
Authentication → Add a platform → Select platform type
```

---

## Web Application Configuration

### Settings for Web Apps

```yaml
Platform: Web

Redirect URIs:
  - https://app.contoso.com/auth/callback
  - https://app.contoso.com/signin-oidc
  - http://localhost:3000/auth/callback  # Development

Front-channel logout URL:
  - https://app.contoso.com/signout-oidc

Implicit grant and hybrid flows:
  □ Access tokens (for implicit flow)
  □ ID tokens (for implicit and hybrid flow)
```

### When to Use Each Setting

| Setting | Purpose | Recommendation |
|---------|---------|----------------|
| Redirect URIs | Where tokens are sent | Required |
| Front-channel logout | Clear browser session | Recommended |
| Access tokens (implicit) | Legacy scenarios | Avoid if possible |
| ID tokens (implicit) | Hybrid flow | Use for OIDC hybrid |

### Example: ASP.NET Core Web App

```yaml
Redirect URIs:
  - https://localhost:5001/signin-oidc
  - https://app.contoso.com/signin-oidc

Front-channel logout URL:
  - https://app.contoso.com/signout-oidc

Implicit grant:
  ✓ ID tokens  # Required for hybrid flow
  □ Access tokens
```

---

## Single-Page Application (SPA)

### Settings for SPAs

```yaml
Platform: Single-page application

Redirect URIs:
  - http://localhost:3000
  - http://localhost:3000/auth/callback
  - https://spa.contoso.com
  - https://spa.contoso.com/auth/callback
```

### Key Differences from Web

| Aspect | Web | SPA |
|--------|-----|-----|
| Secret | Required | Not allowed |
| PKCE | Optional | Required |
| Token storage | Server-side | Browser memory |
| Implicit grant | Optional | Not needed |

### MSAL.js Configuration

```javascript
const msalConfig = {
    auth: {
        clientId: "your-client-id",
        authority: "https://login.microsoftonline.com/your-tenant-id",
        redirectUri: "http://localhost:3000"
    },
    cache: {
        cacheLocation: "sessionStorage", // Or "localStorage"
        storeAuthStateInCookie: false
    }
};
```

---

## Mobile and Desktop Applications

### iOS/macOS Configuration

```yaml
Platform: iOS/macOS

Redirect URI: msauth.com.contoso.app://auth
Bundle ID: com.contoso.app
```

### Android Configuration

```yaml
Platform: Android

Redirect URI: msauth://com.contoso.app/signature-hash
Package name: com.contoso.app
Signature hash: [Generated from your signing key]
```

### Generating Android Signature Hash

```bash
# Debug keystore
keytool -exportcert -alias androiddebugkey \
  -keystore ~/.android/debug.keystore \
  | openssl sha1 -binary | openssl base64

# Release keystore
keytool -exportcert -alias your-key-alias \
  -keystore your-release.keystore \
  | openssl sha1 -binary | openssl base64
```

### Desktop/Console Application

```yaml
Platform: Mobile and desktop applications

Redirect URIs:
  - https://login.microsoftonline.com/common/oauth2/nativeclient
  - http://localhost  # For system browser

Allow public client flows: Yes
```

---

## Public Client Flows

### What is a Public Client?

Applications that cannot securely store secrets:
- Mobile apps
- Desktop apps
- Console applications
- IoT devices

### Enabling Public Client Flows

```
Authentication → Advanced settings → Allow public client flows → Yes
```

### Enabled Flows

| Flow | Description | Use Case |
|------|-------------|----------|
| Device Code | User authenticates on different device | CLI tools, IoT |
| ROPC | Username/password directly | Legacy, testing only |

### Device Code Example

```powershell
# PowerShell with MSAL
$app = [Microsoft.Identity.Client.PublicClientApplicationBuilder]::Create($ClientId)
    .WithAuthority("https://login.microsoftonline.com/$TenantId")
    .WithDefaultRedirectUri()
    .Build()

$scopes = @("User.Read")
$result = $app.AcquireTokenWithDeviceCode($scopes, {
    param($deviceCodeResult)
    Write-Host $deviceCodeResult.Message
}).ExecuteAsync().GetAwaiter().GetResult()

Write-Host "Access Token: $($result.AccessToken)"
```

---

## Implicit Grant Settings

### Understanding Implicit Grant

Implicit grant returns tokens directly in the URL fragment. It's considered legacy and less secure than authorization code flow.

### When Implicit Grant is Used

| Scenario | Implicit Needed | Better Alternative |
|----------|-----------------|-------------------|
| Legacy SPA | Yes | Migrate to MSAL.js 2.x |
| ID token for hybrid | Yes | Authorization code |
| Access token for APIs | Avoid | Auth code + PKCE |

### Hybrid Flow (ID Token)

```yaml
# Enable for ASP.NET Core with OpenID Connect hybrid flow
Implicit grant and hybrid flows:
  □ Access tokens
  ✓ ID tokens  # For hybrid flow
```

### Security Considerations

```yaml
Risks of Implicit Grant:
  - Tokens exposed in URL
  - No refresh tokens
  - Vulnerable to token leakage
  - Limited token lifetime

Recommendations:
  - Use Auth Code + PKCE for SPAs
  - Use Auth Code for server apps
  - Only enable ID tokens for hybrid flow
```

---

## Logout Configuration

### Front-Channel Logout

Browser-based logout that clears session cookies:

```yaml
Front-channel logout URL: https://app.contoso.com/signout-oidc

Behavior:
  1. User clicks logout in one app
  2. Entra ID sends logout request to all apps
  3. Each app clears its session
  4. User is signed out everywhere
```

### Back-Channel Logout

Server-to-server logout notification:

```yaml
Back-channel logout URL: https://app.contoso.com/api/signout

Behavior:
  1. Logout initiated
  2. Entra ID sends POST to back-channel URL
  3. App invalidates server-side session
  4. Works even if browser is closed
```

### ASP.NET Core Logout Handling

```csharp
// Startup.cs
services.AddAuthentication(options => {
    options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = OpenIdConnectDefaults.AuthenticationScheme;
})
.AddCookie()
.AddOpenIdConnect(options => {
    options.SignedOutCallbackPath = "/signout-oidc";
    options.RemoteSignOutPath = "/signout-oidc";
});
```

---

## Advanced Settings

### Supported Account Types (Post-Registration)

```
Authentication → Supported account types
```

You can change account types after registration, but:
- Single → Multi: Allowed
- Multi → Single: Only if no external usage
- Adding Personal: Allowed
- Removing Personal: Only if no external usage

### Redirect URI Limits

```yaml
Limits:
  - Maximum 256 characters per URI
  - Maximum 100 redirect URIs
  - Wildcards only in subdomain (multi-tenant)
  - No query strings or fragments
```

### Wildcard URIs (Multi-Tenant)

```yaml
# Only for multi-tenant apps
Allowed:
  - https://*.contoso.com/callback

Not Allowed:
  - https://app.contoso.com/*
  - https://*/callback
```

---

## PowerShell Configuration

### Configure Web Platform

```powershell
Connect-MgGraph -Scopes "Application.ReadWrite.All"

$appId = "your-app-object-id"

$webSettings = @{
    Web = @{
        RedirectUris = @(
            "https://app.contoso.com/signin-oidc"
            "http://localhost:5000/signin-oidc"
        )
        LogoutUrl = "https://app.contoso.com/signout-oidc"
        ImplicitGrantSettings = @{
            EnableIdTokenIssuance = $true
            EnableAccessTokenIssuance = $false
        }
    }
}

Update-MgApplication -ApplicationId $appId -BodyParameter $webSettings
```

### Configure SPA Platform

```powershell
$spaSettings = @{
    Spa = @{
        RedirectUris = @(
            "http://localhost:3000"
            "https://spa.contoso.com"
        )
    }
}

Update-MgApplication -ApplicationId $appId -BodyParameter $spaSettings
```

### Enable Public Client

```powershell
$publicSettings = @{
    IsFallbackPublicClient = $true
    PublicClient = @{
        RedirectUris = @(
            "https://login.microsoftonline.com/common/oauth2/nativeclient"
        )
    }
}

Update-MgApplication -ApplicationId $appId -BodyParameter $publicSettings
```

---

## Best Practices

### Security

```yaml
Do:
  - Use HTTPS for all production URIs
  - Enable front-channel logout
  - Use Auth Code + PKCE instead of implicit
  - Minimize redirect URI count

Don't:
  - Enable implicit grant unless necessary
  - Use wildcards carelessly
  - Leave development URIs in production
```

### Configuration

```yaml
Recommendations:
  - Separate dev/staging/prod redirect URIs
  - Document each redirect URI purpose
  - Review URIs periodically
  - Remove unused URIs
```

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- Platform configuration options
- Authentication flow by platform type
- Implicit grant decision tree

---

## Key Takeaways

- Each platform type has specific configuration requirements
- SPAs use PKCE and don't need secrets
- Public client flows enable device code and ROPC
- Avoid implicit grant when possible; use auth code + PKCE
- Configure logout URLs for proper session management
- Use wildcards only for multi-tenant scenarios

---

## Navigation

← [6.2 Registering Applications](../6.2-registering-applications/README.md) | [6.4 Credentials Management →](../6.4-credentials-management/README.md)
