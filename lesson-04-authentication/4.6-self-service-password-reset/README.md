# 4.6 Self-Service Password Reset (SSPR)

## Overview

Self-Service Password Reset (SSPR) allows users to reset their own passwords without contacting the helpdesk. This reduces IT support costs, improves user productivity, and provides 24/7 availability for password management.

## Learning Objectives

- Understand SSPR benefits and requirements
- Configure SSPR settings
- Set up authentication methods for SSPR
- Enable password writeback for hybrid environments
- Monitor and troubleshoot SSPR

---

## Benefits of SSPR

| Benefit | Description |
|---------|-------------|
| **Reduced helpdesk costs** | 20-50% fewer password reset tickets |
| **24/7 availability** | Users can reset passwords anytime |
| **Improved productivity** | No waiting for IT support |
| **Enhanced security** | Strong authentication required |
| **Audit trail** | All resets are logged |

---

## SSPR Requirements

### Licensing

| Feature | License Required |
|---------|-----------------|
| Cloud-only SSPR | Microsoft 365 Business Premium, E3, E5 |
| Password writeback | Azure AD Premium P1 or P2 |
| Combined registration | Any license |

### User Requirements

- Registered authentication methods
- Valid license assigned
- Not a member of excluded groups

---

## Enable SSPR

### Portal Navigation

```
Microsoft Entra ID → Password reset → Properties
```

### Enablement Options

| Option | Description |
|--------|-------------|
| None | SSPR disabled |
| Selected | Enabled for specific groups |
| All | Enabled for all users |

### Recommended Rollout

```
Phase 1: Pilot group (IT, early adopters)
Phase 2: Expand to departments
Phase 3: Enable for all users
```

---

## Authentication Methods

### Configure Methods

```
Microsoft Entra ID → Password reset → Authentication methods
```

### Available Methods

| Method | Description | Recommendation |
|--------|-------------|----------------|
| Mobile app notification | Push to Authenticator | Recommended |
| Mobile app code | TOTP from Authenticator | Recommended |
| Email | Code to alternate email | Good |
| Mobile phone | SMS code | Acceptable |
| Office phone | Voice call | Limited use |
| Security questions | Knowledge-based | Not recommended |

### Settings

```yaml
Number of methods required to reset: 2

Methods available to users:
  - Mobile app notification: Yes
  - Mobile app code: Yes
  - Email: Yes
  - Mobile phone: Yes
  - Office phone: No
  - Security questions: No (not recommended)
```

### PowerShell Configuration

```powershell
# Get current SSPR configuration
Get-MgPolicyAuthenticationMethodPolicy |
    Select-Object -ExpandProperty AuthenticationMethodConfigurations

# View SSPR settings
Get-MgPolicyAuthorizationPolicy |
    Select-Object -ExpandProperty DefaultUserRolePermissions
```

---

## Registration Settings

### Portal Configuration

```
Microsoft Entra ID → Password reset → Registration
```

### Settings

```yaml
Require users to register when signing in: Yes
Number of days before users are asked to re-confirm: 180
```

### Registration Campaign

Force users to register for SSPR and MFA:

```
Microsoft Entra ID → Security → Authentication methods → Registration campaign
```

---

## Notifications

### Portal Configuration

```
Microsoft Entra ID → Password reset → Notifications
```

### Settings

```yaml
Notify users on password resets: Yes
Notify all admins when other admins reset their password: Yes
```

---

## Password Writeback (Hybrid)

### Overview

Password writeback enables:
- SSPR passwords written to on-premises AD
- Real-time password synchronization
- Account unlock without password change

### Prerequisites

- Azure AD Connect installed
- Azure AD Premium P1 or P2 license
- Appropriate AD permissions

### Enable in Azure AD Connect

1. Open Azure AD Connect wizard
2. Select "Customize synchronization options"
3. Enable "Password writeback"
4. Complete configuration

### Enable in Portal

```
Microsoft Entra ID → Password reset → On-premises integration
```

```yaml
Write back passwords to your on-premises directory: Yes
Allow users to unlock accounts without resetting their password: Yes
```

### PowerShell Verification

```powershell
# Check password writeback status
Get-ADSyncAADPasswordResetConfiguration -Connector "contoso.com"

# Verify sync service account permissions
Get-ADSyncConnectorAccount
```

---

## Combined Registration

### Overview

Single registration experience for both MFA and SSPR.

### Enable Combined Registration

```
Microsoft Entra ID → User settings → User feature settings
```

```yaml
Users can use the combined security information registration experience: All
```

### User Portal

Users access: [aka.ms/mysecurityinfo](https://aka.ms/mysecurityinfo)

---

## SSPR Process

### User Experience

```
1. User goes to https://passwordreset.microsoftonline.com
2. Enters username
3. Completes CAPTCHA
4. Verifies identity (2 methods required)
5. Enters new password
6. Password updated (and written back if hybrid)
```

### Methods Verification Example

```
Method 1: "Enter code sent to +1 *** ***1234"
User: Enters SMS code

Method 2: "Approve notification on your authenticator app"
User: Approves push notification

Result: Password reset successful
```

---

## Customization

### Helpdesk Link

```
Microsoft Entra ID → Password reset → Customization
```

```yaml
Customize helpdesk link: Yes
Custom helpdesk email or URL: https://helpdesk.contoso.com/password
```

### Custom Helpdesk Message

Provide users with alternative support options if SSPR fails.

---

## Monitoring SSPR

### Password Reset Activity

```
Microsoft Entra ID → Password reset → Audit logs
```

### Key Metrics

| Metric | Description |
|--------|-------------|
| Resets completed | Successful password resets |
| Resets failed | Failed reset attempts |
| Registration completed | Users who registered |
| Registration required | Users who need to register |

### PowerShell Reporting

```powershell
# Get password reset audit logs
$startDate = (Get-Date).AddDays(-30).ToString("yyyy-MM-dd")
Get-MgAuditLogDirectoryAudit -Filter "activityDisplayName eq 'Reset password (self-service)' and activityDateTime ge $startDate" |
    Select-Object ActivityDisplayName, ActivityDateTime, InitiatedBy, Result

# Get registration status
Get-MgReportAuthenticationMethodUserRegistrationDetail |
    Where-Object { $_.IsSsprRegistered -eq $false } |
    Select-Object UserPrincipalName, IsSsprRegistered, IsMfaRegistered
```

---

## Troubleshooting

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| "You can't reset your own password" | SSPR not enabled for user | Add user to SSPR-enabled group |
| "Authentication methods not registered" | User hasn't registered | Direct user to aka.ms/mysecurityinfo |
| "Password doesn't meet policy" | Password complexity | Inform user of requirements |
| "Writeback failed" | Sync or permissions issue | Check Azure AD Connect logs |

### Verification Commands

```powershell
# Check if user is registered for SSPR
Get-MgReportAuthenticationMethodUserRegistrationDetail -UserId "user@contoso.com" |
    Select-Object IsSsprRegistered, IsSsprEnabled, MethodsRegistered

# Check user's authentication methods
Get-MgUserAuthenticationMethod -UserId "user@contoso.com"
```

---

## Best Practices

### Security

- [ ] Require 2 authentication methods
- [ ] Avoid security questions
- [ ] Enable notifications for all resets
- [ ] Monitor for suspicious activity
- [ ] Require registration at sign-in

### User Experience

- [ ] Provide clear registration instructions
- [ ] Communicate SSPR availability
- [ ] Include helpdesk link for exceptions
- [ ] Enable combined registration

### Hybrid Environments

- [ ] Enable password writeback
- [ ] Allow account unlock without reset
- [ ] Test writeback before broad rollout
- [ ] Monitor Azure AD Connect health

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of SSPR.

---

## Key Takeaways

1. SSPR reduces helpdesk costs and improves user experience
2. Require at least 2 authentication methods
3. Avoid security questions (easily compromised)
4. Enable password writeback for hybrid environments
5. Monitor usage and enforce registration

---

## Next Sub-Lesson

[4.7 Authentication Policies →](../4.7-authentication-policies/README.md)
