# 8.4 Entitlement Management

## Overview

Entitlement Management in Microsoft Entra ID is an identity governance capability that automates access request workflows, access assignments, reviews, and expiration. It enables organizations to manage the identity and access lifecycle at scale for both internal employees and external users through access packages, catalogs, and policies.

## Learning Objectives

- Understand the purpose and components of Entitlement Management
- Create and manage catalogs to organize resources
- Build access packages that bundle multiple resources
- Configure request policies with approval workflows
- Implement lifecycle management with expiration and reviews
- Enable self-service access for internal and external users
- Use PowerShell for automation

---

## Why Entitlement Management?

### The Access Management Challenge

```yaml
Traditional access management problems:
  User onboarding:
    - IT creates tickets for each resource
    - Multiple approvers contacted separately
    - Days or weeks to get productive

  Access changes:
    - Role changes require manual updates
    - New projects need new requests
    - No single view of all access

  Offboarding:
    - Access lingers after project ends
    - External users forgotten
    - Audit findings for stale access

  Governance:
    - No visibility into who has what
    - Difficult to demonstrate compliance
    - Time-consuming access reviews
```

### Entitlement Management Solution

```
┌───────────────────────────────────────────────────────────────────┐
│                   Entitlement Management                           │
│                                                                    │
│   ┌──────────────┐    ┌──────────────┐    ┌──────────────┐       │
│   │   Catalog    │───▶│   Access     │───▶│   Policy     │       │
│   │              │    │   Package    │    │              │       │
│   │  Resources   │    │   Bundled    │    │  Who/How     │       │
│   │  organized   │    │   access     │    │  to request  │       │
│   └──────────────┘    └──────────────┘    └──────────────┘       │
│                              │                                     │
│                              ▼                                     │
│   ┌──────────────────────────────────────────────────────────┐   │
│   │                  User Self-Service                        │   │
│   │                myaccess.microsoft.com                     │   │
│   │   Browse packages → Request → Approval → Access granted   │   │
│   └──────────────────────────────────────────────────────────┘   │
│                              │                                     │
│                              ▼                                     │
│   ┌──────────────────────────────────────────────────────────┐   │
│   │                Automatic Lifecycle                        │   │
│   │   Expiration → Review → Extension or Removal              │   │
│   └──────────────────────────────────────────────────────────┘   │
└───────────────────────────────────────────────────────────────────┘
```

---

## Core Components

### Component Overview

| Component | Description | Purpose |
|-----------|-------------|---------|
| Catalog | Container for resources and packages | Organize and delegate management |
| Access Package | Bundle of resource roles | Simplify access requests |
| Resource Role | Specific permission on a resource | Define what access is granted |
| Policy | Rules for requesting access | Control who and how |
| Assignment | Granted access to a user | Track who has what |

### How Components Relate

```yaml
Catalog:
  Name: Marketing Catalog
  Owner: Marketing IT Team

  Contains Resources:
    - Groups: Marketing-All, Marketing-Leads
    - Applications: Adobe Creative Cloud, HubSpot
    - SharePoint: Marketing Assets Site

  Contains Access Packages:
    Package 1: Marketing Team Member
      - Marketing-All (Member)
      - Adobe Creative Cloud (User)
      - Marketing Assets Site (Member)

    Package 2: Marketing Lead
      - Marketing-All (Member)
      - Marketing-Leads (Member)
      - Adobe Creative Cloud (Admin)
      - HubSpot (Admin)
      - Marketing Assets Site (Owner)
```

---

## Catalogs

### Purpose of Catalogs

```yaml
Catalogs enable:
  Organization:
    - Group related resources together
    - Separate by department or project
    - Maintain clear ownership

  Delegation:
    - Delegate to catalog owners
    - Catalog creators can add resources
    - Access package managers create packages

  Control:
    - Limit resource visibility
    - Control who can create packages
    - Manage external access settings
```

### Default Catalog

```yaml
General catalog:
  Created automatically: Yes
  Purpose: Quick start for small organizations
  Owner: Global administrators
  Recommendation: Create custom catalogs for scale
```

### Create a Catalog

```
Identity Governance → Entitlement management → Catalogs → New catalog
```

### Catalog Configuration

```yaml
Basic settings:
  Name: IT Infrastructure Catalog
  Description: Access packages for IT infrastructure resources
  Enabled: Yes
  Enabled for external users: No

Catalog roles:
  Owners:
    - Full control of catalog
    - Add/remove resources and packages
    - Manage other catalog roles

  Catalog readers:
    - View catalog contents
    - Cannot modify anything

  Access package managers:
    - Create and manage access packages
    - Cannot add resources to catalog

  Access package assignment managers:
    - Create and manage assignments
    - Cannot modify packages
```

### Catalog Role Comparison

| Role | Create Packages | Manage Resources | Manage Assignments | View Catalog |
|------|-----------------|------------------|-------------------|--------------|
| Owner | Yes | Yes | Yes | Yes |
| Access package manager | Yes | No | Yes | Yes |
| Assignment manager | No | No | Yes | Yes |
| Reader | No | No | No | Yes |

### PowerShell: Create Catalog

```powershell
Connect-MgGraph -Scopes "EntitlementManagement.ReadWrite.All"

$params = @{
    displayName = "IT Infrastructure Catalog"
    description = "Access packages for IT infrastructure resources"
    isExternallyVisible = $false
}

New-MgEntitlementManagementCatalog -BodyParameter $params
```

---

## Resource Roles

### Adding Resources to Catalogs

Resources must be added to a catalog before they can be included in access packages.

### Supported Resource Types

| Resource Type | Role Types | Example |
|---------------|------------|---------|
| Security groups | Member, Owner | Add to GRP-SEC-Developers |
| Microsoft 365 groups | Member, Owner | Add to Teams |
| Applications | App-defined roles | User, Admin, Reader |
| SharePoint sites | Member, Visitor, Owner | Access to SharePoint |

### Add Groups to Catalog

```
Catalogs → [Catalog] → Resources → Add resources → Groups
```

```yaml
Select groups:
  - GRP-SEC-Finance-Team
  - GRP-SEC-Finance-Admins
  - GRP-M365-Finance-Collaboration

Role types available:
  Security groups: Member
  M365 groups: Member, Owner
```

### Add Applications to Catalog

```yaml
Steps:
  1. Select Applications from resource type
  2. Choose enterprise applications
  3. App roles become available in packages

Example:
  Application: Salesforce
  Roles available:
    - Standard User
    - Administrator
    - Read Only User
```

### Add SharePoint Sites to Catalog

```yaml
Steps:
  1. Select SharePoint Online sites
  2. Choose from registered sites
  3. Standard roles available

Roles:
  - Member: Contribute access
  - Visitor: Read-only access
  - Owner: Full control
```

### PowerShell: Add Resources to Catalog

```powershell
# Get catalog ID
$catalog = Get-MgEntitlementManagementCatalog -Filter "displayName eq 'IT Infrastructure Catalog'"

# Add a group to catalog
$groupParams = @{
    catalogId = $catalog.Id
    requestType = "AdminAdd"
    accessPackageResource = @{
        originId = "group-object-id"
        originSystem = "AadGroup"
    }
}

New-MgEntitlementManagementResourceRequest -BodyParameter $groupParams
```

---

## Access Packages

### What is an Access Package?

```yaml
Access Package definition:
  Purpose: Bundle multiple resources for a specific role or project
  Contains:
    - One or more resource roles
    - One or more policies
    - Lifecycle settings

  Benefits:
    - One request = multiple resources
    - Consistent access for similar users
    - Single point for reviews and expiration
```

### Create Access Package

```
Catalogs → [Catalog] → Access packages → New access package
```

### Step 1: Basics

```yaml
Name: New Employee Starter Pack
Description: |
  Standard access for new employees including:
  - Corporate email and calendar
  - Intranet access
  - Basic applications

Catalog: General Catalog
Hidden: No
```

### Step 2: Resource Roles

```yaml
Resources included:
  Groups:
    Resource: GRP-SEC-All-Employees
    Role: Member

    Resource: GRP-M365-Corporate-Users
    Role: Member

  Applications:
    Resource: Corporate Intranet
    Role: Employee

    Resource: HR Portal
    Role: User

  SharePoint:
    Resource: Company Policies Site
    Role: Visitor
```

### Step 3: Request Policies

See detailed section below.

### Step 4: Requestor Information

```yaml
Questions for requestors:
  Question 1:
    Text: What is your start date?
    Type: Date
    Required: Yes

  Question 2:
    Text: Who is your hiring manager?
    Type: Text
    Required: Yes

  Question 3:
    Text: Which department are you joining?
    Type: Multiple choice
    Required: Yes
    Choices:
      - Engineering
      - Marketing
      - Sales
      - Finance
      - HR
```

### Step 5: Lifecycle

See lifecycle section below.

### PowerShell: Create Access Package

```powershell
$catalog = Get-MgEntitlementManagementCatalog -Filter "displayName eq 'IT Infrastructure Catalog'"

$params = @{
    displayName = "Developer Access Package"
    description = "Access for software developers"
    catalogId = $catalog.Id
    isHidden = $false
}

$package = New-MgEntitlementManagementAccessPackage -BodyParameter $params

# Add resource role to package
$resourceRole = @{
    accessPackageId = $package.Id
    accessPackageResourceRole = @{
        originId = "Member_group-id"
        originSystem = "AadGroup"
        accessPackageResource = @{
            id = "resource-id-from-catalog"
        }
    }
    accessPackageResourceScope = @{
        originId = "group-id"
        originSystem = "AadGroup"
    }
}

New-MgEntitlementManagementAccessPackageResourceRoleScope -BodyParameter $resourceRole
```

---

## Request Policies

### Policy Types

| Policy Type | Who Can Request | Use Case |
|-------------|-----------------|----------|
| No requests | No one | Package under construction |
| Specific users/groups | Named users | Internal team access |
| All members | Any employee | General company resources |
| Connected organizations | External partners | B2B collaboration |
| All users | Anyone with link | Public resources |

### Create Request Policy

```yaml
Policy name: Employee Request Policy

Who can request:
  Type: For users in your directory
  Selection: All members (All Users)

  OR

  Type: For users in your directory
  Selection: Specific users and groups
  Users: GRP-SEC-Full-Time-Employees

Enable new requests: Yes
```

### Approval Workflow

```yaml
Require approval: Yes

Approval stages:
  Stage 1:
    Approvers:
      Type: Manager as approver
      OR
      Type: Choose specific approvers
      Approvers: finance-approvers@contoso.com

    Days to make decision: 7
    If no response: Request denied

  Stage 2 (optional):
    Approvers:
      Type: Choose specific approvers
      Approvers: it-security@contoso.com

    Days to make decision: 5
    If no response: Request denied

Alternate approvers:
  Enabled: Yes
  Add after days: 3
  Alternates: backup-approvers@contoso.com

Justification required: Yes
```

### Approval Workflow Diagram

```
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│   Request    │─────▶│   Stage 1    │─────▶│   Stage 2    │
│   Submitted  │      │   Approval   │      │   Approval   │
└──────────────┘      └──────┬───────┘      └──────┬───────┘
                             │                      │
                    ┌────────┴────────┐    ┌───────┴────────┐
                    ▼                 ▼    ▼                ▼
               ┌─────────┐      ┌─────────┐      ┌─────────┐
               │ Approved│      │ Denied  │      │ Approved│
               └────┬────┘      └─────────┘      └────┬────┘
                    │                                  │
                    └──────────────┬──────────────────┘
                                   ▼
                           ┌─────────────┐
                           │   Access    │
                           │   Granted   │
                           └─────────────┘
```

### Enable Requestor Information

```yaml
Require requestor to provide:
  Justification: Yes
  Custom questions: Yes
  Localized questions: Optional (multiple languages)
  Verified ID credentials: Optional (for high-security)
```

### PowerShell: Create Policy

```powershell
$package = Get-MgEntitlementManagementAccessPackage -Filter "displayName eq 'Developer Access Package'"

$policyParams = @{
    displayName = "Employee Request Policy"
    description = "Policy for internal employees"
    accessPackageId = $package.Id
    canExtend = $true
    durationInDays = 365
    expirationDateTime = $null
    requestorSettings = @{
        scopeType = "AllExistingDirectoryMemberUsers"
        acceptRequests = $true
    }
    requestApprovalSettings = @{
        isApprovalRequired = $true
        isApprovalRequiredForExtension = $true
        isRequestorJustificationRequired = $true
        approvalMode = "Serial"
        approvalStages = @(
            @{
                approvalStageTimeOutInDays = 7
                isApproverJustificationRequired = $true
                isEscalationEnabled = $false
                primaryApprovers = @(
                    @{
                        "@odata.type" = "#microsoft.graph.singleUser"
                        userId = "approver-user-id"
                    }
                )
            }
        )
    }
}

New-MgEntitlementManagementAssignmentPolicy -BodyParameter $policyParams
```

---

## Lifecycle Management

### Expiration Settings

```yaml
Expiration types:
  Never:
    Access does not expire
    Use case: Permanent employee access
    Recommendation: Combine with access reviews

  Number of days:
    Access expires after X days
    Use case: Project-based access
    Example: 90 days, 180 days, 365 days

  Specific date:
    Access expires on exact date
    Use case: Contract end dates
    Example: 2024-12-31

  Hours (preview):
    Short-term access
    Use case: Temporary elevated access
    Example: 8 hours
```

### Configure Expiration

```yaml
Access package lifecycle:
  Expiration:
    Type: Number of days
    Value: 365

  Users can request extension: Yes
  Require approval for extension: Yes
  Extension period: 180 days
```

### Access Reviews for Packages

```yaml
Access reviews:
  Enabled: Yes

  Review frequency:
    Type: Quarterly
    Options: Monthly, Quarterly, Semi-annually, Annually

  Reviewers:
    Type: Manager
    Fallback: Self-review

  If reviewer doesn't respond:
    Action: Remove access

  Show recommendations: Yes
  Require reason: Yes
```

### Lifecycle Timeline

```
Day 1                  Day 270               Day 365
│                      │                      │
▼                      ▼                      ▼
┌──────────────────────────────────────────────────────────────┐
│                    Access Period (365 days)                   │
└──────────────────────────────────────────────────────────────┘
        │                      │                      │
        ▼                      ▼                      ▼
   Access Review          Extension           Expiration
   (quarterly)           Request Window       (automatic)
                         (30 days before)
```

### Separation of Duties

```yaml
Incompatible access packages:
  Purpose: Prevent users from having conflicting access

  Example:
    Package A: Accounts Payable
    Package B: Accounts Receivable
    Relationship: Incompatible

  Result:
    User with Package A cannot request Package B
    User must release one before obtaining other
```

### PowerShell: Configure Lifecycle

```powershell
# Update package with lifecycle settings
$updateParams = @{
    accessPackageId = $package.Id
    displayName = "Employee Request Policy"
    durationInDays = 365
    expirationDateTime = $null
    accessReviewSettings = @{
        isEnabled = $true
        recurrenceType = "quarterly"
        reviewerType = "Manager"
        startDateTime = (Get-Date).AddDays(90).ToString("yyyy-MM-ddTHH:mm:ssZ")
        durationInDays = 14
        isAccessRecommendationEnabled = $true
        isApprovalJustificationRequired = $true
        accessReviewTimeoutBehavior = "removeAccess"
    }
}

Update-MgEntitlementManagementAssignmentPolicy -AccessPackageAssignmentPolicyId $policyId -BodyParameter $updateParams
```

---

## User Experience

### My Access Portal

```
URL: https://myaccess.microsoft.com
```

### User Journey

```yaml
Step 1 - Browse:
  Navigate to: Access packages tab
  View: Available packages user can request
  Filter: By catalog or search

Step 2 - Request:
  Select package → Request access
  Provide:
    - Justification
    - Answers to custom questions
    - Start date (if applicable)

Step 3 - Wait for approval:
  Status: Pending approval
  Notifications: Email updates on progress
  Track: View request status in portal

Step 4 - Access granted:
  Status: Delivered
  Access: Resources available immediately
  Duration: As configured in policy

Step 5 - Renewal (optional):
  Before expiration: Request extension
  Re-approval: If configured

Step 6 - Expiration or removal:
  Automatic: Access removed at expiration
  Manual: User can release access early
```

### My Access Portal Views

| Tab | Purpose |
|-----|---------|
| Access packages | Browse and request packages |
| Request history | View past requests and status |
| Approvals | Pending approvals (for approvers) |
| Access reviews | Complete assigned reviews |

### Requesting Access - User View

```yaml
Package: Developer Tools Access
Description: Access to development resources

Your request:
  Justification:
    "Joining the mobile team as iOS developer,
    need access to development tools and repositories."

  Start date: Immediate

  Custom questions:
    Team: Mobile Development
    Project: iOS Banking App
    Manager approval: john.smith@contoso.com

Submit request → Pending approval
```

---

## Connected Organizations

### External User Access

```yaml
Purpose: Grant access to users from partner organizations
Benefits:
  - Self-service for external users
  - Automatic B2B guest creation
  - Lifecycle management for guests
  - Sponsor approval workflows
```

### Create Connected Organization

```
Identity Governance → Entitlement management → Connected organizations → Add
```

### Connected Organization Configuration

```yaml
Name: Contoso Partners Ltd
Description: External consulting partner

Identity source:
  Type: Azure AD tenant
  Domain: partners.contoso.com
  Tenant ID: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx

  OR

  Type: Any Azure AD tenant (any organization)

  OR

  Type: Social identity providers

State: Configured
  Proposed: Needs sponsor approval
  Configured: Approved and active

Sponsors:
  Internal: partner-managers@contoso.com
  External: admin@partners.contoso.com
```

### External User Policy

```yaml
Policy for connected organizations:
  Who can request:
    Type: For users not in your directory
    Connected organizations: Specific organizations
    Selection: Contoso Partners Ltd

    OR

    Connected organizations: All connected organizations

    OR

    All users (including new guests)

  Approval required: Yes (highly recommended)
  Approvers: Internal sponsor + manager

  Information required:
    Business justification: Yes
    Verified email: Yes
```

### PowerShell: Create Connected Organization

```powershell
$params = @{
    displayName = "Contoso Partners Ltd"
    description = "External consulting partner"
    identitySources = @(
        @{
            "@odata.type" = "#microsoft.graph.azureActiveDirectoryTenant"
            tenantId = "partner-tenant-id"
            displayName = "Contoso Partners"
        }
    )
    state = "configured"
}

New-MgEntitlementManagementConnectedOrganization -BodyParameter $params
```

---

## PowerShell Management

### Common Operations

```powershell
# Connect with required permissions
Connect-MgGraph -Scopes @(
    "EntitlementManagement.ReadWrite.All",
    "User.Read.All",
    "Group.Read.All"
)

# List all catalogs
Get-MgEntitlementManagementCatalog | Select-Object DisplayName, Id, State

# List access packages in a catalog
$catalog = Get-MgEntitlementManagementCatalog -Filter "displayName eq 'IT Catalog'"
Get-MgEntitlementManagementAccessPackage -Filter "catalogId eq '$($catalog.Id)'"

# Get package assignments
$package = Get-MgEntitlementManagementAccessPackage -Filter "displayName eq 'Developer Access'"
Get-MgEntitlementManagementAssignment -Filter "accessPackageId eq '$($package.Id)'"

# Create direct assignment (admin assignment)
$assignmentParams = @{
    requestType = "AdminAdd"
    accessPackageAssignment = @{
        targetId = "user-object-id"
        assignmentPolicyId = "policy-id"
        accessPackageId = $package.Id
    }
}
New-MgEntitlementManagementAssignmentRequest -BodyParameter $assignmentParams

# Remove assignment
$removeParams = @{
    requestType = "AdminRemove"
    accessPackageAssignment = @{
        id = "assignment-id"
    }
}
New-MgEntitlementManagementAssignmentRequest -BodyParameter $removeParams
```

### Bulk Assignment Script

```powershell
# Bulk assign users to an access package
$packageName = "New Employee Starter Pack"
$policyName = "Employee Request Policy"
$users = Import-Csv -Path ".\new_employees.csv"

$package = Get-MgEntitlementManagementAccessPackage -Filter "displayName eq '$packageName'"
$policy = Get-MgEntitlementManagementAssignmentPolicy -Filter "displayName eq '$policyName'"

foreach ($user in $users) {
    $assignmentParams = @{
        requestType = "AdminAdd"
        accessPackageAssignment = @{
            targetId = $user.ObjectId
            assignmentPolicyId = $policy.Id
            accessPackageId = $package.Id
        }
        justification = "Bulk assignment for new employee onboarding"
    }

    try {
        New-MgEntitlementManagementAssignmentRequest -BodyParameter $assignmentParams
        Write-Host "Assigned package to: $($user.UserPrincipalName)" -ForegroundColor Green
    }
    catch {
        Write-Host "Failed to assign to: $($user.UserPrincipalName) - $($_.Exception.Message)" -ForegroundColor Red
    }
}
```

### Reporting Script

```powershell
# Export all access package assignments
$packages = Get-MgEntitlementManagementAccessPackage -All
$report = @()

foreach ($package in $packages) {
    $assignments = Get-MgEntitlementManagementAssignment -Filter "accessPackageId eq '$($package.Id)'" -ExpandProperty target

    foreach ($assignment in $assignments) {
        $report += [PSCustomObject]@{
            PackageName = $package.DisplayName
            UserName = $assignment.Target.DisplayName
            UserEmail = $assignment.Target.Email
            State = $assignment.State
            ExpireDateTime = $assignment.ExpireDateTime
            AssignedDateTime = $assignment.AssignedDateTime
        }
    }
}

$report | Export-Csv -Path ".\entitlement_assignments.csv" -NoTypeInformation
```

---

## Best Practices

### Catalog Organization

```yaml
Do:
  - Create catalogs by business unit or function
  - Assign clear ownership to business stakeholders
  - Use descriptive names and descriptions
  - Limit catalog access appropriately

Don't:
  - Put everything in the General catalog
  - Let catalogs become unmanaged
  - Create too many small catalogs
  - Mix unrelated resources
```

### Access Package Design

```yaml
Do:
  - Design packages around roles or projects
  - Include all resources needed for a task
  - Use clear naming conventions
  - Document package purpose in description
  - Start simple, iterate as needed

Don't:
  - Create packages with single resources
  - Make packages too broad
  - Duplicate resources across many packages
  - Forget to update packages when roles change
```

### Policy Configuration

```yaml
Do:
  - Require approval for sensitive access
  - Set appropriate expiration periods
  - Enable access reviews for all packages
  - Use multi-stage approval for external users
  - Collect meaningful requestor information

Don't:
  - Allow self-approval for privileged access
  - Set indefinite access without reviews
  - Skip justification requirements
  - Ignore approval timeout handling
```

### Lifecycle Management

```yaml
Do:
  - Set expiration for project-based access
  - Enable extension requests before expiration
  - Configure quarterly reviews at minimum
  - Remove denied access automatically
  - Monitor assignment reports regularly

Don't:
  - Grant permanent access without reviews
  - Allow unattended extensions
  - Let expired access linger
  - Forget about inactive assignments
```

### Naming Conventions

```yaml
Catalogs:
  Format: [Department/Function] Catalog
  Examples:
    - Engineering Catalog
    - Finance Catalog
    - Marketing Catalog
    - Project-Phoenix Catalog

Access Packages:
  Format: [Role/Project] - [Access Level]
  Examples:
    - Developer - Standard Access
    - Developer - Admin Access
    - Project Phoenix - Team Member
    - External Contractor - Limited Access

Policies:
  Format: [Audience] - [Type]
  Examples:
    - Employees - Self Service
    - Managers - Direct Assign
    - Partners - Sponsored Access
    - Contractors - Time Limited
```

---

## Common Scenarios

### Scenario 1: Employee Onboarding

```yaml
Package: New Employee Starter Pack
Resources:
  - All-Employees group
  - Corporate email access
  - Intranet application
  - HR self-service portal
  - Training platform

Policy:
  Requestors: HR team (admin assignment)
  Approval: None (pre-approved by HR)
  Expiration: Never (permanent employee)
  Review: Annually
```

### Scenario 2: Project-Based Access

```yaml
Package: Project Alpha - Team Member
Resources:
  - Project Alpha Teams channel
  - Project documentation site
  - Project management tool
  - Development environment

Policy:
  Requestors: All employees
  Approval: Project manager
  Expiration: Project end date or 180 days
  Review: Monthly
```

### Scenario 3: External Consultant Access

```yaml
Package: Consultant - Finance Systems
Resources:
  - Finance-Consultants group
  - Financial reporting app (read-only)
  - Budget SharePoint site

Policy:
  Requestors: Connected organization (Partner Firm)
  Approval:
    Stage 1: Internal sponsor
    Stage 2: Finance director
  Expiration: Contract end date
  Review: Weekly
```

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- Catalog and access package structure
- Request and approval workflow
- Lifecycle management process
- Connected organization integration

---

## Key Takeaways

- Entitlement Management automates the access request lifecycle from request to expiration
- Catalogs organize resources and enable delegated management to business owners
- Access packages bundle multiple resources into a single requestable unit
- Policies control who can request access and the approval workflow required
- Lifecycle settings ensure access expires and is reviewed regularly
- The My Access portal provides self-service for internal and external users
- Connected organizations enable controlled access for B2B partners
- Use PowerShell for bulk operations and reporting

---

## Navigation

[← 8.3 Access Reviews](../8.3-access-reviews/README.md) | [8.5 Security Monitoring →](../8.5-security-monitoring/README.md)
