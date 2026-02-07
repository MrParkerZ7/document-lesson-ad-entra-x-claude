# 5.7 Troubleshooting

## Overview

When users experience access issues, Conditional Access policies are often involved. This lesson covers how to diagnose problems, use sign-in logs effectively, and resolve common Conditional Access issues.

## Learning Objectives

- Use sign-in logs to diagnose access problems
- Understand common Conditional Access errors
- Troubleshoot unexpected blocks and MFA loops
- Maintain emergency access accounts
- Follow a systematic debugging process

---

## Sign-in Logs

### Accessing Sign-in Logs

```
Microsoft Entra ID → Sign-in logs
```

### Key Information in Sign-in Logs

| Field | Description |
|-------|-------------|
| **Date/Time** | When sign-in occurred |
| **User** | Who attempted sign-in |
| **Application** | Target application |
| **Status** | Success, Failure, Interrupted |
| **Conditional Access** | Policies evaluated |
| **IP address** | Source IP |
| **Location** | Geographic location |
| **Device info** | Platform, compliance, trust type |
| **Client app** | Browser, mobile app, desktop |

### Filtering Sign-in Logs

Common filters:
```
Status: Failure
Conditional Access: Failure
User: specific-user@contoso.com
Application: specific-app
Date range: Last 24 hours
```

### Conditional Access Tab

For each sign-in, the Conditional Access tab shows:
- Which policies were evaluated
- Result of each policy (Success/Failure/Not applied)
- Grant controls applied
- Session controls applied

---

## Common Error Messages

### Error: AADSTS53003

**Message**: Access has been blocked by Conditional Access policies.

**Causes**:
- User blocked by policy
- Device not compliant
- Required controls not satisfied

**Resolution**:
1. Check sign-in logs for specific policy
2. Verify user group membership
3. Check device compliance status
4. Review policy exclusions

### Error: AADSTS50076

**Message**: MFA required but not completed.

**Causes**:
- MFA policy applied
- User interrupted MFA flow
- MFA methods not registered

**Resolution**:
1. User completes MFA
2. Verify MFA methods registered
3. Check for MFA service issues

### Error: AADSTS50105

**Message**: User not assigned to application.

**Causes**:
- Application requires assignment
- User not in assigned group

**Resolution**:
1. Add user to application
2. Or disable "User assignment required"

### Error: AADSTS7000218

**Message**: Client assertion contains invalid signature.

**Causes**:
- Certificate-based auth issue
- Token signing problem

**Resolution**:
1. Verify certificate validity
2. Check certificate configuration

---

## Common Issues

### Issue: Unexpected Block

**Symptoms**: User blocked but shouldn't be

**Debugging Steps**:
1. Find sign-in in logs
2. Check Conditional Access tab
3. Identify blocking policy
4. Verify:
   - User group membership
   - Device compliance
   - Location matching
   - Client app type

**Common Causes**:
- User added to included group
- Device became non-compliant
- Location changed
- Using legacy client

### Issue: MFA Loop

**Symptoms**: User prompted for MFA repeatedly

**Debugging Steps**:
1. Check if multiple policies require MFA
2. Verify token refresh settings
3. Check sign-in frequency settings
4. Review session controls

**Common Causes**:
- Conflicting policies
- Very short sign-in frequency
- Session not persisting
- CAE triggering re-auth

### Issue: Complete Lockout

**Symptoms**: All users or admins blocked

**Immediate Actions**:
1. Use emergency access account
2. Sign in to Azure portal
3. Disable problematic policy
4. Investigate root cause

**Prevention**:
- Always exclude emergency accounts
- Test policies in Report-only
- Use What If before enabling

### Issue: Guest Users Blocked

**Symptoms**: B2B guests can't access

**Debugging Steps**:
1. Check guest-specific policies
2. Verify guest MFA capabilities
3. Check device compliance for guests
4. Review cross-tenant settings

**Common Causes**:
- Device compliance required (guest devices not enrolled)
- MFA required but guest can't use MFA
- Location policy blocks guest's country

---

## Debugging Process

### Step 1: Gather Information

```
Questions to ask:
- Who is affected? (specific user, group, everyone)
- What are they trying to access?
- When did it start?
- What changed recently?
- What device/location are they using?
- What error message do they see?
```

### Step 2: Find the Sign-in

```
Sign-in logs → Filter:
- User: affected user
- Status: Failure
- Time range: When issue occurred
```

### Step 3: Analyze Conditional Access

```
Open sign-in → Conditional Access tab:
- Which policies applied?
- Which policy caused failure?
- What control was required?
```

### Step 4: Verify Conditions

```
Check:
- User group membership
- Device compliance (Intune → Devices)
- Location (compare IP to named locations)
- Client app type
- Risk level (if applicable)
```

### Step 5: Test with What If

```
Conditional Access → What If:
- Simulate the exact scenario
- Compare with actual sign-in
- Identify discrepancies
```

### Step 6: Resolve

Options:
- Add user to exclusion group
- Fix device compliance
- Adjust policy conditions
- Add named location
- Grant temporary access (TAP)

---

## Emergency Access Accounts

### Purpose

Break-glass accounts ensure you can always access the tenant, even if Conditional Access blocks all other access.

### Configuration

```yaml
Account: emergency-admin@contoso.com

Properties:
  - Strong, unique password (30+ characters)
  - Stored securely (safe, password manager)
  - No MFA required
  - Excluded from ALL Conditional Access policies
  - Excluded from PIM (always active Global Admin)
  - Monitored for any usage

Security:
  - Usage alerts configured
  - Password split between two people (optional)
  - Tested quarterly
```

### Monitoring Emergency Accounts

```powershell
# Get sign-ins for emergency accounts
Get-MgAuditLogSignIn -Filter "userPrincipalName eq 'emergency-admin@contoso.com'" |
    Select-Object CreatedDateTime, Status, IpAddress, Location

# Create alert in Azure Monitor for any sign-in
```

---

## PowerShell Troubleshooting

### Get Recent Failed Sign-ins

```powershell
Connect-MgGraph -Scopes "AuditLog.Read.All"

Get-MgAuditLogSignIn -Filter "status/errorCode ne 0" -Top 50 |
    Select-Object UserPrincipalName, AppDisplayName,
                  @{N='Error';E={$_.Status.ErrorCode}},
                  @{N='Reason';E={$_.Status.FailureReason}},
                  CreatedDateTime
```

### Get Conditional Access Failures

```powershell
Get-MgAuditLogSignIn -Filter "conditionalAccessStatus eq 'failure'" -Top 50 |
    ForEach-Object {
        [PSCustomObject]@{
            User = $_.UserPrincipalName
            App = $_.AppDisplayName
            Time = $_.CreatedDateTime
            Policies = ($_.AppliedConditionalAccessPolicies |
                Where-Object {$_.Result -eq 'failure'} |
                Select-Object -ExpandProperty DisplayName) -join ", "
        }
    }
```

### Check User's Applied Policies

```powershell
$userId = "user@contoso.com"

Get-MgAuditLogSignIn -Filter "userPrincipalName eq '$userId'" -Top 10 |
    Select-Object CreatedDateTime, AppDisplayName, ConditionalAccessStatus,
        @{N='Policies';E={
            $_.AppliedConditionalAccessPolicies |
            Select-Object DisplayName, Result
        }}
```

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- Troubleshooting workflow
- Common error codes
- Debugging decision tree

---

## Key Takeaways

- Sign-in logs are essential for troubleshooting
- The Conditional Access tab shows exactly which policy caused issues
- Common errors have predictable causes and solutions
- Emergency accounts must be excluded from all policies
- What If tool helps diagnose issues without affecting users
- Document and test policies before enabling

---

## Next Steps

Continue to [5.8 Best Practices](../5.8-best-practices/README.md) to learn about policy design recommendations and the Gap Analyzer.
