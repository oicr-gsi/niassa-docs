---
layout: page
title: Getting Started
nav: true
---
{% include functions.liquid %}


The majority of this guide is dedicated to walking users and developers through 
the basics of calling and developing Niassa workflows respectively. We assume 
that people are most interested in the Niassa Pipeline sub-project and focus 
most of our time on that.  The examples below will all be based on a local VM 
but the environment on our cloud instance is almost identical, so most of the 
examples below will be applicable for both environments.

## By the End of These Tutorials

<!--
Take a look at the "Workflow Development Tutorials" on the internal wiki to make 
sure we've covered all items in the tutorials below.
-->

By the end of these tutorials you will:

* create studies, experiments, and samples in the MetaDB
* run a HelloWorld workflow in Pipeline
* create a new workflow bundle in Pipeline
* install and schedule your new workflow bundle in Pipeline and MetaDB
* generate a report on the outputs of your workflows in Pipeline and Portal
* get an overview of the administration process for Niassa
* be prepared to move on to more detailed documentation for each sub-project

Please launch your local VM and log in as <kbd>Niassa</kbd> at this time. If
you do not have a local VM or Amazon instance currently running please see the
[Installation Guide]({{version_url}}/installation) before following these 
tutorials.

## Getting Started Guides

We recommend you follow the guides in the following order: User, Developer, then Admin guides.

| <img src="{{ version_url }}/images/genome-capable_ico.png"/> | [Users]({{ version_url }}/getting-started/user-tutorial) | explains how you use the Niassa system to launch workflows and analyze data |
| <img src="{{ version_url }}/images/open-source_ico.png"/> | [Developers]({{ version_url }}/getting-started/developer-tutorial) | picks up where the User Tutorial left off and shows you how to create new workflows and make those available to other users of the system |
| <img src="{{ version_url }}/images/cloud-based_ico.png"/> |  [Administrators]({{ version_url }}/getting-started/admin-tutorial) | gives a general overview of how the various Niassa pieces fit together to automate NGS analysis within a large institution like [OICR](http://oicr.on.ca) |

## Further Reading

For other tutorials and reference material, check the [Pipeline]({{ version_url }}/pipeline) page.
