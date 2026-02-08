# 8.1 Identity Protection

## Overview

Microsoft Entra ID Protection uses machine learning to detect, investigate, and remediate identity-based risks. It continuously analyzes sign-in patterns and user behaviors to identify potential compromises and automatically respond to threats.

## Learning Objectives

- Understand user risk vs sign-in risk concepts
- Recognize risk detection types and levels
- Configure Identity Protection policies
- Integrate risk-based Conditional Access
- Investigate and remediate risky users

---

## How Identity Protection Works

### Detection Engine

```yaml
Identity Protection Flow:
  1. Collect signals:
     - Sign-in patterns
     - Device information
     - Location data
     - Threat intelligence

  2. Analyze with ML:
     - Compare to baseline
     - Detect anomalies
     - Correlate events

  3. Assign risk level:
     - None, Low, Medium, High

  4. Trigger response:
     - Policy-based remediation
     - Admin notification
     - User action required
```

### Key Capabilities

| Capability | Description |
|------------|-------------|
| Detection | Identify suspicious activities using ML |
| Investigation | Analyze risk events and user behavior |
| Remediation | Automatic or manual response to risks |
| Reporting | Detailed logs and insights |

---

## Risk Types

### User Risk vs Sign-in Risk

| Aspect | User Risk | Sign-in Risk |
|--------|-----------|--------------|
| Scope | Cumulative account risk | Single sign-in event |
| Persistence | Remains until remediated | Per authentication |
| Examples | Leaked credentials, unusual behavior | Anonymous IP, atypical travel |
| Remediation | Password change, account review | MFA challenge, block |

### Risk Levels

```
None ──▶ Low ──▶ Medium ──▶ High
  │        │         │         │
  │        │         │         └── Immediate action required
  │        │         └── Investigation recommended
  │        └── Monitor closely
  └── Normal behavior
```

---

## Risk Detections

### User Risk Detections

| Detection | Description | Type |
|-----------|-------------|------|
| Leaked credentials | Credentials found on dark web | Offline |
| Anomalous user activity | Unusual behavior patterns | Offline |
| Possible attempt to access PRT | Primary Refresh Token compromise | Real-time |
| Threat intelligence linked user | Microsoft threat data match | Offline |
| Password spray | Same password across accounts | Offline |
| Unfamiliar sign-in properties | New characteristics for user | Real-time |

### Sign-in Risk Detections

| Detection | Description | Type |
|-----------|-------------|------|
| Anonymous IP address | Sign-in from Tor, anonymous VPN | Real-time |
| Atypical travel | Impossible travel between locations | Offline |
| Malware-linked IP | Known malicious IP address | Offline |
| Suspicious browser | Unusual browser characteristics | Real-time |
| Token anomaly | Unusual token characteristics | Real-time |
| Mass access to sensitive files | Unusual file access pattern | Offline |
| Suspicious inbox forwarding | Email forwarding rules | Offline |
| New country | Sign-in from new country | Real-time |

### Premium vs Free Detections

```yaml
Free Tier (P1):
  - Leaked credentials
  - Anonymous IP address
  - Atypical travel

Premium Tier (P2):
  - All Free detections plus:
  - Anomalous user activity
  - Token anomalies
  - Password spray detection
  - Threat intelligence links
  - Custom risk policies
```

---

## Configuring Identity Protection

### Portal Navigation

```
Microsoft Entra ID → Security → Identity Protection
```

### User Risk Policy

```yaml
Policy: POL-IDP-User-Risk-Remediation

Purpose: Force password change for compromised accounts

Assignments:
  Include: All users
  Exclude:
    - Emergency access accounts
    - Service accounts

Conditions:
  User risk level: Medium and above

Controls:
  Access: Allow access
  Require: Password change

State: Enabled
```

### Sign-in Risk Policy

```yaml
Policy: POL-IDP-SignIn-Risk-MFA

Purpose: Require MFA for suspicious sign-ins

Assignments:
  Include: All users
  Exclude:
    - Emergency access accounts

Conditions:
  Sign-in risk level: Medium and above

Controls:
  Access: Allow access
  Require: Multi-factor authentication

State: Enabled
```

---

## Integration with Conditional Access

### Why Use Conditional Access?

Better control and flexibility:
- More conditions available
- Session controls
- Granular targeting
- Combined with other signals

### Risk-Based Conditional Access Policy

```yaml
Name: CA-High-Risk-Block

Assignments:
  Users:
    Include: All users
    Exclude: Emergency access accounts

  Cloud apps: All cloud apps

Conditions:
  User risk: High
  Sign-in risk: High

Access controls:
  Grant: Block access

Session controls:
  Sign-in frequency: Every time
```

### Medium Risk with MFA

```yaml
Name: CA-Medium-Risk-MFA

Assignments:
  Users: All users
  Cloud apps: All cloud apps

Conditions:
  User risk: Medium and above
  Sign-in risk: Medium and above

Access controls:
  Grant:
    Require MFA
    Require password change (for user risk)
```

---

## Investigating Risky Users

### Access Risky Users Report

```
Identity Protection → Risky users
```

### Information Available

| Column | Description |
|--------|-------------|
| User | Display name and UPN |
| Risk state | At risk, Dismissed, Remediated |
| Risk level | Low, Medium, High |
| Risk last updated | Timestamp of last change |
| Risk detail | Detection type |

### Actions on Risky Users

```yaml
Confirm user compromised:
  - Sets risk to High
  - Blocks sign-in
  - Use when: Confirmed breach

Dismiss user risk:
  - Clears risk state
  - Use when: False positive verified

Reset password:
  - Forces password change
  - Use when: Remediation needed

Block user:
  - Prevents all sign-ins
  - Use when: Active threat
```

### Investigation Process

```
1. Review risk detections
         ↓
2. Check sign-in logs
         ↓
3. Verify with user/manager
         ↓
4. Determine if compromised
         ↓
   ┌─────┴─────┐
   ▼           ▼
True Positive  False Positive
   │              │
   ▼              ▼
Confirm &      Dismiss
Remediate      Risk
```

---

## Risky Sign-ins Report

### Access Report

```
Identity Protection → Risky sign-ins
```

### Filter Options

| Filter | Options |
|--------|---------|
| Date | Custom range |
| Risk level | Low, Medium, High |
| Risk state | At risk, Confirmed, Dismissed |
| Risk detail | Detection type |

### Sign-in Details

Click on any sign-in to see:
- Device information
- Location data
- Application accessed
- Risk detections triggered
- Authentication details

---

## PowerShell Management

### Connect and Query Risky Users

```powershell
Connect-MgGraph -Scopes "IdentityRiskyUser.Read.All"

# Get all risky users
$riskyUsers = Get-MgRiskyUser -Filter "riskState eq 'atRisk'"

$riskyUsers | Select-Object UserDisplayName, UserPrincipalName,
    RiskLevel, RiskState, RiskLastUpdatedDateTime
```

### Get Risk Detections

```powershell
Connect-MgGraph -Scopes "IdentityRiskEvent.Read.All"

# Get recent risk detections
$detections = Get-MgRiskDetection -Filter "detectedDateTime ge 2024-01-01"

$detections | Select-Object UserDisplayName, RiskEventType,
    RiskLevel, DetectedDateTime, IpAddress, Location
```

### Dismiss User Risk

```powershell
# Dismiss risk for false positive
Invoke-MgDismissRiskyUser -UserIds @("user-object-id")
```

### Confirm Compromised

```powershell
# Confirm user is compromised
Invoke-MgConfirmRiskyUserCompromised -UserIds @("user-object-id")
```

---

## Notifications and Alerts

### Configure Notifications

```
Identity Protection → Settings → Notifications
```

### Alert Types

```yaml
Users at risk detected emails:
  Recipients: Security team
  Threshold: Weekly digest or immediate

Weekly digest email:
  Recipients: IT admins
  Content: Summary of risky users and detections
```

---

## Best Practices

### Policy Configuration

```yaml
Recommendations:
  - Start with report-only mode
  - Target medium+ risk levels initially
  - Exclude emergency access accounts
  - Use Conditional Access for more control
  - Enable both user and sign-in risk policies
```

### Investigation Process

```yaml
Do:
  - Review detections promptly
  - Verify with users when appropriate
  - Document investigation findings
  - Track false positive patterns

Don't:
  - Dismiss without investigation
  - Ignore low-risk detections
  - Wait for multiple high-risk events
```

### Response Automation

```yaml
Automated responses:
  User Risk High → Block access
  User Risk Medium → Require password change
  Sign-in Risk High → Block access
  Sign-in Risk Medium → Require MFA
```

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- Risk detection flow
- User risk vs sign-in risk
- Policy evaluation
- Investigation process

---

## Key Takeaways

- Identity Protection uses ML to detect identity-based risks
- User risk is cumulative; sign-in risk is per-event
- Configure both user and sign-in risk policies
- Use Conditional Access for advanced control
- Investigate risky users promptly
- Exclude emergency access accounts from policies

---

## Navigation

[← 8.0 Security & Governance Overview](../README.md) | [8.2 Privileged Identity Management →](../8.2-privileged-identity-management/README.md)
