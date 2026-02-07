# 5.6 Testing Policies

## Overview

Testing Conditional Access policies before enabling them is critical to prevent user disruption and lockouts. Entra ID provides Report-only mode and the What If tool to safely validate policies before enforcement.

## Learning Objectives

- Use Report-only mode to test policies without impact
- Analyze Report-only results in sign-in logs
- Use the What If tool for policy simulation
- Understand the testing workflow for new policies
- Know when to enable policies after testing

---

## Report-Only Mode

### What It Does

Report-only mode logs what a policy **would** do without actually enforcing it:
- Policy is evaluated on every sign-in
- Results logged to sign-in logs
- No impact on user access
- Useful for monitoring before enforcement

### Enabling Report-Only Mode

```yaml
Policy state: Report-only
```

### Policy States Comparison

| State | Evaluated | Enforced | Logged |
|-------|-----------|----------|--------|
| Enabled | Yes | Yes | Yes |
| Report-only | Yes | No | Yes |
| Disabled | No | No | No |

---

## Analyzing Report-Only Results

### Accessing Sign-in Logs

```
Microsoft Entra ID → Sign-in logs → [Select sign-in]
→ Conditional Access tab → Report-only
```

### Result States

| Result | Meaning |
|--------|---------|
| **Success** | Policy would allow access |
| **Failure** | Policy would block access |
| **Not applied** | Policy conditions not met |
| **Not enabled** | Policy is disabled |

### Understanding Policy Impact

For each sign-in, review:
1. Which policies applied
2. What the outcome would be
3. Which controls would be required
4. Why policy did/didn't apply

### Example Analysis

```
Sign-in: user@contoso.com → SharePoint Online
Device: Windows, Not compliant
Location: Home network

Report-only results:
┌─────────────────────────────────────────────────────┐
│ Policy: POL-CA-MFA-All-Users                        │
│ Result: Success (would require MFA)                 │
├─────────────────────────────────────────────────────┤
│ Policy: POL-CA-Require-Compliant-Device             │
│ Result: Failure (would block - not compliant)       │
├─────────────────────────────────────────────────────┤
│ Policy: POL-CA-Block-Legacy-Auth                    │
│ Result: Not applied (modern auth used)              │
└─────────────────────────────────────────────────────┘

If enabled: User would be blocked (device not compliant)
```

---

## What If Tool

### What It Does

The What If tool simulates policy evaluation without requiring an actual sign-in:
- Test policies for any user
- Simulate any conditions
- See which policies would apply
- Preview access outcome

### Accessing What If

```
Microsoft Entra ID → Security → Conditional Access → What If
```

### Simulation Parameters

| Parameter | Description |
|-----------|-------------|
| **User** | Select user to test |
| **Cloud apps** | Target application(s) |
| **IP address** | Simulate source IP |
| **Country** | Simulate location |
| **Device platform** | Windows, iOS, Android, etc. |
| **Device state** | Compliant, Hybrid joined, etc. |
| **Client app** | Browser, mobile app, etc. |
| **Sign-in risk** | Simulate risk level |
| **User risk** | Simulate user risk |

### Example Simulation

```yaml
What If parameters:
  User: john@contoso.com
  Cloud apps: Microsoft 365
  IP address: 45.67.89.100 (external)
  Device platform: Windows
  Device state: Not compliant
  Client app: Browser

Results:
  Policies that will apply:
    - POL-CA-MFA-All-Users
    - POL-CA-Require-Compliant-Device

  Expected outcome: Blocked (device not compliant)

  Required grant controls:
    - MFA (from policy 1)
    - Compliant device (from policy 2)
```

---

## Testing Workflow

### Step 1: Create Policy in Report-Only

```yaml
Name: POL-CA-New-MFA-Policy
State: Report-only
# Configure assignments and controls
```

### Step 2: Monitor for 7-14 Days

```powershell
# Get Report-only policy results
Get-MgAuditLogSignIn -Filter "conditionalAccessStatus eq 'reportOnlySuccess' or conditionalAccessStatus eq 'reportOnlyFailure'" |
    Select-Object UserPrincipalName, AppDisplayName, ConditionalAccessStatus
```

### Step 3: Analyze Impact

Review:
- How many users affected?
- Which users would be blocked?
- Any unexpected results?
- Are exclusions working correctly?

### Step 4: Use What If for Edge Cases

Test specific scenarios:
- Emergency admin access
- Guest user access
- Service account access
- Unusual locations

### Step 5: Adjust If Needed

Common adjustments:
- Add exclusions
- Modify conditions
- Change grant controls
- Refine targeting

### Step 6: Enable Policy

```yaml
# Change state
State: Enabled
```

### Step 7: Monitor After Enabling

Watch for:
- Unexpected blocks
- User complaints
- Sign-in failures
- Support tickets

---

## Testing Checklist

### Before Enabling

- [ ] Ran in Report-only for at least 7 days
- [ ] Analyzed all "Failure" results
- [ ] Tested with What If for edge cases
- [ ] Verified emergency accounts are excluded
- [ ] Tested service account access
- [ ] Reviewed guest user impact
- [ ] Communicated to affected users
- [ ] Documented policy purpose

### After Enabling

- [ ] Monitored sign-in logs for first 24 hours
- [ ] Watched for support tickets
- [ ] Verified expected blocks are occurring
- [ ] Confirmed no unexpected impact
- [ ] Ready to disable if issues arise

---

## Common Testing Scenarios

### Scenario 1: New MFA Policy

```
1. Create policy in Report-only
2. Wait 7 days
3. Check how many users would need MFA
4. Identify users without MFA methods registered
5. Run registration campaign
6. Enable policy
```

### Scenario 2: Device Compliance Policy

```
1. Create policy in Report-only
2. Wait 14 days (longer for device policies)
3. Identify non-compliant devices
4. Check if devices can become compliant
5. Plan for exceptions (legacy devices)
6. Enable policy
```

### Scenario 3: Location-Based Policy

```
1. Create policy in Report-only
2. Wait 7 days
3. Identify users accessing from blocked locations
4. Verify legitimate travelers are excluded
5. Check for VPN users
6. Enable policy
```

---

## PowerShell Analysis

### Get Report-Only Results

```powershell
# Connect to Microsoft Graph
Connect-MgGraph -Scopes "AuditLog.Read.All"

# Get recent sign-ins with Report-only results
$signIns = Get-MgAuditLogSignIn -Top 1000 -Filter "createdDateTime ge 2024-01-01"

# Filter for Report-only failures (would be blocked)
$wouldBlock = $signIns | Where-Object {
    $_.ConditionalAccessStatus -eq "reportOnlyFailure"
}

# Group by user
$wouldBlock | Group-Object UserPrincipalName |
    Select-Object Name, Count |
    Sort-Object Count -Descending
```

### Analyze Policy Impact

```powershell
# Get sign-ins that would be blocked by a specific policy
$policyName = "POL-CA-Require-Compliant-Device"

$signIns | Where-Object {
    $_.AppliedConditionalAccessPolicies |
        Where-Object { $_.DisplayName -eq $policyName -and $_.Result -eq "reportOnlyFailure" }
} | Select-Object UserPrincipalName, AppDisplayName, DeviceDetail
```

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- Report-only mode flow
- What If tool usage
- Testing workflow steps

---

## Key Takeaways

- Always test policies in Report-only mode first
- Use What If to simulate edge cases
- Monitor for at least 7 days before enabling
- Analyze "Failure" results to understand impact
- Keep emergency accounts excluded during testing
- Document expected vs. actual results
- Be ready to disable if issues arise after enabling

---

## Next Steps

Continue to [5.7 Troubleshooting](../5.7-troubleshooting/README.md) to learn about diagnosing and resolving Conditional Access issues.
