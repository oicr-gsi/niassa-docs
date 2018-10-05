---
layout: page
title: Pipeline Installation Guide
---
{% include functions.liquid %}

* toc
{:toc}

[Niassa Pipeline]({{version_url}}/pipeline) includes several workflow engines -
systems that run workflows
on different system architectures. In this document, we focus on one particular
workflow engine: SGE-Oozie. This engine uses
[Apache Oozie](http://oozie.apache.org/) as a job manager to run jobs on a
Sun Grid Engine-derived cluster (we use
[Open Grid Scheduler](http://gridscheduler.sourceforge.net/)).

# Requirements

The Oozie-SGE Niassa workflow engine requires:
{% assign oozie_version="4.3.0" %}
{% assign psql_version="9.3.16+" %}
* Java 8
* Apache Oozie {{oozie_version}}
* PostgreSQL {{psql_version}}
* An [installed and configured Web service]({{version_url}}/installation/web-service)
* An [installed and configured MetaDB]({{version_url}}/installation/metadb)

These instructions have been tested on Debian 8.

# Download

For stable releases, the Jar is available for download from Github:
[seqware-distribution-{{version}}-full.jar](https://github.com/oicr-gsi/niassa/releases/download/v2.0.2/seqware-distribution-{{version}}-full.jar)

# Building from source

If you want to build an unstable or development version, see the
[Niassa Build Guide](https://oicr-gsi.github.io/niassa-docs/current/installation/building)
for detailed build instructions. After the build completes, the Jar file
will be in
`seqware-distribution/target/seqware-distribution-{{version}}-full.war`

# Installation

This installation guide is used at OICR in order to install and configure
Oozie-SGE workflow engine for production.

**Note**: these instructions are very rough guidelines. If you have any
questions, contact us through our
[Github issue tracker](https://github.com/oicr-gsi/niassa/issues).

## Install Oozie

We set up a single Oozie node (not HA) and install the Oozie database on the
same host as the Niassa [MetaDB]({{version_url}}/installation/metadb).

Please consult the
[Oozie documentation](http://oozie.apache.org/docs/{{oozie_version}}/index.html) for
installation instructions. We use Oozie version {{oozie_version}}.

### Configuring Oozie

In our setup, Oozie and [Niassa MetaDB]({{version_url}}/metadb) use the same
PostgreSQL {{psql_version}} database. The database is configured for max 100
connections, and so reducing the number of connections prevents connection
refused errors.

As the root user, modify `/etc/oozie/conf.dist/oozie-site.xml`. Change from:

```xml
<property>
    <name>oozie.service.JPAService.pool.max.active.conn</name>
    <value>100</value>
    <description>
         Max number of connections.
    </description>
</property>
```
To:

```xml
<property>
    <name>oozie.service.JPAService.pool.max.active.conn</name>
    <value>50</value>
    <description>
         Max number of connections.
    </description>
</property>
```

Now restart Oozie to apply changes:
```
sudo /etc/init.d/oozie restart
```

## Install/configure SGE

You will need to make sure that the node that runs Oozie also has all SGE tools
(like qsub, qstat, and qacct) hooked up to a real SGE environment. Additionally,
you will want to make sure that Niassa's shared directories (released bundles,
provisioned bundles, and oozie working directory) are available to the execution
nodes.

## Install Oozie-SGE plugin

See the [oozie-sge](https://github.com/SeqWare/oozie-sge) project.

# Good luck!

These instructions are very rough guidelines. If you have any
questions, contact us through our
[Github issue tracker](https://github.com/oicr-gsi/niassa/issues).
