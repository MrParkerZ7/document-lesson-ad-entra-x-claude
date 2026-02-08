# 7.7 Testing and Troubleshooting SSO

## Overview

Proper testing and troubleshooting are essential for successful SSO implementations. This lesson covers built-in testing tools, browser debugging extensions, common errors, and systematic troubleshooting approaches for SAML and OIDC-based SSO.

## Learning Objectives

- Use built-in SSO testing features
- Capture and analyze SAML traffic
- Debug OIDC authentication flows
- Identify and resolve common SSO errors
- Interpret sign-in logs for troubleshooting

---

## Built-in SSO Testing

### SAML Test Feature

```
Enterprise applications → [App] → Single sign-on
→ Test this application
```

### Test Options

| Option | Description |
|--------|-------------|
| Sign in as current user | Test with your account |
| Sign in as someone else | Test with different user |
| Skip test | Manual testing later |

### Test Flow

```
1. Click "Test this application"
          |
2. Select test user mode
          |
3. Portal initiates SSO flow
          |
4. Shows SAML request/response
          |
5. Displays any errors
          |
6. Suggests remediation steps
```

### SAML Response Viewer

The test feature displays:

```yaml
SAML Response Details:
  - Issuer (IdP Entity ID)
  - Destination (ACS URL)
  - Subject (NameID)
  - Conditions (timestamps)
  - Attributes sent
  - Signature status
```

---

## SAML Tracer Tools

### Browser Extensions

| Extension | Browsers | Features |
|-----------|----------|----------|
| SAML-tracer | Firefox, Chrome | Full SAML capture |
| SAML Chrome Panel | Chrome | Built into DevTools |
| SAML Message Decoder | Edge | SAML decoding |

### Installing SAML-tracer

```yaml
Chrome:
  1. Open Chrome Web Store
  2. Search "SAML-tracer"
  3. Click "Add to Chrome"
  4. Enable extension

Firefox:
  1. Open Add-ons Manager
  2. Search "SAML-tracer"
  3. Click "Add to Firefox"
  4. Enable extension
```

### Using SAML-tracer

```yaml
Step 1: Enable Extension
  - Click SAML-tracer icon in toolbar
  - Tracer window opens

Step 2: Initiate SSO Flow
  - Navigate to application
  - Or click app in My Apps

Step 3: Capture Traffic
  - SAML requests/responses captured
  - Rows highlighted in yellow = SAML

Step 4: Analyze
  - Click on row to see details
  - View Summary, SAML, Raw tabs
  - Check assertions and claims
```

### SAML Request Analysis

```xml
<samlp:AuthnRequest
    ID="_abc123"
    Destination="https://login.microsoftonline.com/{tenant}/saml2"
    IssueInstant="2024-01-15T10:30:00Z"
    AssertionConsumerServiceURL="https://app.example.com/saml/acs">
  <saml:Issuer>https://app.example.com</saml:Issuer>
</samlp:AuthnRequest>
```

### SAML Response Analysis

```xml
<samlp:Response
    Destination="https://app.example.com/saml/acs"
    InResponseTo="_abc123">
  <saml:Assertion>
    <saml:Subject>
      <saml:NameID>user@contoso.com</saml:NameID>
    </saml:Subject>
    <saml:Conditions NotBefore="..." NotOnOrAfter="...">
      <saml:AudienceRestriction>
        <saml:Audience>https://app.example.com</saml:Audience>
      </saml:AudienceRestriction>
    </saml:Conditions>
    <saml:AttributeStatement>
      <!-- Claims here -->
    </saml:AttributeStatement>
  </saml:Assertion>
</samlp:Response>
```

---

## OIDC Debugging

### Browser Developer Tools

```yaml
Chrome DevTools (F12):
  - Network tab: Track HTTP requests
  - Console: JavaScript errors
  - Application tab: Cookies, tokens

Key Requests to Watch:
  - /authorize: Authorization request
  - /token: Token exchange
  - Redirect responses
```

### JWT Decoder

Use https://jwt.ms or https://jwt.io to decode tokens:

```yaml
Paste ID Token:
  eyJhbGciOiJSUzI1NiIs...

Decoded Output:
  Header:
    alg: RS256
    typ: JWT
    kid: key-id

  Payload:
    iss: https://login.microsoftonline.com/{tenant}/v2.0
    aud: client-id
    sub: user-object-id
    name: John Doe
    email: john@contoso.com
    exp: 1705318800

  Signature: Valid/Invalid
```

### OIDC Discovery Document

Check endpoint configuration:

```
GET https://login.microsoftonline.com/{tenant}/v2.0/.well-known/openid-configuration
```

```json
{
  "issuer": "https://login.microsoftonline.com/{tenant}/v2.0",
  "authorization_endpoint": "...authorize",
  "token_endpoint": "...token",
  "jwks_uri": "...keys",
  "response_types_supported": ["code", "id_token", "token"],
  "scopes_supported": ["openid", "profile", "email"]
}
```

---

## Common SAML Errors

### Error Reference Table

| Error Code | Description | Solution |
|------------|-------------|----------|
| AADSTS50105 | User not assigned | Assign user to app |
| AADSTS50011 | Reply URL mismatch | Check Reply URL config |
| AADSTS50020 | Guest user issue | Configure guest access |
| AADSTS650052 | App needs permissions | Admin consent required |
| AADSTS70001 | App not found | Check client ID |

### Invalid Signature

```yaml
Error: SAML signature validation failed

Causes:
  - Certificate mismatch
  - Certificate expired
  - Wrong certificate in SP

Solutions:
  1. Download new certificate from Entra ID
  2. Upload to application/SP
  3. Verify signing options match
  4. Check certificate expiry date
```

### Invalid Audience

```yaml
Error: Audience restriction not met / Invalid audience

Causes:
  - Entity ID mismatch
  - SP expects different Identifier

Solutions:
  1. Compare Identifier in Entra ID vs SP config
  2. Update to match exactly (case-sensitive)
  3. Check for trailing slashes
  4. Verify no encoding differences
```

### User Not Assigned (AADSTS50105)

```yaml
Error: AADSTS50105 - User not assigned to application

Causes:
  - Assignment required but user not assigned
  - Group membership not synchronized
  - Dynamic group not evaluating

Solutions:
  1. Assign user directly to app
  2. Add user to assigned group
  3. Or set Assignment required = No
  4. Check group membership sync
```

### Time Skew

```yaml
Error: Assertion conditions not valid / Time-based validation failed

Causes:
  - Clock difference > 5 minutes
  - NotBefore/NotOnOrAfter violated

Solutions:
  1. Sync server clocks with NTP
  2. Check time zone settings
  3. Verify both IdP and SP times
  4. Consider time skew tolerance settings
```

### Reply URL Mismatch (AADSTS50011)

```yaml
Error: AADSTS50011 - Reply URL does not match

Causes:
  - ACS URL not configured
  - URL mismatch (http vs https)
  - Trailing slash difference

Solutions:
  1. Add exact Reply URL to app config
  2. Check protocol (https required)
  3. Match URL exactly including path
  4. Add all environment URLs (dev, prod)
```

---

## Common OIDC Errors

### Error Reference

| Error | Description | Solution |
|-------|-------------|----------|
| invalid_client | App not registered | Verify client ID |
| invalid_grant | Code expired/used | Request new code |
| invalid_scope | Scope not permitted | Check API permissions |
| unauthorized_client | Wrong flow type | Check platform type |
| access_denied | User/admin denied | Check consent settings |

### Invalid Redirect URI

```yaml
Error: The redirect URI is not valid

Causes:
  - URI not registered
  - Protocol mismatch
  - localhost issues in production

Solutions:
  1. Add redirect URI to App Registration
  2. Use HTTPS for production
  3. Check for wildcards (not supported)
  4. Match exact URI including path
```

### Token Validation Failures

```yaml
Error: Token validation failed

Causes:
  - Wrong audience
  - Token expired
  - Invalid signature
  - Wrong issuer

Solutions:
  1. Verify aud claim matches client ID
  2. Check token expiration
  3. Validate using JWKS endpoint
  4. Confirm tenant ID in issuer
```

---

## Sign-in Logs Analysis

### Accessing Sign-in Logs

```
Microsoft Entra ID → Sign-in logs
→ Filter by application
```

### Key Fields to Review

| Field | Purpose |
|-------|---------|
| Status | Success/Failure |
| Sign-in error code | Specific error |
| Failure reason | Error description |
| Resource | Application accessed |
| Conditional Access | Policies applied |
| Authentication Details | MFA status |

### Filtering Sign-in Logs

```yaml
Useful Filters:
  - Application: Filter by app name
  - User: Specific user
  - Status: Failure only
  - Date: Time range
  - Client app: Browser, mobile, etc.
```

### KQL Queries

```kusto
// Failed SSO attempts for an app
SignInLogs
| where TimeGenerated > ago(24h)
| where AppDisplayName == "Salesforce"
| where ResultType != 0
| project TimeGenerated, UserPrincipalName,
          ResultType, ResultDescription,
          AuthenticationDetails
| order by TimeGenerated desc

// SSO errors by type
SignInLogs
| where TimeGenerated > ago(7d)
| where ResultType != 0
| summarize Count = count() by ResultType, ResultDescription
| order by Count desc

// Token issuance details
SignInLogs
| where TimeGenerated > ago(1d)
| where AppDisplayName == "My App"
| extend TokenType = tostring(AuthenticationDetails[0].authenticationMethod)
| project TimeGenerated, UserPrincipalName, TokenType, ResultType
```

---

## Systematic Troubleshooting

### Troubleshooting Checklist

```yaml
Step 1 - Verify User Assignment:
  □ User assigned to application?
  □ Group membership correct?
  □ Assignment required setting?

Step 2 - Check SSO Configuration:
  □ Correct Entity ID?
  □ Reply URL matches?
  □ Sign-on URL correct?
  □ Certificate valid?

Step 3 - Validate Claims:
  □ NameID configured correctly?
  □ Required claims present?
  □ Attribute mapping accurate?

Step 4 - Review Conditional Access:
  □ CA policies blocking?
  □ MFA required but not completed?
  □ Device compliance issues?

Step 5 - Examine Logs:
  □ Sign-in logs errors?
  □ SAML tracer captures?
  □ Application error logs?
```

### Diagnostic Workflow

```
1. Reproduce the issue
          |
2. Capture traffic (SAML tracer / DevTools)
          |
3. Check sign-in logs in Entra ID
          |
4. Identify error code/message
          |
5. Cross-reference with known issues
          |
6. Apply fix
          |
7. Test again
          |
8. Verify in logs
```

---

## Testing Best Practices

### Pre-Production Testing

```yaml
Test Scenarios:
  - New user SSO
  - Existing session SSO
  - Session expiry
  - Logout and re-login
  - Different browsers
  - Mobile devices
  - Claims verification
```

### Test Users

```yaml
Recommended:
  - Create dedicated test accounts
  - Include various user types
  - Test with/without MFA
  - Test assigned vs unassigned users

Avoid:
  - Testing with admin accounts only
  - Skipping guest user testing
  - Ignoring mobile scenarios
```

### Certificate Rotation Testing

```yaml
Before Rotation:
  1. Download new certificate
  2. Add to staging environment
  3. Test SSO with new cert
  4. Verify no issues

During Rotation:
  1. Update production app
  2. Set new certificate active
  3. Immediate testing
  4. Monitor sign-in logs

After Rotation:
  1. Remove old certificate
  2. Update documentation
  3. Set reminder for next rotation
```

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- Troubleshooting workflow
- Common error scenarios
- SAML tracer usage
- Log analysis process

---

## Key Takeaways

- Use built-in test feature for quick SAML validation
- SAML tracer extensions capture full authentication flow
- JWT.ms decodes and validates OIDC tokens
- Common errors have specific error codes and solutions
- Sign-in logs provide detailed failure information
- Systematic troubleshooting saves time
- Test thoroughly before and after changes

---

## Navigation

[7.6 SSO On-Premises](../7.6-sso-on-premises/README.md) | [7.8 Provisioning with SSO](../7.8-provisioning-sso/README.md)
