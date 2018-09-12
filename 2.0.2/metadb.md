---
layout: page
title: MetaDB
nav: false
---
{% include functions.liquid %}

The Niassa MetaDB is a metadatabase modeled on the 
[Short Read Archive](http://www.ncbi.nlm.nih.gov/Traces/sra) schema and built 
with [PostgreSQL](http://www.postgresql.org/). It includes tables to model the 
run of the sequencers, analysis "processing" events, and installed workflows. A
common database for tracking metadata, used by the other Niassa components.

It is not a 100% translation of the SRA data model. There are places where the 
SRA model provides greater flexibility and places where we changed the model 
for our own purposes.

## Tables

The Niassa MetaDB is split into two halves: the top half with sample and sequencing information typically created by migration by the LIMS; and the bottom half that contains information about analysis jobs and used by Niassa to maintain details on what workflows have been installed and run. The two halves are usually* linked by the IUS.

The primary tables in the top half (LIMS/sample information) include:

* `study` : a project, e.g. "Pancreatic Cancer Sequencing Initiative"
* `experiment` : a logical unit of work within a study (we don't use these much)
* `sample` : hierarchical table that describes the biological entities that belong to an experiment and study
* `ius` : indivisible unit of sequencing; usually corresponds to a sequencer run + lane + index/barcode; can point to a unique identifier in a LIMS through a LIMSkey
* `sequencer_run` : distinct run by a sequencing instrument
* `lane` : unit of sequencing on an sequencer run

The primary tables in the bottom half (analysis information) include:

* `workflow` : an installed workflow from pipeline
* `workflow_run` : a particular execution of a workflow
* `processing` : a step in a workflow run
* `file` : an output file associated with a workflow run

\* sometimes by lane in special cases

The diagram below shows the relationships between these tables.

![Simplified DB diagram]({{version_url}}/images/metadb_simple.png)

### Special tables

The `sample` and `processing` tables are 'special' in that they have linker 
tables where they can be parents of other samples/processing events. Navigating
through these tables in the database requires a considerable amount of recursion.

* only the 'root' of a particular `sample` hierarchy will have a reference to an
`experiment`, and only the leaf of the `sample` hierarchy will refer to an `ius`.
* `processing` rows will either have references to the `workflow_run` table in
the `workflow_run_id` column when it is a root `processing` event. All other 
`processing` events will refer to the `workflow_run` in `ancestor_workflow_run_id`.

The example in the next section will show how this works in practice.

### Attributes and linker tables

The following is a more complicated database diagram that includes tables used for 
linking metadata to workflow runs and all attribute tables.

![Complicated DB]({{version_url}}/images/metadb_complicated.png)

Note there are also attribute tables for each of the above tables (of the form 
`X_attribute`) which maintain arbitrary key value information that can be used 
to guide custom behaviour at your particular Niassa site. They can be updated 
using the `seqware annotate` function in the CLI.

Finally, there are a host of linking tables (of the form processing_X) which 
maintain information on which steps in your workflow are attributable to what 
physical entities. 


## Structure of a typical workflow run

Niassa uses the MetaDB to track processing events and to connect samples to
processed data and/or to connect one set of processed data to another. For
example linking between a biological sample, a sequencing run, the generated
raw sequence data, and downstream processed sequence data is possible. 
First, an example of the entities that exist in the MetaDB 
after a "straightforward" workflow run.

![Simple DB](/assets/images/metadb/Study_hierarchy.png)

The 'proper' way to traverse the database from a study or sequencer_run is to
go through ius, processing_ius, and processing. 
So from study:

* traverse through experiment
* sample to sample through the sample_hierarchy table
* through ius
* join ius and processing on processing_ius
* processing to processing through the processing_relationship table
* locate files connected to processings along the way

As previously mentioned, sample and processing are hierarchical.
Each new processing event has the parent of the
previous processing event; for example, if WorkflowA is run on IUSA, and then
WorkflowB is also run on IUSA using the results from WorkflowA, the processing
tree looks like IUSA -> processing_ius -> Processing of step 1,2,3 of WorkflowA
-> Processing of step 1,2,3 of WorkflowB.  Processings are linked to
workflow_runs, which have connections to IUSes through ius_workflow_runs and/or
Lanes through lane_workflow_runs, as a convenience only. These links are not
necessarily present (though we try to make sure they are). The safest way to
find all of the processing events linked with a Study is to traverse through
the IUS to the root Processing through processing_ius, not through
ius_workflow_runs.

Just for fun, there are a number of linker tables that will connect other 
entities in the top half of the database with `processing`. The only other linker
table we use at the moment is `processing_lane`, specifically for associating 
files that are better linked to lane instead of sample (like 
Undetermined_indices files for bcl2fastq).