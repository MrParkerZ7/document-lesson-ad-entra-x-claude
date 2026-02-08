# 10.8 Troubleshooting Issues

## Overview

Effective troubleshooting is essential for maintaining a healthy identity environment. This sub-lesson covers common issues encountered in Microsoft Entra ID, including authentication failures, Conditional Access problems, and synchronization issues. You will learn systematic approaches to diagnose and resolve these issues using built-in tools and log analysis.

## Learning Objectives

- Diagnose and resolve common authentication failures
- Troubleshoot Conditional Access policy issues
- Identify and fix synchronization problems in hybrid environments
- Use the What If tool for policy simulation
- Apply a structured troubleshooting decision tree

---

## Authentication Failures

### Issue: User Cannot Sign In

Authentication failures are the most common support requests. Follow this systematic approach:

#### Error Code Reference

| Error Code | Description | Common Cause | Resolution |
|------------|-------------|--------------|------------|
| 50126 | Invalid username or password | Wrong credentials | Verify username, reset password |
| 50053 | Account is locked | Too many failed attempts | Wait for unlock or admin reset |
| 50057 | Account is disabled | Account disabled in Entra ID | Enable the account |
| 50055 | Password expired | Password expiration policy | Reset password |
| 50056 | Invalid or null password | No password hash | Check password sync or reset |
| 50076 | MFA required but not completed | User cancelled MFA | Complete MFA verification |
| 50074 | Strong authentication required | MFA required by policy | Complete MFA setup |
| 65001 | Consent not granted | App requires consent | Grant admin or user consent |
| 70001 | Application not found | App deleted or misconfigured | Verify app registration |
| 700016 | Application not found in tenant | Wrong tenant ID | Check tenant configuration |

#### Diagnostic Steps

```yaml
Step 1 - Gather Information:
  - User principal name
  - Time of failed attempt
  - Application being accessed
  - Device and location
  - Error message shown to user

Step 2 - Check Sign-in Logs:
  Portal: Microsoft Entra ID → Monitoring → Sign-in logs
  Filter by:
    - User principal name
    - Date/time range
    - Status: Failure

Step 3 - Analyze Error Details:
  Click on the failed sign-in entry:
    - Error code and failure reason
    - Conditional Access results
    - Authentication details
    - Device information

Step 4 - Check User Account:
  Portal: Microsoft Entra ID → Users → [User]
    - Account enabled?
    - Password last changed?
    - MFA registered?
    - Licenses assigned?

Step 5 - Verify Application:
  Portal: Microsoft Entra ID → Enterprise applications → [App]
    - User assigned to app?
    - App enabled?
    - Consent granted?
```

#### KQL Query for Failed Sign-ins

```kusto
SigninLogs
| where TimeGenerated > ago(24h)
| where UserPrincipalName == "user@contoso.com"
| where ResultType != 0
| project
    TimeGenerated,
    ResultType,
    ResultDescription,
    AppDisplayName,
    IPAddress,
    DeviceDetail,
    ConditionalAccessStatus
| order by TimeGenerated desc
```

---

### Issue: MFA Not Working

MFA issues often stem from registration problems or device/app configuration.

#### Common MFA Issues

| Symptom | Possible Cause | Resolution |
|---------|----------------|------------|
| No MFA prompt appears | MFA not required by policy | Check CA policies, MFA settings |
| Authenticator app not receiving codes | Time sync issue | Sync device time, re-add account |
| Push notification not received | Network/notification issue | Check internet, app permissions |
| SMS code not received | Wrong phone number | Verify registered phone |
| OATH token not working | Clock drift | Sync hardware token time |
| MFA keeps prompting | Token not remembered | Check "remember MFA" settings |

#### MFA Troubleshooting Steps

```yaml
Step 1 - Check Registration Status:
  Portal: Microsoft Entra ID → Users → [User] → Authentication methods
  Verify:
    - Methods registered (phone, authenticator, etc.)
    - Default method set
    - Last used dates

Step 2 - Review MFA Policy:
  Check which MFA mechanism applies:
    - Conditional Access (recommended)
    - Per-user MFA (legacy)
    - Security defaults

Step 3 - Test Authentication:
  Options:
    - Have user clear browser cache/cookies
    - Test from different device
    - Test from different network

Step 4 - Reset MFA Registration:
  If needed, reset MFA:
    PowerShell:
      Revoke-MgUserAuthenticationMethodPhoneAuthenticationMethodPhoneNumber
    Portal:
      Users → [User] → Authentication methods → Require re-register MFA
```

#### PowerShell: Check User MFA Methods

```powershell
# Connect to Microsoft Graph
Connect-MgGraph -Scopes "UserAuthenticationMethod.Read.All"

# Get user's registered authentication methods
$userId = "user@contoso.com"
Get-MgUserAuthenticationMethod -UserId $userId |
    Select-Object @{N='Type';E={$_.AdditionalProperties.'@odata.type'}}, Id

# Get specific method details
Get-MgUserAuthenticationPhoneMethod -UserId $userId
Get-MgUserAuthenticationMicrosoftAuthenticatorMethod -UserId $userId
```

---

### Issue: SSO Not Working

Single Sign-On issues vary by SSO type (SAML, OIDC, Seamless SSO).

#### SAML SSO Troubleshooting

| Symptom | Possible Cause | Resolution |
|---------|----------------|------------|
| "AADSTS750054" error | Reply URL mismatch | Verify ACS URL in app config |
| Certificate error | Expired signing certificate | Renew SAML certificate |
| Attribute claim missing | Claim not configured | Add claim mapping |
| User not recognized | Identifier claim wrong | Check NameID format/source |

#### SAML Diagnostic Steps

```yaml
Step 1 - Capture SAML Traffic:
  Tools:
    - Browser extension (SAML tracer/debugger)
    - Fiddler
    - Developer tools (F12)

Step 2 - Check SAML Response:
  Verify:
    - Signature valid
    - Certificate matches
    - Audience restriction correct
    - Subject NameID format
    - Attribute values

Step 3 - Compare Configuration:
  Entra ID side:
    - Reply URL (ACS)
    - Identifier (Entity ID)
    - Signing certificate
    - Claims mapping

  Application side:
    - Entity ID
    - IdP metadata/certificate
    - Attribute mapping
```

#### Seamless SSO Troubleshooting

```yaml
Prerequisites Check:
  - Device domain-joined to on-premises AD
  - User on corporate network
  - Browser supports (Edge, Chrome, Firefox)
  - GPO applied for AZUREADSSO URL

Verification Steps:
  1. Check Group Policy:
     GPO: User Configuration → Admin Templates → Windows Components
          → Internet Explorer → Internet Control Panel → Security Page
          → Site to Zone Assignment List
     Value: https://autologon.microsoftazuread-sso.com → Intranet zone

  2. Verify Kerberos ticket:
     Command: klist
     Look for: AZUREADSSO account ticket

  3. Check DNS resolution:
     nslookup autologon.microsoftazuread-sso.com
```

---

## Conditional Access Issues

### Issue: Unexpected Block

Users being blocked when they should have access is a common CA issue.

#### Diagnostic Process

```yaml
Step 1 - Check Sign-in Log:
  Portal: Sign-in logs → [Failed entry] → Conditional Access tab
  Review:
    - Which policy blocked the user
    - What grant controls failed
    - What conditions matched

Step 2 - Analyze Policy Conditions:
  For the blocking policy, verify:
    - User/group scope (inclusions/exclusions)
    - Cloud apps scope
    - Conditions: locations, devices, client apps, risk
    - Grant controls required

Step 3 - Check User Context:
  Verify user's actual state:
    - Is device compliant?
    - Is device hybrid joined?
    - What location did they sign in from?
    - What client app was used?
    - What was the risk level?
```

#### Common Block Causes

| Cause | Sign | Resolution |
|-------|------|------------|
| Location not trusted | Location shows unknown/non-trusted | Add location to named locations |
| Device not compliant | Device compliance: Not compliant | Fix compliance issues in Intune |
| Device not Entra joined | Join status: Not joined | Join or register device |
| Wrong client app | Client app: Other clients | Use modern auth client |
| MFA not registered | MFA required, not completed | Register MFA methods |
| Risk too high | Risk level: High | Remediate risk, secure account |

#### KQL: Conditional Access Failures

```kusto
SigninLogs
| where TimeGenerated > ago(24h)
| where ConditionalAccessStatus == "failure"
| mv-expand ConditionalAccessPolicies
| where ConditionalAccessPolicies.result == "failure"
| project
    TimeGenerated,
    UserPrincipalName,
    AppDisplayName,
    PolicyName = tostring(ConditionalAccessPolicies.displayName),
    GrantControls = tostring(ConditionalAccessPolicies.enforcedGrantControls),
    SessionControls = tostring(ConditionalAccessPolicies.enforcedSessionControls),
    IPAddress,
    DeviceDetail
| order by TimeGenerated desc
```

---

### Issue: Policy Not Applying

When a CA policy should apply but is not being enforced.

#### Verification Checklist

```yaml
Policy Configuration:
  □ Policy is enabled (not Report-only or Off)
  □ User is in included users/groups
  □ User is NOT in excluded users/groups
  □ Application is in cloud apps scope
  □ All conditions are correctly configured

User Context:
  □ User is signing into the correct app
  □ User is using expected client app type
  □ User is from expected location
  □ Sign-in meets all condition criteria

Policy Conflicts:
  □ No higher-priority policy granting access
  □ No exclusion group membership
  □ No service principal exclusion
```

#### Policy Evaluation Order

```yaml
Evaluation Logic:
  1. All policies that match conditions are evaluated
  2. If ANY policy blocks → Access denied
  3. If policies require controls → All controls must be satisfied
  4. Controls from all matching policies are combined

Priority Considerations:
  - Exclude takes precedence over include
  - Block takes precedence over grant
  - Most restrictive control wins
```

---

## Using the What If Tool

The What If tool simulates Conditional Access policy evaluation without actual sign-in.

### Accessing What If

```
Microsoft Entra ID → Protect & Secure → Conditional Access → What If
```

### What If Parameters

| Parameter | Description | Required |
|-----------|-------------|----------|
| User | User to simulate | Yes |
| Cloud apps | Application(s) to test | Yes (default: All) |
| IP address | Simulate from location | No |
| Country | Simulate from country | No |
| Device platform | OS type | No |
| Device state | Joined/compliant status | No |
| Client app | Browser/app type | No |
| Sign-in risk | Simulated risk level | No |
| User risk | Simulated user risk | No |

### What If Analysis

```yaml
Running a Simulation:
  1. Select the user to test
  2. Select target application
  3. Add relevant conditions (location, device, etc.)
  4. Click "What If"

Understanding Results:
  Policies that will apply:
    - Shows which policies match
    - Lists required grant controls
    - Shows session controls

  Policies that will not apply:
    - Lists non-matching policies
    - Shows why they don't match (condition not met)

  Report-only policies:
    - Shows what would happen if enabled
```

### Common What If Scenarios

```yaml
Scenario 1 - Verify MFA Requirement:
  User: John@contoso.com
  Cloud apps: Office 365
  Device platform: Windows
  Device state: Not compliant
  Expected: MFA policy applies

Scenario 2 - Test Location-Based Access:
  User: Jane@contoso.com
  Cloud apps: Azure Management
  IP address: [External IP]
  Expected: Block if not from trusted location

Scenario 3 - Verify Device Compliance:
  User: Admin@contoso.com
  Cloud apps: Microsoft Admin Portals
  Device state: Compliant, Hybrid Azure AD joined
  Expected: Access granted
```

---

## Sync Issues (Hybrid Environments)

### Issue: User Not Syncing

For hybrid environments using Azure AD Connect or cloud sync.

#### Diagnostic Steps

```yaml
Step 1 - Check Azure AD Connect Status:
  Portal: Microsoft Entra ID → Microsoft Entra Connect → Connect Sync
  Verify:
    - Last sync time
    - Sync status (healthy/error)
    - Export errors

Step 2 - Check Object in Scope:
  On-premises AD:
    - Is user in synced OU?
    - Does user have required attributes?
    - Is user filtered by attribute?

Step 3 - Check Connector Space:
  PowerShell (on AAD Connect server):
    Get-ADSyncCSObject -DistinguishedName "CN=User,OU=Users,DC=contoso,DC=com" `
        -ConnectorName "contoso.com"

Step 4 - Check Sync Errors:
  Portal: Microsoft Entra ID → Microsoft Entra Connect → Connect Health
  Look for:
    - Object-specific errors
    - Attribute sync errors
    - Duplicate attribute conflicts
```

#### Common Sync Issues

| Issue | Cause | Resolution |
|-------|-------|------------|
| User not appearing | Not in sync scope | Add OU to sync, check filtering |
| Attribute not syncing | Not in attribute flow | Check sync rules, add attribute |
| Duplicate object | ProxyAddress conflict | Resolve duplicate values |
| Export error | Invalid attribute value | Fix source attribute format |
| Object deleted unexpectedly | Accidental removal from scope | Check filtering, restore if needed |

#### PowerShell: Sync Diagnostics

```powershell
# On Azure AD Connect server

# Check sync status
Get-ADSyncScheduler

# Force sync cycle
Start-ADSyncSyncCycle -PolicyType Delta

# Preview sync for specific object
$preview = Invoke-ADSyncCSObjectSearch -DistinguishedName "CN=User,OU=Users,DC=contoso,DC=com"

# Check for sync errors
Get-ADSyncSyncError

# Run sync diagnostics
Invoke-ADSyncDiagnostics -PasswordSync
```

---

### Issue: Password Not Syncing

Password hash synchronization (PHS) issues.

#### PHS Troubleshooting

```yaml
Step 1 - Verify PHS Enabled:
  Azure AD Connect wizard → Optional features
  Confirm: Password hash synchronization enabled

Step 2 - Check Password Sync Status:
  PowerShell:
    Invoke-ADSyncDiagnostics -PasswordSync

Step 3 - Review Event Logs:
  On AAD Connect server:
    Event Viewer → Applications and Services Logs →
      Microsoft → AzureADConnect → PasswordSync

Step 4 - Check Specific User:
  PowerShell:
    Get-ADSyncToolsAadObject -DistinguishedName "CN=User..." |
      Select-Object PasswordHash, PasswordLastChanged
```

#### Common PHS Issues

| Issue | Cause | Resolution |
|-------|-------|------------|
| No password sync | PHS not enabled | Enable in AAD Connect wizard |
| Old password works | Sync delay | Wait for sync or force sync |
| Password sync errors | Permissions issue | Verify service account permissions |
| Password writeback fail | Network/permission | Check writeback agent connectivity |

---

## Troubleshooting Decision Tree

Use this decision tree for systematic troubleshooting:

```
User Cannot Access Resource
│
├─► Is user authenticated?
│   │
│   ├─ NO → Check Sign-in Logs
│   │   │
│   │   ├─ Error code 50126? → Credential issue → Reset password
│   │   ├─ Error code 50053? → Account locked → Unlock or wait
│   │   ├─ Error code 50057? → Account disabled → Enable account
│   │   ├─ Error code 50074/50076? → MFA issue → Check MFA registration
│   │   ├─ Error code 53003? → CA blocked → Check CA policies
│   │   └─ Other error? → Research error code → Apply fix
│   │
│   └─ YES → Check authorization
│       │
│       ├─► Is CA blocking?
│       │   │
│       │   ├─ YES → Use What If tool
│       │   │   │
│       │   │   ├─ Device not compliant? → Fix compliance
│       │   │   ├─ Wrong location? → Update named locations
│       │   │   ├─ MFA required? → Complete MFA
│       │   │   └─ Risk too high? → Remediate risk
│       │   │
│       │   └─ NO → Check app assignment
│       │       │
│       │       ├─ User assigned? → Grant access
│       │       ├─ Consent required? → Grant consent
│       │       └─ License required? → Assign license
│       │
│       └─► For Hybrid Users
│           │
│           ├─ User not in Entra ID? → Check sync scope/status
│           ├─ Wrong attributes? → Check attribute flow
│           └─ Password not working? → Check password sync
```

---

## Troubleshooting Tools Summary

| Tool | Purpose | Access |
|------|---------|--------|
| Sign-in Logs | View authentication attempts | Entra ID → Monitoring |
| Audit Logs | View directory changes | Entra ID → Monitoring |
| What If | Simulate CA policies | Conditional Access → What If |
| Log Analytics | Advanced log queries | Azure Monitor |
| Connect Health | Sync monitoring | Entra ID → Connect Health |
| Authentication Methods Activity | MFA usage | Entra ID → Monitoring |
| Diagnostic Settings | Log export config | Entra ID → Monitoring |

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual troubleshooting decision tree and workflow.

---

## Key Takeaways

1. **Systematic Approach**: Always follow a structured diagnostic process starting with sign-in logs
2. **Error Codes Matter**: Learn common error codes for quick identification of issues
3. **What If Tool**: Use this tool extensively to simulate and validate Conditional Access behavior
4. **Check All Layers**: Authentication issues can stem from user, device, policy, or sync problems
5. **Document Resolutions**: Build a knowledge base of common issues and their resolutions

---

## Navigation

[← 10.7 Alerts & Notifications](../10.7-alerts-notifications/README.md) | [10.9 Health & Support →](../10.9-health-status-support/README.md)
