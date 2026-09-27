---
title: Cleartext Credential Transmission over HTTP & SSO Infrastructure Disclosure
target: National Telecommunications & Mobile Carrier
category: Authentication & Transport Layer Security
severity: Medium / P4 (CVSS 6.5 - Bugcrowd Verified)
cvss: 6.5
tags: [broken-authentication, cleartext-http, mitm, siteminder-sso, transport-security, recon]
date: 2026-05-31
---

# 🔓 Cleartext Credential Transmission via Hardcoded HTTP Form Action & SSO Disclosure (Anonymized)

> ⚠️ **Context:** Discovered during an external authentication flow audit on enterprise single sign-on (SSO) gateways.  
> 🛡️ **Confidentiality:** Target organization anonymized as **"National Telecommunications & Mobile Carrier"**. All domains, hostnames, agent hashes, and realm IDs are **[REDACTED]**.

---

## 🎯 Target Scope
- **Organization Type:** National Telecommunications & Mobile Carrier
- **Component:** Enterprise Single Sign-On (SSO) Portal (CA / Broadcom SiteMinder)
- **Vulnerable Endpoint:** `https://www.[redacted-telecom].com.au/signon/portal/login.fcc`
- **Vulnerable Form Action:** `http://www.[redacted-telecom].com.au/siteminderagent/portal/login.fcc`
- **VRT Classification:** Broken Authentication and Session Management > Weak Login Function > Over HTTP
- **CWE References:**
  - **CWE-319:** Cleartext Transmission of Sensitive Information
  - **CWE-523:** Unprotected Transport of Credentials
  - **CWE-352:** Cross-Site Request Forgery (CSRF)
  - **CWE-200:** Exposure of Sensitive Information to an Unauthorized Actor

---

## ⚠️ Vulnerability Summary

The customer authentication portal rendered over secure HTTPS (`https://www.[redacted-telecom].com.au/...`), creating a false sense of transport security for users. However, inspection of the DOM and form attributes revealed that the `<form>` submission endpoint was hardcoded to use unencrypted HTTP:

```html
<FORM NAME="Login" ACTION="http://www.[redacted-telecom].com.au/siteminderagent/portal/login.fcc" METHOD="POST">
```

### The Attack Vector:
1. When the user enters their username and password and clicks submit, the browser transmits the sensitive login credentials in **unencrypted plaintext** over port 80 (HTTP).
2. Any adversary positioned on the local network (shared Wi-Fi, public hotspot, rogue AP, upstream ISP, VPN exit node, or compromised router) can passively sniff and intercept the customer credentials in transit before any encryption takes place.
3. While the Akamai edge reverse proxy terminates the HTTP connection and issues an `HTTP/1.1 302 Moved Temporarily` redirecting the client to `https://...`, the plaintext password payload has **already crossed the unencrypted wire** and been captured.

---

## 🔍 Technical Evidence & Proof of Concept

### 1. Form Action Examination (Source Code)

```html
<FORM NAME="Login" ACTION="http://www.[redacted-telecom].com.au/siteminderagent/portal/login.fcc" METHOD="POST">
    <INPUT TYPE=HIDDEN NAME="SMENC" VALUE="ISO-8859-1">
    <INPUT type=HIDDEN name="SMLOCALE" value="US-EN">
    <input type=hidden name=target value="">
    <input type=hidden name=smauthreason value="">
    <input type=hidden name=smagentname value="">
    <input type=hidden name=postpreservationdata value="">
    <input id="USER" NAME="USER" SIZE="25">
    <input type="password" id="PASSWORD" NAME="PASSWORD" SIZE="25">
</FORM>
```

### 2. HTTP POST Transmission & Premature Redirect

Executing a POST request mimicking form submission directly over cleartext HTTP demonstrates that the edge receives the unencrypted payload:

```http
POST http://www.[redacted-telecom].com.au/siteminderagent/portal/login.fcc HTTP/1.1
Host: www.[redacted-telecom].com.au
Content-Type: application/x-www-form-urlencoded
Content-Length: 108

USER=victim_account&PASSWORD=SensitiveSecret123!&SMENC=ISO-8859-1&SMLOCALE=US-EN&target=&smauthreason=
```

**Observed Response:**
```http
HTTP/1.1 302 Moved Temporarily
Server: AkamaiGHost
Content-Length: 0
Location: https://www.[redacted-telecom].com.au/siteminderagent/portal/login.fcc
Connection: keep-alive
```

> ⚠️ **Key Takeaway:** The edge proxy returns a 302 redirect **only after receiving the full POST body containing plaintext credentials**. Transport encryption was completely bypassed at the moment of transmission.

---

## 🧩 Additional Weaknesses & Reconnaissance Findings

### 1. SiteMinder SSO Internal Topology Disclosure
URL query parameters and hidden form inputs leak proprietary authentication realm parameters:
- **`REALMOID`:** `06-dd57b946-1753-43da-[REDACTED]` (Internal database identifier for the authentication realm)
- **`SMAGENTNAME`:** `-SM-7LF8ifsjXF2F6vC8[REDACTED]` (Encrypted agent configuration identifier)
- **`TYPE`:** `33554433` (SiteMinder form-based authentication routine code)

### 2. Absence of Anti-CSRF Protection
The login submission form lacks any cryptographic anti-CSRF token, leaving the endpoint open to login CSRF attacks where an attacker forces a victim into an attacker-controlled session.

### 3. Client-Side Only Brute-Force Rate Limiting
Investigation of client-side scripts revealed that login attempt tracking relied strictly on a browser cookie:
```javascript
function getCookieValue(cookieName) {
    // SMTRYNO cookie — read and set client-side only
    // No backend rate limiting enforcement or IP throttling
}
```
An attacker can clear or spoof the `SMTRYNO` cookie between automated requests to perform unlimited credential stuffing attempts without encountering account lockout.

---

## 💥 Threat & Business Impact

1. **Passive Credential Harvesting (MITM):** Any attacker on the local network path can capture user and corporate customer credentials in plaintext.
2. **Eavesdropping on Untrusted Networks:** Customers logging in from airports, cafes, hotels, or mobile tethering are fully vulnerable to packet sniffing.
3. **Compromise of Single Sign-On Realm:** Captured credentials grant access across all federated applications managed under the carrier's SiteMinder SSO umbrella.
4. **Credential Stuffing Expansion:** Absence of server-side rate limiting enables high-velocity dictionary and brute-force attacks against customer accounts.

---

## 🛡️ Remediation & Security Hardening

1. **Enforce HTTPS in Form Action:**
   Change the form `ACTION` attribute to strictly use the `https://` protocol:
   ```html
   <FORM NAME="Login" ACTION="https://www.[redacted-telecom].com.au/siteminderagent/portal/login.fcc" METHOD="POST">
   ```
2. **Deploy Strict-Transport-Security (HSTS) with Preload:**
   Issue the HSTS header across all domain assets:
   ```http
   Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
   ```
   This instructs modern browsers to automatically upgrade insecure HTTP requests to HTTPS before transmission.
3. **Implement Content Security Policy `form-action` Directive:**
   Add CSP controls preventing forms from submitting to unencrypted origins:
   ```http
   Content-Security-Policy: form-action 'self' https://www.[redacted-telecom].com.au;
   ```
4. **Implement Cryptographic Anti-CSRF Tokens:**
   Embed a unique, unpredictable per-session CSRF token in the login form and validate it server-side.
5. **Implement Server-Side Authentication Throttling:**
   Track failed authentication attempts server-side by username and IP address, enforcing progressive delays or CAPTCHA challenges instead of relying on client-side cookies.
