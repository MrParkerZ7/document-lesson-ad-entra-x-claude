# 2.4 Company Branding

## Overview

Company branding customizes the sign-in experience with your organization's logos, colors, and messaging. This improves user experience and helps prevent phishing attacks.

## Learning Objectives

- Understand the benefits of custom branding
- Configure sign-in page elements
- Set up localized branding
- Apply branding best practices

---

## Why Configure Branding?

| Benefit | Description |
|---------|-------------|
| **Professional** | Consistent brand experience |
| **Trust** | Users recognize legitimate sign-in |
| **Anti-phishing** | Harder to impersonate |
| **User experience** | Familiar look and feel |

---

## Accessing Branding Settings

```
Microsoft Entra ID → Company branding → Configure
```

Or:
```
https://entra.microsoft.com → Company branding
```

---

## Branding Elements

### Logos

| Element | Size | Max Size | Description |
|---------|------|----------|-------------|
| **Banner logo** | 280×60 px | 10 KB | Horizontal logo on sign-in |
| **Square logo** | 240×240 px | 50 KB | Used in app launcher |
| **Square logo (dark)** | 240×240 px | 50 KB | For dark backgrounds |
| **Favicon** | 32×32 px | 5 KB | Browser tab icon |

### Background

| Element | Specification | Description |
|---------|---------------|-------------|
| **Background image** | 1920×1080 px, <300 KB | Sign-in page background |
| **Background color** | Hex code (#000000) | Fallback if image fails |

### Text

| Element | Max Length | Description |
|---------|------------|-------------|
| **Sign-in page text** | 1024 chars | Welcome message |
| **Username hint** | 64 chars | Placeholder in username field |

---

## Configuration Example

```json
{
  "bannerLogo": "logo-horizontal.png",
  "squareLogo": "logo-square.png",
  "squareLogoDark": "logo-square-dark.png",
  "backgroundColor": "#0078D4",
  "backgroundImage": "office-background.jpg",
  "signInPageText": "Welcome to Contoso! Please sign in with your corporate credentials.",
  "usernameHintText": "user@contoso.com"
}
```

---

## Localized Branding

Configure different branding per language:

```
1. Configure default branding first
2. Add language → Select language (e.g., German, French)
3. Configure language-specific elements
4. User sees branding based on browser language
```

Supported languages: 40+ languages including:
- English, Spanish, French, German
- Chinese, Japanese, Korean
- Arabic, Hebrew (RTL support)

---

## Best Practices

| Practice | Reason |
|----------|--------|
| Use transparent PNG for logos | Blends with any background |
| Keep text concise | Users scan, don't read |
| Test on mobile | Many users sign in on phones |
| Update seasonally (optional) | Keep fresh, show activity |
| Include support info | Help users who have issues |

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of branding elements.

---

## Key Takeaways

1. Branding improves trust and reduces phishing risk
2. Multiple logo types serve different purposes
3. Localized branding supports global organizations
4. Test branding on multiple devices

---

## Next Sub-Lesson

[2.5 Tenant Settings →](../2.5-tenant-settings/README.md)
