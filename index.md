# 👨‍💻 Kanan Gandhi

🛡️ Penetration Tester | Bug Bounty Hunter  

---

## 📊 Dashboard

| Metric | Value |
|-------|------|
| Total Findings | 3+ |
| High Severity | 1 |
| Categories | API, Web |

---

## 🧪 Latest Findings

{% for case in site.cases %}
### 🔐 {{ case.title }}

**Category:** `{{ case.category }}`  
**Severity:** `{{ case.severity }}`  

👉 [View Report]({{ case.url }})

---
{% endfor %}

---

## 🧬 Categories

### 🔗 API Security
{% for case in site.cases %}
{% if case.category == "API Security" %}
- [{{ case.title }}]({{ case.url }})
{% endif %}
{% endfor %}

---

### 🌐 Web Security
{% for case in site.cases %}
{% if case.category == "Web Security" %}
- [{{ case.title }}]({{ case.url }})
{% endif %}
{% endfor %}

---

## 🏷 Tags

{% for case in site.cases %}
{% for tag in case.tags %}
`{{ tag }}`
{% endfor %}
{% endfor %}
