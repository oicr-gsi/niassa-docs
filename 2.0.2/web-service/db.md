---
layout: page
title: Web service - Data Resources
---
{% include functions.liquid %}

This sub-domain is exclusively for low-level access of database tables. Each URI actually retrieves one or more rows from a database table. The output format is exclusively XML produced by JAXB.

For more information about each resource and the correct complete URI, please follow the link to the resource-specific page.

{% assign wwwdb = site.pages | where: "www-bit", "db" %}
| Method | URI | Description |
|--------|-----|-------------|
{% for pg in wwwdb %} |<a href="{{ pg.url }}" alt="{{ pg.title }}">{{ pg.method }}</a> | <a href="{{ pg.url }}" alt="{{ pg.title }}">{{ pg.uri }}</a> | {{pg.summary}} | 
{% endfor %}
