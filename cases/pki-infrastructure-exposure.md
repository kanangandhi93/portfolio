---
title: Public Exposure of Internal PKI Trust Chain via JWKS x5c
target: Multinational Retail & Fashion Conglomerate
category: PKI & Cryptographic Architecture
severity: Critical
cvss: 9.3
tags: [pki, jwks, x509, crl, ocsp, reconnaissance]
date: 2026-02-25
---

# 🛡️ Public Exposure of Internal PKI Trust Hierarchy (Anonymized)

> ⚠️ **Context:** Discovered during security assessment of a global fashion/retail conglomerate.  
> 🛡️ **Confidentiality:** Target organization anonymized as **"Multinational Retail & Fashion Conglomerate"**. All domain names, IP addresses, and certificate serial numbers are **[REDACTED]**.

---

## 🎯 Target Scope
- **Organization Type:** Multinational Retail & Fashion Enterprise
- **Component:** Enterprise Identity & Access Management (OAuth2/OpenAM JWKS)
- **Origin Endpoint:** `https://auth.[redacted-retail].com/openam/oauth2/connect/jwk_uri`
- **Exposed PKI Endpoints:**
  - `http://pki.[redacted-retail].com/CertData/Root_CA.crt`
  - `http://pki.[redacted-retail].com/CertData/Corporate_SubCA.crt`
  - `http://pki.[redacted-retail].com/CertData/Corporate_SubCA.crl`
  - `http://pki.[redacted-retail].com/ocsp`

---

## ⚠️ Impact Summary
- **Severity:** Critical (CVSS 9.3 - `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:L/A:N` - P1)
- **Impact:** Exposure of the complete internal PKI trust hierarchy.
- An unauthenticated external attacker could download internal Root CA certificates, Subordinate CAs, a 15KB Certificate Revocation List (CRL), and query the active OCSP responder.
- Certificate Subject Alternative Names (SANs) revealed internal microservice hostnames (`*.central.[redacted].grp`), internal JWT signing key aliases, and service delivery subnets, laying the groundwork for internal TLS interception and SSRF pivoting.

---

## 🔍 Vulnerability Chain

1. **JWKS `x5c` Parameter Leak:** The public JWKS endpoint returned full X.509 certificate chains in the `x5c` property.
2. **CDP & AIA Extension Extraction:** Parsing the certificates revealed internal CRL Distribution Points (CDP) and Authority Information Access (AIA) URLs pointing to `pki.[redacted-retail].com`.
3. **Public DNS Resolution:** The internal PKI hostname resolved to a public internet IP address without firewall or VPN restrictions.
4. **Public Download of Certificates & Revocation Lists:**
   - Root CA Certificate: `200 OK` (1,519 bytes, DER encoded ASN.1 SEQUENCE)
   - Subordinate CA Certificate: `200 OK` (1,335 bytes, DER encoded)
   - Certificate Revocation List (CRL): `200 OK` (15,494 bytes)
   - OCSP Responder: Active HTTP 200 responses to `POST /ocsp`
5. **Internal Hostname Reconnaissance:** Exposed internal SANs included internal backend servers and internal load balancers.

---

## 🛡️ Remediation
1. **Remove `x5c` from JWKS:** Cease exposing full X.509 certificate chains in public JWKS endpoints. Output only the standard public key parameters (`kty`, `kid`, `use`, `n`, `e`).
2. **Restrict PKI Access to Internal Networks:** Move internal PKI distribution points and OCSP responders behind internal enterprise VPNs/firewalls.
3. **CA Rotation & Revocation:** Rotate exposed intermediate CA certificates and revoke any certificate whose trust parameters were compromised.
