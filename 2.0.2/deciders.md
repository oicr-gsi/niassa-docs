---
layout: page
title: Deciders
---
{% include functions.liquid %}

So far you've seen how to write a workflow that links together multiple 
commands and parameterizes them through an ini file. But the reality of large 
projects dictates that you can't run each workflow manually. That's where 
deciders come in.

Deciders use the metadata in the MetaDB to parameterize a given workflow based 
on a set of rules encoded in the decider. At OICR we have deciders for each of 
our workflows and they let us process data on hourly cron jobs. For example, we 
have a decider for [STAR RNA-seq aligner](https://github.com/alexdobin/STAR) 
that looks in the MetaDB for all RNASeq human samples that have not previously 
been processed through a workflow, pulls back the needed data from the MetaDB, 
and schedules a workflow run for that particular lane. This is the heart of 
what a decider is trying to do. It links metadata to the actual execution of 
workflows in an automated way.

So we created the deciders for:

* automating workflows for specific workflows and projects
* a place for 'business logic': either site-specific (e.g. the location of reference data) or frequently changing code (e.g. configuration for specific projects or data types)

We try to make workflows very clean and generic with little business logic. 
Workflows should rely on INI parameterization for behaviour.

* TOC
{:toc}

## To-Be-Deprecated Note

At OICR, we are starting to move away from per-workflow Java deciders run by 
cron jobs onto a centralised decision-driven action launching system called 
[Shesmu](https://github.com/oicr-gsi/shesmu). Deciders are finicky to write 
and the messier your metadata is, the more custom work you do, and the more 
workflows that exist, the finickier they get to write and maintain. (But they 
work great once they're done!) Nevertheless, we will continue to support our 
existing Java deciders while transitioning to Shesmu and so this documentation 
is provided as a convenience.

## Decider API

When creating your own deciders you will extend 
[BasicDecider](https://github.com/oicr-gsi/niassa/blob/master/seqware-pipeline/src/main/java/net/sourceforge/seqware/pipeline/deciders/BasicDecider.java): 
the decider from which most other deciders came.

If you're working at OICR, we have a child of BasicDecider called 
[OicrDecider](https://github.com/oicr-gsi/pipedev/blob/master/pipedev-decider-utils/src/main/java/ca/on/oicr/pde/deciders/OicrDecider.java). Almost all of our Java deciders extend this class.

OicrDecider implements simplifies BasicDecider's logic as well as providing OICR-specific functionality. OicrDecider and accessory classes have a number of functions that depend on metadata originating from the Geospiza and [MISO LIMS](https://github.com/TGAC/miso-lims) as well as other  conventions defined in-house. You no longer need detailed knowledge about the formatting of Geospiza attributes in order to implement a decider, for example.

## Running Deciders

Deciders are Java JAR files, and can be executed like other jars:

```
java -jar Decider.jar [--arguments]
```

Required parameters of note:

* wf-accession: The SWID for the workflow that you wish to run
* study-name OR sample-name OR all OR sequencer-run-name: Limit the subset of samples to look for files associated with particular study, sample, sequencer-run, or just run over everything.
* parent-wf-accessions OR meta-types: Limit the files under consideration to those of specific file types (such as (gz) files or those generated by particular parent workflows. 

Optional parameters of note:

* check-wf-accessions: Comma-separated accessions of workflows that you may already be satisfied with (for example, satisfactory runs of an older version of a workflow). If the input files in a previous workflow run in this list match the current workflow run under consideration, it will not be re-run.  
* rerun-max: The maximum number of times to re-launch a workflowrun if failed. Default: 5.
* group-by: Group by one of the headings in FindAllTheFilesHeader. Default: FILE_SWA.

When extending the BasicDecider, you may also place the parent-wf-accessions, check-wf-accessions, and wf-accession into a decider.properties file to permanently set these values. 

### The Big List of Decider Arguments

| Command-line option | Description |
|--------------------|--------------|
|\--[arguments]|Override ini options on the command after the separator "\--" with pairs of "\--<key> <value>"|
|\--all|Operate across everything. At least one of [lane-SWID, ius-SWID, study-name, sample-name, root-sample-name, sequencer-run-name, organism, workflow-run-status, processing-status, processing-SWID]  or all is required. |
|\--cf, \--check-file-exists|Optional: only launch on the file if the file exists|
|\--check-wf-accessions|The comma-separated, no spaces, workflow accessions of the workflow that perform the same function (e.g. older versions). Any files that have been processed with these workflows will be skipped.|
|\--force-run-all|Forces the decider to run all matches regardless of whether they've been run before or not|
|\--group-by|Optional: Group by one of the headings in FindAllTheFiles. Default: FILE_SWA. One of LANE_SWA or IUS_SWA.|
|\--ho, \--host|Used only in combination with \--schedule to schedule onto a specific host. If not provided, the default is the local host|
|\--ignore-skip-flag|Ignores any 'skip' flags on lanes, IUSes, sequencer runs, samples, etc. Use caution.|
|\--ius-SWID|ius-SWID. At least one of [lane-SWID, ius-SWID, study-name, sample-name, root-sample-name, sequencer-run-name, organism, workflow-run-status, processing-status, processing-SWID]  or all is required. Specify multiple names by repeating \--ius-SWID|
|\--lane-SWID|lane-SWID. At least one of [lane-SWID, ius-SWID, study-name, sample-name, root-sample-name, sequencer-run-name, organism, workflow-run-status, processing-status, processing-SWID]  or all is required. Specify multiple names by repeating \--lane-SWID|
|\--launch-max|The maximum number of jobs to launch at once.|
|\--meta-types|The comma-separated meta-type(s) of the files to run this workflow with. Alternatively, use parent-wf-accessions.|
|\--no-meta-db, \--no-metadata|Optional: a flag that prevents metadata writeback (which is done by default) by the Decider and that is subsequently passed to the called workflow which can use it to determine if they should write metadata at runtime on the cluster.|
|\--organism|organism. At least one of [lane-SWID, ius-SWID, study-name, sample-name, root-sample-name, sequencer-run-name, organism, workflow-run-status, processing-status, processing-SWID]  or all is required. Specify multiple names by repeating \--organism|
|\--parent-wf-accessions|The workflow accessions of the parent workflows, comma-separated with no spaces. May also specify the meta-type.|
|\--processing-SWID|processing-SWID. At least one of [lane-SWID, ius-SWID, study-name, sample-name, root-sample-name, sequencer-run-name, organism, workflow-run-status, processing-status, processing-SWID]  or all is required. Specify multiple names by repeating \--processing-SWID|
|\--processing-status|processing-status. At least one of [lane-SWID, ius-SWID, study-name, sample-name, root-sample-name, sequencer-run-name, organism, workflow-run-status, processing-status, processing-SWID]  or all is required. Specify multiple names by repeating \--processing-status|
|\--rerun-max|The maximum number of times to re-launch a workflowrun if failed.|
|\--root-sample-name|root-sample-name. At least one of [lane-SWID, ius-SWID, study-name, sample-name, root-sample-name, sequencer-run-name, organism, workflow-run-status, processing-status, processing-SWID]  or all is required. Specify multiple names by repeating \--root-sample-name|
|\--sample-name|sample-name. At least one of [lane-SWID, ius-SWID, study-name, sample-name, root-sample-name, sequencer-run-name, organism, workflow-run-status, processing-status, processing-SWID]  or all is required. Specify multiple names by repeating \--sample-name|
|\--sequencer-run-name|sequencer-run-name. At least one of [lane-SWID, ius-SWID, study-name, sample-name, root-sample-name, sequencer-run-name, organism, workflow-run-status, processing-status, processing-SWID]  or all is required. Specify multiple names by repeating \--sequencer-run-name|
|\--study-name|study-name. At least one of [lane-SWID, ius-SWID, study-name, sample-name, root-sample-name, sequencer-run-name, organism, workflow-run-status, processing-status, processing-SWID]  or all is required. Specify multiple names by repeating \--study-name|
|\--test|Testing mode. Prints the INI files to standard out and does not submit the workflow.|
|\--wf-accession|The workflow accession of the workflow|
|\--workflow-run-status|workflow-run-status. At least one of [lane-SWID, ius-SWID, study-name, sample-name, root-sample-name, sequencer-run-name, organism, workflow-run-status, processing-status, processing-SWID]  or all is required. Specify multiple names by repeating \--workflow-run-status|


### Additional Arguments for OicrDecider

* \--check-file-exists : Only launch on the file if the file exists. By
default, SeqWare will launch on any file path, because it can take advantage
of remote files in S3 or ftp. At OICR, it is unlikely that we will use these
remote paths (the cluster is not internet accessible anyway) and we
occasionally have files that are listed in the metadata database that do not
exist.
* \--skip-status-check : If enabled, will skip the check for the status of
the workflow run. By default, we do not include files from failed workflow
runs. This flag turns that checking off. This flag has no effect on sequencer
runs, lanes, IUSes or any downstream products marked with a 'skip' flag.


# Writing a Decider

## Decision Logic

The decider performs the following steps:

1. Pull down the list of relevant file paths from study down to file (based on a starting point of a specific study, sample, sequencer-run or across all studies)
2. Group file paths based on the grouping strategy determined by "\--group-by". (`separateFiles` and `handleGroupByAttribute`)
3. Files can be omitted from groups in custom deciders by overriding the `checkFileDetails` method;
4. Entire groups can be omitted using the `doFinalCheck` method
5. For each grouping, determine whether there exists a blocking workflow run in the metadb based on the following [decision table](#decision-table). 
6. Create the ini file. This behaviour can be overriden by overriding `modifyIniFile` or `customizeRun`

Each of these steps are detailed below.

## Decider project structure

A decider is a very small Java Maven project. It has one Java file and two resources files as well as one pom.xml file. The relatively deep file structure is required for Maven builds.

```
.
|-- pom.xml
`-- src
    `-- main
	|-- java
	|   `-- ca
	|       `-- on
	|           `-- oicr
	|               `-- seqware
	|                   `-- deciders
	|                       `-- HelloWorldDecider.java
	`-- resources
	    `-- decider.properties
```

The HelloWorldDecider.java is a Java class that extends BasicDecider. Most of the specific decider functionality is built into HelloWorldDecider.java in a series of methods that are extended from BasicDecider. The two resources files are properties files that change more often than the Java code. These are updated when:

* Migrating to a new database install
* A new version of a workflow is released

### Resources folder

There is one file in src/main/resources: decider.properties. 

### Decider.properties file

The decider.properties file contains a number of properties that will be set by a user once and rarely updated. The intention is that the production team update these as new workflows are installed into their database.

	parent-workflow-accessions=
	check-wf-accessions=
	workflow-accession=

The parent-workflow-accessions are the workflows from which the files of interest come from.

The workflow-accession is the accession number of the current workflow from the database install. This is the WORKFLOW_ACCESSION that was printed after successfully installing to the database.

## Methods available to override

The following is a brief guide to the methods available when writing a decider. These methods are called in the order presented below. Note that
not all methods need to be overridden in order to make a functioning decider;
the usual ones are **bolded**.

1. **Constructor** : Add all new arguments to the decider in this method.
2. [**`init()`**](#init) : the first method where the command-line arguments are
  available to store, parse, test and store. Return a ReturnValue with
  a non success exit status if an argument is incorrect.
3. `separateFiles(List, String)` : override this method to modify the
  list of files (i.e. ReturnValues) before they are split into different
  groups.
4. [`handleGroupByAttribute(String)`](#handleGroupByAttribute) : override this method to change
  how files are grouped together. The String passed into the method is the
  unmodified group identifier (the value retrieved from a file with the group
  set by `setGroupBy(Group, boolean)`. You can use this method to modify
  it as desired. Files with the identical String returned from this method will
  be grouped and run together in one workflow run.
5. [**`checkFileDetails`**](#checkFileDetails): Override one of these methods to filter files based 
  on their metadata information.
  * The BasicDecider `checkFileDetails(ReturnValue, FileMetadata)`
    method uses ReturnValue and FileMetadata. ReturnValue
    has a list of attributes that contain the metadata for the file in 
    question.
    These attributes can be retrieved using `FindAllTheFiles.Header`
    objects. A `ReturnValue` represents one processing event in a previous
    workflow run and thereby can represent more than one file. The accompanying
    `FileMetadata` is the specific file that the method is evaluating at
    that time. Lims-specific attributes are somewhat difficult to retrieve with
    this system.
  * The OicrDecider `checkFileDetails(FileAttributes)` method uses
    FileAttributes instead, which simplifies the mechanism to retrieve
    commonly used metadata (e.g. donor, library sample name, sequencer run 
      name) and Geospiza LIMS-specific attributes. See FileAttributes for more
    information.
6. `doFinalCheck(String,String)` : parse a group of files that will
  be run together and do a final check to confirm that they should launch.
7. * **Customize each workflow run** : Override one of the following methods to
  appropriately parameterize each workflow run using the files involved.
  * The BasicDecider [`modifyIniFile(String, String)`](#modifyIniFile) has you return a
    Map which represents the INI file for the workflow.
  * The OicrDecider `customizeRun(WorkflowRun)` hides the process of
    creating an INI file and instead allows you to configure a
    WorkflowRun object. The intention of this method was to a) provide
    easy access to file metadata and b) simplify the creation of a one-step
    workflow by providing convenience methods to specify the script, memory, 
    and params. New properties for the INI are added to the WorkflowRun.

    
See each method for more details about implementation. 

All overridden methods need to call `super.method()` for deciders to work properly.

<a name="init"/>
### Init method

The init method sets up any variables needed for the rest of the setup, including all of those command-line options mentioned earlier. Specifying either the meta-type or the parent-accession is required for the decider to work. Here we specify that the decider only runs on files of type "application/bam". More than one can be specified here. For a full list of possible meta-types, please see [Workflow Conventions]({{version_url}}/workflows/conventions).

```java
@Override
public ReturnValue init() {
        Log.debug("INIT");
        //By default, the decider runs on a per-file basis (one file -> one workflow run)
        //this.setHeader(Header.IUS_SWA);

        //Launches only on BAM files
        this.setMetaType(Arrays.asList("application/bam"));

        //By calling super.init() last, anything defined on the command line at run-time 
        //overrides the defaults set here.
        ReturnValue val = super.init();
        return val;
}
```

<a name="checkFileDetails"/>
### checkFileDetails method

Performs any additional checks on the file before adding it to the list of files to incorporate. This method should be extended for future deciders for custom behavior. You can also pull any details out of the file metadata here. The boolean specifies whether or not the file is added to the list.

```java
protected boolean checkFileDetails(ReturnValue returnValue, FileMetadata fm) {
        Log.debug("CHECK FILE DETAILS:" + fm);
        boolean isValid = super.checkFileDetails(returnValue, fm);
        if (isValid) {
            //If you need access to the information stored in the ReturnValue for each file, uncomment
            //the next line and the line that instantiates the pathToAttributes map before the constructor
            //pathToAttributes.put(fm.getFilePath(), returnValue);
        }
	return isValid;
}
```
<a name="modifyIniFile"/>
### modifyIniFile method

Change the INI file from its defaults as specified in the workflow. The commaSeparatedFilePaths are the files that are going to be grouped together for one particular run of the decider, and the commaSeparatedParentAccessions are the accessions of the processing events that produced the file.

In this particular case, we want to substitute the INI field 'input_file' to be the comma-separated list of files. The rest of the INI file is auto-generated using the decider's INI file and the default INI file from the database.

```java
protected Map<String, String> modifyIniFile(String commaSeparatedFilePaths, String commaSeparatedParentAccessions) {
      Log.debug("INI FILE:" + commaSeparatedFilePaths);
      Map<String, String> iniFileMap = super.modifyIniFile(commaSeparatedFilePaths, commaSeparatedParentAccessions);

    	iniFileMap.put("input_file", commaSeparatedFilePaths);

return iniFileMap;
}
```

Ini attributes are applied in the following order:, where the greater than sign means 'takes precedence over': modifyIniFile method &gt; command line options &gt; Decider INI file &gt;  Default Database INI file

<a name="handleGroupByAttribute"/>
### handleGroupByAttribute method

If the contents of the group-by attribute needs to be modified at all, this is where you should do it.

For example, say that you wanted to group files based on tissue type. This information is captured in the SAMPLE_ATTRIBUTE column. However, the contents of the SAMPLE_ATTRIBUTE column include a variety of information, not all of which is relevant. Here is an example:

```
	SAMPLE_ATTRIBUTE = sample.geo_reaction_id=6223;sample.geo_template_id=17937;sample.geo_library_source_template_type=EX;sample.geo_targeted_resequencing=Agilent SureSelect ICGC/Sanger Exon;sample.geo_tissue_type=n;sample.geo_tissue_origin=Br;
```

In the handleGroupByAttribute, you would use a regular expression to pull out the information "sample.geo_tissue_type=n", which is used to put the file into a bucket and aggregate it with other files.

```java
protected String handleGroupByAttribute(String attribute) {
  return attribute;
}
```
<a name="decision-table"/>
## Decision Table

A general guideline here is that we only count failures when they are associated with workflow runs that ran on the same files (in the metadb) as we are currently attempting to run on. 

| Status of previous workflow run | 1 (Input files are disjoint from previous run)  | 2 (Input files partially overlap a previous run) | 3 (Input files match a previous run exactly) | 4 (Past run contains all current input files) |
|--------------------|--------------|---------------|----------|--------|
|Failed| ignore (allow rerun) | allow re-run | count as failure, re-run if less than max-failures | allow re-run (with warning message)|
|Other (pending, running, no status, etc.) | ignore (allow re-run) | allow re-run | block re-run | block re-run |
|Completed| ignore (allow rerun) | allow re-run | block re-run | do not re-run |

In addition, if for a group of files, we also count up more failures than rerun-max, then we also block a re-run. 

### Search for Relevant Workflow Runs 

When looking for previous workflow runs, we look in a table in the MetaDB, 
`workflow_run_input_files`. This table specifically stores the input files and 
is populated at schedule time by the workflow launcher.

