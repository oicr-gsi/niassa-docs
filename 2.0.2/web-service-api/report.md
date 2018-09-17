---
layout: page
title: Web service - Report Resources
---

{% include functions.liquid %}

Reports are exclusively GET operations that query the state of the Metadata DB and amalgamate information from multiple sources within the MetadataDB and elsewhere (e.g. Server information). They are distinguished from /metadata/db resources when:

* they involve multiple database joins. Populating the values of a particular row would not qualify. For example, displaying an IUS with its lane, processing events and samples would not be a report because all of the information can be acquired with single joins, but reporting all of the samples in a study would be a join of at least 3 tables so would qualify as a report.
* the output format is intended to be more human readable and scriptable than JAXB's XML, e.g. CSV, JSON, or XML with an XSD. 

For more information about each resource and the correct complete URI, please follow the link to the resource-specific page.

{% assign wwwrs = site.pages | where: "www-bit", "report" %}
| Method | URI | Description |
|--------|-----|-------------|
{% for pg in wwwrs %} |<a href="{{ pg.url }}" alt="{{ pg.title }}">{{ pg.method }}</a> | <a href="{{ pg.url }}" alt="{{ pg.title }}">{{ pg.uri }}</a> | {{pg.summary}} | 
{% endfor %}

