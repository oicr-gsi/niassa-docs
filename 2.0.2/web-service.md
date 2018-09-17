---
layout: page
title: Web Service
---
{% include functions.liquid %}

The Niassa Web service in Niassa is a RESTful API to facilitate access to the 
[MetaDB]({{version_url}}/metadb) with user-specific permissions. It is also used
extensively by [Pipeline]({{version_url}}/pipeline) and the 
[MetaDB]({{version_url}}/metadb) to update and maintain analysis status and 
records. It allows privileged users to launch and monitor
workflows and pipelines remotely without having any local Niassa dependencies (other than the `settings` file).


* TOC
{:toc}

## Ways to access the Web Service 

* **Use the [CLI]({{version_url}}/CLI) with the Web service enabled:** The only configuration necessary is to change your .seqware/settings file to point to the Web service. The CLI will use the indicated Web service and user.
* **Use the Java API**: When writing Niassa plugins or workflow modules, you can access the Webservice through the 
[Metadata](https://github.com/oicr-gsi/niassa/blob/develop/seqware-common/src/main/java/net/sourceforge/seqware/common/metadata/Metadata.java) 
object. This object gives you more direct control while hiding the business logic. 
For example, you can install a new workflow, create processing events, and 
schedule workflow runs programmatically through this system.


## Building and Installing

See [Installation]({{version_url}}/installation).


## Environment Configuration
For both users and developers, the [User Settings]({{version_url}}/environment/user-configuration)
file needs to be configured to use the Web service rather than `database` or
`nometadata`. This file is usually located at `~/.seqware/settings` but can also be set using the environment variable `SEQWARE_SETTINGS`. Below is an
example snippet.

```
SW_METADATA_METHOD=webservice
SW_REST_URL=http://localhost:8080/seqware-webservice-{{ version }}
SW_REST_USER=admin@admin.com
SW_REST_PASS=admin
```

## Authentication and Authorization

Usernames and salted passwords are stored in the Niassa 
[MetaDB]({{version_url}}/metadb) and authentication through the happens through 
the Web Service. The usernames and passwords are stored in the 
[User Settings]({{version_url}}/environment/user-configuration) file under the
`SW_REST_USER` and `SW_REST_PASS` properties.

Once authenticated, Niassa has a further level of 
authorization. Not all users should be able to access all studies in the MetaDB. 
Authorization through the Web Service allows those users to view and change only 
those studies that they have permission to view.


## Niassa Web service API

Operations on the metadata resources are primarily for read-only access of the
Niassa metadata database. Any PUT or POST operations add only one row to one
table in the Niassa Metadata database. Queries are added to the URI as query 
parameters, e.g. /metadata/db/studies?title=PCSI to find all of the studies with
the title of PCSI. Retrieving a list of all items in a table is plural 
e.g. /metadata/db/studies. Retrieving a particular table row is singular 
e.g. /metadata/db/study/63. Additional fields can be shown by adding ?show to the query parameters, 
e.g. /metadata/db/lanes/1234?show=workflowRun shows the workflow run associated 
with this lane. 

Types of resources:
* [Data]({{version_url}}/web-service-api/db) - Resources that add, retrieve or 
  update one or more rows from a database table.
* [Reports]({{version_url}}/web-service-api/report) - Resources that query the 
  state of the Metadata DB and amalgamate information from multiple database 
  tables in order to report on the state of a study, sequencer run, sample, etc. 

## Pipeline Resources

These jobs loosely correspond with tasks performed by Niassa Pipeline, for
example, launching workflows, modules, and plugins. Pipeline tasks are
distinguished from Metadata tasks because they trigger more advanced processing
of the data by systems other than those associated with the Metadata DB. For
example, jobs triggered on the pipeline may cause a job to be run on a server,
or a ZIP file to be uploaded and installed as a workflow.

At the moment, a workflow can be launched through a [job](/docs/webservice-api/pipeline/job/) resource.


