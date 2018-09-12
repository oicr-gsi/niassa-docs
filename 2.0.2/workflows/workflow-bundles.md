---
layout: page
title: Workflow Bundles
nav: false
---
{% include functions.liquid %}

## Creating a Java Workflow Bundle

In the [Developer Tutorial](/docs/3-getting-started/developer-tutorial/) you
saw how to create a MyHelloWorld Java workflow using an archetype.

{% include_relative java-archetype.md %}

Alternatively, enter the workflow name and version you want to use.  When 
complete, you can `cd` into the new workflow directory and use `mvn install` to
build the workflow. This copies files to the correct location and pulls in
needed dependencies.

{% include_relative snippets/maven-workflow-build.md %}

You will now have a workflow directory called 
`target/Workflow_Bundle_<WorkflowName>` which contains your assembled workflow.

## Bundle format

{% include_relative workflow-org.md %}