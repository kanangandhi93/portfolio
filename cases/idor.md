---
title: IDOR Vulnerability
category: API Security
severity: High
endpoint: /api/v1/orders?user_id=XXXX
tags: [idor, api, authorization]
date: 2026-01-01
---

# 🔐 IDOR Vulnerability (Anonymized)

> ⚠️ Based on real-world bug bounty experience  
> Details anonymized for confidentiality  

---

## 🎯 Target
Large E-commerce Platform

---

## ⚠️ Impact

> 🔴 **Critical Vulnerability**
> Unauthorized access to sensitive user data

- Data leakage
- Privacy breach

---

## 🔗 Endpoint

`/api/v1/orders?user_id=XXXX`

---

## 🧪 Exploitation Flow

```text
1. Login as normal user
2. Capture request in Burp
3. Modify user_id
4. Access another user's data
