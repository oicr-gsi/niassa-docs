---
layout: page
title: Cron jobs for workflow launching and propagating
---
{% include functions.liquid %}

In the Niassa Pipeline system there are two ways to launch workflows. A user
can use the  `seqware` CLI to directly launch a workflow bundle, or a user can 
"schedule" a workflow through the web service. Another process (usually a cron) 
monitors the web service and launches workflows that have been scheduled. 

Decoupling workflow scheduling from the actual launch allows more flexibility in
where and by whom a workflow is launched. A user can schedule a 
workflow from anywhere they have access to the Niassa web service with only the 
[CLI]({{version_url}}/CLI) and an appropriate `.seqware/settings` file.

Here we detail the needed cron jobs running on the Niassa
Pipeline host (\*-user) that will query the webservice, 
launch scheduled workflows, and update their progress.

## Limitations

The coordination of workflow scheduling and launching happens via the Niassa 
[Web Service]({{version_url}}/web-service).  Each workflow that is
scheduled by a user is associated with that user's Web Service credentials. In
order to launch that scheduled workflow, the CLI must connect
to the Web Service with the same credentials, find the workflow, and then
launch it on the Niassa \*-node box. If workflows are scheduled by multiple
accounts, each account needs its own launcher and monitor cron job to
periodically launch and monitor workflows.

## Configuration

Show table showing user launching and host interactions.  In this example the
user "Bob" logs into server1 and connects to the Niassa Web Service running on
server2 using the login "bob@lab.net". He then schedules a workflow with
WorkflowLauncher and specifies the host to run this workflow on with `--host
server2`.  On server2 another user account, "niassa-bob" runs the daemons
documented below. They connect to the Web Service locally using the same
"bob@lab.net" username and find workflows that have been scheduled to run on
server2. These daemons launch the workflows on server2 and the resulting jobs
run on the cluster connected to server2 as user "niassa-bob", resulting in
files associated with this workflow being owned by the user "niassa-bob".

```
Host - server1         |               |         |Network |    Host - server2  |                      |
---------------------- | -------------- | ------- |--------| ------------------  | -------------------  |
User Scheduling |  Web Service Account  | Host param |   ->      | User Running  | Web Service Account      |
Bob             |  bob@lab.net          | server2    |   ->      | niassa-bob   |  bob@lab.net             |
```

## How to Launch

In our reference Niassa environment, we
typically schedule jobs and then launch them asynchronously via a cronjob. 

A user will schedule workflow runs using a command similar to below:

```
$ seqware workflow schedule --accession 1 --parent-accession 99  --ini workflow.ini --host "$(hostname --long)"
```
Then in a cronjob we use the following command to launch scheduled jobs: 

```
$ seqware workflow-run launch-scheduled --host "$(hostname --long)"
```

Note that in the first command, we specify that the workflow needs to be 
launched on a specific host. When we check for scheduled workflows, we check 
this value in order to determine whether a particular scheduled workflow should 
be launched on this host. While we normally use a fully qualified hostname,
any unique string can be used for launching (for example
on Amazon S3).

## How to Update Workflow Status

Since the engine that executes the workflow is separate from the Niassa MetaDB, 
a separate process is used to propagate statuses between the workflow engine and 
MetaDB:

```
$ seqware workflow-run propagate-statuses
```

Once this is executed, workflow-run reports will reflect the updated status.

## Cron Jobs

Create a cron job that performs both of these functions frequently on the 
\*-user host:

```
$ crontab -l
* * * * * /home/niassa/crons/status.cron >> /home/niassa/logs/status.log
```

This is the script running in the cron:

```
$ cat /home/niassa/crons/status.cron

#!/bin/bash

source /home/niassa/.bash_profile

seqware workflow-run launch-scheduled
seqware workflow-run propagate-statuses --threads 10
```
        
This script runs every minute and uses the first command to launch workflows 
that have been previously scheduled while the second command is used to check 
the status of launched workflows. Note that this command will only launch those
workflows that have been scheduled with the same `--hostname` as the machine
where the cron job is running.

