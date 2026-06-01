# 👨‍💻 Kanan Gandhi

Penetration Tester | Bug Bounty Hunter

---

## 📂 Navigation

- About
- Contact
- API Security
- Web Security

---

## 🚀 About Me

Focused on real-world vulnerabilities like:

- IDOR
- XSS
- Authentication Issues

---

## 📊 Stats

- Total Findings: 3+
- Severity High: 1
- Categories Covered: API, Web

---

## 🧪 Latest Findings

{% for case in site.cases %}
### 🔐 {{ case.title }}

**Category:** `{{ case.category }}`  
**Endpoint:** `{{ case.endpoint }}`  

👉 [View Details]({{ case.url }})

---

{% endfor %}

---

## 🏷 Categories

- API Security
- Web Security
