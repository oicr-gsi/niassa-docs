---
layout: page
title: Documentation
nav: true
---
{% include functions.liquid %}

This is the core documentation for the Niassa project.

## Niassa Overview

Niassa currently provides 3 layers of infrastructure specifically designed to support the analysis necessary for sequencing technologies. The tutorials below will focus mainly on Pipeline with information about other projects provided as needed:

* **MetaDB**: provides a common database to store metadata used by all components.
* **Pipeline**: a workflow engine that is capable of wrapping and combining other tools (BFAST, BWA, SAMtools, etc) into complex pipelines, recording metadata about the analysis, and facilitates automation of pipelines based on metadata.
* **Web Service**: a programmatic API that lets people build new tools on top of the project

<img width="600" src="{{ version_url }}/images/niassa_arch.png"/>

## Niassa Users

There are three major roles that Niassa users can be classified into.

* **User**: injects input data into Niassa, runs workflows, and 
collects the results. These users also use our various command line and Portal
reporting tools.  
* **Developer**: a developer of workflows, the developer makes new workflows and are typically bioinformaticians and experts in NGS analysis techniques and tools
* **Administrator**: typically both a system administrator that installs and configures Niassa components. Also installs new workflows, automates workflow launching, and identifies failed workflows.

Depending on the size of the group these roles may be the same person, distinct people, or even distinct groups within a larger organization.

## Getting Started

We recommend you follow the guides in the following order: User, Developer, then Admin guides.

<img src="{{ version_url }}/images/genome-capable_ico.png"/> [Users]({{ version_url }}/getting-started/user-tutorial)

<img src="{{ version_url }}/images/open-source_ico.png"/> [Developers]({{ version_url }}/getting-started/developer-tutorial)

<img src="{{ version_url }}/images/cloud-based_ico.png"/> [Administrators]({{ version_url }}/getting-started/admin-tutorial)