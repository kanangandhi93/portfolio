---
title: IDOR Vulnerability
category: API Security
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

## 🔗 Endpoint

GET /api/v1/orders?user_id=XXXX

---

## ⚡ Impact

- Unauthorized access to other users’ data  
- Exposure of sensitive information  

---

## 🧪 Proof of Concept

1. Login as normal user  
2. Intercept request (Burp Suite)  
3. Change `user_id`  
4. Access another user's data  

---

## 📊 Severity
High

---

## 🔧 Mitigation

- Implement authorization checks  
- Validate session on backend  
- Avoid exposing direct object IDs  

---

## 🧠 Learning

- Always verify authorization server-side  
- Never trust user input  
``
