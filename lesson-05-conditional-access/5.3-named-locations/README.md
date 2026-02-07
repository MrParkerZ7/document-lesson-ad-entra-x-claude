# 5.3 Named Locations

## Overview

Named locations allow you to define trusted networks, countries, and IP ranges that can be used in Conditional Access policies. This enables location-based access decisions such as allowing access from corporate networks or blocking access from high-risk countries.

## Learning Objectives

- Understand the types of named locations
- Configure IP-based trusted locations
- Set up country-based locations
- Use named locations in Conditional Access policies
- Implement GPS-based location verification

---

## Types of Named Locations

| Type | Description | Use Case |
|------|-------------|----------|
| **IP ranges** | Define specific IP addresses or CIDR ranges | Corporate networks, VPN exit points |
| **Countries/Regions** | Select countries by geographic location | Block high-risk countries |
| **GPS** | Device-reported coordinates | Mobile device location verification |

---

## Accessing Named Locations

```
Microsoft Entra ID → Security → Conditional Access → Named locations
```

---

## IP-Based Locations

### Creating an IP Location

```yaml
Name: Corporate-Network
Type: IP ranges location

Mark as trusted location: Yes

IP ranges (IPv4 or IPv6):
  - 203.0.113.0/24      # Main office
  - 198.51.100.0/24     # Branch office
  - 192.0.2.50/32       # VPN exit point
```

### Trusted Location Benefits

When marked as trusted:
- Can exclude from MFA requirements
- Used to define "inside corporate network"
- Reduces friction for users on trusted networks

### Common IP Location Examples

| Location Name | IP Ranges | Purpose |
|--------------|-----------|---------|
| Corporate-HQ | 10.0.0.0/8 (internal), 203.0.113.0/24 (external) | Main headquarters |
| Branch-Offices | 198.51.100.0/24, 192.0.2.0/24 | Regional offices |
| VPN-Endpoints | Individual IPs | VPN concentrators |
| Partner-Network | Specific ranges | Trusted partner IPs |

### MFA Trusted IPs (Legacy)

Legacy per-user MFA trusted IPs:
```
Microsoft Entra ID → Security → Conditional Access → Named locations
→ Configure MFA trusted IPs
```

> **Note**: Migrate to Conditional Access named locations for new deployments.

---

## Country-Based Locations

### Creating a Country Location

```yaml
Name: Blocked-Countries
Type: Countries/Regions

Determine location by: IP address (geo-IP)

Selected countries:
  - North Korea
  - Iran
  - Russia
  - [Other high-risk countries]
```

### Allowed Countries Example

```yaml
Name: Allowed-Operating-Countries
Type: Countries/Regions

Selected countries:
  - United States
  - Canada
  - United Kingdom
  - Germany
  - [Other operating countries]
```

### Location Detection Methods

| Method | Description | Accuracy |
|--------|-------------|----------|
| **IP address (geo-IP)** | Uses IP geolocation databases | Good for most scenarios |
| **GPS coordinates** | Device-reported location | High accuracy, requires device support |

---

## GPS-Based Locations

### Enabling GPS Verification

GPS locations provide higher accuracy by using device-reported coordinates:

```yaml
Name: Headquarters-GPS
Type: Countries/Regions

Determine location by: GPS coordinates

Include unknown countries/regions: No
```

### GPS Requirements

- Device must have GPS capability
- User must grant location permission
- Only works with supported apps and devices
- Authenticator app can report GPS location

### GPS Use Cases

| Scenario | Benefit |
|----------|---------|
| Verify physical presence | Prevent VPN location spoofing |
| Secure facility access | Ensure user is on-premises |
| Compliance requirements | Prove location for regulatory needs |

---

## Using Named Locations in Policies

### Example 1: Skip MFA on Trusted Network

```yaml
Name: POL-CA-Skip-MFA-Trusted-Network

Users: All users
Cloud apps: All cloud apps

Conditions:
  Locations:
    Include: All locations
    Exclude: All trusted locations

Grant:
  Require multi-factor authentication

# Effect: MFA required except from trusted locations
```

### Example 2: Block Untrusted Countries

```yaml
Name: POL-CA-Block-Untrusted-Countries

Users: All users
Cloud apps: All cloud apps

Conditions:
  Locations:
    Include: All locations
    Exclude:
      - All trusted locations
      - Named location: Allowed-Operating-Countries

Grant:
  Block access

# Effect: Block access from countries not in allowed list
```

### Example 3: Require Compliant Device Outside Network

```yaml
Name: POL-CA-Compliant-Outside-Network

Users: All users
Cloud apps: Office 365

Conditions:
  Locations:
    Include: All locations
    Exclude: Named location: Corporate-Network

Grant:
  Require device to be marked as compliant

# Effect: Require compliance when outside corporate network
```

### Example 4: Block Specific Countries

```yaml
Name: POL-CA-Block-High-Risk-Countries

Users: All users
Cloud apps: All cloud apps

Conditions:
  Locations:
    Include: Named location: Blocked-Countries

Grant:
  Block access

# Effect: Block all access from specified countries
```

---

## Best Practices

### IP Locations

- [ ] Document all IP ranges and their purposes
- [ ] Include both internal and external IPs for offices
- [ ] Update ranges when network changes occur
- [ ] Use CIDR notation for efficient range management
- [ ] Review trusted IPs regularly

### Country Locations

- [ ] Create an "Allowed Countries" list rather than blocking individual countries
- [ ] Consider business travel patterns
- [ ] Review geo-IP accuracy limitations
- [ ] Plan for legitimate access from blocked regions

### General

- [ ] Use descriptive names (e.g., "Corporate-HQ-Seattle" not "Location1")
- [ ] Combine with other conditions for defense in depth
- [ ] Test policies in Report-only mode
- [ ] Document the business reason for each location

---

## Limitations

| Limitation | Description | Workaround |
|------------|-------------|------------|
| IPv6 support | Limited in some scenarios | Include both IPv4 and IPv6 |
| Geo-IP accuracy | Not 100% accurate | Use GPS for high-security scenarios |
| VPN detection | Users can appear from VPN location | Combine with device compliance |
| Dynamic IPs | Home/mobile IPs change | Use country-based or GPS |

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- Named location types
- IP range configuration
- Country-based locations
- Location-based policy flow

---

## Key Takeaways

- Named locations define trusted networks and geographic regions
- IP-based locations are ideal for corporate networks and VPNs
- Country-based locations help block high-risk regions
- GPS verification provides higher accuracy for mobile scenarios
- Always test location-based policies in Report-only mode
- Combine location conditions with other signals for defense in depth

---

## Next Steps

Continue to [5.4 Common Policy Scenarios](../5.4-common-policy-scenarios/README.md) to learn about implementing common Conditional Access use cases.
