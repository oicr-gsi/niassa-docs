---
layout: page
title: Pipeline
nav: false
---
{% include functions.liquid %}

The Niassa Pipeline sub-project is really the heart of the overall Niassa
project. This provides the core functionality of Niassa; it is workflow
developer environment and a series of tools for installing, running, and
monitoring workflows.

Since Niassa is a fork and extension of [SeqWare](http://seqware.io), many parts 
of the system continue to be named SeqWare. Don't panic; this is intentional.

<img src="{{version_url}}/images/pipeline_hpc_oozie.png"/>

We currently support one workflow language (Java) and four 
workflow engines (oozie, oozie-sge, whitestar, and whitestar-sge). 

Our current recommended combination is
Java workflows with the Oozie-SGE engine.

* Oozie uses the Hadoop Workflow Scheduler to schedule steps in workflows on the 
	Hadoop ecosystem (JobTrackers and TaskTrackers). 
* Oozie-sge uses Oozie, but in conjunction with a oozie-sge plugin to schedule 
	steps in workflows on a pre-existing sge cluster. 
* WhiteStar is a synchronous workflow engine used by Niassa developers to debug, 
	it runs steps locally via Bash. 
* WhiteStar-sge runs steps on a local sge cluster.     

## Features

Niassa Pipeline has several key features that distinguish it from other open source and private workflow solutions. These include:

* tool-agnostic
* developer framework focused
* focused on automated analysis
* includes cluster abstraction
* supports detailed provenance tracking
* supports user-created workflows
* implements a self-contained workflow packaging standard
* includes fault tolerance
* focuses on meeting workflow needs of big projects (thousands of samples)
* is open source


## Building and Installing

See [Installation]({{version_url}}/installation)

## Reference for configuration 

Niassa requires a number of configuration options set in its environment. Find the reference here:

* [User Settings]({{version_url}}/environment/user-configuration)
: Configuring .seqware/settings and oozie config files.
* [User Host Configuration]({{version_url}}/environment/host-user)
: Setting up the `user` host that automatically monitors and launches workflows
	and where users can schedule their workflows

## Workflows

Workflows consist of an ordered series of calls to command line tools that 
operate on files read from and written to a shared filesystem. Individual steps
usually run on a cluster node.

* [Java Workflows](/docs/6-pipeline/java-workflows/)
: Java is the workflow language for all workflow development.
* [Workflow Bundle Conventions](/docs/6-pipeline/workflow_bundles/)
: We rely on a bundle format for packaging up and exchanging workflows. This
	document describes the format and directory structure.
* [Workflow Config Files](/docs/6-pipeline/config_files/)
: This document describes the ini configuration file used to describe (and type) workflow parameters.
* [Workflow Metadata File](/docs/6-pipeline/metadata_files/)
: This document describes the metadata XML file used to describe workflows. It provides workflow names, versions, descriptions, and information for running and testing the workflow.
* [File Type Conventions](/docs/6-pipeline/file-types/) 
: This document describes the standardized file meta types (MIME-like types) we use in the project and how to add files to a community-writable file type registration.

## Developer Tutorials

* [Debugging, Troubleshooting, & Restarting Workflow](/docs/6-pipeline/debug-workflows/)
: A guide to debugging, troubleshooting, and restarting failed workflows for the Oozie workflow engine.

* [Developing in Partial Niassa Environments with Whitestar](/docs/6-pipeline/partial_environments/)
: A guide to developing without dependencies such as Oozie, SGE/Hadoop, and metadata (via the Niassa webservice).  

## Modules

Modules are really optional for those interested in workflow development since
most workflows simply refer to command line tools bundled inside the workflow.
For those interested in extending the underlying Niassa system, Modules
provide a way to define new step types and could be useful for writing custom
steps that interact with databases, trigger analysis in other frameworks
(Pig/Hive/MapReduce), make calls to web services, etc. We use Modules to
provide core services in Niassa (such as file provisioning and bash shell
execution). Again, Modules are mainly targeted at core Niassa developers not
general workflow developers.

* [Writing Modules](/docs/6-pipeline/writing_modules/)
: How to extend Niassa with Java tool wrappers. Can be used in workflows or as stand-alone utilities that know how to record provenance data back to Niassa MetaDB.

## Deciders

The Deciders framework allows for the automatic parameterization and calling of workflows in Niassa Pipeline. It allows you to easily encode the parent workflow and file types that, when present, enable a subsequent workflow to be launched.

* [Basic Deciders](/docs/6-pipeline/basic_deciders/)
: A generic Decider that can be used to launch a workflow using criteria like parent workflow and input file type.
* [Making a Custom Decider](/docs/6-pipeline/custom_deciders/)
: How to create a custom decider for your workflow, useful if your logic for running your workflow is more complicated than parent workflow + input file requirements.

## Reporting

A major focus of the Niassa Web Service is providing reporting resources. These are command line tools that are particularly useful for generating reports for Niassa entities such as workflow runs and their outputs.

* `seqware files report`
: Gives you a view of all files and their position in the database hierarchy from study on down
* [Workflow Run Reporter](/docs/19-workflow-run-reporter/)
: Find the identity and library samples and input and output files from one or more workflow runs.

## Other Tools 

Other useful tools used for import, export, and annotation of results.

* [File Import](/docs/22-filelinker/)
: Import files into the Niassa MetaDB so they can be used as inputs to workflows.
* [File Export](/docs/6-pipeline/file_export/)
: Export files that are the outputs from Niassa workflows.
* [Attribute Annotator](/docs/12-attribute-annotator/)
: The underlying Niassa MetaDB supports the annotation of entities in the system using flexible key/value pairs. This guide shows you how to annotate entities in the underlying data model using this approach.
* [Database validation](/docs/24-checkdb/)
: The underlying Niassa MetaDB relies upon a certain number of conventions. This utility allows you to check that these are properly followed.  
* [Batch Metadata Injector](/docs/19/)
: The batch metadata injector is a testing/development tool used to create arbitrary metadata for testing workflows. 
* [Sanity Check](/docs/25-sanity-check-tool/)
: This tool checks whether a Niassa environment is setup properly. It currently runs for user environments and master environments.
* [ProcessingDataStructure2Dot](/docs/6-pipeline/processingdatastructure2dot/)
: This tool converts a section of the processing hierarchy "below" a processing event into a dot file for visualization. 

## Command Line Reference

We have provided a new, simplified command line interface.  The best way to learn its features is to simply add `--help`.

	$ seqware --help

	Usage: seqware [<flag>]
	seqware <command> [--help]

	Commands:
	annotate      Add arbitrary key/value pairs to seqware objects
	bundle        Interact with a workflow bundle during development/admin
	copy          Copy files between local and remote file systems
	create        Create new seqware objects (e.g., study)
	files         Extract information about workflow output files
	study         Extract information about studies
	workflow      Interact with workflows
	workflow-run  Interact with workflow runs
	checkdb       Check the seqware database for convention errors
	check         Check the seqware environment for configuration issues

	Flags:
	--help        Print help out
	--version     Print Seqware's version


	$ seqware workflow --help

	Usage: seqware workflow [--help]
	       seqware workflow <sub-command> [--help]

	Description:
	  Interact with workflows.

	Sub-commands:
	  ini           Generate an ini file for a workflow
	  list          List all installed workflows
	  report        List the details of all runs of a given workflow
	  schedule      Schedule a workflow to be run

Most commands will print the help if no arguments are provided.

The old command line still exists, and its documentation is auto-generated and covers the Plugins (which are utility tools used outside of workflows) and Modules (which model custom steps in workflows and know how to integrate with the Niassa MetaDB for metadata writeback).

* [Plugins](/docs/17-plugins/)
: The command line utilities of Niassa.
* [Modules](/docs/17a-modules/)
: Can be used as custom steps in workflows or on the command line. The most important modules are the GenericCommandRunner and the ProvisionFiles modules. These are used to call individual Bash steps in workflows and to move input/outputs around respectively.


