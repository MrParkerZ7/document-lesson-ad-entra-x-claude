# 9.3 Password Hash Synchronization (PHS)

## Overview

Password Hash Synchronization (PHS) is the simplest and most resilient authentication method for hybrid identity. This sub-lesson explains how PHS works, including the double-hash security process, configuration steps, sync frequency, and why PHS is recommended for disaster recovery scenarios.

## Learning Objectives

- Understand how Password Hash Synchronization works technically
- Learn the security measures that protect synchronized password hashes
- Configure and enable PHS in Azure AD Connect
- Understand sync frequency and timing
- Recognize the disaster recovery benefits of PHS

---

## How Password Hash Sync Works

### The Double-Hash Process

Password Hash Synchronization does not sync your actual password. Instead, it synchronizes a hash of the hash of your password using a secure multi-step process.

```yaml
Password Hash Synchronization Process:

Step 1 - On-Premises AD Storage:
  Original: User password "MyP@ssw0rd!"
  AD Storage: MD4 hash of password (16 bytes)
  Note: AD never stores plaintext passwords

Step 2 - Azure AD Connect Processing:
  Input: MD4 hash from AD
  Process:
    - Adds per-user salt
    - Applies PBKDF2 key derivation (1000 iterations)
    - Produces SHA256 hash
  Output: 32-byte derived hash

Step 3 - Transmission to Entra ID:
  Transport: Encrypted TLS 1.2 connection
  Protocol: HTTPS to Azure endpoints
  Frequency: Every 2 minutes (for changes)

Step 4 - Entra ID Storage:
  Storage: Encrypted hash in Azure
  Access: Used only for authentication validation
  Protection: Azure security controls
```

### Technical Flow Diagram

```yaml
Authentication Flow with PHS:

  User enters password at sign-in
       ↓
  Entra ID receives password
       ↓
  Entra ID computes hash using same algorithm
       ↓
  Compares computed hash with stored hash
       ↓
  If match: Authentication successful
  If no match: Authentication failed
```

### What Gets Synchronized

| Data | Synchronized | Notes |
|------|--------------|-------|
| Plaintext password | No | Never leaves the user's device |
| MD4 hash (AD native) | No | Never transmitted |
| Derived hash (PBKDF2+SHA256) | Yes | Only this is synced |
| Salt | Yes | Unique per user |
| Password history | No | Not synchronized |

---

## Security Considerations

### Security Measures

| Security Layer | Protection |
|----------------|------------|
| **Hashing Algorithm** | PBKDF2 with 1000 iterations + SHA256 |
| **Salting** | Per-user unique salt prevents rainbow table attacks |
| **Transport Encryption** | TLS 1.2 for all communications |
| **Storage Encryption** | Encrypted at rest in Azure |
| **No Reversibility** | Hash cannot be reversed to original password |
| **Isolated Storage** | Password hashes stored separately from user objects |

### Security Best Practices

```yaml
Recommendations:
  Enable Additional Protections:
    - Multi-Factor Authentication (MFA)
    - Conditional Access policies
    - Identity Protection risk policies
    - Leaked Credentials Detection (requires P2)

  Monitor for Threats:
    - Sign-in logs for anomalies
    - Risky sign-in reports
    - Password spray detection
    - Brute force protection

  Regular Reviews:
    - Audit sync service account permissions
    - Review Azure AD Connect Health alerts
    - Check for sync errors regularly
```

### Leaked Credentials Detection

```yaml
Azure AD Identity Protection:
  Capability: Detects if synced credentials appear in known breaches

  How it works:
    - Microsoft monitors dark web and breach dumps
    - Compares hashed credentials against breach data
    - Flags users with potentially compromised passwords

  Requirements:
    - Entra ID P2 license
    - PHS must be enabled

  Actions:
    - Risk event generated
    - User flagged as at-risk
    - Can require password change via Conditional Access
```

---

## Enabling and Configuring PHS

### Enable During Azure AD Connect Installation

```yaml
Express Settings:
  - PHS is enabled by default
  - No additional configuration needed
  - Seamless SSO also configured

Custom Settings:
  Step: User Sign-In page
  Selection: "Password Hash Synchronization"
  Additional: Check "Enable single sign-on" (optional)
```

### Enable PHS After Installation

```yaml
Method 1 - Azure AD Connect Wizard:
  1. Open Azure AD Connect
  2. Click "Configure"
  3. Select "Change user sign-in"
  4. Click Next
  5. Enter Azure AD credentials
  6. Select "Password Hash Synchronization"
  7. Optionally enable Seamless SSO
  8. Complete wizard

Method 2 - PowerShell:
  # Not typically done via PowerShell
  # Use wizard for sign-in method changes
```

### Verify PHS Configuration

```powershell
# Check if password sync is enabled
Get-ADSyncAADPasswordSyncConfiguration -SourceConnector "contoso.com"

# Expected output shows Enabled = True
# SourceConnector   : contoso.com
# Enabled           : True
# TargetConnector   : <tenant>.onmicrosoft.com

# Check password sync status
Get-ADSyncAADPasswordSyncState

# Force password sync for a specific user
$user = Get-ADSyncCSObject -DistinguishedName "CN=John Doe,OU=Users,DC=contoso,DC=com"
Invoke-ADSyncCSPasswordSync -CSObject $user
```

### Enable PHS as Backup with PTA or Federation

```yaml
Best Practice - PHS as Backup:
  Purpose: Maintain authentication if PTA agents or AD FS fail

  Configuration:
    1. Enable PHS in Azure AD Connect
    2. Keep PTA or Federation as primary method
    3. PHS runs silently in background
    4. Hashes sync but not used unless failover

  Failover Scenarios:
    - All PTA agents offline
    - AD FS farm unavailable
    - On-premises network outage

  Activation:
    - Manual: Convert domain to managed
    - Staged Rollout: Move groups to PHS
```

---

## Sync Frequency

### Understanding Sync Timing

| Sync Type | Frequency | Trigger |
|-----------|-----------|---------|
| **Initial Sync** | Once | During installation |
| **Full Sync** | On-demand | Configuration changes |
| **Delta Sync** | Every 30 minutes | Scheduled |
| **Password Sync** | Every 2 minutes | Password change detection |

### Password Change Propagation

```yaml
Password Change Timeline:

User changes password in AD:
  T+0: Password changed on DC

AD Connect detects change:
  T+2 min: Password sync cycle runs

Hash computed and transmitted:
  T+2 min: New hash sent to Entra ID

User can sign in with new password:
  T+2-3 min: Cloud authentication works

Note: May be longer if sync cycle just completed
```

### Sync Cycle Management

```powershell
# Check sync scheduler status
Get-ADSyncScheduler

# Example output:
# AllowedSyncCycleInterval : 00:30:00
# CurrentlyEffectiveSyncCycleInterval : 00:30:00
# CustomizedSyncCycleInterval :
# NextSyncCyclePolicyType : Delta
# NextSyncCycleStartTime : 2/7/2026 10:30:00 AM
# SyncCycleEnabled : True
# MaintenanceEnabled : True

# Force immediate delta sync
Start-ADSyncSyncCycle -PolicyType Delta

# Force full sync (use sparingly)
Start-ADSyncSyncCycle -PolicyType Initial

# Disable sync scheduler temporarily
Set-ADSyncScheduler -SyncCycleEnabled $false

# Re-enable sync scheduler
Set-ADSyncScheduler -SyncCycleEnabled $true
```

---

## Benefits for Disaster Recovery

### Why PHS is Essential for DR

```yaml
Disaster Recovery Scenario:

  Without PHS:
    - On-premises outage occurs
    - PTA agents cannot reach DCs
    - AD FS servers unavailable
    - Users CANNOT authenticate to cloud services
    - Business impact: Complete authentication failure

  With PHS Enabled:
    - On-premises outage occurs
    - Entra ID has password hashes
    - Users CAN authenticate to cloud services
    - Microsoft 365, Azure apps continue working
    - Business impact: Minimal disruption
```

### PHS DR Advantages

| Advantage | Description |
|-----------|-------------|
| **No On-Prem Dependency** | Authentication works without on-premises connectivity |
| **Automatic Availability** | No failover action required |
| **Works Immediately** | Hashes already synchronized |
| **Global Redundancy** | Microsoft's infrastructure ensures availability |
| **No Additional Infrastructure** | No extra servers needed |

### Failover Scenarios

```yaml
Scenario 1 - Data Center Outage:
  Impact: All on-premises servers unavailable
  PHS Result: Cloud apps continue to work
  User Experience: No change for cloud resources

Scenario 2 - Network Connectivity Loss:
  Impact: Azure AD Connect cannot reach cloud
  PHS Result: Last synced hashes still valid
  User Experience: Can authenticate (may not reflect very recent password changes)

Scenario 3 - AD DS Corruption:
  Impact: Active Directory requires rebuild
  PHS Result: Users can still access cloud resources
  Benefit: Business continuity while AD is restored
```

### Implementing PHS for DR

```yaml
DR Configuration Steps:

1. Enable PHS in Azure AD Connect:
   - Even if using PTA or Federation
   - PHS runs in background

2. Test DR Scenario:
   - Stop PTA agents (or AD FS)
   - Verify cloud authentication works
   - Confirm expected behavior

3. Document Failover Process:
   - How to switch from Federation to PHS
   - Staged Rollout configuration
   - Communication plan for users

4. Regular Validation:
   - Monitor password sync health
   - Check Azure AD Connect Health
   - Verify hash sync is current
```

### Staged Rollout (Gradual Migration or DR Testing)

```yaml
Staged Rollout Feature:
  Purpose: Test PHS with specific groups before full deployment

  Configuration:
    Location: Entra ID > Microsoft Entra Connect > Staged Rollout

    Steps:
      1. Enable Staged Rollout
      2. Select authentication method (PHS)
      3. Add test groups
      4. Monitor and validate
      5. Expand to more groups
      6. Eventually convert entire domain

  Use Cases:
    - DR testing without affecting all users
    - Gradual migration from Federation
    - Pilot before full PHS deployment
```

---

## Troubleshooting PHS

### Common Issues and Solutions

```yaml
Issue: Passwords not syncing
  Check:
    - Get-ADSyncAADPasswordSyncConfiguration
    - Ensure Enabled = True
  Solution:
    - Re-enable password sync in wizard
    - Verify connector account permissions

Issue: Specific user password not synced
  Check:
    - User in sync scope (OU filtering)
    - No sync errors for user object
  Solution:
    - Force sync: Invoke-ADSyncCSPasswordSync
    - Check for attribute errors

Issue: Password sync delayed
  Check:
    - Sync scheduler status
    - Network connectivity
  Solution:
    - Run: Start-ADSyncSyncCycle -PolicyType Delta
    - Verify firewall allows outbound 443
```

### Monitoring PHS Health

```powershell
# Check password sync heartbeat
Get-ADSyncAADPasswordSyncState | Select-Object PasswordSyncLastSuccessfulCycleEndTimestamp

# View password sync errors
Get-EventLog -LogName Application -Source "Directory Synchronization" -Newest 50

# Check Azure AD Connect Health in portal
# Navigate: Entra ID > Microsoft Entra Connect > Connect Health
```

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of the Password Hash Synchronization process and security flow.

---

## Key Takeaways

1. PHS synchronizes a hash of the hash, never the actual password or original AD hash
2. The double-hash process (PBKDF2 + SHA256) with salting ensures security
3. Password changes sync within approximately 2 minutes of the change in AD
4. PHS provides essential disaster recovery capability even when using PTA or Federation
5. Leaked Credentials Detection (Entra ID P2) only works with PHS enabled
6. Always enable PHS as a backup authentication method for business continuity

---

## Navigation

[<- 9.2 Azure AD Connect](../9.2-azure-ad-connect/README.md) | [9.4 Pass-through Authentication ->](../9.4-pass-through-authentication/README.md)
