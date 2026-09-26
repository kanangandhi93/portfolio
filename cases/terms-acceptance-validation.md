---
title: Missing Server-Side Enforcement of Terms Acceptance
target: EdTech & Online Learning Platform
category: Business Logic & Compliance QA
severity: Low / Compliance
cvss: 3.8
tags: [business-logic, input-validation, compliance, gdpr, manual-qa]
date: 2026-03-25
---

# ⚖️ Missing Server-Side Validation on Terms Acceptance (Anonymized)

> ⚠️ **Context:** Discovered during security and QA auditing of a user onboarding and registration service.  
> 🛡️ **Confidentiality:** Target organization anonymized as **"EdTech & Online Learning Platform"**. Hostnames, tokens, and personal credentials are **[REDACTED]**.

---

## 🎯 Target Scope
- **Organization Type:** EdTech & Online Learning Platform
- **Component:** User Registration & Onboarding Service
- **Affected Endpoint:** `POST /api/users/register/`
- **Vulnerable Parameter:** `has_accepted_terms_and_condition` (Boolean)

---

## ⚠️ Impact Summary
- **Severity:** Low / Informational (Compliance, Legal, & Business Logic Risk)
- **CWE:** CWE-20 (Improper Input Validation), CWE-840 (Business Logic Errors)
- **Impact:** Client-side only enforcement of mandatory user legal consent.
- An adversary or automated script could register accounts by submitting `"has_accepted_terms_and_condition": false`.
- The server accepted the payload, provisioned the account, persisted `terms_accepted_at: null`, and issued fully authenticated JWT access and refresh tokens.
- **Regulatory Risk:** Processing user data without verifiable consent introduces compliance exposures under GDPR and CCPA, while weakening contractual enforceability of terms of service.

---

## 🔍 Proof of Concept Flow (Sanitized)

### Step 1: Intercept Registration Request
Populate signup form and capture outbound request before transmission using an intercepting proxy (Burp Suite).

### Step 2: Alter Consent Parameter
Modify the JSON payload parameter `"has_accepted_terms_and_condition"` from `true` to `false`:

```http
POST /api/users/register/ HTTP/2
Host: app.[redacted-edtech].com
Content-Type: application/json

{
  "email": "audit_user@example.com",
  "password": "[REDACTED_PASSWORD]",
  "firstName": "Test",
  "lastName": "User",
  "preferred_language": "en",
  "remember": false,
  "has_accepted_terms_and_condition": false
}
```

### Step 3: Observed Response
The server responds with `HTTP 201 Created` and issues production tokens despite user declining terms:

```http
HTTP/2 201 Created
Content-Type: application/json

{
  "access_token": "eyJhbGciOiJIUzI1[REDACTED]...",
  "refresh_token": "eyJhbGciOiJIUzI1[REDACTED]...",
  "user": {
    "id": 182895,
    "email": "audit_user@example.com",
    "full_name": "Test User",
    "access_role": "student",
    "has_accepted_terms_and_condition": false,
    "terms_accepted_at": null
  }
}
```

---

## 🛡️ Remediation Guidance

Enforce server-side schema validation to reject any registration request where the terms flag is not explicitly `true`.

### Node.js / Express Example Fix
```javascript
// Middleware / Route Validator
if (req.body.has_accepted_terms_and_condition !== true) {
  return res.status(400).json({
    error_code: "TERMS_NOT_ACCEPTED",
    detail: "You must accept the Terms and Conditions to create an account."
  });
}
```

### Python / Django REST Framework Example Fix
```python
from rest_framework import serializers

class UserRegistrationSerializer(serializers.ModelSerializer):
    has_accepted_terms_and_condition = serializers.BooleanField(required=True)

    def validate_has_accepted_terms_and_condition(self, value):
        if value is not True:
            raise serializers.ValidationError("Acceptance of Terms and Conditions is required.")
        return value
```
