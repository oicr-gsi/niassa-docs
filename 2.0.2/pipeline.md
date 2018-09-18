---
layout: page
title: Pipeline
---
{% include functions.liquid %}

The Niassa Pipeline sub-project is the heart of the overall Niassa
project. Niassa is a workflow engine, a Java workflow API, and a series of tools for installing, running, and monitoring workflows.

{% include figure.html img="images/pipeline_hpc_oozie.png" title="Niassa pipeline diagram" caption="" %}

{% include seqware-note.md %}


## Features

Niassa Pipeline has several key features that distinguish it from other open source and private workflow solutions. These include:

* developer framework focused
* supports user-created workflows
* focused on automated analysis
* includes cluster abstraction
* implements a self-contained workflow packaging standard
* supports detailed provenance tracking
* includes fault tolerance
* focuses on meeting workflow needs of big projects (thousands of samples)
* is open source

# Reference

The following document contains links to all reference material for working with Niassa Pipeline. To get started with Niassa, we recommend our [Getting Started]({{version_url}}/getting-started) guides.

A complete [command line interface (CLI)]({{version_url}}/CLI) reference is available.

* TOC
{:toc}


## Environment Configuration 

Niassa requires a number of configuration options set in its environment. Find the reference here:

* [User Settings]({{version_url}}/environment/user-configuration)
: Configuring .seqware/settings and oozie config files.
* [User Host Configuration]({{version_url}}/environment/host-user)
: Setting up the `user` host that automatically monitors and launches 
	workflows	and where users can schedule their workflows

## Workflow engines

We currently support four workflow engines: oozie, oozie-sge, whitestar, and whitestar-sge. Our current recommended combination is Java workflows with the Oozie-SGE engine.

* Oozie uses the Hadoop Workflow Scheduler to schedule steps in workflows on 
	the Hadoop ecosystem (JobTrackers and TaskTrackers). 
* Oozie-sge uses Oozie, but in conjunction with a oozie-sge plugin to schedule 
	steps in workflows on a pre-existing sge cluster. 
* WhiteStar is a synchronous workflow engine used by Niassa developers to 
	debug. It runs steps locally via Bash. 
* WhiteStar-AGE runs steps on a local Sun Grid Engine (or equivalent) cluster.     

## Workflows

Workflows consist of an ordered series of calls to command line tools that 
operate on files read from and written to a shared filesystem. Individual steps
usually run on a cluster node.

* [Java Workflows]({{version_url}}/workflows/java-workflows)
: Java is the workflow language for all workflow development.
* [Workflow Bundles]({{version_url}}/workflows/workflow_bundles)
: We rely on a bundle format for packaging up and exchanging workflows. This
	document describes the format and directory structure.
* [Workflow INI Files]({{version_url}}/workflows/ini-files)
: This document describes the INI configuration file used to describe 
	workflow parameters.
* [Workflow Manifest File]({{version_url}}/workflows/manifest)
: This document describes the XML file manifest used to describe workflows. 
	It provides workflow names, versions, descriptions, and information for running and testing the workflow.
* [Workflow Conventions]({{version_url}}/workflows/conventions) 
: exit codes and metatypes

## Developer Tutorials

* [Debugging, Troubleshooting, & Restarting Workflows]({{version_url}}/workflows/debugging-workflows)
: A guide to debugging, troubleshooting, and restarting failed workflows for the Oozie workflow engine.

* [Developing in Partial Niassa Environments with Whitestar]({{version_url}}/workflows/partial-environments)
: A guide to developing without dependencies such as Oozie, SGE/Hadoop, and metadata (via the Niassa webservice).  

## Automation

* [Deciders]({{version_url}}/deciders)
: Java-based deciders allow for automatic parameterization and calling of workflows in Niassa Pipeline. It allows you to easily encode the parent workflow and file types that, when present, enable a subsequent workflow to be launched. An extensive reference on BasicDecider and OicrDecider classes.
* Rules-based automation with [Shesmu](https://github.com/oicr-gsi/shesmu)
: A centralised decision-making system that accepts metadata from [Cerberus](https://github.com/oicr-gsi/cerberus) and other services and uses simplified configuration files to take actions such as launching workflows. 

## Reporting

A major focus of the Niassa Web Service is providing reporting resources. These are command line tools that are particularly useful for generating reports for Niassa entities such as workflow runs and their outputs.

* `seqware workflow-run report`
: The details of a given workflow-run, including the identity and library samples, input and output files from one or more workflow runs.
* File Provenance through Cerberus
: For practical purposes, we recommend setting up [Cerberus](https://github.com/oicr-gsi/cerberus), which provides as streaming, filterable view of Niassa objects.
