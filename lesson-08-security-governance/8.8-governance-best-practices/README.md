# 8.8 Governance Best Practices

## Overview

This sub-lesson consolidates best practices for implementing and maintaining identity governance in Microsoft Entra ID. It covers strategic guidance for Identity Protection, Privileged Identity Management (PIM), Access Reviews, Entitlement Management, emergency access procedures, separation of duties, and audit compliance. Following these practices ensures a robust, secure, and compliant identity governance framework.

## Learning Objectives

- Apply best practices for each governance component
- Implement emergency access account procedures
- Establish separation of duties controls
- Configure audit and compliance reporting
- Create an actionable implementation checklist
- Avoid common governance mistakes

---

## Identity Protection Best Practices

### Risk Policy Configuration

```yaml
User Risk Policy:
  recommendations:
    - Enable for all users (except emergency accounts)
    - Set threshold to Medium and above
    - Require password change on remediation
    - Use Conditional Access for advanced control

  settings:
    threshold: Medium
    action: Allow with password change
    exclude: Emergency access accounts

Sign-in Risk Policy:
  recommendations:
    - Enable for all users (except emergency accounts)
    - Set threshold to Medium and above
    - Require MFA for risky sign-ins
    - Block High risk when possible

  settings:
    threshold: Medium
    action: Require MFA
    high_risk: Block access
```

### Investigation Process

```yaml
Investigation Checklist:
  - [ ] Review risk detection details
  - [ ] Check sign-in logs for context
  - [ ] Verify user location and device
  - [ ] Contact user or manager if needed
  - [ ] Document investigation findings
  - [ ] Take appropriate action (confirm/dismiss)
  - [ ] Update risk state

Remediation Actions:
  true_positive:
    - Confirm user compromised
    - Reset password immediately
    - Revoke all sessions
    - Review recent activity
    - Enable additional MFA methods

  false_positive:
    - Dismiss user risk
    - Document reason
    - Consider adding location/device to trusted
    - Review detection patterns
```

### Monitoring and Alerts

| Alert Type | Trigger | Response SLA |
|------------|---------|--------------|
| High user risk | Risk level = High | 1 hour |
| High sign-in risk | Risk level = High | 4 hours |
| Multiple medium risks | 3+ in 24 hours | Same day |
| Leaked credentials | Detection triggered | Immediate |

---

## PIM Best Practices

### Role Settings by Sensitivity

```yaml
Critical Roles (Global Admin, Privileged Role Admin):
  activation:
    max_duration: 1-2 hours
    require_mfa: Yes
    require_justification: Yes
    require_ticket: Yes
    require_approval: Yes
    approvers: Security team (2+ people)

  assignment:
    allow_permanent_eligible: No
    eligible_duration: 6 months max
    allow_permanent_active: No
    require_mfa_on_assignment: Yes

High Privilege Roles (Security Admin, Exchange Admin):
  activation:
    max_duration: 4 hours
    require_mfa: Yes
    require_justification: Yes
    require_approval: Recommended

  assignment:
    eligible_duration: 12 months max
    review_frequency: Quarterly

Standard Admin Roles (User Admin, Groups Admin):
  activation:
    max_duration: 8 hours
    require_mfa: Yes
    require_justification: Yes
    require_approval: Optional

  assignment:
    eligible_duration: 12 months
    review_frequency: Semi-annually
```

### Approval Requirements

```yaml
Approval Configuration:
  approvers:
    primary: Direct security team members
    fallback: Security manager
    count: At least 2 potential approvers

  settings:
    require_reason: Yes
    timeout: 24 hours
    notify_on_pending: Yes
    notify_on_approval: Yes

  escalation:
    if_no_response: Escalate to fallback
    max_wait_time: 4 hours for critical roles
```

### Activation Monitoring

```yaml
Monitor PIM Activations:
  kql_query: |
    AuditLogs
    | where Category == "RoleManagement"
    | where ActivityDisplayName has "PIM activation"
    | project TimeGenerated,
              User = InitiatedBy.user.userPrincipalName,
              Role = TargetResources[0].displayName,
              Justification = AdditionalDetails
    | order by TimeGenerated desc

  alerts:
    - Global Admin activation (any)
    - After-hours activation
    - Multiple role activations same user
    - Activation without approval (if configured)

  review_frequency: Daily for critical roles
```

---

## Access Reviews Best Practices

### Review Frequency Guidelines

| Access Type | Frequency | Duration | Reviewers |
|-------------|-----------|----------|-----------|
| Global Administrator | Quarterly | 14 days | Security team |
| Security roles | Quarterly | 14 days | Security team |
| Privileged roles | Quarterly | 14 days | Role owners |
| Sensitive data groups | Quarterly | 14 days | Data owners |
| Application access | Semi-annually | 21 days | Managers |
| Standard groups | Annually | 30 days | Group owners |
| Guest users | Monthly | 7 days | Sponsors |

### Reviewer Selection

```yaml
Reviewer Types:
  group_owners:
    use_for: Resource-specific groups
    pros: Know members, business context
    cons: May rubber-stamp approvals

  managers:
    use_for: Employee access to applications
    pros: Know direct reports
    cons: May not know all systems

  security_team:
    use_for: Privileged access
    pros: Objective, security-focused
    cons: May lack business context

  multi_stage:
    use_for: Sensitive or high-risk access
    stage_1: Business owner/manager
    stage_2: Security team

Best Practices:
  - Use group owners for group membership
  - Use managers for application access
  - Use security team for admin roles
  - Configure fallback reviewers
  - Avoid self-review for privileged access
```

### Automation Settings

```yaml
Recommended Settings:
  auto_apply_results: Yes
  if_no_response: Remove access (for privileged)
  if_no_response: Take recommendations (for standard)
  show_recommendations: Yes
  require_reason_on_approval: Yes
  mail_notifications: Yes
  reminders: Yes

  decision_helpers:
    show_last_sign_in: Yes
    show_inactivity_days: 90

Completion Monitoring:
  - Track completion percentage daily
  - Send reminders at 50% and 75% of duration
  - Escalate incomplete reviews
  - Report on reviewer performance
```

---

## Entitlement Management Best Practices

### Catalog Organization

```yaml
Catalog Strategy:
  by_department:
    - IT Resources Catalog
    - Finance Resources Catalog
    - HR Resources Catalog
    - Engineering Resources Catalog

  by_sensitivity:
    - Public Resources Catalog
    - Internal Resources Catalog
    - Confidential Resources Catalog

  by_project:
    - Project Alpha Resources
    - Project Beta Resources

Catalog Settings:
  catalog_owners:
    - IT team for IT catalog
    - Department heads for department catalogs
    - Project leads for project catalogs

  external_users:
    - Disable for internal-only resources
    - Enable with approval for collaboration
```

### Access Package Policies

```yaml
Standard Access Package:
  requestors:
    - All employees
    - Specific groups based on role

  approval:
    stages: 1
    approvers: Manager
    timeout: 7 days

  lifecycle:
    expiration: 365 days
    access_review: Semi-annually
    auto_extend: No

Sensitive Access Package:
  requestors:
    - Specific groups only

  approval:
    stages: 2
    stage_1: Manager
    stage_2: Resource owner
    timeout: 3 days per stage

  lifecycle:
    expiration: 180 days
    access_review: Quarterly
    auto_extend: No

Guest Access Package:
  requestors:
    - Connected organizations
    - Specific external domains

  approval:
    stages: 2
    stage_1: Internal sponsor
    stage_2: Security team

  lifecycle:
    expiration: 90 days
    access_review: Monthly
    require_justification: Yes
```

---

## Emergency Access (Break-Glass) Accounts

### Account Configuration

```yaml
Break-Glass Account Requirements:
  count: Minimum 2 accounts
  naming: Avoid predictable names (not admin@, emergency@)

  authentication:
    - Cloud-only (no federation dependency)
    - FIDO2 security keys (primary)
    - Long complex password (backup)
    - Exclude from all Conditional Access
    - Exclude from MFA policies

  permissions:
    - Global Administrator role
    - Permanent active assignment (not PIM eligible)

  storage:
    - Split password/FIDO2 key in separate secure locations
    - Physical safe with dual control
    - Documented access procedures

Account Example:

  Account 1:
    upn: bg-alpha-7x9k@contoso.onmicrosoft.com
    role: Global Administrator (permanent)
    auth: FIDO2 key in safe A + password in safe B

  Account 2:
    upn: bg-beta-3m2p@contoso.onmicrosoft.com
    role: Global Administrator (permanent)
    auth: FIDO2 key in safe B + password in safe A
```

### Monitoring and Testing

```yaml
Monitoring Configuration:
  alert_on_sign_in:
    - Any sign-in attempt (success or failure)
    - Immediate notification to security team
    - SMS/phone call for critical alert

  kql_query: |
    SigninLogs
    | where UserPrincipalName in ("bg-alpha-7x9k@contoso.onmicrosoft.com",
                                   "bg-beta-3m2p@contoso.onmicrosoft.com")
    | project TimeGenerated, UserPrincipalName, ResultType,
              IPAddress, Location, AppDisplayName

Testing Schedule:
  frequency: Every 90 days
  procedure:
    - [ ] Notify security team of scheduled test
    - [ ] Retrieve credentials from secure storage
    - [ ] Sign in to Azure portal
    - [ ] Verify Global Admin access
    - [ ] Sign out and clear sessions
    - [ ] Return credentials to secure storage
    - [ ] Document test results
    - [ ] Rotate password if used
```

### Break-Glass Procedures

```yaml
When to Use:
  - All other admin accounts locked out
  - Federation service failure
  - Conditional Access misconfiguration
  - PIM service unavailable
  - Critical security incident response

Usage Procedure:
  1. Attempt normal admin access first
  2. Document reason for break-glass use
  3. Notify security team before access
  4. Retrieve credentials with witness
  5. Perform only necessary actions
  6. Document all actions taken
  7. Sign out and revoke sessions
  8. Rotate credentials immediately
  9. Conduct post-incident review
```

---

## Separation of Duties

### Conflicting Roles Matrix

| Role A | Role B | Conflict Level | Mitigation |
|--------|--------|----------------|------------|
| Global Admin | Security Admin | High | Different people |
| User Admin | Privileged Role Admin | High | Different people |
| Application Admin | Cloud App Security Admin | Medium | Review justification |
| Groups Admin | User Admin | Medium | Approval required |
| Exchange Admin | SharePoint Admin | Low | Monitor usage |

### Controls Implementation

```yaml
Separation of Duties Controls:

  administrative:
    - Document role assignment rationale
    - Require approval for conflicting roles
    - Regular review of role combinations

  technical:
    - PIM for just-in-time access
    - Conditional Access for context
    - Audit logging of all admin actions

  detective:
    - Alert on unusual role combinations
    - Review role assignments quarterly
    - Monitor cross-role activities

Implementation Checklist:
  - [ ] Define conflicting role pairs
  - [ ] Document exceptions and approvals
  - [ ] Configure alerts for violations
  - [ ] Review assignments quarterly
  - [ ] Update matrix as roles change
```

### Role Assignment Guidelines

```yaml
Assignment Principles:
  least_privilege:
    - Assign minimum required permissions
    - Use scoped roles when available
    - Prefer task-based over broad roles

  just_in_time:
    - Use PIM for all admin roles
    - Avoid permanent active assignments
    - Set appropriate activation durations

  accountability:
    - One person per role assignment
    - No shared accounts for admin access
    - Document assignment justification

  review:
    - Regular access reviews for all roles
    - Remove unused assignments promptly
    - Validate business need annually
```

---

## Audit and Compliance

### Log Retention Configuration

```yaml
Log Retention Requirements:
  entra_id_logs:
    default_retention: 30 days (portal)
    recommended: 2+ years (Log Analytics)

  log_types:
    - AuditLogs
    - SignInLogs
    - NonInteractiveUserSignInLogs
    - ServicePrincipalSignInLogs
    - ManagedIdentitySignInLogs
    - RiskyUsers
    - RiskDetections
    - UserRiskEvents

Diagnostic Settings:
  destination: Log Analytics workspace
  additional: Storage account (long-term archive)

  configuration:
    workspace_retention: 730 days (2 years)
    archive_retention: 2555 days (7 years)
```

### Compliance Reporting

```yaml
Standard Reports:
  daily:
    - High-risk sign-ins
    - PIM activations
    - Break-glass account activity

  weekly:
    - Failed sign-in summary
    - New admin role assignments
    - Access review progress

  monthly:
    - Identity Secure Score trend
    - Risk detection summary
    - Privileged access changes

  quarterly:
    - Access review completion rates
    - PIM usage statistics
    - Guest user inventory
    - Role assignment changes

Audit Trail Requirements:
  what_to_log:
    - All administrative actions
    - Role assignments and changes
    - Policy modifications
    - Access review decisions
    - Consent grants

  retention: Per regulatory requirements
  access: Restrict to audit team
  integrity: Immutable storage recommended
```

### KQL Queries for Compliance

```kusto
// Administrative Actions Summary
AuditLogs
| where TimeGenerated > ago(30d)
| where Category in ("RoleManagement", "Policy", "UserManagement")
| summarize ActionCount = count() by Category, ActivityDisplayName,
    Actor = InitiatedBy.user.userPrincipalName
| order by ActionCount desc

// Role Assignment Changes
AuditLogs
| where TimeGenerated > ago(30d)
| where Category == "RoleManagement"
| where ActivityDisplayName has "member"
| project TimeGenerated, Activity = ActivityDisplayName,
    Actor = InitiatedBy.user.userPrincipalName,
    Target = TargetResources[0].userPrincipalName,
    Role = TargetResources[0].displayName

// Privileged Operations Audit
AuditLogs
| where TimeGenerated > ago(7d)
| where Category == "RoleManagement" or Category == "Policy"
| where Result == "success"
| project TimeGenerated, Category, Activity = ActivityDisplayName,
    Actor = InitiatedBy.user.userPrincipalName
| order by TimeGenerated desc

// Consent Grant Audit
AuditLogs
| where TimeGenerated > ago(30d)
| where ActivityDisplayName has "Consent"
| project TimeGenerated, Activity = ActivityDisplayName,
    User = InitiatedBy.user.userPrincipalName,
    App = TargetResources[0].displayName,
    Permissions = AdditionalDetails
```

---

## Governance Framework

### Roles and Responsibilities

```yaml
Governance Roles:

  Identity Governance Owner:
    responsibilities:
      - Overall governance strategy
      - Policy approval
      - Risk acceptance decisions
      - Executive reporting
    typical_role: CISO or IT Director

  Identity Administrator:
    responsibilities:
      - Day-to-day administration
      - Policy implementation
      - Access review management
      - Incident response
    typical_role: Identity team lead

  Security Analyst:
    responsibilities:
      - Risk investigation
      - Alert triage
      - Compliance monitoring
      - Report generation
    typical_role: Security operations team

  Resource Owners:
    responsibilities:
      - Access review decisions
      - Access package approval
      - Business justification
      - User attestation
    typical_role: Department managers

  Auditors:
    responsibilities:
      - Compliance verification
      - Control testing
      - Finding remediation tracking
      - Report review
    typical_role: Internal audit team
```

### Governance Processes

```yaml
Regular Processes:

  daily:
    - Review high-risk alerts
    - Monitor PIM activations
    - Check break-glass account alerts

  weekly:
    - Review Identity Secure Score
    - Check access review progress
    - Review pending approvals

  monthly:
    - Governance metrics review
    - Policy effectiveness assessment
    - Guest user review

  quarterly:
    - Access review completion
    - Role assignment audit
    - Policy review and update
    - Governance board meeting

  annually:
    - Full governance assessment
    - Policy refresh
    - Training update
    - Disaster recovery test
```

### Metrics and KPIs

| Metric | Target | Measurement |
|--------|--------|-------------|
| Identity Secure Score | >80% | Monthly trend |
| Access review completion | >95% | Per review cycle |
| PIM activation compliance | 100% | Weekly check |
| High-risk remediation time | <4 hours | Per incident |
| Orphaned account cleanup | <30 days | Monthly audit |
| Role assignment documentation | 100% | Quarterly review |

---

## Implementation Checklist

### Identity Protection

```yaml
Phase 1 - Foundation:
  - [ ] Enable user risk policy (report-only first)
  - [ ] Enable sign-in risk policy (report-only first)
  - [ ] Configure notifications for security team
  - [ ] Document investigation procedures
  - [ ] Train security team on investigation

Phase 2 - Enforcement:
  - [ ] Enable policies in enforced mode
  - [ ] Set appropriate risk thresholds
  - [ ] Integrate with Conditional Access
  - [ ] Configure automated remediation
  - [ ] Establish SLAs for risk response

Phase 3 - Optimization:
  - [ ] Tune based on false positive rates
  - [ ] Implement advanced detections
  - [ ] Automate investigation workflows
  - [ ] Regular policy review and update
```

### PIM Implementation

```yaml
Phase 1 - Critical Roles:
  - [ ] Enable PIM for Global Administrator
  - [ ] Configure approval requirements
  - [ ] Set activation duration limits
  - [ ] Enable MFA on activation
  - [ ] Train eligible admins

Phase 2 - Expand Coverage:
  - [ ] Enable PIM for Security roles
  - [ ] Enable PIM for Exchange/SharePoint admins
  - [ ] Configure role-specific settings
  - [ ] Set up access reviews for PIM roles
  - [ ] Document activation procedures

Phase 3 - Full Deployment:
  - [ ] Enable PIM for all admin roles
  - [ ] Enable PIM for Azure resources
  - [ ] Configure groups in PIM
  - [ ] Implement monitoring and alerting
  - [ ] Regular access reviews
```

### Access Reviews

```yaml
Phase 1 - Priority Reviews:
  - [ ] Configure admin role reviews (quarterly)
  - [ ] Configure sensitive group reviews
  - [ ] Train reviewers on process
  - [ ] Set up completion monitoring
  - [ ] Document review procedures

Phase 2 - Expand Scope:
  - [ ] Configure application access reviews
  - [ ] Configure guest user reviews (monthly)
  - [ ] Enable auto-apply for routine reviews
  - [ ] Configure multi-stage reviews for sensitive access
  - [ ] Implement reviewer reminders

Phase 3 - Automation:
  - [ ] Enable recommendations
  - [ ] Configure fallback reviewers
  - [ ] Automate reporting
  - [ ] Integrate with governance dashboards
  - [ ] Regular process optimization
```

### Emergency Access

```yaml
Setup:
  - [ ] Create 2+ break-glass accounts
  - [ ] Configure FIDO2 + password authentication
  - [ ] Exclude from all Conditional Access
  - [ ] Assign Global Administrator (permanent)
  - [ ] Store credentials securely (split control)

Operations:
  - [ ] Configure sign-in alerts
  - [ ] Document usage procedures
  - [ ] Train authorized personnel
  - [ ] Schedule quarterly testing
  - [ ] Establish rotation procedures
```

---

## Common Mistakes to Avoid

### Identity Protection Mistakes

```yaml
Mistakes:
  1. Dismissing risks without investigation
     fix: Always investigate before dismissing

  2. Setting thresholds too high
     fix: Start with Medium and adjust based on data

  3. Not excluding emergency accounts
     fix: Add break-glass accounts to exclusions

  4. Ignoring low-risk detections
     fix: Review all detections for patterns

  5. No integration with Conditional Access
     fix: Use CA for advanced control and reporting
```

### PIM Mistakes

```yaml
Mistakes:
  1. Setting activation duration too long
     fix: 1-4 hours for critical roles, 8 hours max

  2. Not requiring approval for critical roles
     fix: Always require approval for Global Admin

  3. Allowing permanent active assignments
     fix: Use eligible only, activate when needed

  4. Too many eligible users
     fix: Limit eligible users, use groups if needed

  5. Not monitoring activations
     fix: Configure alerts for all activations
```

### Access Reviews Mistakes

```yaml
Mistakes:
  1. Review periods too long
     fix: 14-21 days max for most reviews

  2. Wrong reviewers selected
     fix: Choose reviewers with context

  3. Not enabling auto-apply
     fix: Enable for routine reviews

  4. Ignoring completion rates
     fix: Monitor and follow up on pending

  5. No consequences for non-response
     fix: Configure appropriate defaults (remove access)
```

### Emergency Access Mistakes

```yaml
Mistakes:
  1. Including in Conditional Access policies
     fix: Explicitly exclude from ALL policies

  2. Using federation-dependent accounts
     fix: Use cloud-only accounts only

  3. Storing credentials together
     fix: Split password and FIDO2 key

  4. Not testing regularly
     fix: Test every 90 days minimum

  5. No sign-in monitoring
     fix: Alert on any sign-in attempt
```

### General Governance Mistakes

```yaml
Mistakes:
  1. No clear ownership
     fix: Assign governance owner and document RACI

  2. Inconsistent policies
     fix: Standardize across all governance areas

  3. Poor documentation
     fix: Document all procedures and decisions

  4. Reactive only approach
     fix: Proactive monitoring and regular reviews

  5. No metrics tracking
     fix: Define KPIs and track monthly
```

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- Governance framework overview
- Best practice areas and relationships
- Implementation lifecycle phases
- Roles and responsibilities matrix

---

## Key Takeaways

- Enable Identity Protection with appropriate risk thresholds and investigation processes
- Configure PIM with approval requirements, MFA, and time-limited activations
- Conduct access reviews at appropriate frequencies with the right reviewers
- Organize entitlement management catalogs by department or sensitivity
- Maintain properly configured and tested emergency access accounts
- Implement separation of duties controls for conflicting roles
- Configure comprehensive audit logging with appropriate retention
- Establish clear governance roles, responsibilities, and processes
- Use implementation checklists to ensure complete coverage
- Avoid common mistakes through proper planning and monitoring

---

## Navigation

[← 8.7 Identity Secure Score](../8.7-identity-secure-score/README.md) | [Lesson 08 Overview](../README.md)
