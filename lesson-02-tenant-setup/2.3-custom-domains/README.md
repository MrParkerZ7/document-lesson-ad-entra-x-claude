# 2.3 Custom Domain Setup

## Overview

Custom domains allow users to sign in with your organization's domain (user@yourcompany.com) instead of the default .onmicrosoft.com domain. This sub-lesson covers domain verification and configuration.

## Learning Objectives

- Understand why custom domains matter
- Add and verify custom domains
- Configure DNS records
- Set a primary domain

---

## Why Custom Domains?

| Benefit | Description |
|---------|-------------|
| **Professional** | user@contoso.com vs user@contoso.onmicrosoft.com |
| **Branding** | Consistent with your organization |
| **User-friendly** | Easier for users to remember |
| **Required** | Some integrations require custom domains |

---

## Domain Setup Process

### Step 1: Add Domain

```
Microsoft Entra ID → Custom domain names → Add custom domain
```

Enter your domain: `contoso.com`

### Step 2: Get Verification Record

Microsoft provides a unique verification value:

```
MS=ms12345678
```

### Step 3: Add DNS Record

Choose one verification method:

**Option A: TXT Record (Recommended)**
```
Type:  TXT
Host:  @
Value: MS=ms12345678
TTL:   3600
```

**Option B: MX Record**
```
Type:     MX
Host:     @
Value:    ms12345678.msv1.invalid
Priority: 32767
TTL:      3600
```

### Step 4: Verify Domain

- Wait for DNS propagation (15-60 minutes, up to 72 hours)
- Click "Verify" in Azure Portal
- Domain status changes to "Verified"

### Step 5: Set as Primary (Optional)

```
Select domain → Make primary
```

---

## DNS Records for Microsoft Services

After verification, add these records for full functionality:

| Service | Type | Host | Value |
|---------|------|------|-------|
| Exchange (mail) | MX | @ | contoso-com.mail.protection.outlook.com |
| Autodiscover | CNAME | autodiscover | autodiscover.outlook.com |
| SPF | TXT | @ | v=spf1 include:spf.protection.outlook.com -all |
| DKIM | CNAME | selector1._domainkey | selector1-contoso-com._domainkey.contoso.onmicrosoft.com |
| DKIM | CNAME | selector2._domainkey | selector2-contoso-com._domainkey.contoso.onmicrosoft.com |

---

## Multiple Domains

You can add multiple domains:

```
Primary:    contoso.com
Additional: contoso.co.uk
            contoso.de
            subsidiary.com
```

Each domain needs separate verification.

---

## Subdomain Considerations

| Scenario | Behavior |
|----------|----------|
| Verify `contoso.com` | Subdomains auto-verified |
| Verify `sub.contoso.com` | Only that subdomain verified |

---

## PowerShell Commands

```powershell
# Connect
Connect-MgGraph -Scopes "Domain.ReadWrite.All"

# List all domains
Get-MgDomain | Select-Object Id, IsDefault, IsVerified

# Add new domain
New-MgDomain -Id "contoso.com"

# Get verification DNS records
Get-MgDomain -DomainId "contoso.com" |
    Select-Object -ExpandProperty ServiceConfigurationRecords

# Verify domain
Confirm-MgDomain -DomainId "contoso.com"

# Set as primary
Update-MgDomain -DomainId "contoso.com" -IsDefault $true
```

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual overview of domain setup.

---

## Key Takeaways

1. Custom domains provide professional user identities
2. Verification proves domain ownership via DNS
3. TXT record verification is recommended
4. Additional DNS records needed for email services
5. One domain can be set as primary

---

## Next Sub-Lesson

[2.4 Company Branding →](../2.4-company-branding/README.md)
