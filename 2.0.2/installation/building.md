---
layout: page
title: Build Guide
---
{% include functions.liquid %}

* toc
{:toc}

# Prerequisites
{% assign psql_version="9.3.16+" %}
* Linux (preferably Debian or Ubuntu)
* Java 8
* Maven 3.3.9+
* Postgres {{psql_version}} (for integration testing only)

# Get the source code 

Our source code is available from [GitHub](https://github.com/oicr-gsi/niassa) 
or the "Fork me on GitHub" banner at the upper right of our website

To get a copy of of our source code you will first need to install Git 
(`sudo apt-get install git` in Ubuntu) and then clone our repository.

```
git clone git://github.com/oicr-gsi/niassa.git
```

By default, this will land you on the `develop` branch.


<a name="test_db"/>
# Set up for integration testing

Niassa Integration testing requires PostgreSQL to be installed, for a database to be 
created for testing purposes, connection information entered in 
the [Settings]({{version_url}}/environment/user-configuration) file or supplied at 
build time, and for the database to be pre-populated with testing data.

You do *not* need to set up PostgreSQL if you want to build without tests or 
with only unit tests.

## Create the database

All integration tests require a PostgreSQL {{psql_version}} database for testing. 
These are the instructions to create this database.

Replace the DB variables below as appropriate for your system.

```bash
# the name of the MetaDB
export DB_NAME="test_db"
# the PostgreSQL user which will access the MetaDB
export DB_USER="postgres"
# the password for the PostgreSQL user
# make sure to change this!
export DB_PASSWORD="???" 
# the host IP or address where PostgreSQL is running
export DB_HOST="127.0.0.1"
# the port where PostgreSQL is running
export DB_PORT="5432"

# add to pgpass
# echo "${DB_HOST}:${DB_PORT}:${DB_NAME}:${DB_USER}:${DB_PASSWORD}" >> "${HOME}/.pgpass"

psql -h "${DB_HOST}" -U "${DB_USER}" -p "${DB_PORT}" -d postgres -c "create database ${DB_NAME}"
```

## Configure Niassa to use the new DB

Once you have created the database, you will need to instruct Niassa how to 
connect to your new database. You configure either access through the 
`~/.seqware/settings` file or by specifying at build time.

### Modifying settings 

The following line appends the settings file using 
the previously defined environment variables: 

```bash
cat << EOF > ${HOME}/.seqware/settings
SW_METADATA_METHOD=webservice
SW_REST_URL=http://localhost:8889/seqware-webservice
SW_REST_USER=admin@admin.com
SW_REST_PASS=admin
 
EXTENDED_TEST_DB_NAME=${DB_NAME}
EXTENDED_TEST_DB_USER=${DB_USER}
EXTENDED_TEST_DB_PASSWORD=${DB_PASSWORD}
EXTENDED_TEST_DB_HOST=${DB_HOST}
EXTENDED_TEST_DB_PORT=${DB_PORT}
EOF
```

For more information on this file, see 
[Settings]({{version_url}}/environment/user-configuration).

### Specifying on the fly

Alternatively, if you don't want to edit your settings file, add these variables
to your Maven commands to override them on the fly:

```
-Dseqware_meta_db_host="${DB_HOST}" \
-Dseqware_meta_db_port="${DB_PORT}" \
-Dseqware_meta_db_name="${DB_NAME}" \
-Dseqware_meta_db_user="${DB_USER}" \
-Dseqware_meta_db_password="${DB_PASSWORD}"
```


## Load test data

In the testing database, create the tables and populated with the built-in 
testing data. These SQL files are available in the Niassa seqware-meta-db 
module. Make sure `NIASSA_PATH` is set to the correct location.

```
export NIASSA_PATH="${HOME}/niassa"
 
psql -h "${DB_HOST}" -U "${DB_USER}" -p "${DB_PORT}" -d "${DB_NAME}" -f "${NIASSA_PATH}/seqware-meta-db/src/main/resources/io/seqware/metadb/util/seqware_meta_db.sql"
psql -h "${DB_HOST}" -U "${DB_USER}" -p "${DB_PORT}" -d "${DB_NAME}" -f "${NIASSA_PATH}/seqware-meta-db/src/main/resources/io/seqware/metadb/util/seqware_meta_db_data.sql"
psql -h "${DB_HOST}" -U "${DB_USER}" -p "${DB_PORT}" -d "${DB_NAME}" -f "${NIASSA_PATH}/seqware-meta-db/src/main/resources/io/seqware/metadb/util/seqware_meta_db_testdata.sql"
```


# Building without tests

Build without running any tests:

```
mvn clean install -DskipTests
```

# Build and run unit tests

Build and run unit tests (does not require PostgreSQL or Tomcat):

```
mvn clean install  
```

# Build and run integration tests

Make sure you have PostgreSQL installed and Niassa configured 
as per [the above instructions](#test_db). 

**WARNING:**  While integration and extended tests are running, launching 
workflows and workflow status checking MUST not occur.  If you have a cronjob 
performing these tasks it MUST be disabled prior to integration and extended 
testing.

```bash
# ensures that enough memory is allocated for integration tests (presumes 8G RAM is available on your machine)
export MAVEN_OPTS="-Xms8192m -Xmx8192m"
# runs all unit tests and integration tests that only require postgres as a prerequisite
mvn clean install -DskipITs=false -Denforcer.skip=true
```

# Build and run extended integration tests

The extended integration tests profile is used to trigger integration tests that
run our command line utilities. In order to point your command-line tools at 
the web service brought up by the integration tests, you will need to comment 
out your crontab. Make sure you have PostgreSQL installed and Niassa configured 
as per [the above instructions](#test_db).

**WARNING:**  While integration and extended tests are running, launching 
workflows and workflow status checking MUST not occur.  If you have a cronjob 
performing these tasks it MUST be disabled prior to integration and extended 
testing.

Run all unit tests and all integration tests including those that require Oozie:

```bash
# ensures that enough memory is allocated for integration tests (presumes 8G RAM is available on your machine)
export MAVEN_OPTS="-Xms8192m -Xmx8192m"
mvn clean install -DskipITs=false -P extITs,embeddedTomcat -Denforcer.skip=true
```


# Build and run long integration tests

Runs just the long integration tests that take longer than can run on Travis-CI.
Make sure you have PostgreSQL installed and Niassa configured as per 
[the above instructions](#test_db).

**WARNING:**  While integration and extended tests are running, launching 
workflows and workflow status checking MUST not occur.  If you have a cronjob 
performing these tasks it MUST be disabled prior to integration and extended 
testing.

```bash
# ensures that enough memory is allocated for integration tests (presumes 8G RAM is available on your machine)
export MAVEN_OPTS="-Xms8192m -Xmx8192m"
mvn clean install -DskipITs=false -P longITs,embeddedTomcat -Denforcer.skip=true
```  
  


# Build and run extended integration tests against running Tomcat and PostgreSQL instances

It is possible to disable our embedded tomcat instance and run against both a 
remote postgres and Tomcat instance. Note that you will need to deploy the 
seqware-webservice war yourself. 

**WARNING:**  While integration and extended tests are running, launching 
workflows and workflow status checking MUST not occur.  If you have a cronjob 
performing these tasks it MUST be disabled prior to integration and extended 
testing.

Set the following variables in your `.seqware/settings` to override these 
settings for the extended integration 
tests:

```
EXTENDED_TEST_DB_HOST=otherserver.ca
EXTENDED_TEST_DB_NAME=test_seqware_meta_db
EXTENDED_TEST_DB_USER=seqware
EXTENDED_TEST_DB_PASSWORD=seqware
SW_REST_URL=https://otherserver.ca/seqware-webservice 
```

Then run the command:

```bash
# ensures that enough memory is allocated for integration tests (presumes 8G RAM is available on your machine)
export MAVEN_OPTS="-Xms8192m -Xmx8192m"
mvn clean install -DskipITs=false -P 'extITs,!embeddedTomcat -Denforcer.skip=true'
```

# Build and run integration tests and connect using a debugger

If you wish to debug unit tests, you can connect a debugger on port 5005 after 
starting the tests with:

```bash
# ensures that enough memory is allocated for integration tests (presumes 8G RAM is available on your machine)
export MAVEN_OPTS="-Xms8192m -Xmx8192m"
mvn clean install -Dmaven.surefire.debug -DskipITs=false -Denforcer.skip=true
```

If you run the full suite of tests, you will have to reconnect the debugger for each module.