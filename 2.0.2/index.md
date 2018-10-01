---
layout: page
title: Documentation
nav: true
---
{% include functions.liquid %}

This is the core documentation for the Niassa project.

* TOC
{:toc}

# Niassa Components

Niassa currently provides 3 layers of infrastructure specifically designed to
support the analysis necessary for sequencing technologies.

<img width="600" src="{{ version_url }}/images/niassa_arch.png" alt="Niassa overall architecture"/>

Click on each component for lots and lots of documentation on each component.

## [MetaDB]({{version_url}}/metadb)
Provides a common database to store metadata used by all components.

## [Pipeline]({{version_url}}/pipeline)
A workflow engine capable of wrapping and combining other tools (BFAST, BWA, SAMtools, etc) into complex pipelines, recording metadata about the analysis, and facilitates automation of pipelines based on metadata.

## [Web Service]({{version_url}}/web-service)
A programmatic API that lets people build new tools on top of the project


# Reference Material

* [Installation]({{version_url}}/installation)
* [Command line interface (CLI)]({{version_url}}/CLI)
* [Getting Started]({{version_url}}/getting-started)
* [For Niassa Administrators]({{version_url}}/admins)
