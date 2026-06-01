# 🔐 IDOR Vulnerability (Anonymized)

> Based on real-world bug bounty experience  
> Details anonymized for confidentiality  

---

## 🎯 Target
Large E-commerce Platform

---

## 🐞 Vulnerability
IDOR (Insecure Direct Object Reference)

---

## 🔗 Endpoint

GET /api/v1/orders?user_id=XXXX

---

## ⚡ Impact

Unauthorized access to user order data

---

## 🧪 Proof of Concept

1. Login as user  
2. Intercept request  
3. Change user_id  
4. Access another user data  

---

## 🔧 Fix

- Implement authorization checks  
- Validate user session  
