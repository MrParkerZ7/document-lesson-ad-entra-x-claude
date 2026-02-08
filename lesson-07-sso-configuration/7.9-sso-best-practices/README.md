# 7.9 SSO Best Practices

## Overview

Implementing SSO effectively requires careful planning, security considerations, and operational excellence. This lesson consolidates best practices for SSO security, operations, and planning to help organizations maximize the benefits of Single Sign-On while minimizing risks.

## Learning Objectives

- Apply security best practices for SSO
- Implement operational excellence
- Plan SSO rollouts effectively
- Maintain SSO configurations
- Monitor and audit SSO access

---

## Security Best Practices

### Authentication Method Selection

| Priority | Method | Security Level |
|----------|--------|----------------|
| 1st | SAML / OIDC | High (federated) |
| 2nd | Header-based with App Proxy | Medium-High |
| 3rd | Password-based SSO | Medium |
| 4th | Linked SSO | Low (redirect only) |

### Federation Security Guidelines

```yaml
SAML Security:
  - Sign both response AND assertion
  - Use SHA-256 signing algorithm
  - Rotate certificates before expiry
  - Validate audience restrictions
  - Use appropriate NameID format

OIDC Security:
  - Use authorization code flow
  - Enable PKCE for public clients
  - Request minimum required scopes
  - Validate ID tokens properly
  - Avoid implicit flow
```

### Certificate Management

```yaml
Certificate Lifecycle:
  - Default validity: 3 years
  - Plan rotation 30-60 days before expiry
  - Set notification emails
  - Test new certificate before production
  - Document rotation procedure

Best Practices:
  - Maintain certificate inventory
  - Automate expiry alerts
  - Create rotation runbook
  - Have rollback plan
```

### PowerShell: Monitor Certificate Expiry

```powershell
# Check all SAML app certificates
Connect-MgGraph -Scopes "Application.Read.All"

$apps = Get-MgApplication -Filter "signInAudience eq 'AzureADMyOrg'"

foreach ($app in $apps) {
    $sp = Get-MgServicePrincipal -Filter "appId eq '$($app.AppId)'"

    $sp.KeyCredentials | Where-Object { $_.Type -eq "AsymmetricX509Cert" } | ForEach-Object {
        $daysRemaining = ($_.EndDateTime - (Get-Date)).Days
        if ($daysRemaining -lt 60) {
            [PSCustomObject]@{
                AppName = $sp.DisplayName
                Expiry = $_.EndDateTime
                DaysRemaining = $daysRemaining
                Status = if ($daysRemaining -lt 0) { "EXPIRED" }
                         elseif ($daysRemaining -lt 30) { "CRITICAL" }
                         else { "WARNING" }
            }
        }
    }
}
```

---

## Conditional Access Integration

### Recommended Policies

```yaml
Policy 1: Require MFA for All SSO Apps
  Assignments:
    Users: All users
    Apps: All enterprise applications
  Access controls:
    Grant: Require MFA

Policy 2: Block Legacy Authentication
  Assignments:
    Users: All users
    Apps: All enterprise applications
  Conditions:
    Client apps: Other clients, Exchange ActiveSync
  Access controls:
    Block access

Policy 3: Compliant Device for Sensitive Apps
  Assignments:
    Users: All users
    Apps: [High-security apps]
  Access controls:
    Grant: Require compliant device

Policy 4: Named Location Restrictions
  Assignments:
    Users: All users
    Apps: [Critical apps]
  Conditions:
    Locations: Exclude trusted locations
  Access controls:
    Grant: Require MFA
```

### Session Controls

```yaml
Recommended Settings:
  Sign-in frequency: 8-12 hours
  Persistent browser session: No (for sensitive apps)

Use Cases:
  - Standard apps: 24-hour sign-in frequency
  - Sensitive apps: 4-8 hour frequency
  - Highly sensitive: Each session requires auth
```

---

## Operational Excellence

### SSO Inventory Management

```yaml
Documentation:
  For Each App:
    - SSO method (SAML/OIDC/Password)
    - Entity ID / Client ID
    - Reply URL / Redirect URI
    - Certificate expiry date
    - Claims configuration
    - Provisioning status
    - Owner/Contact
    - User count
```

### Regular Audits

| Audit Type | Frequency | Focus |
|------------|-----------|-------|
| Access reviews | Quarterly | User assignments |
| Certificate check | Monthly | Expiry dates |
| Configuration review | Semi-annually | Settings accuracy |
| Sign-in analysis | Weekly | Error patterns |
| Unused apps | Quarterly | App rationalization |

### Access Reviews

```yaml
Access Review Configuration:
  Reviewers: Application owners
  Duration: 14 days
  Recurrence: Quarterly

  Settings:
    - Auto-apply results
    - Remove access for non-responses
    - Notify reviewers
```

### PowerShell: Identify Unused Apps

```powershell
# Find apps with no sign-ins in 90 days
$apps = Get-MgServicePrincipal -All

foreach ($app in $apps) {
    $signIns = Get-MgAuditLogSignIn -Filter "appId eq '$($app.AppId)'" -Top 1

    if (-not $signIns) {
        [PSCustomObject]@{
            AppName = $app.DisplayName
            AppId = $app.AppId
            LastSignIn = "Never or >90 days"
        }
    }
}
```

---

## Monitoring and Alerting

### Key Metrics to Monitor

| Metric | Description | Threshold |
|--------|-------------|-----------|
| SSO failure rate | Failed sign-ins | > 5% |
| Certificate expiry | Days remaining | < 60 days |
| Provisioning errors | Failed provisions | Any failures |
| Consent requests | Admin consent needed | New requests |
| Unusual sign-ins | Risk detections | Any high-risk |

### KQL Monitoring Queries

```kusto
// SSO success rate by app
SignInLogs
| where TimeGenerated > ago(7d)
| summarize
    Total = count(),
    Success = countif(ResultType == 0),
    Failed = countif(ResultType != 0)
    by AppDisplayName
| extend SuccessRate = Success * 100.0 / Total
| where SuccessRate < 95
| order by Total desc

// Top SSO errors
SignInLogs
| where TimeGenerated > ago(7d)
| where ResultType != 0
| summarize Count = count() by ResultType, ResultDescription
| order by Count desc
| take 10

// Users with repeated SSO failures
SignInLogs
| where TimeGenerated > ago(24h)
| where ResultType != 0
| summarize FailureCount = count() by UserPrincipalName, AppDisplayName
| where FailureCount > 5
| order by FailureCount desc
```

### Alert Configuration

```yaml
Azure Monitor Alerts:

Certificate Expiring:
  Query: Check cert expiry < 30 days
  Frequency: Daily
  Action: Email to security team

High SSO Failure Rate:
  Query: Failure rate > 10% per app
  Frequency: Hourly
  Action: Teams notification

Provisioning Failures:
  Query: Provisioning errors detected
  Frequency: Every 15 minutes
  Action: Create ServiceNow ticket
```

---

## Planning Best Practices

### SSO Rollout Planning

```yaml
Phase 1 - Assessment:
  - Inventory all applications
  - Identify SSO capability per app
  - Determine priority order
  - Assess integration complexity

Phase 2 - Pilot:
  - Select 3-5 apps for pilot
  - Choose mix of SSO types
  - Pilot with small user group
  - Gather feedback

Phase 3 - Production Rollout:
  - Wave-based deployment
  - Start with low-risk apps
  - Communication plan
  - Support readiness

Phase 4 - Optimization:
  - Monitor performance
  - Gather user feedback
  - Continuous improvement
```

### App Prioritization Matrix

| Priority | Criteria | Examples |
|----------|----------|----------|
| High | High user count, daily use | Microsoft 365, Salesforce |
| Medium | Department apps, regular use | HR systems, Finance apps |
| Low | Occasional use, small audience | Specialty tools |

### Change Management

```yaml
Communication:
  - Announce SSO rollout
  - Explain benefits to users
  - Provide training materials
  - Set expectations

User Training:
  - My Apps portal usage
  - Extension installation
  - Password SSO first-time setup
  - Troubleshooting basics

Support Preparation:
  - Update knowledge base
  - Train helpdesk
  - Create escalation path
  - Common issue documentation
```

---

## Testing Best Practices

### Pre-Production Testing

```yaml
Test Scenarios:
  □ New user SSO (never accessed before)
  □ Existing session SSO (already signed in)
  □ Session expiry and re-authentication
  □ Logout and new login
  □ Claims verification
  □ Group-based access
  □ MFA enforcement
  □ Conditional Access scenarios
  □ Mobile device access
  □ Multiple browser testing
```

### Test User Strategy

```yaml
Test Accounts:
  - Dedicated test users
  - Representative of user types
  - Various group memberships
  - With/without MFA registered

Avoid:
  - Testing only with admin accounts
  - Skipping external user testing
  - Ignoring edge cases
```

### Certificate Rotation Testing

```yaml
Before Production Rotation:
  1. Create new certificate
  2. Upload to test/staging environment
  3. Test SSO with new certificate
  4. Verify all scenarios work

During Rotation:
  1. Schedule maintenance window
  2. Upload to production app
  3. Set new certificate active
  4. Verify immediately

After Rotation:
  1. Monitor for 24-48 hours
  2. Check sign-in success rates
  3. Remove old certificate
  4. Update documentation
```

---

## Documentation Requirements

### SSO Configuration Documentation

```yaml
For Each Application:
  Basic Information:
    - App name and vendor
    - SSO method configured
    - Primary contact/owner
    - Support URL

  Technical Details:
    - Entity ID / Client ID
    - Reply URLs / Redirect URIs
    - Claims configuration
    - Certificate thumbprint and expiry
    - Provisioning configuration

  Access Information:
    - User assignment method
    - Groups with access
    - Conditional Access policies applied
    - Approval workflow details
```

### Runbook Contents

```yaml
Standard Runbooks:

  Certificate Rotation:
    - Steps to create new certificate
    - Upload to application
    - Testing procedure
    - Rollback steps

  Troubleshooting:
    - Common error codes
    - Resolution steps
    - Escalation path
    - Log locations

  Onboarding New App:
    - SSO method selection
    - Configuration steps
    - Testing requirements
    - Go-live checklist
```

---

## Disaster Recovery

### SSO Recovery Planning

```yaml
Scenarios:

Certificate Compromise:
  1. Generate new certificate immediately
  2. Update all dependent apps
  3. Revoke compromised certificate
  4. Investigate breach

Application Unavailable:
  1. Check app status with vendor
  2. Communicate to users
  3. Enable bypass if critical
  4. Document incident

Entra ID Service Issue:
  1. Check service health
  2. Follow Microsoft guidance
  3. User communication
  4. Document for post-incident
```

### Break-Glass Procedures

```yaml
Emergency Access:
  - Maintain emergency access accounts
  - Exclude from CA policies if needed
  - Regular testing of break-glass
  - Secure storage of credentials

Local Admin Access:
  - Some apps may need local admin
  - Document and secure
  - Regular rotation
  - Audit usage
```

---

## Summary Checklist

### Security Checklist

```yaml
□ Use SAML/OIDC over password-based when possible
□ Sign SAML response and assertion
□ Enable MFA via Conditional Access
□ Block legacy authentication
□ Rotate certificates before expiry
□ Configure session controls
□ Regular access reviews
□ Monitor sign-in logs
```

### Operations Checklist

```yaml
□ Document all SSO configurations
□ Maintain certificate expiry calendar
□ Set up monitoring and alerts
□ Create troubleshooting runbooks
□ Regular audit of unused apps
□ Test disaster recovery procedures
□ Train support staff
□ User communication plan
```

### Planning Checklist

```yaml
□ Complete application inventory
□ Prioritize by business value
□ Plan phased rollout
□ Prepare user training
□ Test thoroughly before production
□ Have rollback procedures
□ Plan for ongoing maintenance
□ Schedule regular reviews
```

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- SSO security architecture
- Best practices framework
- Operational workflow
- Planning and rollout phases

---

## Key Takeaways

- Prefer federated SSO (SAML/OIDC) over password-based methods
- Integrate SSO with Conditional Access for enhanced security
- Manage certificate lifecycle proactively
- Maintain comprehensive documentation
- Monitor continuously and alert on anomalies
- Plan rollouts carefully with proper testing
- Conduct regular access reviews
- Prepare for disaster recovery scenarios

---

## Navigation

[7.8 Provisioning with SSO](../7.8-provisioning-sso/README.md) | [Lesson 07 Overview](../README.md)
