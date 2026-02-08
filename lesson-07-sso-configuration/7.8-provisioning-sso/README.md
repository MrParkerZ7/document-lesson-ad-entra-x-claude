# 7.8 Provisioning with SSO

## Overview

User provisioning automates the creation, management, and removal of user accounts in applications. When combined with SSO, provisioning ensures users can access applications seamlessly without manual account creation. SCIM (System for Cross-domain Identity Management) is the standard protocol for automatic provisioning.

## Learning Objectives

- Understand the relationship between SSO and provisioning
- Configure SCIM-based automatic provisioning
- Set up attribute mappings
- Configure provisioning scopes and filters
- Troubleshoot provisioning issues

---

## SSO and Provisioning Relationship

### Why Provisioning Matters for SSO

```yaml
SSO Only:
  - User authenticates via Entra ID
  - App receives identity token
  - App may not have user account
  - Manual account creation needed

SSO + Provisioning:
  - User assigned in Entra ID
  - User automatically created in app
  - Attributes synced to app
  - User authenticates via SSO
  - Account already exists - seamless access
```

### SSO + Provisioning Flow

```
1. User assigned to app in Entra ID
          |
2. Provisioning service detects assignment
          |
3. SCIM call creates user in application
          |
4. User accesses application
          |
5. SSO authenticates user
          |
6. App matches SSO identity to provisioned account
          |
7. Access granted
```

---

## Provisioning Modes

### Available Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| Automatic | SCIM-based sync | Apps with SCIM support |
| Manual | Admin creates accounts | No SCIM support |
| None | No provisioning | SSO only |

### Automatic Provisioning Benefits

```yaml
Benefits:
  - No manual account creation
  - Consistent attribute mapping
  - Automatic deprovisioning
  - Audit trail
  - Reduced errors
  - Time savings
```

---

## SCIM Overview

### What is SCIM?

SCIM (System for Cross-domain Identity Management) is a standard protocol for automating identity management:

| Aspect | Description |
|--------|-------------|
| Protocol | REST-based HTTP |
| Format | JSON |
| Version | SCIM 2.0 |
| Operations | Create, Read, Update, Delete |
| Resources | Users, Groups |

### SCIM Endpoints

```yaml
Base URL: https://app.example.com/scim/v2

Endpoints:
  /Users     - User management
  /Groups    - Group management
  /Schemas   - Schema discovery
  /ResourceTypes - Resource type discovery
  /ServiceProviderConfig - Configuration

Operations:
  POST /Users     - Create user
  GET /Users      - List users
  GET /Users/{id} - Get specific user
  PATCH /Users/{id} - Update user
  DELETE /Users/{id} - Delete user
```

### SCIM User Resource

```json
{
  "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
  "userName": "john.doe@contoso.com",
  "name": {
    "givenName": "John",
    "familyName": "Doe"
  },
  "emails": [
    {
      "value": "john.doe@contoso.com",
      "type": "work",
      "primary": true
    }
  ],
  "displayName": "John Doe",
  "active": true,
  "externalId": "user-object-id"
}
```

---

## Configuring Provisioning

### Portal Navigation

```
Enterprise applications → [App] → Provisioning
```

### Step-by-Step Configuration

```yaml
Step 1 - Set Provisioning Mode:
  Provisioning Mode: Automatic

Step 2 - Configure Admin Credentials:
  Tenant URL: https://app.example.com/scim/v2
  Secret Token: [API token from application]

Step 3 - Test Connection:
  Click "Test Connection"
  Verify: "Credentials authorized successfully"

Step 4 - Configure Mappings:
  - User mappings
  - Group mappings (if supported)

Step 5 - Set Scope:
  - All users and groups
  - Assigned users and groups only

Step 6 - Enable Provisioning:
  Provisioning Status: On
```

### Credential Configuration

```yaml
Tenant URL:
  - SCIM endpoint provided by application
  - Include /scim/v2 or as specified
  - Use HTTPS

Secret Token:
  - API token or OAuth token
  - Generated in target application
  - Store securely
  - May require renewal
```

---

## Attribute Mappings

### Default User Mappings

| Entra ID Attribute | SCIM Attribute | Description |
|--------------------|----------------|-------------|
| userPrincipalName | userName | Primary identifier |
| mail | emails[type eq "work"].value | Email address |
| displayName | displayName | Display name |
| givenName | name.givenName | First name |
| surname | name.familyName | Last name |
| objectId | externalId | External reference |
| accountEnabled | active | Account status |

### Customizing Mappings

```
Provisioning → Mappings → Provision Azure Active Directory Users
```

```yaml
Mapping Options:
  - Source attribute: Entra ID attribute
  - Target attribute: SCIM attribute
  - Matching precedence: For matching existing users
  - Apply mapping: Always / Object creation / Object update
  - Default value: If source is null
```

### Expression-Based Mappings

```yaml
Examples:

Join two attributes:
  [givenName] & " " & [surname]
  Result: "John Doe"

Conditional mapping:
  IIF([department]="IT", "Technology", [department])
  Result: "Technology" if IT, otherwise department value

Substring extraction:
  Left([userPrincipalName], InStr([userPrincipalName], "@")-1)
  Result: Username without domain
```

### Common Expressions

| Expression | Description | Example Result |
|------------|-------------|----------------|
| `ToLower([mail])` | Lowercase email | john@contoso.com |
| `Replace([upn], "@", "_")` | Replace @ with _ | john_contoso.com |
| `SelectUniqueValue()` | Generate unique value | john.doe1 |
| `Switch([country], ...)` | Map values | USA → US |

---

## Provisioning Scope

### Scope Options

| Option | Description |
|--------|-------------|
| Sync all users and groups | Provisions everyone |
| Sync only assigned | Only assigned users/groups |

### Scoping Filters

```
Provisioning → Mappings → Source Object Scope
```

```yaml
Filter Examples:

Only active users:
  Attribute: accountEnabled
  Operator: EQUALS
  Value: true

Specific department:
  Attribute: department
  Operator: EQUALS
  Value: "Sales"

Multiple conditions (AND):
  - department EQUALS "IT"
  - accountEnabled EQUALS true

Users with email:
  Attribute: mail
  Operator: IS NOT NULL
```

---

## Provisioning Cycles

### Initial Cycle

```yaml
Initial Cycle:
  - Scans all users in scope
  - Creates users in target app
  - May take hours for large directories
  - Run once when enabled

Behavior:
  - Matches existing users by matching attribute
  - Creates new users
  - Updates existing matches
  - Marks orphans for attention
```

### Incremental Cycles

```yaml
Incremental Cycle:
  - Runs every 40 minutes
  - Processes changes only
  - Faster than initial cycle
  - Continues until disabled

Triggers:
  - User attribute changes
  - New user assignments
  - User unassignments
  - Manual sync request
```

### Manual Sync

```
Provisioning → Restart provisioning
```

```yaml
Use Manual Sync When:
  - Testing configuration
  - After mapping changes
  - Urgent user updates needed
  - Troubleshooting issues
```

---

## Deprovisioning

### What Happens on Unassignment

| Action | Result |
|--------|--------|
| Disable user | SCIM PATCH sets active=false |
| Delete user | SCIM DELETE removes user |
| Do nothing | User remains in app |

### Configuring Deprovisioning

```
Provisioning → Mappings → Target Object Actions
```

```yaml
Options:
  When user goes out of scope:
    - Disable (recommended)
    - Delete

  Delete behavior:
    - Soft delete (if supported)
    - Hard delete
```

### Accidental Deletion Prevention

```yaml
Settings:
  Accidental deletion threshold: 500

  Purpose:
    - Prevents mass deletion
    - Triggers if > threshold users affected
    - Pauses provisioning
    - Requires admin review
```

---

## Provisioning with SSO Types

### SAML + Provisioning

```yaml
Configuration:
  SSO: SAML
  Provisioning: SCIM

User Matching:
  - SCIM creates user with email
  - SAML NameID uses same email
  - App matches by email

Important:
  - NameID must match provisioned user
  - Use same identifier attribute
```

### OIDC + Provisioning

```yaml
Configuration:
  SSO: OIDC
  Provisioning: SCIM

User Matching:
  - SCIM creates user with objectId as externalId
  - OIDC token contains sub claim (objectId)
  - App matches by externalId

Important:
  - externalId links Entra user to app user
  - Consistent identifier critical
```

---

## Troubleshooting Provisioning

### Provisioning Logs

```
Enterprise applications → [App] → Provisioning logs
```

### Log Information

| Field | Description |
|-------|-------------|
| Date | When action occurred |
| Action | Create, Update, Delete, Other |
| Status | Success, Failure, Skipped |
| Source User | Entra ID user |
| Target User | App user identifier |
| Modified Properties | Changed attributes |

### Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| InvalidCredentials | Token invalid | Regenerate API token |
| DuplicateUser | User exists in app | Set matching attribute |
| MissingRequiredAttribute | Required field null | Add default value or mapping |
| QuotaExceeded | Rate limiting | Wait and retry |
| ServiceUnavailable | App endpoint down | Check app status |

### Troubleshooting Checklist

```yaml
Connection Issues:
  □ Test Connection successful?
  □ Tenant URL correct?
  □ Token valid and not expired?
  □ SCIM endpoint accessible?

User Not Provisioning:
  □ User assigned to app?
  □ User in scope?
  □ Scoping filters correct?
  □ Provisioning enabled?

Attribute Not Syncing:
  □ Mapping configured?
  □ Source attribute has value?
  □ Mapping applied correctly?
  □ Transform working?

Deprovisioning Not Working:
  □ Target object actions set?
  □ User actually unassigned?
  □ App supports disable/delete?
```

### KQL: Provisioning Analysis

```kusto
// Failed provisioning operations
AuditLogs
| where TimeGenerated > ago(24h)
| where OperationName contains "Provision"
| where ResultDescription contains "Error"
| project TimeGenerated, OperationName, ResultDescription,
          TargetResources[0].displayName
| order by TimeGenerated desc

// Provisioning success rate
AuditLogs
| where TimeGenerated > ago(7d)
| where OperationName contains "Provision"
| summarize
    Success = countif(Result == "success"),
    Failure = countif(Result == "failure")
| extend SuccessRate = Success * 100.0 / (Success + Failure)
```

---

## Best Practices

### Configuration Best Practices

```yaml
Mapping:
  - Map all required app attributes
  - Use externalId for matching
  - Test with small user set first
  - Document custom expressions

Scoping:
  - Start with assigned users only
  - Use scoping filters cautiously
  - Test filters before production

Maintenance:
  - Monitor provisioning logs weekly
  - Review quarantine status
  - Update tokens before expiry
  - Test after app updates
```

### Performance Considerations

```yaml
Large Directories:
  - Initial sync may take hours
  - Limit scope if possible
  - Monitor cycle duration
  - Consider off-peak provisioning

Rate Limiting:
  - Apps may throttle requests
  - Entra respects rate limits
  - Large changes may queue
```

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- Provisioning and SSO integration flow
- SCIM communication
- Attribute mapping process
- Provisioning cycle

---

## Key Takeaways

- Provisioning creates user accounts before SSO access
- SCIM is the standard protocol for automatic provisioning
- Attribute mappings connect Entra attributes to app fields
- Scoping filters control which users are provisioned
- Initial cycle syncs all users; incremental syncs changes
- Deprovisioning can disable or delete users
- Matching attributes link SSO identity to provisioned account
- Monitor logs and quarantine for issues

---

## Navigation

[7.7 Testing & Troubleshooting](../7.7-testing-troubleshooting/README.md) | [7.9 SSO Best Practices](../7.9-sso-best-practices/README.md)
