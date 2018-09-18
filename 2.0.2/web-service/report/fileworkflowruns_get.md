---
title: GET the workflow run objects for a particular file or set of files.
uri: /reports/fileworkflowruns
method: GET
summary: Retrieves the workflow runs associated with a file or a set of files using one of three search patterns.
layout: page
www-bit: report
---

{% include_relative resource-template.md summary=page.summary method=page.method uri=page.uri %}

* files
: SWIDs (SeqWare accessions) for the files that we wish to base the search on, comma-delimited if more than one file is needed to be considered
search
* search
: one of "CHILDREN_VIA_PROCESSING_RELATIONSHIP", "CHILDREN_VIA_IUS_WORKFLOW_RUN", "CHILDREN_VIA_LANE_WORKFLOW_RUN." In the first search pattern, workflow_runs found by joining the File, Processing, processing_relationship, and Workflow_Run tables are reported. In the second search pattern, children found by looking at potentially related children found by looking through workflow_runs that share an IUS but have not been successfully linked to the processing hierarchy are returned. In the third pattern, children found by looking at the equivalent, except sharing a Lane instead of an IUS are returned. 


## Example Requests

`/reports/fileworkflowruns&#63;files=287714&amp;search=CHILDREN_VIA_PROCESSING_RELATIONSHIP`
`/reports/fileworkflowruns&#63;files=10,20,30&amp;search=CHILDREN_VIA_IUS_WORKFLOW_RUN`