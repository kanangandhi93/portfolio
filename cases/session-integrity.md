---
title: Session Replay & Account Switching via Session Overwrite
target: Healthcare & Genetic Analytics Platform
category: Session Management & Integrity
severity: High
cvss: 8.0
tags: [session-replay, session-fixation, cookie-security, auth-bypass]
date: 2026-02-15
---

# ⏱️ Session Replay & Account Switching via Session Overwrite (Anonymized)

> ⚠️ **Context:** Discovered during security evaluation of a healthcare and genetic analytics web application.  
> 🛡️ **Confidentiality:** Target organization anonymized as **"Healthcare & Genetic Analytics Platform"**. All session tokens and user identifiers are **[REDACTED]**.

---

## 🎯 Target Scope
- **Organization Type:** Healthcare & Personal Genomics Platform
- **Component:** User Profile & Authentication Middleware
- **Affected Endpoint:** `GET /user/`
- **Vulnerable Parameter:** `Cookie: sessionid=[REDACTED]`

---

## ⚠️ Impact Summary
- **Severity:** High (CVSS 8.0 - `CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:U/C:H/I:H/A:N`)
- **Impact:** Dynamic account switching and session replay.
- The server relied solely on client-supplied `sessionid` cookies without context or environment binding.
- More critically, supplying a different user's valid session cookie caused the server to return responses that forced the browser to overwrite the active session, dynamically switching the logged-in user state.

---

## 🔍 Attack Flow & Reproduction

### 1. Session Replay Without Environment Binding
1. Authenticate as User A and capture the `sessionid` cookie via intercepting proxy (Burp Suite).
2. Open a clean browser session from a different IP address and user-agent.
3. Inject the captured `sessionid` cookie and navigate to `/user/`.
4. Observe immediate authenticated access to User A's private genetic reports and health data without MFA or context challenge.

### 2. Dynamic Session Overwrite
1. Authenticate as User B in a browser session.
2. Intercept an outbound request to `GET /user/`.
3. Replace the cookie with User A's `sessionid`.
4. Forward the request.
5. The backend accepts the modified cookie and returns a `Set-Cookie` header that instructs the browser to overwrite the active session cookie with User A's identity, switching the browser session completely.

---

## 🛡️ Remediation
1. **Context Binding:** Bind session tokens to cryptographic device fingerprints or client TLS session context to prevent off-host replay.
2. **Session ID Regeneration:** Always regenerate session identifiers upon any change in privilege or state.
3. **Reject Arbitrary Session Overwrite:** Enforce server-side session integrity validation to ensure sessions cannot be dynamically reassigned via modified client headers.
4. **Strict Revocation:** Invalidate all active user sessions globally upon logout or security events.
