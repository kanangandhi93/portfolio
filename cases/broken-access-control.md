---
title: Broken Access Control in Multi-Tenant API
category: API Security
severity: Critical
cvss: 9.1
tags: [bola, idor, api-security, multi-tenant]
date: 2026-03-15
---

# 🔐 Broken Access Control in Multi-Tenant API (Anonymized)

> ⚠️ Discovered during commercial penetration test & QA security audit  
> Details sanitized for client confidentiality

---

## 🎯 Scope & Context
Multi-tenant enterprise SaaS platform providing financial and customer analytics to enterprise organizations.

---

## ⚠️ Impact Summary
- **Severity**: Critical (CVSS 9.1)
- Complete breakdown of multi-tenant isolation
- Unauthorized horizontal data exfiltration across organizations
- Regulatory violation (SOC2 / GDPR compliance impact)

---

## 🔗 Target Endpoint
`GET /api/v2/tenants/{tenant_id}/invoices`  
`GET /api/v2/tenants/{tenant_id}/customers`

---

## 🧪 Exploitation Flow
1. Authenticate with low-privileged account in Tenant A.
2. Intercept authenticated HTTP request via Burp Suite.
3. Replace `tenant_id` path parameter with Tenant B's identifier (`tenant_uuid`).
4. Microservice backend fails to validate organizational claims in the JWT token against requested resource URI.
5. Response returns Tenant B's confidential billing and customer records with HTTP 200 OK.

---

## 🛡️ Remediation
1. **JWT Claim Binding**: Validate tenant claim (`token.tenant_id == request.path.tenant_id`) at API Gateway middleware before dispatching.
2. **Database Row Level Security (RLS)**: Enforce RLS policies so queries automatically isolate rows by active tenant context.
3. **Automated Regression**: Integrate negative authorization assertions into Newman/Postman CI test suites.
