# 7.3 OpenID Connect SSO

## Overview

OpenID Connect (OIDC) is a modern authentication layer built on OAuth 2.0. It's the recommended protocol for new applications due to its simplicity, mobile support, and JSON-based tokens. This lesson covers OIDC configuration for SSO.

## Learning Objectives

- Understand OIDC vs SAML differences
- Configure OIDC-based applications
- Work with ID tokens and claims
- Set up token configuration
- Implement OIDC SSO flows

---

## OIDC vs SAML

### Comparison

| Aspect | SAML | OIDC |
|--------|------|------|
| Format | XML | JSON |
| Token | Assertion | JWT (ID Token) |
| Complexity | Higher | Lower |
| Mobile support | Limited | Excellent |
| API access | Separate | Built-in (OAuth) |
| Best for | Enterprise legacy | Modern apps |

### When to Use OIDC

```yaml
Choose OIDC when:
  - Building new applications
  - Mobile or SPA applications
  - Need API access alongside SSO
  - Want simpler implementation
  - Using modern frameworks

Choose SAML when:
  - Application only supports SAML
  - Migrating from on-premises ADFS
  - Enterprise vendor requires SAML
```

---

## OIDC Concepts

### Key Terminology

| Term | Description |
|------|-------------|
| ID Token | JWT containing user identity claims |
| Access Token | Token for API authorization |
| Authorization Endpoint | Where authentication starts |
| Token Endpoint | Where tokens are issued |
| UserInfo Endpoint | Additional user claims |
| Scopes | Permissions for user data |

### Standard Scopes

| Scope | Claims Included |
|-------|-----------------|
| openid | Subject (sub) - Required for OIDC |
| profile | name, family_name, given_name, etc. |
| email | email, email_verified |
| address | address |
| phone | phone_number, phone_number_verified |

---

## OIDC Authentication Flow

### Authorization Code Flow

```
1. User accesses application
          ↓
2. App redirects to /authorize
   (with client_id, redirect_uri, scope, response_type=code)
          ↓
3. User authenticates at Entra ID
          ↓
4. User consents to permissions
          ↓
5. Entra ID redirects to redirect_uri
   (with authorization code)
          ↓
6. App exchanges code for tokens at /token
   (with client_id, client_secret, code)
          ↓
7. App receives ID Token + Access Token
          ↓
8. App validates ID Token, creates session
```

### ID Token Structure

```json
{
  "typ": "JWT",
  "alg": "RS256",
  "kid": "key-id"
}
.
{
  "aud": "client-id",
  "iss": "https://login.microsoftonline.com/{tenant}/v2.0",
  "iat": 1609459200,
  "exp": 1609462800,
  "sub": "user-object-id",
  "name": "John Doe",
  "preferred_username": "john@contoso.com",
  "email": "john@contoso.com",
  "oid": "user-object-id",
  "tid": "tenant-id"
}
.
[signature]
```

---

## Configuring OIDC SSO

### For Gallery Apps with OIDC

Some gallery apps support OIDC. Configuration is simpler:

```
Enterprise applications → Add → Search for app
→ Create → Single sign-on → OpenID Connect
```

### For Custom Applications

Configuration is done in App Registration:

```
App registrations → [App] → Authentication
```

### Platform Configuration

```yaml
Web Application:
  Platform: Web
  Redirect URIs:
    - https://app.contoso.com/signin-oidc
    - http://localhost:5000/signin-oidc
  Front-channel logout URL: https://app.contoso.com/signout-oidc
  ID tokens: ✓ Enabled

SPA:
  Platform: Single-page application
  Redirect URIs:
    - https://spa.contoso.com
    - http://localhost:3000
  (Uses PKCE, no secrets needed)

Mobile/Desktop:
  Platform: Mobile and desktop applications
  Redirect URIs:
    - msauth://com.contoso.app
    - https://login.microsoftonline.com/common/oauth2/nativeclient
```

---

## Token Configuration

### Adding Optional Claims

```
App registrations → [App] → Token configuration
→ Add optional claim
```

### ID Token Claims

| Claim | Description |
|-------|-------------|
| email | User's email address |
| family_name | Last name |
| given_name | First name |
| upn | User principal name |
| preferred_username | Display username |
| groups | Group memberships |
| roles | App roles assigned |

### Adding Claims

```yaml
Add optional claim:
  Token type: ID
  Claims:
    - auth_time: Time of authentication
    - email: Email address
    - family_name: Surname
    - given_name: First name
    - ipaddr: IP address
    - upn: UPN

Token type: Access
  Claims:
    - idtyp: Token type identifier
```

### Groups Claim

```
Token configuration → Add groups claim
```

Options:
```yaml
Groups assigned to application:
  - Only groups assigned to this app

Security groups:
  - All security groups

All groups:
  - Security + Microsoft 365 groups

Emit as:
  - Group ID (GUID)
  - sAMAccountName (requires hybrid)
  - NetBIOSDomain\sAMAccountName
```

---

## Endpoints

### Discovery Document

```
https://login.microsoftonline.com/{tenant}/v2.0/.well-known/openid-configuration
```

Returns all endpoints and supported features.

### Key Endpoints

```yaml
Authorization:
  https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize

Token:
  https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token

Logout:
  https://login.microsoftonline.com/{tenant}/oauth2/v2.0/logout

UserInfo:
  https://graph.microsoft.com/oidc/userinfo

JWKS (for token validation):
  https://login.microsoftonline.com/{tenant}/discovery/v2.0/keys
```

---

## Code Examples

### ASP.NET Core

```csharp
// Program.cs
builder.Services.AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApp(builder.Configuration.GetSection("AzureAd"));

// appsettings.json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "TenantId": "your-tenant-id",
    "ClientId": "your-client-id",
    "ClientSecret": "your-client-secret",
    "CallbackPath": "/signin-oidc"
  }
}
```

### Node.js (passport-azure-ad)

```javascript
const OIDCStrategy = require('passport-azure-ad').OIDCStrategy;

passport.use(new OIDCStrategy({
    identityMetadata: `https://login.microsoftonline.com/${tenantId}/v2.0/.well-known/openid-configuration`,
    clientID: clientId,
    responseType: 'code',
    responseMode: 'query',
    redirectUrl: 'http://localhost:3000/auth/callback',
    clientSecret: clientSecret,
    scope: ['openid', 'profile', 'email']
  },
  (iss, sub, profile, accessToken, refreshToken, done) => {
    return done(null, profile);
  }
));
```

### React SPA (MSAL React)

```javascript
import { PublicClientApplication } from "@azure/msal-browser";
import { MsalProvider, useMsal } from "@azure/msal-react";

const msalConfig = {
    auth: {
        clientId: "your-client-id",
        authority: "https://login.microsoftonline.com/your-tenant-id",
        redirectUri: "http://localhost:3000"
    }
};

const pca = new PublicClientApplication(msalConfig);

// In component
const { instance, accounts } = useMsal();

const handleLogin = () => {
    instance.loginPopup({
        scopes: ["openid", "profile", "User.Read"]
    });
};
```

---

## ID Token Validation

### Validation Steps

```yaml
1. Verify signature using JWKS
2. Check issuer (iss) matches expected
3. Check audience (aud) matches client_id
4. Check expiration (exp) not passed
5. Check not before (nbf) is past
6. Verify nonce if used
```

### MSAL Handles Validation

MSAL libraries automatically validate tokens. Manual validation only needed for custom implementations.

---

## Logout

### Front-Channel Logout

```
https://login.microsoftonline.com/{tenant}/oauth2/v2.0/logout
?post_logout_redirect_uri=https://app.contoso.com
```

### ASP.NET Core

```csharp
// Configure in authentication
options.SignedOutCallbackPath = "/signout-oidc";
options.SignedOutRedirectUri = "/";

// Controller action
public IActionResult Logout()
{
    return SignOut(
        new AuthenticationProperties { RedirectUri = "/" },
        CookieAuthenticationDefaults.AuthenticationScheme,
        OpenIdConnectDefaults.AuthenticationScheme
    );
}
```

---

## Best Practices

### Security

```yaml
Do:
  - Use authorization code flow (not implicit)
  - Use PKCE for public clients
  - Validate tokens properly
  - Use HTTPS for redirect URIs
  - Store tokens securely

Don't:
  - Store tokens in localStorage for SPAs
  - Skip token validation
  - Use implicit flow for new apps
  - Expose client secrets in frontend
```

### Configuration

```yaml
Recommendations:
  - Request minimum scopes needed
  - Use optional claims sparingly
  - Configure logout properly
  - Set appropriate token lifetimes
```

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- OIDC authentication flow
- Token structure
- Claims configuration

---

## Key Takeaways

- OIDC is preferred for modern applications
- ID tokens are JWTs containing user claims
- Configure claims in Token configuration blade
- Use MSAL libraries for implementation
- PKCE is required for public clients (SPA, mobile)
- Always validate ID tokens properly

---

## Navigation

← [7.2 SAML SSO](../7.2-saml-sso/README.md) | [7.4 Password & Linked SSO →](../7.4-password-linked-sso/README.md)
