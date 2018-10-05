---
layout: page
title: MetaDB Installation Guide
---
{% include functions.liquid %}

* toc
{:toc}

# Requirements

The [Niassa MetaDB]({{version_url}}/metadb) requires:
{% assign psql_version="9.3.16+" %}
* PostgreSQL {{psql_version}}

These instructions have been tested on Debian 8.

# Download stable releases

For stable releases, the PostgreSQL MetaDB schema, controlled vocabulary and 
test data can be downloaded from Github releases:

* [seqware_meta_db.sql](https://github.com/oicr-gsi/niassa/releases/download/v{{version}}/seqware_meta_db.sql) : 
	the MetaDB schema without data
* [seqware_meta_db_data.sql](https://github.com/oicr-gsi/niassa/releases/download/v{{version}}/seqware_meta_db_data.sql) :
	Controlled vocabulary for the MetaDB, e.g. populate the 'Platforms' table with different sequencing instruments
* [seqware_meta_db_testdata](https://github.com/oicr-gsi/niassa/releases/download/v{{version}}/seqware_meta_db_testdata.sql) :
	Testing data

# Download unstable or development versions

If you want to use an unstable or development version of the MetaDB, navigate to
the branch on Github or switch to your branch of choice using git and use the
SQL files located in `seqware-meta-db/src/main/resources/io/seqware/metadb/util`.
There is no need to build the source.

```
git checkout -b branch_of_interest origin/branch_of_interest
```

# Installation

This installation guide is used at OICR in order to install and configure the 
MetaDB for production.

## Modify postgresql.conf

Make sure PostgreSQL is configured to accept TCP/IP connections, look in file 
`/etc/postgresql/9.3/main/postgresql.conf` for the following and uncomment 
`listen_address` and `port`.

```
# - Connection Settings -

listen_addresses = '*'          # what IP address(es) to listen on; 
					# comma-separated list of addresses;
					# defaults to 'localhost', '*' = all
port = 5432
```

Add the following to the bottom of the postgresql.conf file - these settings 
will override postgresql settings.

```
max_connections = 100
shared_buffers = 6GB
effective_cache_size = 18GB
work_mem = 629145kB
maintenance_work_mem = 1536MB
checkpoint_segments = 32
checkpoint_completion_target = 0.7
wal_buffers = 16MB
default_statistics_target = 500
```

## pg_hba.conf

Make sure `/etc/postgresql/9.3/main/pg_hba.conf` has an appropriate 
authentication line for your user. Keep in mind you need to put the more 
specific rules first, for example your md5 auth for `seqware` should be the first 
line and then you can have another line for all other databases/user after it. 
Take a look at the [PostgreSQL docs](http://www.postgresql.org/docs/) for more 
info about this file. 
You may want something like:

```
# Database administrative login by Unix domain socket
local   all             postgres                                ident

#HOST   DATABASE          USER                              AUTH
local   all     seqware         md5

# IPv4 local connections:
host    all     seqware         127.0.0.1/32    md5
```

By default PostgreSQL uses “ident” to ensure that the user logging in via psql 
is the same as the user logged into the shell. Turn this off for the seqware 
user so you can log in as seqware under any users account. Note that the 
seqware user needs to be able to log on both to ‘seqware_meta_db’ and 
‘test_seqware_meta_db’ (if you would like to run the tests).

## Set environment variables

First, to make things a little easier, set a number of environment variables.
Use values appropriate for your setup.

```bash
HOST="my.hostname"
PORT="5432"
DB_NAME="name_of_new_db"
DB="/.mounts/labs/seqprodbio/private/backups/sqwprod2-db.hpc.oicr.on.ca/database/20170505-03:30.sql.gz"

#this is the "admin@admin.com" user's password that is stored in the registration table
SEQWARE_ADMIN_PASSWORD="??????"
#this is the postgres user you will create in the next step
DB_USER="postgres_user_with_perms"
DB_USER_PASSWORD="???????"
```

## Create a database user

As the `postgres` user, create a database user with the following command. 
`${DB_USER}` is the username and `${DB_USER_PASSWORD}` is the password for this 
user. 

```bash
$ psql -h "${HOST}" -p "${PORT}" -d "postgres" -c "CREATE USER ${DB_USER} WITH PASSWORD ${DB_USER_PASSWORD} CREATEDB;"
```

## Create a .pgpass file to avoid password prompts

Run the following command to append your new username and password to your 
PGPASS file. This file will prevent PostgreSQL prompting you for a password when 
you execute psql commands for the rest of this guide. 

```bash
echo "${HOST}:${PORT}:*:${DB_USER}:${DB_USER_PASSWORD}" >> "${HOME}/.pgpass"
```

### Set up Postgres Language

Once you create the database you may have to load a stored procedure programming 
language in order for stored procedures in the database to work properly. From 
the commandline, run:

```
$ sudo -u postgres createlang plpgsql seqware_meta_db
```

In order to enable this for all of the databases created on your instance of PostgreSQL in the future, also apply this to the template1 database. This step is required for the tests to work properly.

```
$ sudo -u postgres createlang plpgsql template1
```

## Restart PostgreSQL

Make sure to restart postgres after modifying the PostgreSQL configuration files.

## Create the database

```bash
#create new database
psql -h "${HOST}" -U "${DB_USER}" -p "${PORT}" -d "postgres" -c "CREATE DATABASE ${DB_NAME} WITH OWNER ${DB_USER} TEMPLATE=template0 LC_COLLATE='en_CA.UTF-8' LC_CTYPE='en_CA.UTF-8'"
```

## Populate the database

### For an empty database

There are two SQL files that are used for building and populating an empty MetaDB database.

* seqware_meta_db.sql – this is the schema for the database without any data
* seqware_meta_db_data.sql – this contains the basic data for the database, 
	including organisms, admin registrations, etc.

Run the following command to create the database and populate it with basic
data. Make sure that `seqware_meta_db.sql` and `seqware_meta_db_data.sql` point
to real locations.

```bash
# construct the schema
psql -h "${HOST}" -U "${DB_USER}" -p "${PORT}" -d "${DB_NAME}"  -f seqware_meta_db.sql 
# populate the database with basic data
psql -h "${HOST}" -U "${DB_USER}" -p "${PORT}" -d "${DB_NAME}"  -f seqware_meta_db_data.sql

```

### Restore from a backup

An additional environment variable, the location of a database backup file:

```bash
DB="/.mounts/labs/seqprodbio/private/backups/sqwprod2-db.hpc.oicr.on.ca/database/20170505-03:30.sql.gz"
```

Run the following to load this into your newly created database:

```bash
#restore backup to the new database
gunzip -c  "${DB}" | psql -h "${HOST}" -p "${PORT}" -d "${DB_NAME}" -U "${USER}"
#cancel workflow runs that were scheduled or running at the time of the backup
psql -h "${HOST}" -p "${PORT}" -d "${DB_NAME}" -U "${USER}" -c "UPDATE workflow_run SET status = 'cancelled' WHERE status IN ('pending', 'submitted', 'running','submitted_cancel') OR status IS NULL"
```

## Set a new admin password

For security purposes you should reset the Niassa admin password (for the 
`admin@admin.com` user) immediately after database creation. Note that this should
be the plain-text password: we will salt and encrypt it next.

```bash
#set new seqware webservice admin password
psql -h "${HOST}" -p "${PORT}" -d "${DB_NAME}" -U "${USER}" -c "UPDATE registration SET password ='"${SEQWARE_ADMIN_PASSWORD}"' WHERE email = 'admin@admin.com'"
```

Run the salting-encrypting utility 
[RegistrationMigrationPlugin](https://github.com/oicr-gsi/niassa/blob/develop/seqware-pipeline/src/main/java/net/sourceforge/seqware/pipeline/plugins/RegistrationMigrationPlugin.java) 
to encrypt the password. You will need to have a copy of the 
seqware-distribution jar and set up your settings file according to the 
[client-only](https://oicr-gsi.github.io/niassa-docs/current/installation/client-only) 
installation instructions. If you don't have this set up yet, you can come back 
to this step later.

```bash
java -jar seqware-distribution-{{version}}-full.jar --plugin net.sourceforge.seqware.pipeline.plugins.RegistrationMigrationPlugin
```

## Test the database
Once created, you should be able to log in to the db and list the database tables:

```
psql -h "${HOST}" -U "${DB_USER}" -p "${PORT}" -d "${DB_NAME}" 
Password for user seqware: 
psql ({{psql_version}})
Type "help" for help.

seqware_meta_db=> \dt
		      List of relations
 Schema |               Name               | Type  |  Owner  
--------+----------------------------------+-------+---------
 public | experiment                       | table | seqware
 public | experiment_attribute             | table | seqware
 public | experiment_library_design        | table | seqware
 public | experiment_link                  | table | seqware
 ....
```

# For Developers

## Populating the database with test data

* seqware_meta_db_testdata.sql -- this contains the testing data for the 
	database, including sequencer_runs, lanes, studies, and processing events.

Testing databases are built using all three SQL files in sequence. The test data 
SQL file contains only testing data - none of the basic data from the second SQL 
file should be contained within it at all. There should not be any overlap 
between the two data files (data and testdata) in order to prevent errors and 
duplicated rows.

Follow the directions as for [empty databases](#for-an-empty-database), but 
additionally load the testdata SQL file:

```
psql -h "${HOST}" -U "${DB_USER}" -p "${PORT}" -d "${DB_NAME}" -f seqware_meta_db_testdata.sql
```


### Upgrading your Database Version
If you have previously installed MetaDB and want to upgrade to the 
latest version, you can do so by running the upgrade scripts available 
from the repository.

In the MetaDB module, there are a number of upgrade scripts 
that allow you to update between versions. The numeric sql scripts, for example 
1.0.x_to_1.2.sql, contain the statements required to update the database schema 
between revisions. You must apply each of the successive patches in order. 

The files can be applied by using the script as an input to the psql call. For 
example:

```
$ psql -U seqware -W -f 1.0.x_to_1.2.sql seqware_meta_db
```

We have done our best to avoid potential errors during upgrades. If you run into 
any problems, please contact the developers.