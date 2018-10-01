---
layout: page
title: Client Installation
---
{% include functions.liquid %}

If you are within an environment that already has Niassa configured and set up
(hi OICR!), you will only need to configure your environment to interact with 
the system. As long as you have access to the Niassa webservice host and the 
appropriate permissions, you can write new workflows, install those workflows on
the remote Niassa system, schedule workflows remotely, etc.

The following section assumes tests are being run on the machine 
`nsdev-user.hpc.oicr.on.ca`, under your own user account (as opposed to a shared
account such as `sqwdev`). It is possible to run on other machines with access 
to the Niassa architecture, but this may require additional configuration, such 
as setting `JAVA_HOME` to an appropriate Java 8 installation path.

* TOC
{:toc}

# Disclaimer: Don't Panic
In order to build and launch workflows on Niassa, a number of elements must 
work together correctly: software versions, network hosts, configuration files, 
etc.

It is possible that changes to the pipeline or cluster, or a variation in user 
environment, will cause a step in this tutorial to fail. If that happens, seek 
help--and 
[update this page](https://github.com/oicr-gsi/niassa-docs/edit/master/{{version}}/installation/0-client-only.md) 
if needed.

This tutorial includes a few workarounds and ad hoc modifications. Future 
updates may make the workarounds unnecessary; or more elegant alternatives to 
the workarounds may exist. Again, this page can be updated as appropriate.

# What you need to get started

## A cooperative Niassa administrator

Make sure your local administrator follows the guide on 
[adding new users]({{version_url}}/admins/new-user).

## Java 8

Niassa uses Java 8. If on a machine where you can directly upgrade Java, use 
your favourite package manager (apt-get or yum) to download Java 1.8 and update 
your `java` bin to point to the new location.

If installing on your own: Download 1.8 JRE from Oracle or OpenJDK. Unzip it in 
a stable location, like your home directory. Set `JAVA_HOME` to be the root 
directory, and `JAVA_CMD` to be the direct location of the `java` executable.


# CLI setup

All users will need to have the `seqware` CLI client installed and configured.

## Download and configure the `seqware` client

All you need to get started is the Niassa 'bin' script. 

{% capture release_url %}https://github.com/oicr-gsi/niassa/releases/download/{{ version }}/seqware{% endcapture %}

1. Get the most recent version here: [https://github.com/oicr-gsi/niassa/releases](https://github.com/oicr-gsi/niassa/releases). 
  * In your browser: [{{release_url}}]({{release_url}}); 
  * In your terminal:
    ```bash
curl -L {{release_url}} > seqware
    ```
2. Put the script into your `~/bin` directory. Check to make sure `~/bin` is in your path.
  ```bash
$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games:/oicr/cluster/bin:/oicr/cluster/sbin:/oicr/cluster/openmpi/bin:/oicr/local/bin:/oicr/local/sbin:/oicr/local/analysis/bin
  ```
3. Make sure the binary is executable.
  ```bash
chmod +x ~/bin/seqware
  ```
4. Run the script to set up your environment
  ```bash
$ seqware
Downloading SeqWare to /home/mtaschuk/.seqware/self-installs/seqware-distribution-{{version}}-full.jar now...
...
Downloading SeqWare Check to /home/mtaschuk/.seqware/self-installs/seqware-sanity-check-{{version}}-jar-paired-with-distribution.jar now...
...
  ```
  
## User Configuration : `settings` file

Running the `seqware` command for the first time will create a hidden `.seqware` 
directory in your home directory and populate it with the seqware-distribution 
and seqware-sanity-check JAR files as well as a `settings` file. The `settings`
file is where you will configure your access to Niassa: servers, databases, 
web services, usernames, passwords, etc. Right now the file is full of dummy
values to localhost.

Your local Niassa administrator will provide you with a `settings` file with 
most of these values pre-configured. You may have to modify the following values
for your specific user, and these values will also be given to you by an admin.

* `SW_REST_USER` and `SW_REST_PASS`: a Niassa web service username and password
* `OOZIE_APP_PATH`: the complete URI to your user's HDFS directory.
* `OOZIE_WORK_DIR`: the working directory for Niassa pipeline. It must exist and
  be writeable.


For more information on the `settings` file, see 
[User Configuration]({{version_url}}/environment/user-configuration).

## Check to see if the CLI works

Having the CLI set up properly means that you will be able to query from and 
create metadata in the [MetaDB]({{version_url}}/metadb), but not deal with 
workflows. Here's a query to test the system is set up properly.

```
seqware --metadata
```

An example result is below. If you get a connection exception, check your 
`settings` file is correct.

```
{
  "database.DateStyle": "ISO, MDY",
  "database.IntervalStyle": "postgres",
  "database.TimeZone": "UTC",
  "database.allow_system_table_mods": "

...

  "metadata": "webservice",
  "tomcat.built": "Jan 11 2016 11:40:31",
  "tomcat.version": "7.0.28.0",
  "version": "1.1.1-gsi"
}
```

One more command:

```
seqware workflow list
```

```
-[ RECORD 0 ]------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------
Name                     | BamFilterMergeCollapse
Version                  | 1.3
Creation Date            | Fri Jun 23 14:08:11 EDT 2017
SeqWare Accession        | 9797849

...

Description              | The Xenome one step workflow.
Current Working Directory| /u/sqwprod/provisioned-bundles/Workflow_Bundle_Xenome_1.1_SeqWare_0.13.6.9
Bundle Location          | /u/sqwprod/released-bundles/Workflow_Bundle_Xenome_1.1_SeqWare_0.13.6.9.zip
```

# Running, scheduling, and monitoring workflows

The following section will be needed for most users who are running and 
scheduling Niassa workflows. Note that you will need to follow the CLI setup 
instructions first for these steps to work.

## Scratch and working directories

Most workflows will need quite a lot of space to deposit intermediate files. We 
recommend having a dedicated 'scratch' area with at least a couple hundred 
gigabytes free. After workflows complete successfully, this area can be cleared.

Configure this working directory using the `.seqware/settings` file under the
`OOZIE_WORK_DIR` property. It must exist, be writeable, and have sufficient 
space.

## Access to Oozie 

If you are using an Oozie-based workflow engine (your Niassa administrator will 
know this and it will be in your supplied `settings` file under 
`SW_DEFAULT_WORKFLOW_ENGINE`), we find user access to Oozie to be useful for 
debugging workflows.

Most of these settings are environment variables. You can either set them in 
your profile (e.g. `~/.bash_profile`) or in a file that you source when you
want to work with Oozie.

1. Add the required Hadoop dependencies (hadoop and hadoop-hdfs) to the 
  environment variable `$CLASSPATH`.
```bash
export CLASSPATH="/.mounts/labs/PDE/public/bin/hadoop/hadoop/*:/.mounts/labs/PDE/public/bin/hadoop/hadoop/lib/*:/.mounts/labs/PDE/public/bin/hadoop/hadoop-hdfs/*:/.mounts/labs/PDE/public/bin/hadoop/hadoop-hdfs/lib/*:$CLASSPATH"
```
2. Set your PATH to include the Oozie executables.
```bash
export PATH="/.mounts/labs/PDE/public/bin/oozie/bin:$PATH"
```
3. Set more environment variables to specify what Oozie instance to use. Note
  that the timezone should correspond to the local 
  time for your Niassa installation.
```bash
export OOZIE_URL="http://nsdev-node.hpc.oicr.on.ca:11000/oozie"
export OOZIE_TIMEZONE="EST"
```
4. Test Oozie client and Oozie server connectivity and that your client is 
  configured properly.
```bash
oozie admin -version
Oozie server build version: 3.3.2-cdh4.5.0
```

## Workaround for Niassa's locking system

Niassa's locking system is slightly overzealous, so we have a workaround 
until we fix it: we use an in-memory location for its lock files.

```bash
mkdir -p /var/tmp/${USER}/junique
ln -s /var/tmp/${USER}/junique ${HOME}/.junique
```




# Developing new workflows

This final section is only for workflow developers. If you plan to create your
own workflows or modify existing workflows, you will need to follow the steps in 
this section.

## Setting up Maven

We use Maven as our build, testing and dependency management system for 
workflows.  

Environment variables to set for Maven:

* `M2_HOME` : dir to Apache Maven 3.1.1+ 
* `JAVA_HOME` : Java JDK 1.8.0_91 +
* `PATH="${JAVA_HOME}/bin:${M2_HOME}/bin:$PATH"`

The steps below involve copying Maven config files to your ~/.m2 directory. 
This will of course replace any existing config files. If you wish to preserve 
an existing Maven configuration, the files will need to be merged manually.


### settings.xml

The [gsi-config](https://github.com/oicr-gsi/gsi-config) Github repository 
contains the definitive settings.xml file, as well as a README with more 
details.

1. Download: https://raw.githubusercontent.com/oicr-gsi/gsi-config/master/settings.xml 
2. Copy to: ~/.m2/settings.xml

### archetype-catalog.xml

This file is needed to generate workflows using 
[archetypes]({{version_url}}/getting-started-developer-tutorial#creating-a-new-workflow).

1. Download: https://raw.githubusercontent.com/oicr-gsi/niassa/develop/seqware-pipeline/bin/archetype-catalog.xml 
2. Copy to: ~/.m2/archetype-catalog.xml
3. Manually correct artifactory URL in  ~/.m2/archetype-catalog.xml. We need to 
  replace the (obsolete) wrench.res.oicr.on.ca with a working server such as 
  wrench2.oicr.on.ca.
  ```bash
mv ~/.m2/archetype-catalog.xml ~/.m2/archetype-catalog.xml.tmp
sed s_wrench.res.oicr.on.ca/artifactory/seqware-release_wrench2.oicr.on.ca/artifactory/remote-repos_g < ~/.m2/archetype-catalog.xml.tmp > ~/.m2/archetype-catalog.xml
rm ~/.m2/archetype-catalog.xml.tmp
  ```
  
### Check to see if Maven will build

In this example, we clone one of OICR's public workflows, 
[bam-qc](https://github.com/oicr-gsi/bam-qc), and make sure it builds. You will
need to have git installed. 

```bash
mkdir code
cd code
git clone git@github.com:oicr-gsi/bam-qc.git
cd bam-qc/workflow-bam-qc/
```

Build the workflow.  This command cleans, downloads dependencies, compiles the 
workflow java code, and packages the project into a seqware "bundle" directory - 
but does not run long running integration tests

```bash
mvn clean install
```

And the successful result:

```
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building workflow_BamQC 2.5
[INFO] ------------------------------------------------------------------------

...

[INFO] --- maven-install-plugin:2.4:install (default-install) @ workflow-bam-qc ---
[INFO] Installing /u/mlaszloffy/code/bam-qc/workflow-bam-qc/target/workflow-bam-qc-2.5.jar to /u/mlaszloffy/.m2/repository/ca/on/oicr/pde/workflows/workflow-bam-qc/2.5/workflow-bam-qc-2.5.jar
[INFO] Installing /u/mlaszloffy/code/bam-qc/workflow-bam-qc/pom.xml to /u/mlaszloffy/.m2/repository/ca/on/oicr/pde/workflows/workflow-bam-qc/2.5/workflow-bam-qc-2.5.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 2:14.704s
[INFO] Finished at: Fri Sep 01 14:33:52 EDT 2017
[INFO] Final Memory: 36M/491M
[INFO] ------------------------------------------------------------------------
```

## Continuous integration for workflows

If you are building a workflow written at OICR, it will have built-in 
integration tests. Note: these probably won't work outside OICR, because they
have local dependencies like the human genome reference. But if you _are_ at 
OICR, here's what you need to do.

1. Set yet more environment variables. Note that `WORKING_DIR` must exist and be 
  writeable.
  ```
WEBSERVICE="http://nsdev-www.hpc.oicr.on.ca:8080/seqware-webservice"
WEB_USER="admin@admin.com"
WEB_PASSWORD="$(cat /.mounts/labs/PDE/secrets/dev_webservice_password)"
OOZIE_HOST="nsdev-node.hpc.oicr.on.ca"
DB_HOST="sqwci-docker.gsi.oicr.on.ca"
DB_PORT="54323"
DB_USER="postgres"
DB_PASSWORD="$(cat /.mounts/labs/PDE/secrets/dev_sqwci_docker_password)"
WORKING_DIR="/scratch2/users/${USER}/seqware_tmp/"
PROVENANCE_SETTINGS="/.mounts/labs/PDE/secrets/dev_provenance_settings.json"
export MAVEN_OPTS="-Xmx8g -XX:+UseG1GC -XX:+UseStringDeduplication"
  ```
2. Check that it's working properly. In the workflow you previously checked out 
above, run the integration tests:
  ```
mvn clean install \
  -DskipITs=false \
  -DskipRunITs=false \
  -DwebserviceUrl="${WEBSERVICE}" \
  -DwebserviceUser="${WEB_USER}" \
  -DwebservicePassword="${WEB_PASSWORD}" \
  -DdbHost="${DB_HOST}" \
  -DdbPort="${DB_PORT}" \
  -DdbUser="${DB_USER}" \
  -DschedulingHost="${OOZIE_HOST}" \
  -DworkingDirectory="${WORKING_DIR}" \
  -DrunTestThreads=2 \
  -Djava_opts="-Xmx2g -XX:+UseG1GC -XX:+UseStringDeduplication" \
  -DprovenanceSettingsPath="${PROVENANCE_SETTINGS}"
  ```
  The result:
  ```
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building workflow_BamQC 2.5
[INFO] ------------------------------------------------------------------------
...

    Test case = [WorkflowRunTest_9_BamQC-2.5]
      group = [preExecution], method = [installWorkflow], execution time = [20.62s], status = [SUCCESS]
      group = [preExecution], method = [initializeEnvironment], execution time = [0.66s], status = [SUCCESS]
      group = [preExecution], method = [scheduleWorkflow], execution time = [41.14s], status = [SUCCESS]
      group = [execution], method = [executeWorkflow], execution time = [595.40s], status = [SUCCESS]
      group = [postExecution], method = [checkWorkflowOutputExists], execution time = [0.00s], status = [SUCCESS]
      group = [postExecution], method = [calculateOutputMetrics], execution time = [1.08s], status = [SUCCESS]
      group = [], method = [compareOutputToExpected], execution time = [0.04s], status = [SUCCESS]
================================================================================
Tests run: 77, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] --- exec-maven-plugin:1.3.2:exec (test_bundle) @ workflow-bam-qc ---
[INFO] skipping execute as per configuraion
[INFO]
[INFO] --- maven-failsafe-plugin:2.18.1:verify (verify) @ workflow-bam-qc ---
[INFO] Failsafe report directory: /u/mlaszloffy/code/oicr-gsi/bam-qc/workflow-bam-qc/target/failsafe-reports
[INFO]
[INFO] --- maven-install-plugin:2.4:install (default-install) @ workflow-bam-qc ---
[INFO] Installing /u/mlaszloffy/code/oicr-gsi/bam-qc/workflow-bam-qc/target/workflow-bam-qc-2.5.jar to /u/mlaszloffy/.m2/repository/ca/on/oicr/pde/workflows/workflow-bam-qc/2.5/workflow-bam-qc-2.5.jar
[INFO] Installing /u/mlaszloffy/code/oicr-gsi/bam-qc/workflow-bam-qc/pom.xml to /u/mlaszloffy/.m2/repository/ca/on/oicr/pde/workflows/workflow-bam-qc/2.5/workflow-bam-qc-2.5.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 12:35.673s
[INFO] Finished at: Fri Sep 01 15:56:04 EDT 2017
[INFO] Final Memory: 30M/491M
[INFO] ------------------------------------------------------------------------
```

## Installing this workflow

Now that the workflow has been built and tested, you can install the workflow bundle:

Make sure your `SW_BUNDLE_DIR` and `SW_BUNDLE_REPO_DIR` in your `settings` file 
exist and are accessible.

```
seqware bundle install-dir-only --dir target/Workflow_Bundle_BamQC_2.5_SeqWare_1.1.0/
```

```
Installing Bundle (Working Directory Only)
Bundle: target/Workflow_Bundle_BamQC_2.5_SeqWare_1.1.0/
Added 'BamQC' (SWID: 10378445)
Bundle Has Been Installed to the MetaDB and Provisioned to target/Workflow_Bundle_BamQC_2.5_SeqWare_1.1.0/!
```

Make note of the SWID (workflow accession) for use in the next section.


# Test that everything is set up properly

We're going to use a workflow that retrieved from `seqware workflow list`: 
BamQC. If you followed the instructions for 
[Developing new workflows](#developing-new-workflows), you can use the workflow 
accession (or SWID) that you got from installing that workflow. Otherwiser, ask
ask your friendly Niassa administrator for a convenient workflow to use for 
testing.

0. Find the workflow accession. In the example below, the workflow accession is 
  10378445.
  ```
$ seqware workflow list --tsv | grep BamQC
BamQC	2.5	Fri Sep 11 06:03:06 EDT 2015	10378445	The BAM QC Workflow takes a BAM file as input and applies the samStats.pl script to generate a JSON stats file. https://...	/.mounts/labs/seqprodbio/private/seqware/hsqwprod/provisioned-bundles/Workflow_Bundle_BamQC_2.5_SeqWare_1.1.0	/.mounts/labs/seqprodbio/private/seqware/hsqwprod/released-bundles/Workflow_Bundle_BamQC_2.5_SeqWare_1.1.0.zip
  ```
1. Customize ini. First get the workflow's default ini (note how the accession 
  from bundle install above step is being used):
  ```
seqware workflow ini --accession 10378445 --out "${HOME}/bam-qc-defaults.ini"
Created '/u/mlaszloffy/bam-qc-defaults.ini'.
  ``` 
2. Modify the ini with the properties that you want to run the workflow with. 
  For bam-qc workflow, add the following to the bam-qc-defaults.ini:
  ```
input_file=/.mounts/labs/PDE/data/RegressionTests/BamQC/workflow/development/input_data/pde-806/SWID_1237537_THB_0004_nn_n_PE_329_TS_140926_M00146_0007_000000000-A55RY_TGACCA_L001_R1_001.annotated.bam
target_bed=/.mounts/labs/PDE/data/RegressionTests/BamQC/workflow/development/input_data/pde-806/RainDance_Thunderbolts_Cancer_Panel_target_sorted.bed
  ```
3. Schedule a workflow run. Now that the workflow has been installed and the ini 
  is ready, a workflow run can be scheduled:
  ```
seqware workflow schedule --accession 10378445 --host "$(hostname)" --ini "${HOME}/bam-qc-defaults.ini"
Created workflow run with SWID: 10378446
  ```
4. Launch the workflow run. The following command launches the workflow run 
  (note that the accession is from the above "Schedule a workflow run" step):
  ```
seqware workflow-run launch-scheduled --accession 10378446
  ```

```
[2017/09/06 11:45:23] | Number of submitted workflows: 7
Working Run: 10378446
Valid run by host check: 10378446
Launching via new launcher: 10378446
Using working directory: /scratch2/users/mlaszloffy/work_dir/oozie-ed595189-7889-4fd7-83e6-5b6b318c7b61
[SeqWare Pipeline] WARN [2017/09/06 11:45:25] | Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Files copied to /scratch2/users/mlaszloffy/work_dir/oozie-ed595189-7889-4fd7-83e6-5b6b318c7b61
Submitted Oozie job: 0000119-170825094912765-oozie-oozi-W
...
Monitoring workflow run progress
Workflow run progress can be monitored using the  seqware client:

seqware workflow-run propagate-statuses --accession 10378446 && seqware workflow-run report --accession 10378446
[2017/09/06 11:49:30] | Propagated workflow engine statuses
-[ RECORD 0 ]------------------+-------------------------------------------------------------------------------
Workflow                       | BamQC 2.5
Workflow Run SWID              | 10378446
Workflow Run Status            | running
Workflow Run Create Timestamp  | 2017-09-06 15:42:41.678
Workflow Run Host              | sqwdev-user
Workflow Run Working Dir       | /scratch2/users/mlaszloffy/work_dir/oozie-ed595189-7889-4fd7-83e6-5b6b318c7b61
Workflow Run Engine ID         | 0000119-170825094912765-oozie-oozi-W
Library Sample Names           |
Library Sample SWIDs           |
Identity Sample Names          |
Identity Sample SWIDs          |
IUS-LimsKeys                   | []
Input File Meta-Types          |
Input File SWIDs               |
Input File Paths               |
Immediate Input File Meta-Types|
Immediate Input File SWIDs     |
Immediate Input File Paths     |
Output File Meta-Types         |
Output File SWIDs              |
Output File Paths              |
Workflow Run Time              | 3m 9.0s
```



# Success!

If you've gotten this far, you're ready for prime-time. Head on over to the 
[Getting Started]({{version_url}}/getting-started) guides if you are new to 
Niassa, or head over to the central [Documentation]({{version_url}}/index).