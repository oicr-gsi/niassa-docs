---
title: GET workflowparams
uri: /workflowparams
method: GET
summary: Retrieves a list of workflow parameters.
layout: page
www-bit: db
---

{% include_relative resource-template.md summary=page.summary method=page.method uri=page.uri %}


* id
: Returns the WorkflowParam with the given id (primary key, not SWID, since workflow parameters do not have SWIDs).
