---
title: Direct Origin Access & WAF Bypass via Header IP Disclosure
target: Global Connected Vehicle & EV Enterprise
category: Infrastructure Security & WAF Bypass
severity: High
cvss: 7.5
tags: [waf-bypass, origin-disclosure, cdn, infrastructure, dos, akamai]
date: 2026-06-01
---

# 🛡️ Direct Origin Access & WAF Bypass via Header IP Disclosure (Anonymized)

> ⚠️ **Context:** Discovered during external reconnaissance and edge security auditing of a multinational electric vehicle and clean energy platform.  
> 🛡️ **Confidentiality:** Target organization anonymized as **"Global Connected Vehicle & EV Enterprise"**. All production hostnames, origin IP addresses, and internal datacenter identifiers are strictly **[REDACTED]**.

---

## 🎯 Target Scope
- **Organization Type:** Global Connected Vehicle & EV Enterprise
- **Components Audited:**
  - Core Authentication Gateway (`auth.[redacted-automotive].com`)
  - Vehicle Telematics & Fleet API (`owner-api.[redacted-automotive].com`)
  - Account Management Portal (`accounts.[redacted-automotive].com`)
- **Primary Vulnerability:** Server Security Misconfiguration > Web Application Firewall (WAF) Bypass (CVSS 7.5 - `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:L/A:L`)

---

## ⚠️ Impact Summary
- **Complete WAF Circumvention:** By connecting directly to discovered backend origin IPs, an attacker completely bypasses the Akamai WAF perimeter, evading all rate limiting, bot management, automated threat detection, and IP reputation filters on the enterprise authentication gateway.
- **Unthrottled Authentication Attacks:** Allows high-speed credential stuffing, OAuth token brute-forcing, and API fuzzing without interference or blocking by the CDN layer.
- **Denial of Service Vector:** Sending internal IP structures via the `Client-IP` header triggered unhandled exceptions and persistent HTTP 500 Internal Server Errors on the backend Ruby on Rails service.
- **Internal Infrastructure & Datacenter Exposure:** Leaked internal datacenter naming conventions (`x-datacenter`, `x-hosting-region`), internal Varnish caching tiers, and session cookies provisioned on HTTP 404 error responses.

---

## 🔍 Attack Chain Breakdown

```
[Origin IP Disclosure in Headers]
       │
       ▼
[Direct Origin TLS Connection via --resolve]
       │
       ▼
[Akamai WAF Layer Completely Bypassed]
       │
       ├─► [Unrestricted OAuth Token Brute-Force & Fuzzing]
       ├─► [Internal IP Spoofing via X-Forwarded-For]
       └─► [Backend Exception / DoS via Client-IP Header]
```

### Step 1: Origin IP Discovery via CDN Latency Headers
The edge CDN leaked internal origin IP addresses within HTTP response headers (`OriginIP` and latency profiling headers):
- `auth.[redacted-automotive].com` $\rightarrow$ `66.17.x.x` (Origin behind Akamai WAF)
- `owner-api.[redacted-automotive].com` $\rightarrow$ `66.17.x.x` (Direct Envoy Gateway)

### Step 2: Direct Origin Access & WAF Bypass
By forcing DNS resolution directly to the leaked origin IP address while preserving the legitimate SNI hostname, the request reached the origin backend directly without traversing the Akamai edge:

```bash
# Direct origin connection bypassing Akamai WAF rules
curl -v --resolve "auth.[redacted-automotive].com:443:[REDACTED_ORIGIN_IP]" \
  https://auth.[redacted-automotive].com/oauth2/v1/token

# Response: HTTP 200 OK — Direct origin access verified
```

### Step 3: Internal IP Spoofing & Denial of Service
Testing internal subnet ranges (`192.168.90.0/24`) revealed differential backend handling:
1. **Locale Manipulation:** Sending `X-Forwarded-For: 192.168.90.100` caused the backend Envoy proxy to route traffic to an internal localized cluster.
2. **Backend Crash / DoS:** Sending `Client-IP: 192.168.90.100` caused the backend to throw an unhandled internal exception:
```http
GET /api/1/vehicles HTTP/1.1
Host: owner-api.[redacted-automotive].com
Client-IP: 192.168.90.100

HTTP/1.1 500 Internal Server Error
```

### Step 4: Cross-Region Auth Infrastructure Disclosure
Probing regional SSO nodes disclosed internal data center architecture via custom headers:
- `x-datacenter: [redacted-datacenter-iam]`
- `x-hosting-region: [REDACTED]`
- `Set-Cookie: [redacted]-auth.sid` provisioned on HTTP 404 responses.

---

## 🛡️ Remediation Guidance

1. **Origin Shielding & Ingress Filtering:** Configure origin firewalls/security groups to drop all inbound traffic that does not originate from verified Akamai/CDN IP CIDR blocks.
2. **Strip Diagnostic Headers:** Ensure edge reverse proxies remove `OriginIP` and internal latency headers before returning responses to clients.
3. **Defensive Client-IP Handling:** Drop or sanitize client-supplied `Client-IP` and `X-Forwarded-For` headers at the edge reverse proxy so invalid formats cannot crash upstream microservices.
4. **Suppress Internal Header Leaks:** Strip `x-datacenter` and `x-hosting-region` headers from public responses.
