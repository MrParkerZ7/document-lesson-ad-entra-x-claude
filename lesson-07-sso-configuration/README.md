# Lesson 07: Single Sign-On (SSO) Configuration

## Overview

This lesson covers how to configure Single Sign-On (SSO) in Microsoft Entra ID. SSO enables users to sign in once and access multiple applications without re-authenticating, improving user experience and security.

## Learning Objectives

By the end of this lesson, you will:
- Understand SSO concepts and authentication flows
- Configure SAML-based SSO with claims mapping
- Set up OIDC-based SSO and token configuration
- Implement password-based and linked SSO
- Customize the My Apps portal
- Configure SSO for on-premises applications
- Test and troubleshoot SSO issues
- Implement provisioning with SSO
- Apply SSO best practices

---

## Sub-Lessons

### [7.1 SSO Concepts](./7.1-sso-concepts/README.md)
Introduction to Single Sign-On concepts, benefits, authentication methods, and flow diagrams.

### [7.2 SAML SSO](./7.2-saml-sso/README.md)
SAML-based SSO configuration including assertions, claims mapping, and certificate management.

### [7.3 OIDC SSO](./7.3-oidc-sso/README.md)
OpenID Connect SSO configuration, token settings, and comparison with SAML.

### [7.4 Password & Linked SSO](./7.4-password-linked-sso/README.md)
Password-based SSO for non-federation apps and linked SSO for external providers.

### [7.5 My Apps Portal](./7.5-my-apps-portal/README.md)
My Apps portal customization, collections, and user self-service features.

### [7.6 SSO for On-Premises](./7.6-sso-on-premises/README.md)
Application Proxy SSO including Integrated Windows Authentication and Kerberos delegation.

### [7.7 Testing & Troubleshooting](./7.7-testing-troubleshooting/README.md)
SSO testing methods, common errors, diagnostic tools, and resolution strategies.

### [7.8 Provisioning with SSO](./7.8-provisioning-sso/README.md)
SCIM provisioning, attribute mappings, and provisioning lifecycle with SSO.

### [7.9 SSO Best Practices](./7.9-sso-best-practices/README.md)
Security, operational, and planning best practices for SSO implementations.

---

## Key Topics Covered

| Sub-Lesson | Key Topics |
|------------|------------|
| 7.1 SSO Concepts | SSO benefits, authentication methods, flow diagrams |
| 7.2 SAML SSO | SAML assertions, Entity ID, ACS URL, claims, certificates |
| 7.3 OIDC SSO | ID tokens, access tokens, scopes, PKCE, MSAL |
| 7.4 Password & Linked | Password vault, field detection, linked providers |
| 7.5 My Apps Portal | Collections, customization, self-service groups |
| 7.6 On-Premises SSO | App Proxy, IWA, KCD, header-based auth |
| 7.7 Testing | SAML tracer, sign-in logs, diagnostic tools |
| 7.8 Provisioning | SCIM, attribute mapping, sync cycles |
| 7.9 Best Practices | Security, operations, planning checklists |

---

## Prerequisites

- Understanding of Lesson 06: Application Integration
- Access to Microsoft Entra admin center
- Test users and applications for SSO configuration

---

## Navigation

← [Lesson 06: Application Integration](../lesson-06-application-integration/README.md) | [Lesson 08: Security & Governance →](../lesson-08-security-governance/README.md)
