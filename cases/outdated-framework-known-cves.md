---
title: EOL Apache Cocoon Exposure & Known CVEs in Enterprise Portal
target: Major National Telecommunications Provider
category: Component Vulnerabilities & Perimeter Hardening
severity: Medium / P5 (CVSS 5.3 - Bugcrowd Verified)
cvss: 5.3
tags: [known-vulnerabilities, outdated-components, eol-framework, apache-cocoon, waf-analysis, reconnaissance]
date: 2026-05-31
---

# 📦 Exposure of EOL Apache Cocoon Framework & Unpatched CVEs (Anonymized)

> ⚠️ **Context:** Identified during an external perimeter attack surface evaluation on enterprise portal endpoints.  
> 🛡️ **Confidentiality:** Target organization anonymized as **"Major National Telecommunications Provider"**. All domain names, hostnames, and internal references are **[REDACTED]**.

---

## 🎯 Target Scope
- **Organization Type:** Major National Telecommunications Provider
- **Component:** Enterprise Facility & Resource Management System (eFRAMS Portals)
- **Target Endpoints:**
  - `https://portal-fm1.[redacted-telecom].net/eFRAMS3/` (Instance FM1)
  - `https://portal-fm2.[redacted-telecom].net/eFRAMS3/` (Instance FM2)
- **VRT Classification:** Using Components with Known Vulnerabilities > Outdated Software Version
- **Identified Stack:** Apache Cocoon `2.0.4` running on Apache Tomcat (`Apache-Coyote/1.1`) behind Akamai Edge WAF

---

## ⚠️ Summary of Findings

During external reconnaissance of enterprise edge assets, both production facility management portals were observed exposing exact framework signatures via HTTP response headers:
- `X-Cocoon-Version: 2.0.4`
- `Server: Apache-Coyote/1.1`

**Apache Cocoon 2.0.4** is a retired and unmaintained XML publishing framework that contains multiple unpatched high and critical severity CVEs. Furthermore, comparative edge rule auditing revealed inconsistent WAF enforcement between the two identical portal instances.

---

## 🔍 Unpatched CVE Exposure in Apache Cocoon 2.0.4

Because version `2.0.4` has reached End-of-Life (EOL) without official maintenance, the running service is exposed to published vulnerability vectors:

| CVE Identifier | Severity | CVSS | Vulnerability Type & Technical Description |
| :--- | :--- | :--- | :--- |
| **CVE-2023-49733** | **Critical** | **9.8** | **XML External Entity (XXE) Injection** — Failure to restrict external entity references in XML processing components. |
| **CVE-2022-45135** | **High** | **8.8** | **SQL Injection** — Improper neutralization of special elements in database serialization and SQL transformers. |
| **CVE-2020-11991** | **High** | **7.5** | **XXE Injection** — Vulnerability in `StreamGenerator` allowing arbitrary file disclosure and SSRF. |
| **CVE-2025-24783** | **High** | **7.5** | **Predictable PRNG Continuation IDs** — Weak pseudo-random number generator leading to session hijacking. |
| **CVE-2003-1172** | **High** | **7.5** | **Directory Traversal** — Arbitrary local file disclosure via unauthenticated `view-source` sample handlers. |

---

## 🔬 Proof of Concept & Evidence

### 1. HTTP Response Header Fingerprinting

Sending a standard HTTP GET request to the primary portal reveals detailed component versions:

```http
GET /eFRAMS3/ HTTP/1.1
Host: portal-fm1.[redacted-telecom].net
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36

HTTP/1.1 200 OK
X-Cocoon-Version: 2.0.4
Server: Apache-Coyote/1.1
X-OneAgent-JS-Injection: true
Content-Type: text/html; charset=ISO-8859-1
Set-Cookie: JSESSIONID=[REDACTED]; Path=/eFRAMS3; HttpOnly
Accept-Ranges: bytes
```

### 2. Information Disclosure via Public Login Interface

Instance FM1 publicly renders the authentication interface (`authenticate.html`), revealing internal application metadata:
- **Application Build Version:** `3.7.6.0`
- **Authentication Handler:** `processlogin.html` (POST)
- **Active XML Namespaces:**
  - `xmlns:sql="http://apache.org/cocoon/SQL/2.0"` (Cocoon SQL database transformer active)
  - `xmlns:java="http://xml.apache.org/xslt/java"` (XSLT Java language binding enabled — potential RCE path)
  - `xmlns:portal="http://[redacted-telecom].net.au/eFRAMS/1.0"`
- **APM Telemetry:** Dynatrace Real User Monitoring (RUM) agent injection (`ruxitagentjs_[REDACTED]`)
- **Internal Administrative Contact:** `feedback@[redacted-telecom].net`

---

## ⚖️ Comparative Edge WAF Analysis (FM1 vs FM2)

A differential security audit was conducted to evaluate the consistency of perimeter firewall rules protecting the two cluster instances:

| HTTP Probe Vector | Instance FM1 (portal-fm1) | Instance FM2 (portal-fm2) | Backend Behavior Observation |
| :--- | :--- | :--- | :--- |
| `GET /eFRAMS3/` | **200 OK** (Empty Body) | **403 Forbidden** (WAF #18) | FM1 allows unauthenticated edge ingress; FM2 enforces edge block. |
| `GET controller.html` | **302 Redirect** $\rightarrow$ `authenticate.html` | **403 Forbidden** (WAF #18) | Differential routing rule at edge layer. |
| `GET authenticate.html` | **200 OK** (Login Portal UI) | **403 Forbidden** (WAF #18) | Internal portal UI fully exposed on FM1. |
| `POST` (No Content-Type) | **411 Length Required** | **411 Length Required** | Bypasses edge; reaches identical Apache Tomcat backend. |
| `POST` (with XML / JSON) | Connection Terminated (Bot Mgr) | **403 Forbidden** (WAF #18) | FM1 relies on heuristic bot mitigation; FM2 enforces static CIDR rule. |
| `PUT / PATCH` | **501 Not Implemented** | **501 Not Implemented** | Reaches backend servlet container. |
| `PROPFIND / COPY` | **501 Not Implemented** | **403 Forbidden** (WAF #18) | FM1 passes WebDAV methods directly to backend. |
| `WEB-INF/web.xml` | **403 Forbidden** (WAF) | **403 Forbidden** (WAF) | WAF rule blocks traversal attempts on both instances. |

**Observation:** While Instance FM2 correctly applies Akamai WAF Rule #18 across all unauthenticated endpoints, Instance FM1 exhibits a perimeter misconfiguration, permitting public reconnaissance and exposing the legacy backend.

---

## 💥 Potential Threat Impact

1. **Remote Exploitation Risk:** If an edge bypass or internal network pivot is achieved, unpatched XXE (`CVE-2023-49733`) and SQLi (`CVE-2022-45135`) allow arbitrary database access and local file read.
2. **Session Hijacking:** Predictable continuation IDs (`CVE-2025-24783`) allow session token forecasting.
3. **Attack Surface Expansion:** Accurate framework and build identification (`Apache Cocoon 2.0.4`, `Apache-Coyote/1.1`, build `3.7.6.0`) enables adversaries to construct targeted exploit payloads.

---

## 🛡️ Remediation & Hardening Plan

1. **Framework Modernization:** Decommission or migrate away from retired Apache Cocoon versions toward modern, actively maintained web architectures.
2. **Standardize Edge Firewall Policies:** Replicate FM2's strict Akamai WAF policy onto FM1 to block unauthenticated public access to administrative portals.
3. **Information Disclosure Prevention:** Strip `X-Cocoon-Version` and `Server` response headers at the edge reverse proxy / CDN tier.
4. **XML Parser Hardening:** Disable inline DTD processing, external general entities (`DOCTYPE`), and external parameter entities within XML pipelines.
