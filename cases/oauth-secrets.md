---
title: Exposed OAuth2 Client Credentials in Client-Side JavaScript Bundle
target: Global Hospitality Management System
category: Authentication & Secrets Management
severity: High
cvss: 8.2
tags: [oauth, secrets-leak, javascript, pkce, hospitality]
date: 2026-03-01
---

# 🔑 Exposed OAuth2 Client Credentials in Client-Side Bundle (Anonymized)

> ⚠️ **Context:** Discovered during source code review and authentication flow auditing of a staging/UAT single-page application.  
> 🛡️ **Confidentiality:** Target organization anonymized as **"Global Hospitality Management System"**. All live client IDs, client secrets, and internal hostnames are **[REDACTED]**.

---

## 🎯 Target Scope
- **Organization Type:** Global Hospitality Management & Hotel Reservation Network
- **Component:** Production/UAT Single-Page Application (React SPA)
- **Asset:** Publicly accessible JavaScript bundle (`main.[hash].js`)

---

## ⚠️ Impact Summary
- **Severity:** High (CVSS 8.2 - `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:L/A:N`)
- **Impact:** Extraction of confidential OAuth 2.0 client credentials (`client_id`, `client_secret`) and internal API gateway routes.
- An attacker could leverage the valid client secret to mint OAuth tokens, bypass client-origin restrictions, map internal PingFederate/SAML infrastructure, and interact with internal APIs.

---

## 🔍 Vulnerability Details & Evidence

Static analysis of the compiled 2.8MB React bundle revealed hardcoded environment variables containing production/staging secrets:

```javascript
// Extracted from compiled SPA client bundle (Sanitized)
REACT_APP_CLIENT_ID: "[REDACTED_CLIENT_ID]"
REACT_APP_GRANT_TYPE: "authorization_code"
REACT_APP_CLIENT_SECRET: "[REDACTED_HIGH_ENTROPY_SECRET_64_CHARS]"
REACT_APP_TOKEN_URL: "https://auth-api.[redacted-hospitality].com/v1/auth-service/token.auth"
REACT_APP_API_URL: "https://internal-api.[redacted-hospitality].com/v1"
REACT_APP_LOGIN_URL: "https://sso.[redacted-hospitality].com/as/authorization.oauth2"
REACT_APP_REDIRECT_URL: "https://app.[redacted-hospitality].com"
```

### Verification via Token Exchange API
Using the extracted client secret in an authorization exchange request:
```bash
curl -s -D - "https://auth-api.[redacted-hospitality].com/v1/auth-service/token.auth" \
  -H "Origin: https://app.[redacted-hospitality].com" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -X POST \
  -d "grant_type=authorization_code&client_id=[REDACTED]&client_secret=[REDACTED]&code=test_probe&redirect_uri=https://app.[redacted-hospitality].com"
```
**API Gateway Response:**
```json
HTTP/1.1 400 Bad Request
Access-Control-Allow-Origin: https://app.[redacted-hospitality].com
Content-Type: application/json

{"error_description":"Authorization code is malformed.","error":"invalid_grant"}
```
**Analysis:**
- The API gateway accepted the grant type `authorization_code`.
- The credentials were recognized and verified as valid (returning `invalid_grant` for the test code rather than `invalid_client`).
- CORS origin was accepted and verified.

---

## 🛡️ Remediation
1. **Migrate to OAuth 2.0 PKCE:** Public client applications (such as React SPAs and mobile apps) must never hold confidential client secrets. Implement RFC 7636 (Proof Key for Code Exchange).
2. **Immediate Credential Invalidation:** Revoke and rotate all exposed client IDs and secret keys across both staging and production.
3. **Automated Secret Detection:** Embed automated scanners (TruffleHog, GitGuardian) into pre-commit hooks and GitHub Actions to fail builds whenever secrets are detected.
