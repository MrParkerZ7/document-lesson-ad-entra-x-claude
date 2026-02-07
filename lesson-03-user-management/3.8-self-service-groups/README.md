# 3.8 Self-Service Group Management

## Overview

Self-service group management allows users to create and manage their own groups, reducing IT overhead while maintaining governance controls.

## Learning Objectives

- Enable self-service group management
- Configure naming policies
- Set up group expiration
- Manage access to group creation

---

## Self-Service Capabilities

| Capability | Description |
|------------|-------------|
| Create groups | Users create their own groups |
| Manage membership | Owners add/remove members |
| Request to join | Users request group access |
| Approve requests | Owners approve membership |

---

## Enabling Self-Service Groups

### Via Portal

```
Microsoft Entra ID → Groups → General settings
```

### Configuration Settings

| Setting | Options | Recommendation |
|---------|---------|----------------|
| Users can create security groups | Yes/No | Restrict (No) |
| Users can create M365 groups | Yes/No | Allow with controls |
| Group owners can manage membership | Yes/No | Yes |
| Restrict group creation | All/Selected | Selected users only |

---

## Restricting Group Creation

### Allow Only Specific Users

```powershell
# Create a group of allowed creators
$allowedCreators = New-MgGroup -DisplayName "GRP-AllowedGroupCreators" `
    -MailEnabled:$false `
    -SecurityEnabled:$true `
    -MailNickname "allowed-creators"

# Set the restriction
$settings = @{
    "EnableGroupCreation" = $false
    "GroupCreationAllowedGroupId" = $allowedCreators.Id
}
# Apply via directory settings
```

### Via Portal

```
Microsoft Entra ID → Groups → General settings
1. Set "Users can create Microsoft 365 groups" = No
2. Select group of allowed creators
```

---

## Group Naming Policy

### Purpose

- Enforce consistent naming conventions
- Prevent inappropriate names
- Add organizational prefixes/suffixes

### Configure Naming Policy

```
Microsoft Entra ID → Groups → Naming policy
```

### Policy Components

#### Prefixes and Suffixes

| Type | Example | Result |
|------|---------|--------|
| String prefix | "GRP-" | GRP-Marketing |
| String suffix | "-Team" | Marketing-Team |
| Attribute prefix | [Department] | Sales-Team |
| Attribute suffix | [Country] | Marketing-US |

#### Blocked Words

```
Common blocked words:
- CEO, CFO, CTO
- Admin, Administrator
- Payroll, HR-Confidential
- [Custom organization-specific terms]
```

### Example Configuration

```
Prefix: GRP-[Department]-
Suffix: -[Country]

User creates: "Marketing Team"
Result: GRP-Sales-Marketing Team-US
```

---

## Group Expiration Policy

### Purpose

- Automatically clean up unused groups
- Reduce orphaned resources
- Maintain governance

### Configure Expiration

```
Microsoft Entra ID → Groups → Expiration
```

### Settings

| Setting | Options | Description |
|---------|---------|-------------|
| Group lifetime | 180, 365, custom days | Time until expiration |
| Email contact | Owners, specific email | Who gets notified |
| Enable for | All, selected groups | Scope of policy |

### Expiration Process

```
Day 0: Group created
Day X-30: First renewal notification
Day X-15: Second notification
Day X-1: Final warning
Day X: Group expires (soft delete)
Day X+30: Permanent deletion
```

### Renewal Options

- Owner clicks renewal link in email
- Owner accesses group (activity extends life)
- Admin renews manually

---

## Self-Service Join Requests

### Enable Join Requests

```
Group settings → Allow users to request to join
```

### Request Flow

```
1. User finds group in directory
2. User clicks "Request to join"
3. Owner receives notification
4. Owner approves/denies request
5. User receives notification
```

### PowerShell Configuration

```powershell
# Update group to allow join requests
Update-MgGroup -GroupId $groupId -AdditionalProperties @{
    "membershipType" = "Assigned"
    "accessType" = "ControlledByOwner"
}
```

---

## Access Reviews for Groups

### Purpose

Periodic review of group membership to ensure accuracy.

### Configure Access Reviews

```
Microsoft Entra ID → Identity Governance → Access Reviews → New review
```

### Review Settings

| Setting | Options |
|---------|---------|
| Reviewers | Owners, specific users, self-review |
| Frequency | Weekly, monthly, quarterly, yearly |
| Duration | 3-14 days |
| Action on no response | Remove access, keep access |

---

## Best Practices

### Governance

- [ ] Restrict group creation to specific users
- [ ] Implement naming policies
- [ ] Enable group expiration
- [ ] Conduct regular access reviews
- [ ] Require group owners

### User Experience

- [ ] Provide self-service training
- [ ] Document naming conventions
- [ ] Enable join requests for public groups
- [ ] Set reasonable expiration periods

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of self-service groups.

---

## Key Takeaways

1. Self-service reduces IT overhead
2. Naming policies ensure consistency
3. Expiration policies prevent group sprawl
4. Restrict creation to trusted users
5. Access reviews maintain governance

---

## Lesson 03 Complete

You've completed Lesson 03! You now understand:
- User types and creation methods
- User properties and lifecycle
- Group types and dynamic membership
- Group-based licensing
- Self-service group management

---

## Next Lesson

[Lesson 04: Authentication Methods →](../../lesson-04-authentication/README.md)
