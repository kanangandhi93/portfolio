---
title: Hardcoded OAuth Secrets & Client Invalidation
category: Authentication & Secrets
severity: High
cvss: 8.2
tags: [oauth, secrets-leak, javascript, auth-bypass]
date: 2026-01-20
---

# 🔑 Hardcoded OAuth Secrets & Client Invalidation (Anonymized)

> ⚠️ Discovered during client-side JavaScript source code audit & authentication analysis  
> Details sanitized for client confidentiality

---

## 🎯 Scope & Context
Single-Page Application (SPA) frontend utilizing an external OAuth 2.0 authorization server for user single sign-on (SSO).

---

## ⚠️ Impact Summary
- **Severity**: High (CVSS 8.2)
- Exposure of OAuth client credentials in public production web assets
- Threat actors could impersonate the first-party application and mint rogue authorization tokens

---

## 🔗 Target Scope
`https://app.[target].com/static/js/main.[hash].chunk.js`

---

## 🧪 Exploitation Flow
1. Review minified JavaScript bundles and source maps fetched during application boot.
2. Search for common secret patterns (e.g. `client_secret`, `app_secret`, `api_key`).
3. Discovered hardcoded production OAuth `client_secret` value embedded in build configuration constants.
4. Crafted direct HTTP POST request to OAuth `/oauth/token` endpoint using the compromised secret.
5. Successfully exchanged authorization codes and issued tokens without user-origin verification.

---

## 🛡️ Remediation
1. **Migrate to PKCE**: Transition frontend SPA to OAuth 2.0 Authorization Code Flow with PKCE (Proof Key for Code Exchange), which does not require a client secret on public clients.
2. **Rotate Secrets**: Immediately revoke exposed OAuth client credentials and regenerate new keys.
3. **CI/CD Secret Scanning**: Implemented GitGuardian and TruffleHog in GitHub Actions to block secret commits at push time.
