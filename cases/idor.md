---
title: IDOR Vulnerability
category: API Security
endpoint: /api/v1/orders?user_id=XXXX
tags: [idor, api, auth]
---

# 🔐 IDOR Vulnerability (Anonymized)

> Based on real-world bug bounty experience  
> Details anonymized for confidentiality  

---

## 🎯 Target

Large E-commerce Platform

---

## 🐞 Vulnerability

Insecure Direct Object Reference (IDOR)

---

## ⚠️ Impact

> ⚠️ **Critical Vulnerability**  
> Unauthorized access to other users' sensitive data

- Data exposure
- Privacy breach

---

## 🔗 Endpoint

`/api/v1/orders?user_id=XXXX`

---

## 🧪 Proof of Concept

> 💡 **PoC Summary**  
> By modifying `user_id`, attacker accessed another user's data

1. Login as normal user  
2. Intercept request using Burp Suite  
3. Modify `user_id` parameter  
4. Access unauthorized data  

---

## 📊 Severity

`High`

---

## 🔧 Fix

- Implement strict authorization checks  
- Validate user session on backend  
- Avoid exposing direct object references  

---

## 🧠 Learning

- Never trust user input  
- Always enforce server-side authorization  
