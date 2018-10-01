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

* TOC
{:toc}

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

## Cron jobs for workflow launching and propagating

See [Cron jobs for workflow launching and propagating]({{version_url}}/admins/launching-propagating)

## See Also

There's a list of references 
[for Niassa Administrators]({{version_url}}/admins).