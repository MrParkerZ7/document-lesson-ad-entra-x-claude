# 4.2 Multi-Factor Authentication (MFA)

## Overview

Multi-Factor Authentication (MFA) requires users to verify their identity using two or more authentication factors. This significantly increases security by ensuring that stolen passwords alone cannot grant access.

## Learning Objectives

- Understand MFA concepts and factors
- Compare different MFA methods
- Enable MFA via Conditional Access (recommended)
- Configure MFA registration campaigns
- Migrate from legacy per-user MFA

---

## What is MFA?

MFA combines multiple verification factors:

| Factor Type | Description | Examples |
|-------------|-------------|----------|
| **Something you know** | Knowledge-based | Password, PIN |
| **Something you have** | Possession-based | Phone, security key |
| **Something you are** | Biometric | Fingerprint, face |

---

## MFA Methods Comparison

| Method | Pros | Cons | Best For |
|--------|------|------|----------|
| Authenticator App (Push) | Secure, convenient | Requires smartphone | Most users |
| Authenticator App (TOTP) | Works offline | Manual code entry | All users |
| SMS | Easy to use | Less secure (SIM swap) | Basic scenarios |
| Voice Call | No smartphone needed | Less secure | Accessibility |
| FIDO2 Key | Most secure | Cost, can be lost | High security |

---

## Enable MFA Methods

### Legacy: Per-User MFA (Not Recommended)

```
Microsoft Entra ID → Users → Per-user MFA
```

> **Warning**: Per-user MFA is legacy. Use Conditional Access for new deployments.

### Recommended: Conditional Access MFA

```
Microsoft Entra ID → Security → Conditional Access → New policy
```

---

## Example Conditional Access Policy

### Require MFA for All Users

```yaml
Name: POL-CA-Require-MFA-All-Users

Assignments:
  Users:
    Include: All users
    Exclude:
      - Break-glass accounts
      - Service accounts

  Cloud apps:
    Include: All cloud apps

  Conditions:
    Locations:
      Exclude: Trusted locations (optional)

Access controls:
  Grant:
    - Require multi-factor authentication

Session:
  Sign-in frequency: 7 days

State: Report-only → Enabled
```

### Require MFA for Risky Sign-ins

```yaml
Name: POL-CA-MFA-Risky-SignIn

Assignments:
  Users:
    Include: All users

  Conditions:
    Sign-in risk: Medium and above

Access controls:
  Grant:
    - Require multi-factor authentication
```

---

## MFA Registration Campaign

Force users to register for MFA within a specific timeframe.

### Configure Registration

```
Microsoft Entra ID → Security → Authentication methods → Registration campaign
```

### Settings

| Setting | Description | Recommendation |
|---------|-------------|----------------|
| State | Enable campaign | Enabled |
| Target | Users/groups | All users |
| Snooze duration | Days user can postpone | 3 days |
| Enforcement | Hard deadline | Set date |

### PowerShell Configuration

```powershell
# Get current registration campaign settings
Get-MgPolicyAuthenticationMethodPolicy | Select-Object -ExpandProperty RegistrationEnforcement

# Update registration enforcement
$params = @{
    registrationEnforcement = @{
        authenticationMethodsRegistrationCampaign = @{
            snoozeDurationInDays = 3
            state = "enabled"
            includeTargets = @(
                @{
                    id = "all_users"
                    targetType = "group"
                }
            )
        }
    }
}
Update-MgPolicyAuthenticationMethodPolicy -BodyParameter $params
```

---

## Migration from Legacy MFA

### Migration States

| State | Description |
|-------|-------------|
| Pre-migration | Legacy MFA settings active |
| Migration in Progress | Both policies evaluated |
| Migration Complete | Only new policy active |

### Check Migration Status

```
Microsoft Entra ID → Security → Authentication methods → Policies → Manage migration
```

### Migration Steps

1. **Audit current state**: Document existing per-user MFA settings
2. **Create CA policies**: Build equivalent Conditional Access policies
3. **Enable report-only**: Test CA policies without enforcement
4. **Start migration**: Enable both policy evaluation
5. **Complete migration**: Switch to new policies only
6. **Clean up**: Remove legacy per-user MFA settings

---

## MFA Best Practices

### Security Recommendations

- [ ] Use Conditional Access, not per-user MFA
- [ ] Exclude break-glass accounts from MFA policies
- [ ] Disable SMS/Voice for privileged accounts
- [ ] Enable number matching for Authenticator
- [ ] Implement phishing-resistant MFA for admins

### User Experience

- [ ] Start with report-only policies
- [ ] Communicate changes to users in advance
- [ ] Provide self-service registration portal
- [ ] Allow reasonable snooze periods
- [ ] Offer multiple MFA methods as backup

---

## Troubleshooting MFA

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| User can't receive codes | Phone/app issue | Verify phone number, reinstall app |
| MFA prompt every time | Session expired | Check sign-in frequency settings |
| MFA not triggered | Policy misconfigured | Review CA policy conditions |
| "Access denied" | Account blocked | Check user's MFA status |

### View MFA Status

```powershell
# Get user's MFA registration status
Get-MgUserAuthenticationMethod -UserId "user@contoso.com"

# Get MFA registration report
Get-MgReportAuthenticationMethodUserRegistrationDetail
```

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of MFA concepts.

---

## Key Takeaways

1. MFA combines multiple factors for stronger security
2. Use Conditional Access instead of per-user MFA
3. Authenticator app is the recommended method for most users
4. Avoid SMS/Voice for privileged accounts
5. Plan and communicate MFA rollout carefully

---

## Next Sub-Lesson

[4.3 Microsoft Authenticator →](../4.3-microsoft-authenticator/README.md)
