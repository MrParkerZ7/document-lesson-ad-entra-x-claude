# 10.2 Sign-in Logs

## Overview

Sign-in logs are the most frequently used logs in Microsoft Entra ID, providing detailed information about every authentication attempt. These logs are essential for troubleshooting access issues, detecting security threats, and understanding user behavior patterns.

## Learning Objectives

- Understand the different sign-in log categories
- Know the key fields available in sign-in logs
- Apply filters to find specific sign-in events
- Interpret common error codes and their resolutions
- Navigate sign-in log details for troubleshooting

---

## Sign-in Log Categories

Microsoft Entra ID captures four distinct types of sign-in events:

| Category | Description | Examples |
|----------|-------------|----------|
| **Interactive** | User directly authenticates with credentials | Portal login, MFA prompt, password change |
| **Non-interactive** | Background authentication without user action | Token refresh, silent SSO, OAuth refresh |
| **Service Principal** | Application authenticating with its own credentials | API calls, daemon apps, service-to-service |
| **Managed Identity** | Azure resource using system/user-assigned identity | VM accessing Key Vault, Function App calling SQL |

### Interactive User Sign-ins

```yaml
Interactive Sign-in Scenarios:
  Browser-based:
    - Office 365 portal login
    - Azure portal access
    - SaaS application login

  Desktop Applications:
    - Microsoft 365 apps (Word, Excel, Outlook)
    - Azure CLI login
    - Custom desktop applications

  Mobile Applications:
    - Outlook mobile
    - Teams mobile
    - Authenticator app prompts

Captured Events:
  - Initial authentication
  - MFA challenge and response
  - Password reset flows
  - Consent prompts
```

### Non-interactive User Sign-ins

```yaml
Non-interactive Sign-in Scenarios:
  Token Operations:
    - Access token refresh
    - Silent token acquisition
    - Primary Refresh Token (PRT) renewal

  SSO Operations:
    - Seamless SSO from domain-joined devices
    - Web session continuation
    - Cross-application SSO

High Volume:
  - Typically 10-100x more than interactive
  - Consider filtering in queries
  - May need sampling for large tenants
```

### Service Principal Sign-ins

```yaml
Service Principal Scenarios:
  Application Authentication:
    - Client credentials flow (client_secret)
    - Certificate-based authentication
    - Federated identity credentials

Common Use Cases:
  - Background job authentication
  - API-to-API calls
  - Automation scripts
  - CI/CD pipelines
```

### Managed Identity Sign-ins

```yaml
Managed Identity Scenarios:
  System-assigned:
    - Azure VM accessing Azure resources
    - App Service calling Key Vault
    - Logic Apps accessing Storage

  User-assigned:
    - Shared identity across resources
    - Cross-resource group access
    - Specific permission scoping
```

---

## Key Fields in Sign-in Logs

| Field | Description | Example Values |
|-------|-------------|----------------|
| **Date** | Timestamp of the sign-in attempt | 2024-01-15 14:32:45 UTC |
| **Request ID** | Unique identifier for support tickets | a1b2c3d4-e5f6-7890-... |
| **Correlation ID** | Links related events together | Same across token refresh chain |
| **User** | User principal name (UPN) | john.doe@contoso.com |
| **User ID** | Unique object ID | a1b2c3d4-e5f6-7890-abcd-... |
| **Application** | Target app display name | Microsoft Office 365 Portal |
| **Application ID** | App registration GUID | 00000003-0000-0ff1-ce00-... |
| **Status** | Result of the sign-in | Success, Failure, Interrupted |
| **Error Code** | Numeric error identifier | 50126, 53003, 0 (success) |
| **IP Address** | Client IP address | 192.168.1.100 |
| **Location** | Geo-location based on IP | Seattle, WA, United States |
| **Device** | Device info if available | DESKTOP-ABC123 |
| **Device State** | Compliance and join status | Compliant, Hybrid Azure AD joined |
| **Browser** | Browser/client app type | Chrome 120.0.0, Edge 121.0.0 |
| **Operating System** | Client OS | Windows 11, iOS 17.2, macOS 14 |
| **Client App** | Authentication client type | Browser, Mobile app, Desktop client |
| **MFA Result** | MFA status | Satisfied, Required, Not required |
| **Conditional Access** | CA policy results | Applied, Not applied, Failure |
| **Risk Level** | Identity Protection risk | None, Low, Medium, High |

---

## Filtering Options

### Common Filter Scenarios

```yaml
By Status:
  - Status: Success
  - Status: Failure
  - Status: Interrupted (user abandoned)

By User:
  - User: john.doe@contoso.com
  - User: contains "admin"
  - User ID: [specific GUID]

By Application:
  - Application: Microsoft Azure Management
  - Application: contains "SharePoint"
  - Application ID: [specific app ID]

By Time Range:
  - Last 24 hours
  - Last 7 days
  - Last 30 days
  - Custom range

By Client App:
  - Browser
  - Mobile Apps and Desktop clients
  - Exchange ActiveSync
  - Other clients (legacy auth)

By Location:
  - Country: United States
  - City: Seattle
  - IP Address: 192.168.1.0/24
```

### Advanced Filters

```yaml
Conditional Access Filters:
  - Policy applied: [Policy Name]
  - Grant control: Block, MFA, Compliant device
  - CA Status: Success, Failure, Not applied

Device Filters:
  - Device: Compliant
  - Device: Managed
  - Device: Hybrid Azure AD joined
  - Device: Azure AD registered

Risk Filters (P2 Required):
  - Sign-in risk level: High
  - User risk level: Medium
  - Risk state: At risk, Confirmed compromised

Authentication Filters:
  - Authentication requirement: MFA
  - MFA result: Satisfied, Denied
  - Auth method: Authenticator app, SMS, Phone call
```

---

## Common Error Codes and Resolutions

| Error Code | Error Name | Description | Resolution |
|------------|------------|-------------|------------|
| **50126** | InvalidUserNameOrPassword | Invalid username or password | Verify credentials, check account exists |
| **50053** | IdsLocked | Account locked due to failed attempts | Wait for lockout duration or admin unlock |
| **50055** | InvalidPasswordExpiredPassword | Password expired | User must reset password |
| **50057** | UserDisabled | Account is disabled | Enable account in Entra ID |
| **50058** | UserNotFound | User not found in directory | Verify UPN, check tenant |
| **50074** | StrongAuthRequired | MFA required but not completed | Complete MFA registration |
| **50076** | MFARequired | MFA challenge not satisfied | Re-authenticate with MFA |
| **50079** | MFARegistrationRequired | MFA registration needed | Register MFA methods |
| **50105** | EntitlementGrantsNotFound | User not assigned to application | Assign user to app |
| **53000** | DeviceNotCompliant | Device not meeting compliance | Update device, run sync |
| **53001** | DeviceNotDomainJoined | Device not domain-joined | Join device to domain/Entra ID |
| **53003** | BlockedByConditionalAccess | CA policy blocked access | Review CA policy conditions |
| **65001** | DelegationDoesNotExist | Consent not granted | Grant admin or user consent |
| **70001** | UnauthorizedClient | App not authorized | Check app registration |
| **70011** | InvalidScope | Invalid scope requested | Verify API permissions |
| **700016** | UnauthorizedClient_DoesNotMatchRequest | App not found in tenant | Register app in tenant |
| **7000218** | InvalidClient | Client secret expired | Rotate client secret |
| **AADSTS90072** | PassThroughUserMfaError | External MFA provider issue | Check external MFA config |
| **AADSTS90094** | AdminConsentRequired | Admin consent needed | Global admin must consent |

### Error Investigation Workflow

```yaml
Step 1 - Identify the Error:
  - Note the error code from sign-in log
  - Check the failure reason description
  - Note the correlation ID for related events

Step 2 - Check User State:
  - Is the account enabled?
  - Is the password expired?
  - Is MFA registered?
  - Is the user assigned to the app?

Step 3 - Check Device State:
  - Is the device registered?
  - Is the device compliant?
  - Is the device hybrid joined?

Step 4 - Check CA Policies:
  - Which policies evaluated?
  - Which policy blocked (if any)?
  - What grant controls required?
  - Use What If tool to simulate

Step 5 - Check Application:
  - Is the app registered?
  - Does user have access?
  - Are permissions consented?
  - Is client secret valid?
```

---

## Sign-in Log Details View

When you click on a sign-in event, you get a detailed view with multiple tabs:

### Basic Info Tab

```yaml
Basic Info:
  Date: 2024-01-15T14:32:45Z
  Request ID: a1b2c3d4-e5f6-7890-abcd-1234567890ab
  Correlation ID: b2c3d4e5-f6a7-8901-bcde-2345678901bc

User:
  User: john.doe@contoso.com
  User ID: c3d4e5f6-a7b8-9012-cdef-3456789012cd
  User type: Member

Application:
  Application: Microsoft Azure Portal
  Application ID: c44b4083-3bb0-49c1-b47d-974e53cbdf3c
  Resource: Windows Azure Service Management API
  Resource ID: 797f4846-ba00-4fd7-ba43-dac1f8f63013

Status:
  Status: Failure
  Error code: 53003
  Failure reason: Blocked by Conditional Access
```

### Location Tab

```yaml
Location Details:
  IP Address: 203.0.113.50
  City: Seattle
  State: Washington
  Country: United States
  Coordinates: 47.6062, -122.3321

Network:
  Autonomous System Number: AS12345
  ISP: Contoso ISP
```

### Device Info Tab

```yaml
Device Information:
  Device ID: d4e5f6a7-b8c9-0123-defa-4567890123de
  Display Name: DESKTOP-JOHN01
  Operating System: Windows 11
  Browser: Edge 121.0.2277.83

Device State:
  Registered: Yes
  Compliant: Yes
  Managed: Yes
  Join Type: Hybrid Azure AD joined
  Trust Type: Azure AD Joined
```

### Authentication Details Tab

```yaml
Authentication Methods:
  - Method: Password
    Result: Success
    Detail: Password validated

  - Method: Microsoft Authenticator (push notification)
    Result: Success
    Detail: Push notification approved

Authentication Requirement: Multi-factor authentication
MFA Result: MFA completed

Token Details:
  Access Token Issued: Yes
  Refresh Token Issued: Yes
  Token Lifetime: 1 hour
```

### Conditional Access Tab

```yaml
Policies Evaluated:

Policy: Require MFA for All Users
  Result: Success
  State: Enabled
  Grant Controls:
    - Required: Multi-factor authentication
    - Result: Satisfied
  Session Controls: None

Policy: Block Legacy Authentication
  Result: Not Applied
  Reason: Conditions not matched

Policy: Require Compliant Device for Admins
  Result: Not Applied
  Reason: User not in scope
```

---

## PowerShell Queries

```powershell
# Connect to Microsoft Graph
Connect-MgGraph -Scopes "AuditLog.Read.All"

# Get recent failed sign-ins
Get-MgAuditLogSignIn -Filter "status/errorCode ne 0" -Top 50 |
    Select-Object CreatedDateTime, UserPrincipalName, AppDisplayName,
                  @{N='ErrorCode';E={$_.Status.ErrorCode}},
                  @{N='FailureReason';E={$_.Status.FailureReason}}

# Get sign-ins for specific user
Get-MgAuditLogSignIn -Filter "userPrincipalName eq 'john.doe@contoso.com'" -Top 100

# Get sign-ins blocked by Conditional Access
Get-MgAuditLogSignIn -Filter "conditionalAccessStatus eq 'failure'" -Top 50

# Export sign-ins to CSV
$signIns = Get-MgAuditLogSignIn -Filter "createdDateTime ge 2024-01-01" -All
$signIns | Export-Csv "SignInLogs.csv" -NoTypeInformation
```

---

## KQL Queries for Log Analytics

```kusto
// Failed sign-ins in the last 24 hours
SigninLogs
| where TimeGenerated > ago(24h)
| where ResultType != 0
| project TimeGenerated, UserPrincipalName, AppDisplayName,
          ResultType, ResultDescription, IPAddress, Location
| order by TimeGenerated desc

// Sign-ins blocked by Conditional Access
SigninLogs
| where TimeGenerated > ago(7d)
| where ConditionalAccessStatus == "failure"
| mv-expand ConditionalAccessPolicies
| where ConditionalAccessPolicies.result == "failure"
| project TimeGenerated, UserPrincipalName, AppDisplayName,
          PolicyName = ConditionalAccessPolicies.displayName
| summarize Count = count() by PolicyName

// Top users with failed sign-ins
SigninLogs
| where TimeGenerated > ago(7d)
| where ResultType != 0
| summarize FailedCount = count() by UserPrincipalName
| order by FailedCount desc
| take 20

// Sign-ins from unusual locations
SigninLogs
| where TimeGenerated > ago(24h)
| where ResultType == 0
| summarize Countries = make_set(Location) by UserPrincipalName
| where array_length(Countries) > 1
```

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of sign-in log categories and key fields.

---

## Key Takeaways

1. Sign-in logs are divided into four categories: interactive, non-interactive, service principal, and managed identity
2. Key fields include user, application, status, IP address, location, device, and Conditional Access results
3. Error codes provide specific information for troubleshooting failed sign-ins
4. The details view offers comprehensive information across multiple tabs
5. Use filters effectively to find specific events in large log volumes

---

[<- 10.1 Monitoring Overview](../10.1-monitoring-overview/README.md) | [10.3 Audit Logs ->](../10.3-audit-logs/README.md)
