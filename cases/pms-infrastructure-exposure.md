---
title: Internet-Exposed Property Management System with Unhandled Error State
target: Global Hospitality Property Management System
category: Infrastructure & Error Handling
severity: Medium
cvss: 6.5
tags: [iis, pms, information-disclosure, session-leak, error-handling]
date: 2026-01-25
---

# 🏨 Exposed Property Management System with Unhandled Error State (Anonymized)

> ⚠️ **Context:** Discovered during external attack surface reconnaissance of a global hotel group.  
> 🛡️ **Confidentiality:** Target organization anonymized as **"Global Hospitality Property Management System"**. Hostnames and IP ranges are **[REDACTED]**.

---

## 🎯 Target Scope
- **Organization Type:** Global Hospitality Property Management System (PMS)
- **Component:** Core Property Management & Room Inventory System (IIS/10.0 + ASP.NET)
- **Affected Endpoint:** `POST /login`

---

## ⚠️ Impact Summary
- **Severity:** Medium (CVSS 6.5 - `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:L/A:N`)
- **Impact:** Core hotel management system (handling guest records, room allocations, and billing) accessible on the public internet.
- A broken IIS URL Rewrite rule caused consistent HTTP 500 errors on the `/login` handler while continuing to issue active `ASP.NET_SessionId` session cookies on error responses.
- Server response headers leaked internal technology stack details (`Microsoft-IIS/10.0`, ASP.NET viewstate architecture, PingFederate SSO integration).

---

## 🔍 Vulnerability Details

1. **Internal PMS Exposure:** The Property Management System, which handles guest reservations and operational billing, was exposed directly to the public web rather than restricted to internal hotel property networks.
2. **Broken Rewrite / Unhandled Exception:** Submitting POST credentials to `/login` consistently triggered an unhandled server error:
   ```http
   POST /login HTTP/1.1
   Host: pms.[redacted-hospitality].com
   Content-Type: application/x-www-form-urlencoded

   username=test&password=test
   ```
   **Response:**
   ```http
   HTTP/1.1 500 Internal Server Error
   Server: Microsoft-IIS/10.0
   Set-Cookie: ASP.NET_SessionId=[REDACTED]; path=/; HttpOnly
   Content-Length: 75

   The page cannot be displayed because an internal server error has occurred.
   ```
3. **Session Cookie Generation on Errors:** The server continuously provisioned new ASP.NET session state even when returning HTTP 500 error pages.

---

## 🛡️ Remediation
1. **Network Boundary Enforcement:** Restrict access to internal PMS portals exclusively to corporate VPNs and internal property IP ranges.
2. **Custom Error Handling:** Enable `<customErrors mode="On" />` in `web.config` to prevent internal framework disclosure.
3. **Suppress Server Headers:** Strip the `Server: Microsoft-IIS/10.0` header via IIS URL Rewrite rules.
4. **Harden Session Cookie Lifecycles:** Ensure session identifiers are only initialized after successful user authentication.
