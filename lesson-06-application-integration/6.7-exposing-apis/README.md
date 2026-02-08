# 6.7 Exposing APIs

## Overview

When you build your own API and want other applications to access it, you need to expose it through Microsoft Entra ID. This lesson covers how to configure your application to act as an API, define scopes, and authorize client applications.

## Learning Objectives

- Configure Application ID URI
- Define custom scopes for your API
- Pre-authorize client applications
- Validate tokens in your API
- Implement proper scope-based authorization

---

## Exposing an API

### Portal Navigation

```
App registrations → [Your App] → Expose an API
```

### Configuration Steps

1. Set Application ID URI
2. Add scopes (permissions)
3. Authorize client applications (optional)

---

## Application ID URI

### What is it?

The Application ID URI uniquely identifies your API in token requests.

### Default Format

```yaml
Default: api://{application-client-id}
Example: api://a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

### Custom Format

```yaml
Custom URIs (multi-tenant apps):
  - https://api.contoso.com
  - https://contoso.com/api

Requirements:
  - Must be globally unique
  - HTTPS required for custom URLs
  - Must use verified domain (for custom URLs)
```

### Setting the URI

```
Expose an API → Set → Application ID URI → Save
```

---

## Defining Scopes

### What are Scopes?

Scopes define the permissions that client applications can request to access your API.

### Adding a Scope

```
Expose an API → Add a scope
```

### Scope Configuration

| Field | Description | Example |
|-------|-------------|---------|
| Scope name | Permission identifier | `access_as_user` |
| Who can consent | Admin only or Admins and users | Admins and users |
| Admin consent display name | Name shown to admins | Access API as user |
| Admin consent description | Description for admins | Allows the app to access the API on behalf of the user |
| User consent display name | Name shown to users | Access API |
| User consent description | Description for users | Allow this app to access the API on your behalf |
| State | Enabled or Disabled | Enabled |

### Example Scopes

```yaml
Read-only access:
  Scope: api://contoso-api/Data.Read
  Description: Read data from the API

Write access:
  Scope: api://contoso-api/Data.Write
  Description: Write data to the API

Admin access:
  Scope: api://contoso-api/Admin.Full
  Description: Full administrative access
  Who can consent: Admins only
```

### Scope Naming Best Practices

```yaml
Format: {Resource}.{Action}

Examples:
  - Tasks.Read
  - Tasks.Write
  - Tasks.Delete
  - Users.Manage
  - Reports.Generate
  - Settings.Configure

Avoid:
  - Generic names like "access" or "full"
  - Overly broad scopes
```

---

## Pre-authorizing Client Applications

### Purpose

Pre-authorization allows known client applications to access your API without user consent prompts for the specified scopes.

### When to Use

```yaml
Use for:
  - Your own frontend calling your backend
  - Trusted first-party applications
  - Partner applications with established trust

Don't use for:
  - Unknown third-party applications
  - When user consent is important for compliance
```

### Adding Pre-authorized Clients

```
Expose an API → Add a client application
```

Configuration:
```yaml
Client ID: {frontend-app-client-id}
Authorized scopes:
  ✓ api://contoso-api/Data.Read
  ✓ api://contoso-api/Data.Write
```

### Example Scenario

```yaml
Backend API: api://contoso-backend
Frontend SPA: Client ID = abc123...

Pre-authorization:
  Client: abc123...
  Scopes:
    - api://contoso-backend/Data.Read
    - api://contoso-backend/Data.Write

Result: SPA users see no additional consent for these scopes
```

---

## Token Validation

### Validating Access Tokens

When your API receives a request, validate the access token:

```yaml
Validation Steps:
  1. Verify token signature
  2. Check issuer (iss claim)
  3. Check audience (aud claim)
  4. Verify token not expired
  5. Check required scopes (scp claim)
```

### Token Claims

```json
{
  "aud": "api://contoso-api",
  "iss": "https://login.microsoftonline.com/{tenant}/v2.0",
  "iat": 1609459200,
  "exp": 1609462800,
  "azp": "abc123...",
  "scp": "Data.Read Data.Write",
  "sub": "user-object-id",
  "name": "John Doe",
  "preferred_username": "john@contoso.com"
}
```

### Key Claims

| Claim | Description | Example |
|-------|-------------|---------|
| aud | Audience - your API's ID | `api://contoso-api` |
| scp | Scopes granted (delegated) | `Data.Read Data.Write` |
| roles | Roles granted (application) | `API.ReadWrite.All` |
| azp | Client app that requested token | `abc123...` |
| sub | User identifier | `xyz789...` |

---

## Code Examples

### ASP.NET Core API

```csharp
// Program.cs
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApi(builder.Configuration.GetSection("AzureAd"));

// appsettings.json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "TenantId": "your-tenant-id",
    "ClientId": "your-api-client-id",
    "Audience": "api://contoso-api"
  }
}

// Controller with scope authorization
[Authorize]
[RequiredScope("Data.Read")]
[ApiController]
[Route("api/[controller]")]
public class DataController : ControllerBase
{
    [HttpGet]
    public IActionResult Get()
    {
        return Ok(new { data = "Protected data" });
    }

    [HttpPost]
    [RequiredScope("Data.Write")]
    public IActionResult Post([FromBody] DataModel data)
    {
        return Ok(new { message = "Data saved" });
    }
}
```

### Node.js/Express API

```javascript
const express = require('express');
const jwt = require('jsonwebtoken');
const jwksClient = require('jwks-rsa');

const app = express();

// JWKS client for token validation
const client = jwksClient({
    jwksUri: `https://login.microsoftonline.com/${process.env.TENANT_ID}/discovery/v2.0/keys`
});

// Middleware to validate token
const validateToken = async (req, res, next) => {
    const authHeader = req.headers.authorization;
    if (!authHeader || !authHeader.startsWith('Bearer ')) {
        return res.status(401).json({ error: 'Missing token' });
    }

    const token = authHeader.split(' ')[1];

    try {
        const decoded = jwt.decode(token, { complete: true });
        const key = await client.getSigningKey(decoded.header.kid);

        const verified = jwt.verify(token, key.getPublicKey(), {
            audience: process.env.API_AUDIENCE,
            issuer: `https://login.microsoftonline.com/${process.env.TENANT_ID}/v2.0`
        });

        req.user = verified;
        next();
    } catch (error) {
        return res.status(401).json({ error: 'Invalid token' });
    }
};

// Middleware to check scopes
const requireScope = (requiredScope) => {
    return (req, res, next) => {
        const scopes = req.user.scp ? req.user.scp.split(' ') : [];
        if (!scopes.includes(requiredScope)) {
            return res.status(403).json({ error: 'Insufficient scope' });
        }
        next();
    };
};

// Protected routes
app.get('/api/data', validateToken, requireScope('Data.Read'), (req, res) => {
    res.json({ data: 'Protected data' });
});

app.post('/api/data', validateToken, requireScope('Data.Write'), (req, res) => {
    res.json({ message: 'Data saved' });
});
```

### Python/Flask API

```python
from flask import Flask, request, jsonify
from functools import wraps
import jwt
from jwt import PyJWKClient

app = Flask(__name__)

TENANT_ID = os.environ['TENANT_ID']
API_AUDIENCE = os.environ['API_AUDIENCE']
JWKS_URL = f'https://login.microsoftonline.com/{TENANT_ID}/discovery/v2.0/keys'

jwks_client = PyJWKClient(JWKS_URL)

def validate_token(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        auth_header = request.headers.get('Authorization', '')
        if not auth_header.startswith('Bearer '):
            return jsonify({'error': 'Missing token'}), 401

        token = auth_header.split(' ')[1]

        try:
            signing_key = jwks_client.get_signing_key_from_jwt(token)
            payload = jwt.decode(
                token,
                signing_key.key,
                algorithms=['RS256'],
                audience=API_AUDIENCE,
                issuer=f'https://login.microsoftonline.com/{TENANT_ID}/v2.0'
            )
            request.user = payload
        except jwt.exceptions.InvalidTokenError as e:
            return jsonify({'error': str(e)}), 401

        return f(*args, **kwargs)
    return decorated

def require_scope(required_scope):
    def decorator(f):
        @wraps(f)
        def decorated(*args, **kwargs):
            scopes = request.user.get('scp', '').split()
            if required_scope not in scopes:
                return jsonify({'error': 'Insufficient scope'}), 403
            return f(*args, **kwargs)
        return decorated
    return decorator

@app.route('/api/data', methods=['GET'])
@validate_token
@require_scope('Data.Read')
def get_data():
    return jsonify({'data': 'Protected data'})
```

---

## PowerShell Configuration

### Add Scope to API

```powershell
Connect-MgGraph -Scopes "Application.ReadWrite.All"

$appId = "your-app-object-id"

# Define scope
$scope = @{
    AdminConsentDescription = "Allows the app to read data"
    AdminConsentDisplayName = "Read data"
    Id = (New-Guid).ToString()
    IsEnabled = $true
    Type = "User"
    UserConsentDescription = "Allow this app to read your data"
    UserConsentDisplayName = "Read data"
    Value = "Data.Read"
}

# Get current app
$app = Get-MgApplication -ApplicationId $appId

# Add scope to API
$api = @{
    Oauth2PermissionScopes = @($scope)
    RequestedAccessTokenVersion = 2
}

Update-MgApplication -ApplicationId $appId -Api $api
```

### Set Application ID URI

```powershell
$uri = "api://contoso-api"

Update-MgApplication -ApplicationId $appId -IdentifierUris @($uri)
```

### Pre-authorize Client

```powershell
$preAuth = @{
    AppId = "frontend-client-id"
    DelegatedPermissionIds = @("scope-id-1", "scope-id-2")
}

Update-MgApplication -ApplicationId $appId -Api @{
    PreAuthorizedApplications = @($preAuth)
}
```

---

## Best Practices

### Scope Design

```yaml
Do:
  - Create granular scopes (Read, Write, Delete separately)
  - Use descriptive names
  - Document each scope's purpose
  - Set appropriate consent levels

Don't:
  - Create a single "full access" scope
  - Mix read and write in same scope
  - Use vague scope names
```

### Security

```yaml
Always:
  - Validate tokens on every request
  - Check audience matches your API
  - Verify token signature
  - Check required scopes before processing
  - Log failed authorization attempts
```

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- API exposure configuration
- Scope definition
- Token flow with scopes

---

## Key Takeaways

- Set Application ID URI to uniquely identify your API
- Create granular scopes for fine-grained access control
- Pre-authorize trusted client applications
- Always validate tokens in your API
- Check scopes match required permissions
- Use established libraries for token validation

---

## Navigation

← [6.6 OAuth/OIDC Flows](../6.6-oauth-oidc-flows/README.md) | [6.8 Enterprise Applications →](../6.8-enterprise-applications/README.md)
