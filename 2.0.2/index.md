---
layout: page
title: Documentation
nav: true
---
{% include functions.liquid %}

This is the core documentation for the Niassa project.

* TOC
{:toc}

## Niassa Overview

Niassa currently provides 3 layers of infrastructure specifically designed to support the analysis necessary for sequencing technologies.

<img width="600" src="{{ version_url }}/images/niassa_arch.png"/>

* [MetaDB]({{version_url}}/metadb): provides a common database to store metadata used by all components.
* [Pipeline]({{version_url}}/pipeline): a workflow engine that is capable of wrapping and combining other tools (BFAST, BWA, SAMtools, etc) into complex pipelines, recording metadata about the analysis, and facilitates automation of pipelines based on metadata.
* [Web Service]({{version_url}}/web-service): a programmatic API that lets people build new tools on top of the project


## Things of note

* [Installation]({{version_url}}/installation)
* [Command line interface (CLI)]({{version_url}}/CLI)
