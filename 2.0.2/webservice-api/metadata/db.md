---
layout: page
title: Metadata - Data Resources
nav: true
---
{% include functions.liquid %}

This sub-domain is exclusively for low-level access of database tables. Each URI actually retrieves one or more rows from a database table. The output format is exclusively XML produced by JAXB.

Retrieving a list of all items in a table is plural e.g. /metadata/db/studies

* GET: Grouped resources are of the type TableNameList, where TableName is the singular name of the table.
* POST: Add a new row to the table
* PUT and DELETE are not permitted

Queries on the table as a whole can be added to the URI as query parameters, e.g. /metadata/db/studies?title=PCSI to find all of the studies with the title of PCSI 

Retrieving a particular table row is singular e.g. /metadata/db/study/63

* GET: returns the table row with the given SWID
* PUT: updates the row in the table with the row in the request
* POST and DELETE are not permitted

Additional fields can be shown by adding ?show to the query parameters, e.g. /metadata/db/lanes/1234?show=workflowRun shows the workflow run associated with this lane. 

Resources
---------------
For more information about each resource and the correct complete URI, please follow the link to the resource-specific page.

{% assign wwwdb = site.pages | where: "www-bit", "db" %}
| Method | URI | Description |
|--------|-----|-------------|
{% for pg in wwwdb %} |<a href="{{ pg.url }}" alt="{{ pg.title }}">{{ pg.method }}</a> | <a href="{{ pg.url }}" alt="{{ pg.title }}">{{ pg.uri }}</a> | {{pg.summary}} | 
{% endfor %}