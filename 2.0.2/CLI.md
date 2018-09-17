---
layout: page
title: Command Line Interface
nav: false
---

This document describes the Niassa Command Line Interface (CLI).

{% include_relative seqware-note.md %}

```
Usage: seqware [<flag>]
       seqware <command> [--help]

Commands:
  annotate      Add arbitrary key/value pairs to seqware objects
  query         Display ad-hoc information about seqware objects
  bundle        Interact with a workflow bundle during development/admin
  copy          Copy files between local and remote file systems
  create        Create new seqware objects (e.g., study)
  files         Extract information about workflow output files
  study         Extract information about studies
  workflow      Interact with workflows
  workflow-run  Interact with workflow runs
  checkdb       Check the seqware database for convention errors
  check         Check the seqware environment for configuration issues
  dev           Advanced commands that are useful for developers or debugging

Flags:
  --help        Print help out
  --version     Print Seqware's version
  --metadata    Print metadata environment
```


<h1>Commands</h1>

* TOC
{:toc}



# annotate

Add arbitrary key/value pairs to seqware objects. 

Allows you to annotate [MetaDB]({{version_url}}/metadb) rows with Key-Value pairs or 'skip' fields. The 'skip' field is a boolean that indicates to a decider that the results from that lane, IUS or sequencer run should be skipped in downstream processing. When the 'skip' field is enabled through this plugin, a key-value pair can also be added that describes the reason for skipping the lane, ius or sequencer run.

```
Usage: seqware annotate [--help]
       seqware annotate <object> --accession <swid> --key <key> --val <value>
       seqware annotate <object> --accession <swid> --skip [--reason <text>]
       seqware annotate <object> --csv <file>
```

| Parameter  | Description  |
|---|---|
| _object_ | A table from the [MetaDB]({{version_url}}/metadb). Permitted: experiment, file, lane, ius, processing, sample, sequencer-run, study, workflow, workflow-run |
| \--accession <swid>  |  The SWID of the object to annotate. Required. |
|\--csv <file>  | Bulk annotation from CSV file of: accession, key, value  |
| \--key <key>   | The identifier of the annotation  |
| \--reason <text>  | The reason the object is skipped  |
| \--skip  | Sets the skip attribute flag on the object  |
| \--val <value>  | The value of the annotation  |

Objects (tables in the MetaDB):







**Examples**

Set the 'skip' value of lane with SWID 1234 to 'true'. No key-value annotations are added.

```
seqware annotate lane --accession 1234 --skip
```

Set the 'skip' value of sequencer run with SWID 3456 to 'true' and enter a key-value pair with the key "skip" and value "Improperly entered into the LIMS". Notice that the key is not specified but defaults to 'skip'.

```
seqware annotate sequencer-run --accession 3456 --key 'skip' --val 'Improperly entered into the LIMS'
```

Annotate the IUS with SWID 2345 with a key-value pair of key "dual-barcodes" and value "barcode 1". The 'skip' field is not affected.


```
seqware annotate ius --accession 2345 --key "dual-barcodes" --val "barcode 1"
```

Bulk insert annotation on experiments with file file.csv.

```
$ cat file.csv
106288,t11,v11
106288,t21,v21
106288,t31,v11
1234,key1,value1
1000,key1,value1
321,key1,value1
$ seqware annotate experiment --csv file.csv
```

# query

Display an object in JSON format.

```
Usage: seqware query [--help]
       seqware query <object> --accession <swid>
```

Objects (tables in the MetaDB):
* experiment
* file
* lane
* ius
* processing
* sample
* sequencer-run
* study
* workflow
* workflow-run

| Parameter  | Description  |
|---|---|
|\--accession <swid>  | The SWID of the object to display   |
|\--annotations  | Display only the annotations from the object   |

**Examples**

Return information about IUS with accession 11776576l

```
$ seqware query ius --accession 11776576
{
  "iusId": 629358,
  "lane": null,
  "owner": null,
  "sample": null,
  "limsKey": null,
  "name": null,
  "alias": null,
  "description": null,
  "tag": null,
  "swAccession": 11776576,
  "createTimestamp": "Mar 20, 2018 8:08:15 AM",
  "updateTimestamp": "Mar 20, 2018 8:08:15 AM",
  "processings": [],
  "workflowRuns": [],
  "iusAttributes": [],
  "skip": false,
  "isHasFile": false,
  "isSelected": false
}
```



# bundle

Interact with a workflow bundle.

```
Usage: seqware bundle [--help]
       seqware bundle <sub-command> [--help]
```

## dry-run
Perform all steps to prepare for a launch, but not actually launch

```
Usage: seqware bundle dry-run [--help]
       seqware bundle dry-run --dir <bundle-dir> <params>
```

| Parameter  | Description  |
|---|---|
|\--dir <bundle-dir>  | The root directory of the bundle  |
|\--engine <type>  | The engine that will process the workflow run. May be one of: oozie, oozie-sge, whitestar, whitestar-parallel, whitestar-sge. Defaults to the value of SW_DEFAULT_WORKFLOW_ENGINE or 'oozie' if not specified.  |
|\--ini <ini-file>  | An ini file to configure the workflow run. Repeat this parameter to provide multiple files. Defaults to the value of the 'config' node in metadata.xml  |
|\--name <wf-name>  | The name of the workflow in the bundle  |
|\--version <ver>  | The version of the workflow in the bundle  |


## install
Inform the Seqware system of the availability of a standard bundle. 

The install process will first unzip the workflow bundle into the directory specified by `SW_BUNDLE_DIR` in the .seqware/settings file (skipping files that already exist) and install the unzipped location in Niassa. After installation, the workflow SWID will be printed to standard out.

```
Usage: seqware bundle install [--help]
       seqware bundle install --zip <bundle-zip>
```

| Parameter  | Description  |
|---|---|
|\--zip <bundle-zip>  | The zip file of the bundle  |

## install-dir-only
Inform the Seqware system of the availability of an unzipped bundle. Installs the workflow in Niassa. After installation, the workflow SWID will be printed to standard out.

```
Usage: seqware bundle install-dir-only [--help]
       seqware bundle install-dir-only --dir <bundle-dir>
```

| Parameter  | Description  |
|---|---|
|\--dir <bundle-dir>  | The directory containing a Niassa workflow |


## launch
Launch a specified workflow in a bundle directory.

```
Usage: seqware bundle launch [--help]
       seqware bundle launch --dir <bundle-dir> <params>
```

| Parameter  | Description  |
|---|---|
|\--dir <bundle-dir>  | The root directory of the bundle  |
|\--engine <type>  | The engine that will process the workflow run. May be one of: oozie, oozie-sge, whitestar, whitestar-parallel, whitestar-sge. Defaults to the value of SW_DEFAULT_WORKFLOW_ENGINE or 'oozie' if not specified.  |
|\--ini <ini-file>  | An ini file to configure the workflow run. Repeat this parameter to provide multiple files. Defaults to the value of the 'config' node in metadata.xml  |
|\--name <wf-name>  | The name of the workflow in the bundle  |
|\--version <ver>  | The version of the workflow in the bundle  |
|\--override <key=value>   |  Override specific parameters from the workflow.ini |
|\--no-metadata  | Run without per-step workflow metadata tracking   |


## list
List workflows within a bundle directory

```
Usage: seqware bundle list [--help]
       seqware bundle list --dir <bundle-dir>
```

| Parameter  | Description  |
|---|---|
|\--dir <bundle-dir>  | The root directory of the bundle  |

## package
Package a bundle directory into a zip file

```
Usage: seqware bundle package [--help]
       seqware bundle package --dir <bundle-dir> <params>
```

| Parameter  | Description  |
|---|---|
|\--dir <bundle-dir>  | The root directory of the bundle  |
|\--to <dir>  | The directory to place the zip. Defaults to the current directory  |

# copy 
Convenience tool to copy files between local and remote file systems, e.g. S3. If destination is a local directory, the input file's name will be used for the output file.

```
Usage: seqware copy [--help]
       seqware copy <source> <destination>
```

# create

Create new seqware objects (e.g., study) in the [MetaDB]({{version_url}}/metadb). It is highly recommended to be familiar with the layout of the MetaDB before creating objects through the CLI.

```
Usage: seqware create [--help]
       seqware create <sub-command> [--help]
```

## experiment

Create an experiment in the MetaDB. Note: run with \--interactive to see what the controlled vocabulary is in your database.

```
Usage: seqware create experiment [--help]
       seqware create experiment --interactive
       seqware create experiment <fields>
```

| Parameter  | Description  |
|---|---|
|\--description <val>  |   |
|\--platform-id <val>  |  Dynamic-valued column |
|\--study-accession <val>  |   |
|\--title <val>  |   |

## file

Not implemented.

```
seqware: 'create file' is not a seqware command. See 'seqware create --help'.
```


## ius

Create an IUS in the MetaDB.

```
Usage: seqware create ius [--help]
       seqware create ius --interactive
       seqware create ius --barcode <val> --description <val> --lane-accession <val> --name <val> --sample-accession <val> --skip <val>
```

| Parameter  |
|---|---|
|\--barcode <val>  |
|\--description <val>  |
|\--lane-accession <val>  |
|\--name <val>  |
|\--sample-accession <val>  |
|\--skip <val> |

## ius-lims-key

Create an IUS LIMS key in the MetaDB.
```
Usage: seqware create ius-lims-key --help
       seqware create ius-lims-key --provider <val> --id <val> --version <val> --last-modified <val> 
```

| Parameter  | Description  |
|---|---|
|\--provider <val>  |   |
|\--id <val>  |   |
|\--version <val>  |   |
|\--last-modified <val>  |   |

## lane

Create a lane in the MetaDB. Note: run with \--interactive to see what the controlled vocabulary is in your database.

```
Usage: seqware create lane [--help]
       seqware create lane --interactive
       seqware create lane --cycle-descriptor <val> --description <val> --lane-number <val> --library-selection-accession <val> --library-source-accession <val> --library-strategy-accession <val> --name <val> --sequencer-run-accession <val> 
```

| Parameter  | Description  |
|---|---|
|\--cycle-descriptor <val>  |   |
|\--description <val>  |   |
|\--lane-number <val>  |   |
|\--library-selection-accession <val>  |  Dynamic-valued field  |
|\--library-source-accession <val>  |   Dynamic-valued field |
|\--library-strategy-accession <val>  |  Dynamic-valued field  |
|\--name <val>  |   |
|\--sequencer-run-accession <val>  |   |
|\--skip <val>  |   |
|\--study-type-accession <val>  | Dynamic-valued field  |


## sample

Create a sample in the MetaDB. Note: run with \--interactive to see what the controlled vocabulary is in your database.

```
Usage: seqware create sample [--help]
       seqware create sample --interactive
       seqware create sample --description <val> --experiment-accession <val> --organism-id <val> --title <val>

```

| Parameter  | Description  |
|---|---|
|\--description <val>  |   |
|\--experiment-accession <val>  |   |
|\--organism-id <val>  | Dynamic-valued field  |
|\--title <val>  |   |

## sequencer-run

Create a sequencer run in the MetaDB. Note: run with \--interactive to see what the controlled vocabulary is in your database.

```
Usage: seqware create sequencer-run [--help]
       seqware create sequencer-run --interactive
       seqware create sequencer-run --description <val> --file-path <val> --name <val> --paired-end <val> --platform-accession <val>
```

| Parameter  | Description  |
|---|---|
|\--description <val>  |   |
|\--file-path <val>  |   |
|\--paired-end <val>  |   |
|\--platform-accession <val>  | Dynamic-valued field  |
|\--skip <val>  |   |

## study

Create a study in the MetaDB. Note: run with \--interactive to see what the controlled vocabulary is in your database.

```
Usage: seqware create study [--help]
       seqware create study --interactive
       seqware create study --accession <val> --center-name <val> --center-project-name <val> --description <val> --study-type <val> --title <val>
```

| Parameter  | Description  |
|---|---|
|\--accession <val>  |   |
|\--center-name <val>  |   |
|\--center-project-name <val>  |   |
|\--description <val>  |   |
|\--study-type <val>  |  Dynamic-valued field |
|\--title <val>  |   |

## workflow

Create a workflow in the MetaDB. Note: run with \--interactive to see what the controlled vocabulary is in your database.

```
Usage: seqware create workflow [--help]
       seqware create workflow --interactive
       seqware create workflow --name <val> --version <val>  --description <val>
```



| Parameter  | Description  |
|---|---|
|\--name <val>  |   |
|\--version <val>  |   |
|\--description <val>  |   |

## workflow-run

Create a workflow run in the MetaDB. Note: run with \--interactive to see what the controlled vocabulary is in your database.

```
Usage: seqware create workflow-run [--help]
       seqware create workflow-run --interactive
       seqware create workflow-run --workflow-accession <val> --parent-accession <swid> [ parameters ]
```

| Parameter  | Description  |
|---|---|
|\--workflow-accession <val>  | The SWID of a parent to the workflow run  |
|\--parent-accession <swid>  | Repeat this parameter to provide multiple parents  |
|\--file <type::meta-type::path>  | Add (output) files as a part of the workflow run. Repeat this parameter to add multiple files  |
|\--input-file <swid>  | Add (input) files as a part of the workflow run. Repeat this parameter to add multiple files.  |

# files
Extract information about workflow output files.

```
Usage: seqware files --help
       seqware files <sub-command> [--help]
```

## report
A report of the provenance of output files.

```
Usage: seqware files report --help
       seqware files report <params>
```

| Parameter  | Description  |
|---|---|
|\--out <file>  | The name of the output file  |
|\--lane-SWID <value>  | Limit files to the specified Lane sw_accession. Can occur multiple times.  |
|\--ius-SWID <value>  | Limit files to the specified IUS sw_accession. Can occur multiple times.  |
|\--study-name <value>  | Limit files to the specified Full study name. Can occur multiple times.  |
|\--sample-name <value>  | Limit files to the specified Full sample name. Can occur multiple times.  |
|\--root-sample-name <value>  |  Limit files to the specified Full root sample name. Can occur multiple times. |
|\--sequencer-run-name <value>  |  Limit files to the specified Full sequencer run name. Can occur multiple times. |
|\--organism <value>  | Limit files to the specified organism id. Can occur multiple times.  |
|\--workflow-run-status <value>  | Limit files to the specified Workflow run status. Can occur multiple times.  |
|\--processing-status <value>  | Limit files to the specified Processing status. Can occur multiple times.  |
|\--processing-SWID <value>  |  Limit files to the specified processing sw_accession. Can occur multiple times. |
|\--lane-name <value>  |  Limit files to the specified Full lane name. Can occur multiple times. |


## refresh
Refresh the provenance report table AND/OR take down the production databse.

Takes no arguments, including `--help`. Just gets at it and makes memory usage 
on the database host spike.

```
# seqware files refresh # DO NOT RUN THIS. TRUST ME. YOU WILL REGRET IT.
```


# study

Extract information about studies.

```
Usage: seqware study --help
       seqware study <sub-command> [--help]
```

## list
List all studies.

```
Usage: seqware study list --help
       seqware study list [params]
```

| Parameter  | Description  |
|---|---|
|\--tsv  | Emit a tab-separated values list  |

# workflow
Interact with workflows.

```
Usage: seqware workflow [--help]
       seqware workflow <sub-command> [--help]
```

## ini
Generate an ini file for a workflow.

```
Usage: seqware workflow ini --help
       seqware workflow ini --accession <swid> <params>
```

| Parameter  | Description  |
|---|---|
|\--accession <swid>  | The SWID of the workflow  |
|\--out <file>  |  Where to write the file (defaults to 'workflow.ini') |


## list
List all installed workflow.

```
Usage: seqware workflow list --help
       seqware workflow list [params]
```

| Parameter  | Description  |
|---|---|
|\--tsv  | Emit a tab-separated values list  |

## report
List the details of all runs of a given workflow.

```
Usage: seqware workflow report --help
       seqware workflow report --accession <swid> <params>
       seqware workflow report --status <status> <params> 
```

| Parameter  | Description  |
|---|---|
|\--accession <swid>  | The SWID of the workflow  |
|\--status <status>  | One of [submitted, pending, running, failed, completed, submitted_cancel, cancelled, submitted_retry]  |
|\--out <file>  |  The name of the report file |
|\--tsv  | Emit a tab-separated values report  |
|\--when <date>  | The date or date-range of runs to include. If omitted, all runs included. Dates are in the form YYYY-MM-DD. Date ranges are in the form YYYY-MM-DD:YYYY-MM-DD  |

## schedule
Schedule a workflow to be run.

```
Usage: seqware workflow schedule [--help]
       seqware workflow schedule --accession <swid> --host <host> <params>

```

| Parameter  | Description  |
|---|---|
|\--accession <swid>  | The SWID of the workflow to be run  |
|\--host <host>  | The host on which to launch the workflow run  |
|\--engine <type>  | The engine that will process the workflow run. May be one of: oozie, oozie-sge, whitestar, whitestar-parallel, whitestar-sge. Defaults to the value of SW_DEFAULT_WORKFLOW_ENGINE or 'oozie' if not specified.  |
|\--parent-accession <swid>  | The SWID of a parent to the workflow run. Repeat this parameter to provide multiple parents  |
|\--override <key=value>  | Override specific parameters from the workflow.ini  |
|\--ini <ini-file>  | An ini file to configure the workflow run. Repeat this parameter to provide multiple files  |
|\--input-file <input-file>  | Track input files to workflow runs. Repeat this parameter to provide multiple files   |

# workflow-run

Interact with workflow runs.

```
Usage: seqware workflow-run --help
       seqware workflow-run <sub-command> [--help]
```

## cancel
Cancel a submitted or running workflow run

```
Usage: seqware workflow-run cancel --help
       seqware workflow-run cancel --accession <swid>
```

| Parameter  | Description  |
|---|---|
|\--accession <swid>  |  The SWID of the workflow run, comma separated (no-spaces) for multiple SWIDs |


## launch-scheduled
Launch scheduled workflow runs

```
Usage: seqware workflow-run launch-scheduled --help
       seqware workflow-run launch-scheduled [<params>]
```

| Parameter  | Description  |
|---|---|
|\--accession <swid>  | Launch the specified workflow-run. Repeat this to provide multiple runs.  |
|\--host <value>  | Use the specified value instead of the local when selecting which workflow-runs to launch.  |

## propagate-statuses
Propagate workflow engine statuses to seqware meta DB

```
Usage: seqware workflow-run propagate-statuses --help
       seqware workflow-run propagate-statuses <params>
```

| Parameter  | Description  |
|---|---|
|\--accession <swid>  | Propagate the status of the specified workflow-run. Repeat this parameter to specify multiple workflow-runs.  |
|\--host <value>  | Use the specified value instead of the local hostname when selecting which workflow-runs to check.  |
|\--threads <num>  | The number of concurrent worker threads (default 1)  |

## retry
Retry a failed or cancelled workflow run, skipping completed steps

```

Usage: seqware workflow-run retry --help
       seqware workflow-run retry --accession <swid>

```

| Parameter  | Description  |
|---|---|
|\--accession <swid> | The SWID of the workflow run, comma separated (no-spaces) for multiple SWIDs  |

## reschedule
Reschedule a workflow-run to re-run from scratch as a new run

```
Usage: seqware workflow-run reschedule [--help]
       seqware workflow-run reschedule --accession <swid> <params>
```

| Parameter  | Description  |
|---|---|
|\--accession <swid>  | The SWID of the workflow-run to be rescheduled  |
|\--host <host>  | The host on which to launch the workflow run  |
|\--engine <type>  | The engine that will process the workflow run. May be one of: oozie, oozie-sge, whitestar, whitestar-parallel, whitestar-sge. Defaults to the value of SW_DEFAULT_WORKFLOW_ENGINE or 'oozie' if not specified.  |

## stderr
Obtain the stderr output of the run

```
Usage: seqware workflow-run stderr --help
       seqware workflow-run stderr --accession <swid> <params>
```

| Parameter  | Description  |
|---|---|
|\--accession <swid>  | The SWID of the workflow run  |
|\--out <file>  |   The name of the file to write the stderr. Defaults to `swid`.err |

## stdout
Obtain the stdout output of the run

```
Usage: seqware workflow-run stdout --help
       seqware workflow-run stdout --accession <swid> <params>

```

| Parameter  | Description  |
|---|---|
|\--accession <swid>  | The SWID of the workflow run  |
|\--out <file>  | The name of the file to write the stdout. Defaults to `swid`.out  |

## report
The details of a given workflow-run. Generates a tab-separated file containing information about one or more workflow runs. The workflow runs can either be retrieved according to time period, workflow type, or by workflow run accession.

```
Usage: seqware workflow-run report --help
       seqware workflow-run report --accession <swid> <params>
```

| Parameter  | Description  |
|---|---|
|\--accession <swid>  | The SWID of the workflow run  |
|\--out <file>  |  The name of the report file |
|\--tsv  |  Emit a tab-separated values report |

## watch
Watch a workflow-run in progress

```
Usage: seqware workflow-run watch --help
       seqware workflow-run watch --accession <swid>
```

| Parameter  | Description  |
|---|---|
|\--accession <swid  | The SWID of the workflow run  |

## ini
Output the effective ini for a workflow run

```
Usage: seqware workflow-run ini --help
       seqware workflow-run ini --accession <swid> <params>
```

| Parameter  | Description  |
|---|---|
|\--accession <swid>  | The SWID of the workflow run  |
|\--out <file>  |  The name of the ini file |

## delete
Recursively delete workflow runs based on the SWID of an ancestral sequencer run, lane, or workflow run.

```
Usage: seqware workflow-run delete --help
       seqware workflow-run delete --accession <swid> <params>
```

| Parameter  | Description  |
|---|---|
|\--accession <swid>  | The SWID of the desired target  |
|\--key <file>   | Delete workflow runs given a key file containing records to be deleted in one transaction.  |
|\--out <file>  |  Override the filename for where to write a key file containing records. |

# checkdb
Using a direct database connection, check whether the meta db contains any content that deviates from recommended conventions.

```
Usage: seqware checkdb --help
       seqware checkdb
```

# check
Check to see if your seqware environment is running. Requires direct access to 
the metaDB. Required in SEQWARE_SETTINGS: SW_DB_SERVER, SW_DB, SW_DB_USER, SW_DB_PASS

```
Usage: seqware check--help
       seqware check <params>
```

| Parameter  | Description  |
|---|---|
|\--help, h, ?  | Provides help message  |
|\--master, m  |  Include this parameter to test if you are on a master node and not a user one  |
|\--tutorial, t |  Include this parameter to test by going through the seqware tutorials |

# dev
Advanced commands for debugging and developers.

```
Usage: seqware dev --help
       seqware dev <sub-command> [--help]
```

## map
Map from various unique identifiers to unique identifiers.

```
Usage: seqware dev map --help
       seqware dev map <params>
```

| Parameter  | Description  |
|---|---|
|\--engine-id <engine-id>  | Convert from an engine ID (for oozie this is a job ID) to workflow run accession  |
|\--action-id <sge-id>  |  Convert from an external ID (for oozie-sge this is a SGE ID) to workflow run accession (expensive) |

## files2workflowruns
Identify workflow runs that used specified files as input for workflow runs

```
Usage: seqware dev files2workflowruns --help
       seqware dev files2workflowruns <params>
```

| Parameter  | Description  |
|---|---|
|\--file <file-swa> | List of files by sw_accession, repeat for multiple files  |
|\--workflow <workflow-swa>  | List of workflows, repeat for multiple workflows  |