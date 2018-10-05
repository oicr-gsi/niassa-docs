---
layout: page
title: For Niassa Administrators
---
{% include functions.liquid %}

In this section we cover the information needed for Niassa administrators, who
are responsible for setting up and maintaining Niassa for other users. For this
reason, we assume that Niassa Pipeline is configured to use a full PostgreSQL
[MetaDB]({{version_url}}/metadb), one of the Oozie workflow engines (either
`oozie` or `oozie-sge`), and a Tomcat
[Web Service]({{version_url}}/web-service).

For reasons of load and configuration, we expect a 'production-like' environment
to have different servers hosting each Niassa component. This allows scheduling
and running workflows to be decoupled (different machines, different users,
different user roles, etc).

<img src="{{version_url}}/images/multihost_arch.png" alt="Niassa multi-host architecture"/>

In a typical production setup, there are four servers:

* \*-user : users and developers log on to in order to schedule
  and launch workflows. The \*-user node also hosts cron jobs that launch and
  update workflow status.
* \*-www : where the Tomcat [Web Service]({{version_url}}/web-service) is hosted
* \*-db : where the PostgreSQL [MetaDB]({{version_url}}/metadb) is hosted
* \*-node : where Oozie is installed and runs

# Installation

See [Installation]({{version_url}}/installation).

## Changing a user's password

Passwords are changed in the MetaDB `registration` table. You will need the
username and password for the Postgres MetaDB.

First, change the
password in the table to be the plain-text password of the user's choice.

```bash
#set new seqware webservice admin password
psql -h "${HOST}" -p "${PORT}" -d "${DB_NAME}" -U "${DB_USER}" -c "UPDATE registration SET password ='"${NEW_PASSWORD}"' WHERE email = '${USER_EMAIL}'"
```

Run the salting-encrypting utility
[RegistrationMigrationPlugin](https://github.com/oicr-gsi/niassa/blob/develop/seqware-pipeline/src/main/java/net/sourceforge/seqware/pipeline/plugins/RegistrationMigrationPlugin.java)
to encrypt the password. You will need to have a copy of the
seqware-distribution jar and set up your settings file according to the
[client-only](https://oicr-gsi.github.io/niassa-docs/current/installation/client-only)
installation instructions.

```bash
java -jar seqware-distribution-{{version}}-full.jar --plugin net.sourceforge.seqware.pipeline.plugins.RegistrationMigrationPlugin
```


# Backups

Niassa MetaDB should be regularly backed up. On our deployment, we have a cron
script which calls the Files Report and `pg_dump` nightly to do back-up.

Note the variables in the script below: `{BACKUP_DIR}` is the location of your
nightly backups; `${DBCATALOG}` is the name of the MetaDB; `${DBUSER}` is the
name of the PostgreSQL user with access to the MetaDB; `${DBHOST}` is the
\*-db server name where the MetaDB is hosted.

```bash
#!/bin/bash

OUT_DIR="${BACKUP_DIR}/database/$(date -u +%Y-%m)/"
mkdir -p "${OUT_DIR}"
cd "${OUT_DIR}"

DATE="$(date +%Y%m%d-%H:%M)"
PG_SIZE_FILE=/tmp/pg-db-size
pg_dump --encoding utf8 --no-privileges --no-owner "${DBCATALOG}" \
          -U "${DBUSER}" \
          -h "${DBHOST}" \
          -T file_provenance_report \
        | tee >(wc -c > ${PG_SIZE_FILE}) \
        | gzip > "$DATE.sql.gz"
rm -f ${PG_SIZE_FILE}
```

# Reference material

* [Getting Started: Admins]({{version_url}}/getting-started-admin-tutorial)
* [Adding new users]({{version_url}}/admins/new-user)
* [Installing Workflows]({{version_url}}/getting-started-admin-tutorial#how-to-install-a-workflow)
* [Cron jobs for workflow launching and propagating]({{version_url}}/admins/launching-propagating)
