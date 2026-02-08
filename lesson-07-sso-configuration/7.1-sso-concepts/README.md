# 7.1 SSO Concepts

## Overview

Single Sign-On (SSO) allows users to authenticate once and access multiple applications without re-entering credentials. This lesson introduces SSO fundamentals, benefits, and the different methods available in Microsoft Entra ID.

## Learning Objectives

- Understand what SSO is and why it matters
- Learn the benefits of SSO for users and organizations
- Identify the SSO methods available in Entra ID
- Understand the basic SSO flow

---

## What is Single Sign-On?

### Definition

SSO enables users to:
- Sign in once to their identity provider
- Access multiple applications seamlessly
- Avoid re-authenticating for each application

### The Problem SSO Solves

Without SSO:
```
User → App 1 Login → Enter credentials
User → App 2 Login → Enter credentials again
User → App 3 Login → Enter credentials again
...
Result: Password fatigue, security risks, poor experience
```

With SSO:
```
User → Entra ID Login → Enter credentials once
User → App 1 → Automatic access
User → App 2 → Automatic access
User → App 3 → Automatic access
...
Result: Single authentication, seamless access
```

---

## Benefits of SSO

### For Users

| Benefit | Description |
|---------|-------------|
| Reduced password fatigue | One set of credentials for all apps |
| Faster access | No repeated logins |
| Better experience | Seamless application switching |
| Fewer password resets | Less to remember |

### For Organizations

| Benefit | Description |
|---------|-------------|
| Enhanced security | Stronger authentication policies |
| Centralized control | Single point for access management |
| Reduced helpdesk costs | Fewer password reset tickets |
| Better compliance | Unified audit trail |
| Faster offboarding | Disable one account, revoke all access |

### Security Improvements

```yaml
Without SSO:
  - Users create weak passwords
  - Password reuse across apps
  - No MFA enforcement per app
  - Credentials scattered everywhere

With SSO:
  - Strong password policies enforced
  - MFA applied at IdP level
  - Conditional Access policies
  - Centralized credential management
```

---

## SSO Flow Overview

### Basic SSO Process

```
1. User accesses application
          ↓
2. Application redirects to Identity Provider (Entra ID)
          ↓
3. User authenticates (if no existing session)
          ↓
4. Entra ID issues security token
          ↓
5. Token sent to application
          ↓
6. Application validates token
          ↓
7. User granted access
```

### Session Behavior

```yaml
First Application Access:
  1. User signs in to Entra ID
  2. Session cookie created
  3. Token issued for App 1
  4. User accesses App 1

Subsequent Application Access:
  1. User accesses App 2
  2. Entra ID detects existing session
  3. No login prompt needed
  4. Token issued for App 2
  5. User accesses App 2 immediately
```

---

## SSO Methods in Entra ID

### Available Methods

| Method | Protocol | Best For |
|--------|----------|----------|
| SAML | SAML 2.0 | Enterprise applications |
| OpenID Connect | OAuth 2.0/OIDC | Modern applications |
| Password-based | N/A | Apps without federation |
| Linked | Redirect | External identity providers |
| Header-based | HTTP Headers | Legacy on-premises apps |

### Method Comparison

| Aspect | SAML | OIDC | Password-based |
|--------|------|------|----------------|
| Security | High | High | Medium |
| Modern apps | Limited | Excellent | N/A |
| Mobile support | Limited | Excellent | Limited |
| Setup complexity | Medium | Low | Low |
| Token format | XML | JWT | N/A |

### Choosing the Right Method

```yaml
Use SAML when:
  - Enterprise app supports SAML
  - Migrating from on-premises IdP
  - Vendor provides SAML metadata

Use OIDC when:
  - Building modern applications
  - Mobile or SPA applications
  - Need API access tokens

Use Password-based when:
  - App has no SSO support
  - Quick integration needed
  - Legacy applications

Use Linked when:
  - App uses external IdP
  - Redirect to another portal
```

---

## Key Terminology

### Identity Concepts

| Term | Description |
|------|-------------|
| Identity Provider (IdP) | System that authenticates users (Entra ID) |
| Service Provider (SP) | Application that relies on IdP |
| Relying Party | Same as Service Provider |
| Federation | Trust relationship between IdP and SP |

### Token Concepts

| Term | Description |
|------|-------------|
| Assertion | SAML security token |
| ID Token | OIDC token for authentication |
| Access Token | Token for API authorization |
| Claims | Attributes about the user in token |

### Configuration Concepts

| Term | Description |
|------|-------------|
| Entity ID | Unique identifier for IdP or SP |
| ACS URL | Where SAML responses are sent |
| Redirect URI | Where OIDC responses are sent |
| Reply URL | Same as ACS URL or Redirect URI |

---

## SSO and Conditional Access

### Enhanced Security

SSO integrates with Conditional Access for:
- MFA enforcement at sign-in
- Location-based restrictions
- Device compliance requirements
- Risk-based authentication

### Flow with Conditional Access

```
User → App → Entra ID → Conditional Access Evaluation
                              ↓
               ┌──────────────┴──────────────┐
               ↓                              ↓
         Policy matched                  No policy match
               ↓                              ↓
         Apply controls                  Grant access
         (MFA, compliance)
               ↓
         Controls satisfied
               ↓
         Issue token
```

---

## SSO Session Management

### Session Lifetime

```yaml
Entra ID Session:
  - Persistent browser session (90 days default)
  - Can be reduced via Conditional Access
  - Sign-in frequency controls

Application Session:
  - Managed by application
  - Can differ from IdP session
  - Token refresh behavior varies
```

### Token Lifetime

| Token Type | Default Lifetime |
|------------|------------------|
| Access Token | 60-90 minutes |
| ID Token | 60 minutes |
| Refresh Token | 90 days (sliding) |
| SAML Assertion | 60 minutes |

### Sign-Out Behavior

```yaml
Local Sign-Out:
  - Signs out of single application
  - IdP session remains active

Single Logout (SLO):
  - Signs out of all applications
  - Ends IdP session
  - Supported in SAML with SLO URL
```

---

## My Apps Portal

### Overview

The My Apps portal (https://myapps.microsoft.com) provides:
- Central launch point for SSO applications
- User-friendly application tiles
- Self-service capabilities

### Features

```yaml
Application Access:
  - All assigned applications in one place
  - Click to launch with SSO
  - Collections for organization

Self-Service:
  - Request access to applications
  - Manage group memberships
  - Update profile information
```

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- SSO flow overview
- SSO method comparison
- Session and token lifecycle

---

## Key Takeaways

- SSO enables single authentication for multiple applications
- Benefits include better security, user experience, and cost savings
- Entra ID supports SAML, OIDC, password-based, and linked SSO
- Choose OIDC for modern apps, SAML for enterprise apps
- SSO integrates with Conditional Access for enhanced security
- My Apps portal provides centralized application access

---

## Navigation

[Lesson 07 Overview](../README.md) | [7.2 SAML SSO →](../7.2-saml-sso/README.md)
