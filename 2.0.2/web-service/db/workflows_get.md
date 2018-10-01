---
title: GET /workflows
uri: /workflows
method: GET
summary: Retrieves a list of workflows.
layout: page
www-bit: db
---

{% include_relative resource-template.md summary=page.summary method=page.method uri=page.uri %}


* id
: Returns the Workflow with the given id (primary key, not SWID)
* name
: Returns the Workflow(s) with the given name
* version
: Must be used in combination with the "name" parameter. Returns the named workflow with the given version.
