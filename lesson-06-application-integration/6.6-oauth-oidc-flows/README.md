# 6.6 OAuth 2.0 and OpenID Connect Flows

## Overview

OAuth 2.0 and OpenID Connect are the modern protocols used for authorization and authentication in Microsoft Entra ID. This lesson covers the different flows available, when to use each, and provides code examples for implementation.

## Learning Objectives

- Understand OAuth 2.0 vs OpenID Connect
- Choose the appropriate flow for your application type
- Implement authorization code flow with PKCE
- Implement client credentials flow for services
- Use Microsoft Authentication Libraries (MSAL)

---

## Protocol Overview

### OAuth 2.0 vs OpenID Connect

| Protocol | Purpose | Result |
|----------|---------|--------|
| OAuth 2.0 | Authorization | Access Token |
| OpenID Connect | Authentication | ID Token + Access Token |

```yaml
OAuth 2.0:
  Question: "What can you access?"
  Answer: Access token with scopes/permissions

OpenID Connect:
  Question: "Who are you?"
  Answer: ID token with user identity + Access token
```

### Key Endpoints

```yaml
Authorization Endpoint:
  URL: https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize
  Purpose: User authentication and consent

Token Endpoint:
  URL: https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token
  Purpose: Exchange code for tokens

Logout Endpoint:
  URL: https://login.microsoftonline.com/{tenant}/oauth2/v2.0/logout
  Purpose: End user session

Tenant values:
  - Your tenant ID (GUID)
  - Your domain (contoso.onmicrosoft.com)
  - "common" - Any org + personal
  - "organizations" - Any org account
  - "consumers" - Personal accounts only
```

---

## Available Flows

### Flow Selection Guide

| Application Type | Recommended Flow |
|-----------------|------------------|
| Web Application | Authorization Code |
| Single-Page Application | Authorization Code + PKCE |
| Mobile/Desktop | Authorization Code + PKCE |
| Daemon/Service | Client Credentials |
| CLI Tool | Device Code |
| Legacy (avoid) | Implicit, ROPC |

---

## Authorization Code Flow

### When to Use

- Web applications with server-side processing
- Confidential clients (can store secrets securely)

### Flow Diagram

```
┌──────┐                              ┌──────────┐                    ┌─────────┐
│ User │                              │   App    │                    │ Entra   │
└──┬───┘                              └────┬─────┘                    └────┬────┘
   │ 1. Click "Sign In"                    │                               │
   │──────────────────────────────────────▶│                               │
   │                                       │                               │
   │ 2. Redirect to /authorize             │                               │
   │◀──────────────────────────────────────│                               │
   │                                       │                               │
   │ 3. Authenticate (username/password/MFA)                               │
   │──────────────────────────────────────────────────────────────────────▶│
   │                                       │                               │
   │ 4. Consent to permissions             │                               │
   │──────────────────────────────────────────────────────────────────────▶│
   │                                       │                               │
   │ 5. Redirect with authorization code   │                               │
   │◀──────────────────────────────────────────────────────────────────────│
   │                                       │                               │
   │ 6. Code passed to app                 │                               │
   │──────────────────────────────────────▶│                               │
   │                                       │                               │
   │                                       │ 7. POST /token (code + secret)│
   │                                       │──────────────────────────────▶│
   │                                       │                               │
   │                                       │ 8. Access + ID + Refresh token│
   │                                       │◀──────────────────────────────│
```

### Authorization Request

```http
GET https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?
    client_id=YOUR_CLIENT_ID
    &response_type=code
    &redirect_uri=https://app.contoso.com/callback
    &response_mode=query
    &scope=openid profile User.Read
    &state=random_state_value
```

### Token Request

```http
POST https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token
Content-Type: application/x-www-form-urlencoded

client_id=YOUR_CLIENT_ID
&scope=openid profile User.Read
&code=AUTHORIZATION_CODE
&redirect_uri=https://app.contoso.com/callback
&grant_type=authorization_code
&client_secret=YOUR_CLIENT_SECRET
```

---

## Authorization Code Flow with PKCE

### When to Use

- Single-page applications (SPAs)
- Mobile and desktop applications
- Any public client (can't store secrets)

### What is PKCE?

```yaml
PKCE (Proof Key for Code Exchange):
  Purpose: Protect against code interception attacks
  How: Uses code_verifier and code_challenge

Flow:
  1. App generates random code_verifier (43-128 chars)
  2. App creates code_challenge = SHA256(code_verifier)
  3. Send code_challenge in /authorize
  4. Send code_verifier in /token
  5. Server verifies SHA256(code_verifier) == code_challenge
```

### PKCE Authorization Request

```http
GET https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?
    client_id=YOUR_CLIENT_ID
    &response_type=code
    &redirect_uri=http://localhost:3000
    &response_mode=fragment
    &scope=openid profile User.Read
    &state=random_state_value
    &code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM
    &code_challenge_method=S256
```

### PKCE Token Request

```http
POST https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token
Content-Type: application/x-www-form-urlencoded

client_id=YOUR_CLIENT_ID
&scope=openid profile User.Read
&code=AUTHORIZATION_CODE
&redirect_uri=http://localhost:3000
&grant_type=authorization_code
&code_verifier=dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk
```

---

## Client Credentials Flow

### When to Use

- Daemon applications
- Background services
- Application-to-API calls (no user)

### Flow Diagram

```
┌─────────────┐                              ┌─────────┐
│   Service   │                              │ Entra   │
└──────┬──────┘                              └────┬────┘
       │                                          │
       │ 1. POST /token (client_id + secret)      │
       │─────────────────────────────────────────▶│
       │                                          │
       │ 2. Access token                          │
       │◀─────────────────────────────────────────│
       │                                          │
       │ 3. Call API with access token            │
       │─────────────────────────────────────────▶│
```

### Token Request

```http
POST https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token
Content-Type: application/x-www-form-urlencoded

client_id=YOUR_CLIENT_ID
&scope=https://graph.microsoft.com/.default
&client_secret=YOUR_CLIENT_SECRET
&grant_type=client_credentials
```

### Important Notes

```yaml
Scope format: Use ".default" suffix
  - https://graph.microsoft.com/.default
  - api://your-api-id/.default

No refresh token: Request new access token when expired
No ID token: No user identity, only app identity
```

---

## Device Code Flow

### When to Use

- CLI applications
- IoT devices with no keyboard
- TV applications
- Devices with limited input capabilities

### Flow Diagram

```
┌──────────┐          ┌─────────┐          ┌──────┐
│ Device   │          │ Entra   │          │ User │
│ (CLI)    │          │         │          │(Phone)│
└────┬─────┘          └────┬────┘          └──┬───┘
     │                     │                  │
     │ 1. Request device   │                  │
     │    code             │                  │
     │────────────────────▶│                  │
     │                     │                  │
     │ 2. Device code +    │                  │
     │    user code +      │                  │
     │    verification URL │                  │
     │◀────────────────────│                  │
     │                     │                  │
     │ Display: "Go to URL │                  │
     │ and enter code ABC" │                  │
     │─────────────────────────────────────▶│
     │                     │                  │
     │ 3. Poll /token      │ 4. User visits  │
     │    (waiting)        │    URL, enters  │
     │────────────────────▶│    code, signs  │
     │                     │◀─────in─────────│
     │                     │                  │
     │ 5. Access token     │                  │
     │    (after user      │                  │
     │    completes auth)  │                  │
     │◀────────────────────│                  │
```

### Device Code Request

```http
POST https://login.microsoftonline.com/{tenant}/oauth2/v2.0/devicecode
Content-Type: application/x-www-form-urlencoded

client_id=YOUR_CLIENT_ID
&scope=User.Read
```

### Response

```json
{
    "device_code": "device_code_value",
    "user_code": "ABCD-EFGH",
    "verification_uri": "https://microsoft.com/devicelogin",
    "expires_in": 900,
    "interval": 5,
    "message": "To sign in, use a web browser to open..."
}
```

---

## Code Examples

### JavaScript/Node.js (MSAL)

```javascript
const msal = require('@azure/msal-node');

// Configuration
const config = {
    auth: {
        clientId: process.env.CLIENT_ID,
        authority: `https://login.microsoftonline.com/${process.env.TENANT_ID}`,
        clientSecret: process.env.CLIENT_SECRET // For confidential clients
    }
};

// Confidential Client (Web App)
const cca = new msal.ConfidentialClientApplication(config);

// Authorization Code Flow
async function getAuthUrl() {
    return await cca.getAuthCodeUrl({
        scopes: ["User.Read"],
        redirectUri: "http://localhost:3000/redirect"
    });
}

async function getTokenByCode(authCode) {
    return await cca.acquireTokenByCode({
        code: authCode,
        scopes: ["User.Read"],
        redirectUri: "http://localhost:3000/redirect"
    });
}

// Client Credentials Flow
async function getTokenForApp() {
    return await cca.acquireTokenByClientCredential({
        scopes: ["https://graph.microsoft.com/.default"]
    });
}
```

### Python (MSAL)

```python
from msal import ConfidentialClientApplication, PublicClientApplication
import os

# Confidential Client (Web App/Service)
app = ConfidentialClientApplication(
    client_id=os.environ["CLIENT_ID"],
    authority=f"https://login.microsoftonline.com/{os.environ['TENANT_ID']}",
    client_credential=os.environ["CLIENT_SECRET"]
)

# Client Credentials Flow
def get_token_for_app():
    result = app.acquire_token_for_client(
        scopes=["https://graph.microsoft.com/.default"]
    )
    return result.get("access_token")

# Device Code Flow (Public Client)
public_app = PublicClientApplication(
    client_id=os.environ["CLIENT_ID"],
    authority=f"https://login.microsoftonline.com/{os.environ['TENANT_ID']}"
)

def get_token_device_code():
    flow = public_app.initiate_device_flow(scopes=["User.Read"])
    print(flow["message"])  # Display to user
    result = public_app.acquire_token_by_device_flow(flow)
    return result.get("access_token")
```

### C# (.NET)

```csharp
using Microsoft.Identity.Client;

// Confidential Client Application
var app = ConfidentialClientApplicationBuilder
    .Create(clientId)
    .WithClientSecret(clientSecret)
    .WithAuthority($"https://login.microsoftonline.com/{tenantId}")
    .Build();

// Client Credentials Flow
var result = await app
    .AcquireTokenForClient(new[] { "https://graph.microsoft.com/.default" })
    .ExecuteAsync();

string accessToken = result.AccessToken;

// Device Code Flow (Public Client)
var publicApp = PublicClientApplicationBuilder
    .Create(clientId)
    .WithAuthority($"https://login.microsoftonline.com/{tenantId}")
    .Build();

var deviceCodeResult = await publicApp
    .AcquireTokenWithDeviceCode(
        new[] { "User.Read" },
        deviceCodeCallback => {
            Console.WriteLine(deviceCodeCallback.Message);
            return Task.CompletedTask;
        })
    .ExecuteAsync();
```

---

## Token Details

### Access Token

```yaml
Purpose: Access protected resources
Format: JWT (JSON Web Token)
Lifetime: ~1 hour (default)
Refresh: Use refresh token to get new access token
```

### ID Token

```yaml
Purpose: User identity information
Format: JWT
Contains: User claims (name, email, sub, etc.)
Lifetime: ~1 hour
```

### Refresh Token

```yaml
Purpose: Get new access tokens without re-authentication
Format: Opaque string
Lifetime: 90 days (default), sliding window
Revocation: Can be revoked by admin
```

### Token Caching

```yaml
Best Practices:
  - Always cache tokens
  - Check cache before requesting new token
  - MSAL handles caching automatically
  - Implement distributed cache for web farms
```

---

## Best Practices

### Security

```yaml
Do:
  - Always use HTTPS
  - Validate state parameter
  - Use PKCE for public clients
  - Store tokens securely
  - Validate token signatures

Don't:
  - Log tokens
  - Store tokens in localStorage (use sessionStorage)
  - Use implicit flow for new apps
  - Hardcode credentials
```

### Performance

```yaml
Recommendations:
  - Cache tokens appropriately
  - Request minimum scopes
  - Use refresh tokens
  - Implement silent token acquisition
```

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- Authorization Code flow
- Client Credentials flow
- PKCE enhancement
- Token lifecycle

---

## Key Takeaways

- OAuth 2.0 is for authorization, OIDC adds authentication
- Use Authorization Code + PKCE for most applications
- Use Client Credentials for services/daemons
- Device Code for CLI and limited-input devices
- Always use MSAL libraries instead of raw HTTP
- Cache tokens and use refresh tokens

---

## Navigation

← [6.5 API Permissions](../6.5-api-permissions/README.md) | [6.7 Exposing APIs →](../6.7-exposing-apis/README.md)
