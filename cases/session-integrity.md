---
title: Session Integrity & Race Condition Flaws
category: Business Logic & Session Security
severity: High
cvss: 7.8
tags: [race-condition, toctou, session-management, turbo-intruder]
date: 2026-01-05
---

# ⏱️ Session Integrity & Race Condition Flaws (Anonymized)

> ⚠️ Discovered during business logic fuzzing & concurrency QA testing  
> Details sanitized for client confidentiality

---

## 🎯 Scope & Context
High-traffic e-commerce checkout and membership subscription system.

---

## ⚠️ Impact Summary
- **Severity**: High (CVSS 7.8)
- Direct financial exploitation via voucher double-spend race condition
- Stale session persistence after credential reset, failing complete account lockout

---

## 🔗 Target Scope
`POST /api/cart/apply-voucher`  
`POST /api/auth/password-reset`

---

## 🧪 Exploitation Flow
1. Load a one-time promo voucher worth \$50 discount into session cart.
2. Script a concurrent burst using Burp Suite Turbo Intruder (30 parallel HTTP requests).
3. Backend checked voucher validity (`isValid = true`) concurrently before persisting redemption state (`used = true`).
4. Over 20 requests succeeded within a 45ms window, applying cumulative \$1,000+ discounts against a single code.
5. In addition, when testing password reset in Session A, existing bearer tokens in Session B remained fully valid indefinitely until manual token expiry.

---

## 🛡️ Remediation
1. **Atomic Locks**: Implemented Redis distributed locks (`SET key val NX EX 5`) around voucher validation and wallet balance transactions.
2. **Session Token Versioning**: Added user-level `token_version` column in the database; incrementing version on password change immediately invalidates all active JWT sessions.
3. **Automated Concurrency QA**: Added load & race-condition test cases to automated QA pipeline.
