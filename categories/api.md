# 🔗 API Security Findings

{% for case in site.cases %}
{% if case.category == "API Security" %}
- [{{ case.title }}]({{ case.url }})
{% endif %}
{% endfor %}
