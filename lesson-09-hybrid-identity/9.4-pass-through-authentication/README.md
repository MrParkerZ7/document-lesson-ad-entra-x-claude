# 9.4 Pass-through Authentication (PTA)

## Overview

Pass-through Authentication (PTA) is a hybrid authentication method that validates user passwords directly against on-premises Active Directory. Unlike Password Hash Synchronization, passwords never leave your on-premises environment, providing organizations with enhanced control over credential security while still enabling cloud authentication.

## Learning Objectives

- Understand how PTA validates passwords on-premises
- Recognize the benefits and considerations of PTA
- Install and configure PTA agents
- Implement high availability for authentication reliability

---

## How Pass-through Authentication Works

### Authentication Flow

When a user signs in to Microsoft Entra ID with PTA enabled:

```
1. User enters credentials at Entra ID sign-in page
2. Entra ID encrypts the password with the agent's public key
3. Encrypted password placed in queue for on-premises agent
4. PTA Agent retrieves and decrypts the password
5. Agent validates credentials against Active Directory
6. AD returns validation result (success/failure + reason)
7. Agent forwards result to Entra ID
8. User granted or denied access
```

### Detailed Flow Diagram

```yaml
Authentication Steps:
  Step 1 - User Sign-in:
    Location: "Microsoft 365 portal or Azure app"
    Action: "User enters username and password"

  Step 2 - Password Encryption:
    Location: "Microsoft Entra ID"
    Action: "Password encrypted using PTA agent's public key"

  Step 3 - Queue Placement:
    Location: "Azure Service Bus"
    Action: "Encrypted credentials placed in tenant-specific queue"

  Step 4 - Agent Retrieval:
    Location: "On-premises PTA Agent"
    Action: "Agent pulls request via outbound HTTPS (443)"

  Step 5 - Decryption:
    Location: "On-premises PTA Agent"
    Action: "Agent decrypts password with private key"

  Step 6 - AD Validation:
    Location: "Active Directory Domain Controller"
    Action: "Win32 LogonUser API validates credentials"

  Step 7 - Result Return:
    Location: "Microsoft Entra ID"
    Action: "Validation result returned to cloud"

  Step 8 - Access Decision:
    Location: "Microsoft Entra ID"
    Action: "User authenticated or denied based on result"
```

---

## Benefits of Pass-through Authentication

### Security Benefits

| Benefit | Description |
|---------|-------------|
| **No cloud passwords** | Password hashes never stored in Microsoft Entra ID |
| **On-premises policy enforcement** | AD password policies applied in real-time |
| **Real-time account state** | Disabled, locked, or expired accounts denied immediately |
| **Logon hour restrictions** | AD logon hours enforced |
| **Immediate password changes** | New passwords effective immediately |

### Operational Benefits

| Benefit | Description |
|---------|-------------|
| **Simple deployment** | Lightweight agents, no infrastructure changes |
| **Outbound only** | No inbound firewall rules required |
| **Automatic updates** | Agents auto-update |
| **No password sync lag** | Unlike PHS, no sync delay |

### Compliance Benefits

```yaml
Regulatory Alignment:
  - Passwords remain on-premises (data residency)
  - Existing audit infrastructure maintained
  - Password never transmitted in clear text
  - No password derivatives stored in cloud
```

---

## Considerations and Requirements

### Technical Considerations

| Consideration | Details |
|---------------|---------|
| **Agent availability** | Authentication fails if all agents offline |
| **Network dependency** | Requires reliable connectivity to Entra ID |
| **DC accessibility** | Agents must reach domain controllers |
| **No offline access** | Unlike PHS, no cloud fallback |
| **Latency** | Adds slight delay vs PHS |

### Infrastructure Requirements

```yaml
Server Requirements:
  Operating System: "Windows Server 2016 or later"
  Memory: "4 GB RAM minimum"
  Storage: "Minimal (agent only)"
  Network: "Outbound HTTPS (443) to Azure"
  Domain: "Domain-joined server"

Active Directory:
  Forest Level: "Windows Server 2003 or later"
  Domain Level: "Windows Server 2003 or later"

Network Requirements:
  Outbound URLs:
    - "*.msappproxy.net"
    - "*.servicebus.windows.net"
    - "login.windows.net"
    - "secure.aadcdn.microsoftonline-p.com"
    - "*.microsoftonline.com"
    - "*.microsoftonline-p.com"
    - "*.msauth.net"
    - "*.msftauth.net"
    - "*.msauthimages.net"
```

### Security Considerations

| Factor | Recommendation |
|--------|----------------|
| **Agent server security** | Treat as Tier 0 (like DCs) |
| **Service account** | Use managed service accounts |
| **Agent updates** | Enable automatic updates |
| **Monitoring** | Monitor agent health in portal |

---

## Installing PTA Agents

### Primary Agent Installation

The primary PTA agent is installed with Azure AD Connect:

```yaml
Installation via Azure AD Connect:
  1. Run Azure AD Connect wizard
  2. Select "Change user sign-in"
  3. Choose "Pass-through Authentication"
  4. Optionally enable "Seamless SSO"
  5. Enter Global Admin credentials
  6. Complete wizard

Result:
  - PTA agent installed on Connect server
  - Agent registered with tenant
  - Certificate provisioned for encryption
```

### Additional Agent Installation

For high availability, install additional agents:

```yaml
Download Additional Agent:
  Portal Path: "Microsoft Entra ID > Microsoft Entra Connect > Pass-through authentication"
  Action: "Download agent"
  File: "AADConnectAuthAgentSetup.exe"

Installation Steps:
  1. Run installer on additional server
  2. Accept license terms
  3. Sign in with Global Administrator
  4. Installation completes automatically
  5. Agent appears in portal within minutes
```

### PowerShell Installation

```powershell
# Download agent (via browser or script)
$agentUrl = "https://download.msappproxy.net/subscription/xxx/connector/download"

# Silent installation (after download)
.\AADConnectAuthAgentSetup.exe /quiet

# Verify installation
Get-Service -Name "AzureADConnectAuthenticationAgent"
```

---

## High Availability Configuration

### Recommended Architecture

```yaml
Minimum Deployment:
  Agents: 3
  Reason: "N+1 redundancy for maintenance and failures"

Placement Strategy:
  Agent 1: "Azure AD Connect server"
  Agent 2: "Separate member server (same site)"
  Agent 3: "Member server (secondary site/DR)"

Geographic Distribution:
  - Place agents near domain controllers
  - Consider network latency to DCs
  - Distribute across fault domains
```

### High Availability Topology

| Agent | Server | Location | Purpose |
|-------|--------|----------|---------|
| Agent 1 | AADC01 | Primary DC | Azure AD Connect + PTA |
| Agent 2 | APP01 | Primary DC | Dedicated PTA agent |
| Agent 3 | APP02 | Secondary DC | DR/Failover agent |
| Agent 4 | APP03 | Primary DC | Additional capacity (optional) |

### Load Distribution

```yaml
Agent Behavior:
  Load Balancing: "Automatic by Entra ID"
  Method: "Round-robin across healthy agents"
  Failover: "Automatic if agent unresponsive"
  Health Check: "Every 10 minutes"

No Additional Configuration:
  - No load balancer required
  - No port configuration needed
  - Agents self-register
```

---

## Monitoring and Troubleshooting

### Portal Monitoring

```yaml
Health Status Path:
  "Microsoft Entra ID > Microsoft Entra Connect > Pass-through authentication"

Dashboard Shows:
  - Agent name and version
  - Server hostname
  - Status (Active/Inactive)
  - Last contact time
```

### Agent Health Commands

```powershell
# Check agent service status
Get-Service -Name "AzureADConnectAuthenticationAgent"

# View agent event logs
Get-WinEvent -LogName "Application" |
    Where-Object {$_.ProviderName -eq "AzureADConnectAuthenticationAgentService"} |
    Select-Object -First 20

# Test connectivity
Test-NetConnection -ComputerName "autologon.microsoftazuread-sso.com" -Port 443
```

### Common Issues and Solutions

| Issue | Cause | Solution |
|-------|-------|----------|
| Agent shows Inactive | Network connectivity | Check firewall, proxy settings |
| Authentication fails | All agents offline | Verify agent services running |
| Slow authentication | DC unreachable | Check DC health and connectivity |
| Agent won't register | Certificate issue | Reinstall agent |
| Intermittent failures | Agent overloaded | Add more agents |

### Event Log Analysis

```yaml
Key Event IDs:
  12000: "Agent started successfully"
  12001: "Authentication request received"
  12002: "Authentication succeeded"
  12003: "Authentication failed - invalid credentials"
  12004: "Authentication failed - account disabled"
  12005: "Authentication failed - account locked"
  12006: "Agent connection error"
```

---

## Best Practices

### Deployment Best Practices

- [ ] Deploy minimum 3 PTA agents for production
- [ ] Place agents on domain-joined Windows Servers
- [ ] Ensure agents can reach domain controllers
- [ ] Keep agents updated (enable auto-update)
- [ ] Document agent server locations

### Security Best Practices

- [ ] Treat PTA agent servers as Tier 0
- [ ] Limit administrative access to agent servers
- [ ] Monitor agent health continuously
- [ ] Enable Microsoft Defender on agent servers
- [ ] Regular security patching

### Operational Best Practices

- [ ] Enable PHS as backup authentication method
- [ ] Set up alerts for agent failures
- [ ] Test DR scenarios regularly
- [ ] Maintain documentation of agent configuration
- [ ] Plan agent updates during maintenance windows

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of the PTA authentication flow and high availability architecture.

---

## Key Takeaways

1. **PTA keeps passwords on-premises** - credentials never stored in the cloud
2. **Real-time validation** - password changes and account states apply immediately
3. **Deploy 3+ agents** - ensure high availability for authentication
4. **Outbound only** - no inbound firewall rules needed
5. **Consider PHS backup** - enable as fallback for agent failures

---

[← 9.3 Password Hash Sync](../9.3-password-hash-sync/README.md) | [9.5 Federation with AD FS →](../9.5-federation-adfs/README.md)
