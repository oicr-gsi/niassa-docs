---
layout: page
title: Administrator Tutorial
---
{% include functions.liquid %}

This guide assumes you have installed Niassa already. If you have not, see 
[Installation]({{version_url}}/installation) for directions. We also recommend
you follow the [User Tutorial]({{version_url}}/getting-started-user-tutorial) and
[Developer Tutorial]({{version_url}}/getting-started-developer-tutorial) before this
guide.

This guide is intended for a Niassa administrator. Currently, it covers the
tools required to install workflows, monitor workflows globally, and launch
scheduled jobs. We also cover tools that are required for cancelling workflows
that have started and restarting workflows.

<!--In the near future, this guide will also include information on how to setup
Niassa at your site or on the cloud.  It focuses on what you need to do to get
“real” work done e.g. to run workflows you create on datasets that require
multiple nodes to analyze the data in a reasonable amount of time.  There are
basically two approaches for this, connect the VirtualBox VM to a cluster at
your local site or to launch a full Niassa cluster on EC2 using Starcluster.
Either of these approaches will leave you with a system that can process large
amounts of data. This guide assumes you are an IT admin at your site or are
working with an admin since some of the steps will require “root” privileges.
-->

By the end of these tutorials you will:

* install workflows
* monitor workflows
* see how to connect a local VM to a local cluster for running large-scale workflows
* see how to launch a cluster on Amazon’s cloud for running large-scale workflows

## How to Install a Workflow

<!-- make this install from a zip for the admin guide --> 
When provided with a tested workflow bundle from a workflow developer, the next step
is to install it, this means it will be inserted into the MetaDB via a running
web service.  During this process it will copy the bundle into your
released-bundles directory and provision it into your provisioned-bundles
directory. The provisioned bundles directory is where running workflows will
access their files.

Here is an example showing how this process works on the VM and what is
happening in the database and your released-bundles directory as you do this.

See the [Developer Tutorial]({{version_url}}/getting-started-developer-tutorial) for
how to make the zipped workflow bundle. After the zip bundle is created, the
bundle can be provided to the admin for install as below.

```
$ seqware bundle install --zip ~/packaged-bundles/Workflow_Bundle_MyHelloWorld_1.0_SeqWare_{{ version }}.zip 
Now transferring /home/niassa/packaged-bundles/Workflow_Bundle_MyHelloWorld_1.0_SeqWare_{{ version }}.zip to the directory: /home/niassa/released-bundles Please be aware, this process can take hours if the bundle is many GB in size.
Processing input: /home/niassa/packaged-bundles/Workflow_Bundle_MyHelloWorld_1.0_SeqWare_{{ version }}.zip
	output-dir: /home/niassa/released-bundles
	
WORKFLOW_ACCESSION: 16
Bundle Has Been Installed to the MetaDB and Provisioned to /home/niassa/packaged-bundles/Workflow_Bundle_MyHelloWorld_1.0_SeqWare_{{ version }}.zip!
```

What happens here is the `Workflow_Bundle_MyHelloWorld_1.0_SeqWare_{{ version }}.zip` copied to your released-bundles directory and unzip'd into your provisioned-bundles directory. The metadata about the workflow is then saved to the database.

# Monitor configuration

{% include_relative environment/admin-monitor-configuration.md %}

## See Also

**Note:**
Before proceeding further, it is worth noting that the Niassa MetaDB should be regularly backed-up. 
On our deployment, we have a cron script which calls the Files Report and `pg_dump` nightly to do back-up. 


As an admin the next steps are to explore the various sub-project guides in
this documentation.  Also take a look at the 
[installation guide]({{version_url}}/installation) which provides low-level, technical
details on how to install the components of the Niassa software stack. 


<!--
## Coming Soon

We are also preparing guides which will walk administrators through

* Hooking up to an SGE cluster (Oozie)
* Hooking up to an Oozie cluster
* Hooking up to an LSF cluster
-->

