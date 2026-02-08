# 8.7 Identity Secure Score

## Overview

Microsoft Entra Identity Secure Score is a measurement of your organization's identity security posture, expressed as a percentage from 0-100%. It provides actionable recommendations to improve security, benchmarks against similar organizations, and helps prioritize security investments based on impact and effort.

## Learning Objectives

- Understand what Identity Secure Score measures and how it's calculated
- Identify high-impact improvement actions
- Compare your score to similar organizations
- Track score improvements over time
- Prioritize security improvements based on impact
- Understand the relationship to Microsoft Secure Score
- Use PowerShell to retrieve and monitor scores

---

## What is Identity Secure Score?

### Purpose

Identity Secure Score helps organizations:
- **Measure** current identity security posture objectively
- **Benchmark** against similar organizations
- **Prioritize** security improvements based on impact
- **Track** progress over time
- **Report** security posture to stakeholders

### Score Range

```
0% ─────────────────────────────────────────────────── 100%
 │                                                       │
 └── Poor Security Posture         Excellent Posture ───┘
     High Risk                      Low Risk
```

### Portal Access

```
Microsoft Entra ID → Security → Identity Secure Score
```

---

## Score Calculation

### How Points Are Assigned

Each improvement action has a maximum point value based on:
- **Security impact** - How much the action reduces risk
- **User impact** - Effect on user experience
- **Implementation complexity** - Effort required

### Score Formula

```
                 Points Achieved
Score (%) = ─────────────────────── × 100
             Total Available Points
```

### Point Categories

| Category | Description | Example Actions |
|----------|-------------|-----------------|
| Identity | User and admin account security | MFA, password protection |
| Devices | Device compliance and management | Conditional Access for devices |
| Apps | Application security | Consent policies, app registration |
| Infrastructure | Directory and role security | PIM, least privilege |

### Score Status Types

| Status | Description |
|--------|-------------|
| To address | Action not yet implemented |
| Planned | Action scheduled for implementation |
| Risk accepted | Decision made to accept the risk |
| Resolved through third party | Mitigated by external solution |
| Resolved through alternate mitigation | Different approach used |
| Completed | Fully implemented |

---

## Key Improvement Actions

### High-Impact Actions

| Action | Impact | Points | Description |
|--------|--------|--------|-------------|
| Require MFA for all users | High | 10.00 | Enable MFA for all user accounts |
| Block legacy authentication | High | 8.00 | Prevent legacy protocols |
| Enable sign-in risk policy | High | 7.00 | Automate risk response |
| Enable user risk policy | High | 7.00 | Protect compromised accounts |
| Designate more than one global admin | High | 5.00 | Ensure admin redundancy |

### Medium-Impact Actions

| Action | Impact | Points | Description |
|--------|--------|--------|-------------|
| Ensure all admins use MFA | Medium | 5.00 | Protect privileged accounts |
| Enable password protection | Medium | 4.00 | Ban weak passwords |
| Use least privileged admin roles | Medium | 4.00 | Minimize standing privileges |
| Enable PIM for admin roles | Medium | 4.00 | Just-in-time access |
| Remove stale accounts | Medium | 3.00 | Clean up unused accounts |

### Standard Actions

| Action | Impact | Points | Description |
|--------|--------|--------|-------------|
| Enable self-service password reset | Standard | 2.00 | Reduce helpdesk load |
| Configure sign-in frequency | Standard | 2.00 | Control session duration |
| Require approved apps | Standard | 2.00 | Control app access |
| Enable Conditional Access | Standard | 2.00 | Context-based access |
| Review access regularly | Standard | 1.00 | Maintain least privilege |

---

## Detailed Key Actions

### 1. Require MFA for All Users

```yaml
Action: Enable MFA for all users
Impact: High (10 points)
Effort: Medium

Implementation:
  Option 1: Security Defaults
    - Microsoft Entra ID → Properties → Security defaults → Enable

  Option 2: Conditional Access
    - Create policy requiring MFA for all users
    - Exclude emergency access accounts

Best Practices:
  - Use Microsoft Authenticator app
  - Enable number matching
  - Register backup methods
```

### 2. Block Legacy Authentication

```yaml
Action: Block legacy authentication protocols
Impact: High (8 points)
Effort: Low

Why Important:
  - Legacy protocols don't support MFA
  - Common attack vector for password spray
  - 99% of password spray attacks use legacy auth

Implementation:
  Conditional Access Policy:
    Name: CA-Block-Legacy-Auth
    Conditions:
      Client apps: Exchange ActiveSync, Other clients
    Access controls:
      Block access
```

### 3. Enable Risk Policies

```yaml
Action: Enable sign-in and user risk policies
Impact: High (7 points each)
Effort: Low

Sign-in Risk Policy:
  Trigger: Medium and above
  Action: Require MFA

User Risk Policy:
  Trigger: Medium and above
  Action: Require password change

Configuration:
  Identity Protection → Policies → Configure
```

### 4. Use Least Privileged Roles

```yaml
Action: Minimize permanent admin assignments
Impact: Medium (4 points)
Effort: Medium

Current State Audit:
  - Review Global Administrator count
  - Identify over-privileged accounts
  - Map tasks to minimum required roles

Role Recommendations:
  Instead of Global Admin, use:
    - User Administrator (user management)
    - Application Administrator (app management)
    - Security Reader (security monitoring)
    - Billing Administrator (licensing)

Target:
  - Maximum 5 Global Administrators
  - All other admins use specific roles
```

### 5. Enable Password Protection

```yaml
Action: Enable Azure AD Password Protection
Impact: Medium (4 points)
Effort: Low

Features:
  - Block common passwords
  - Block organization-specific terms
  - Enforce password requirements

Configuration:
  Microsoft Entra ID → Security → Authentication methods
  → Password protection
    - Enable custom banned password list
    - Add organization-specific terms
    - Mode: Enforced
```

---

## Comparing to Similar Organizations

### Benchmark Categories

The score compares your organization against peers based on:

| Factor | Categories |
|--------|------------|
| Industry | Technology, Healthcare, Finance, etc. |
| Size | Small, Medium, Large enterprise |
| Region | Geographic location |
| License tier | P1, P2, E3, E5, etc. |

### Understanding the Comparison

```
Your Score: 65%
├── Industry Average: 58%  ─── You're above average
├── Size Comparison: 62%   ─── Slightly above peers
└── Top Performers: 85%    ─── Room for improvement
```

### Interpreting Benchmarks

| Your Position | Interpretation | Action |
|---------------|----------------|--------|
| Below average | Security gaps exist | Prioritize high-impact actions |
| At average | Meeting baseline | Focus on medium-impact items |
| Above average | Strong posture | Fine-tune and maintain |
| Top quartile | Excellent security | Continuous monitoring |

---

## Tracking Score Over Time

### Score History

View score trends in the portal:

```
Identity Secure Score → Score history
```

### Key Metrics to Track

```yaml
Track:
  - Current score percentage
  - Score change over 30/60/90 days
  - Number of completed actions
  - Pending improvement actions
  - Regression events (score decreases)
```

### Score Changes

| Change Type | Cause | Action |
|-------------|-------|--------|
| Increase | Actions completed | Document success |
| Decrease | New vulnerabilities discovered | Investigate and remediate |
| Decrease | Configuration drift | Review and restore |
| No change | Stagnation | Prioritize next actions |

### Creating a Score Dashboard

```kusto
// KQL Query for Score Tracking
IdentitySecureScore
| where TimeGenerated > ago(90d)
| summarize AvgScore = avg(Score) by bin(TimeGenerated, 1d)
| render timechart
```

---

## Prioritizing Improvements

### Prioritization Matrix

| Impact | Effort | Priority | Examples |
|--------|--------|----------|----------|
| High | Low | Immediate | Block legacy auth, risk policies |
| High | Medium | High | MFA for all users |
| Medium | Low | High | Password protection |
| High | High | Medium | PIM implementation |
| Medium | Medium | Medium | Access reviews |
| Low | Low | Standard | Documentation |
| Low | High | Defer | Complex customizations |

### Decision Framework

```yaml
Step 1: Filter by Impact
  - Focus on High impact first
  - Then Medium impact
  - Low impact as time permits

Step 2: Consider Effort
  - Quick wins (low effort) first
  - Plan medium effort items
  - Schedule high effort projects

Step 3: Evaluate Dependencies
  - Some actions require prerequisites
  - Group related actions together
  - Consider rollout timing

Step 4: Assess User Impact
  - Balance security with usability
  - Plan user communication
  - Consider phased rollout
```

---

## Relationship to Microsoft Secure Score

### Score Comparison

| Aspect | Identity Secure Score | Microsoft Secure Score |
|--------|----------------------|------------------------|
| Scope | Identity only | Entire M365 environment |
| Location | Entra ID portal | Defender portal |
| Focus | Users, auth, governance | Devices, data, apps, identity |
| Actions | ~50 identity actions | 200+ total actions |

### How They Relate

```
Microsoft Secure Score
├── Identity Score ←── Identity Secure Score
├── Device Score
├── Apps Score
└── Data Score

Identity Secure Score actions also improve Microsoft Secure Score
```

### Using Both Scores

```yaml
Identity Secure Score:
  Use for:
    - Identity-specific improvements
    - Entra ID team metrics
    - Identity governance reporting

Microsoft Secure Score:
  Use for:
    - Overall security posture
    - Cross-workload view
    - Executive reporting
    - Compliance frameworks
```

---

## PowerShell to Retrieve Score

### Connect and Get Current Score

```powershell
# Connect to Microsoft Graph
Connect-MgGraph -Scopes "SecurityEvents.Read.All"

# Get current Identity Secure Score
$secureScore = Get-MgSecuritySecureScore -Top 1

# Display score details
$secureScore | Select-Object @{
    Name = "CurrentScore"
    Expression = { $_.CurrentScore }
}, @{
    Name = "MaxScore"
    Expression = { $_.MaxScore }
}, @{
    Name = "Percentage"
    Expression = { [math]::Round(($_.CurrentScore / $_.MaxScore) * 100, 2) }
}, CreatedDateTime
```

### Get Control Scores (Individual Actions)

```powershell
# Get all control scores
$controlScores = $secureScore.ControlScores

# Filter for identity-related controls
$identityControls = $controlScores | Where-Object {
    $_.ControlCategory -eq "Identity"
}

# Display controls needing attention
$identityControls | Where-Object {
    $_.Score -lt $_.MaxScore
} | Select-Object ControlName, Score, MaxScore, Description |
    Sort-Object MaxScore -Descending |
    Format-Table -AutoSize
```

### Get Score History

```powershell
# Get score history for trending
$scoreHistory = Get-MgSecuritySecureScore -Top 30 |
    Sort-Object CreatedDateTime

# Calculate trend
$scoreHistory | Select-Object @{
    Name = "Date"
    Expression = { $_.CreatedDateTime.ToString("yyyy-MM-dd") }
}, @{
    Name = "Score"
    Expression = { [math]::Round(($_.CurrentScore / $_.MaxScore) * 100, 2) }
} | Format-Table
```

### Export Improvement Actions

```powershell
# Get secure score profiles (improvement actions)
$profiles = Get-MgSecuritySecureScoreControlProfile

# Export to CSV for analysis
$profiles | Select-Object @{
    Name = "Action"
    Expression = { $_.Title }
}, @{
    Name = "Category"
    Expression = { $_.ControlCategory }
}, @{
    Name = "MaxScore"
    Expression = { $_.MaxScore }
}, @{
    Name = "Status"
    Expression = { $_.ControlStateUpdates[-1].State }
}, @{
    Name = "Remediation"
    Expression = { $_.RemediationDescription }
} | Export-Csv -Path "SecureScoreActions.csv" -NoTypeInformation
```

### Automated Score Monitoring

```powershell
# Script to monitor score changes
$currentScore = (Get-MgSecuritySecureScore -Top 1)
$percentage = [math]::Round(
    ($currentScore.CurrentScore / $currentScore.MaxScore) * 100, 2
)

# Alert if score drops below threshold
$threshold = 60
if ($percentage -lt $threshold) {
    Write-Warning "Identity Secure Score ($percentage%) is below threshold ($threshold%)"

    # Send notification (example)
    # Send-MailMessage -To "security@contoso.com" ...
}
```

---

## Implementation Roadmap

### Phase 1: Quick Wins (Week 1-2)

```yaml
Actions:
  1. Block legacy authentication
     - Create Conditional Access policy
     - Monitor for impact
     - Expected score increase: +8 points

  2. Enable risk policies
     - Configure sign-in risk policy (Medium+)
     - Configure user risk policy (Medium+)
     - Expected score increase: +14 points

  3. Enable password protection
     - Add custom banned passwords
     - Enable enforcement
     - Expected score increase: +4 points

Estimated Total Increase: ~25 points
```

### Phase 2: Foundation (Week 3-4)

```yaml
Actions:
  1. MFA for all users
     - Audit current MFA status
     - Communication plan
     - Phased rollout
     - Expected score increase: +10 points

  2. Reduce Global Administrators
     - Audit current assignments
     - Assign specific roles
     - Document role mapping
     - Expected score increase: +5 points

  3. Enable MFA for all admins
     - Verify admin MFA registration
     - Enforce MFA for privileged roles
     - Expected score increase: +5 points

Estimated Total Increase: ~20 points
```

### Phase 3: Advanced Security (Month 2)

```yaml
Actions:
  1. Implement PIM
     - Enable for critical roles
     - Configure activation requirements
     - Train administrators
     - Expected score increase: +4 points

  2. Configure access reviews
     - Quarterly admin role reviews
     - Application access reviews
     - Expected score increase: +3 points

  3. Least privilege enforcement
     - Role assignment audit
     - Remove unnecessary permissions
     - Expected score increase: +4 points

Estimated Total Increase: ~11 points
```

### Phase 4: Optimization (Month 3+)

```yaml
Actions:
  1. Self-service password reset
     - Enable SSPR
     - Configure authentication methods
     - Expected score increase: +2 points

  2. Conditional Access optimization
     - Device compliance policies
     - Location-based policies
     - App protection policies
     - Expected score increase: +4 points

  3. Continuous monitoring
     - Regular score reviews
     - Address new recommendations
     - Maintain security posture

Estimated Total Increase: ~6 points
```

### Roadmap Summary

| Phase | Timeline | Expected Score Increase | Cumulative |
|-------|----------|------------------------|------------|
| Quick Wins | Week 1-2 | +25 points | 25% |
| Foundation | Week 3-4 | +20 points | 45% |
| Advanced | Month 2 | +11 points | 56% |
| Optimization | Month 3+ | +6 points | 62%+ |

---

## Best Practices

### Score Management

```yaml
Do:
  - Review score weekly
  - Document completed actions
  - Communicate progress to stakeholders
  - Track regressions immediately
  - Celebrate milestones

Don't:
  - Ignore low-impact actions entirely
  - Accept risk without documentation
  - Implement without testing
  - Skip user communication
```

### Continuous Improvement

```yaml
Monthly:
  - Review score trend
  - Address new recommendations
  - Audit recent changes

Quarterly:
  - Executive reporting
  - Benchmark comparison
  - Roadmap adjustment

Annually:
  - Comprehensive security review
  - Policy updates
  - Training refresh
```

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- Score components and calculation
- Improvement actions by impact level
- Score improvement journey
- Prioritization matrix

---

## Key Takeaways

- Identity Secure Score measures identity security from 0-100%
- Focus on high-impact, low-effort actions first
- Block legacy authentication and enable risk policies for quick wins
- MFA for all users is the highest-impact single action
- Compare your score against similar organizations for benchmarking
- Track score over time to demonstrate security improvements
- Use PowerShell for automated monitoring and reporting
- Follow a phased implementation roadmap for systematic improvement

---

## Navigation

[← 8.6 Security Alerts](../8.6-security-alerts/README.md) | [8.8 Governance Best Practices →](../8.8-governance-best-practices/README.md)
