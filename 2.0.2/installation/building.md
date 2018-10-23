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

To get a copy of our source code,
clone our repository.

```
git clone https://github.com/oicr-gsi/niassa.git
```

By default, this will land you on the `develop` branch.

# Set up for building

We use Maven 3 to build and run tests.

## Setting up Maven for Java 8

Maven requires a toolchains file to set the Java version. You can find this at 
[.travis/toolchains.xml](https://github.com/oicr-gsi/niassa/blob/develop/.travis/toolchains.xml)
in the Niassa repository. Maven also requires a certain amount of memory, so set
that in a `~/.mavenrc` file.

```bash
# note that this will NOT overwrite any existing toolchain files
cp -n .travis/toolchains.xml ~/.m2
echo "MAVEN_OPTS='-Xms6144m -Xmx6144m'" >> ~/.mavenrc
```

## Building without tests

Build without running any tests:

```bash
mvn clean install -DskipTests
```

<a name="basic_tests"/>
# Basic Testing

Basic testing runs local unit tests.

## Niassa settings file

All Niassa testing requires a settings file to exist at 
`~/.seqware/settings`. Note that the `SEQWARE_SETTINGS` environment variable 
does not work for testing.

If you have never installed Niassa (or SeqWare) before running tests, create
a settings file:

```bash
# create the directory if it doesn't already exist
# note if there is already a file at ~/.seqware/settings this will NOT overwrite it
mkdir -p ~/.seqware && cp -n .travis/settings ~/.seqware/settings
```

If you do have a `settings` file, ensure the following variables are set:

```
SW_METADATA_METHOD=webservice
SW_REST_URL=http://localhost:8889/seqware-webservice
SW_REST_USER=admin@admin.com
SW_REST_PASS=admin
```

## Build and run unit tests

Build and run unit tests (does not require PostgreSQL or Tomcat):

```bash
mvn clean install  
```

## Build and run tests and connect using a debugger

If you wish to debug tests, you can connect a debugger on port 5005 after 
starting the tests with:

```bash
# ensures that enough memory is allocated for integration tests (presumes 8G RAM is available on your machine)
export MAVEN_OPTS="-Xms8192m -Xmx8192m"
mvn clean install -Dmaven.surefire.debug
```

If you run the full suite of tests, you will have to reconnect the debugger for 
each module.


<a name="integrated_tests"/>
# Integration testing

Niassa integration testing requires PostgreSQL, a test database to be 
created, connection information entered in the 
[Settings]({{version_url}}/environment/user-configuration) file and supplied at 
build time, and for the database to be pre-populated with testing data.

This part of the guide assumes that you're already set up for 
[basic testing](#basic_tests) with Maven and Niassa settings files.
All integration tests require a PostgreSQL {{psql_version}} database for testing. 
Replace the DB variables below as appropriate for your system.

You do *not* need to set up PostgreSQL if you want to build without tests or 
with only unit tests.

```bash
# the name of the MetaDB
export DB_NAME="test_niassa_meta_db"
# the PostgreSQL user which will access the MetaDB
# this MUST remain 'postgres'
export DB_USER="postgres"
# the password for the PostgreSQL user. 
# Note that this generates a random string for a password
export DB_PASSWORD=$(openssl rand -base64 12)
# the host IP or address where PostgreSQL is running
export DB_HOST="127.0.0.1"
# the port where PostgreSQL is running
export DB_PORT="5433"

# add to pgpass
echo "${DB_HOST}:${DB_PORT}:${DB_NAME}:${DB_USER}:${DB_PASSWORD}" >> "${HOME}/.pgpass"
```

## Start PostgreSQL Docker container

Start a PostgreSQL {{psql_version}} container. Because I have a local installation of 
PostgreSQL, I map 5432 in the container to 5433 on the localhost. This command
also creates a database and user with the names above and sets their password
to be the randomly generated one above.

From the root of the Niassa directory, run the 
[start-postgres.sh](https://github.com/oicr-gsi/niassa/blob/develop/.travis/start-postgres.sh) 
script:

```bash
bash .travis/start-postgres.sh $(pwd) ${DB_NAME} ${DB_USER} ${DB_PASSWORD} ${DB_HOST} ${DB_PORT} 
```

Check the container is running and your database is set up properly. If your 
`.pgpass` file is set up properly, it should not ask for a password and should 
connect you directly to the metadb_test database.

```bash
psql -h ${DB_HOST} -p ${DB_PORT} -U ${DB_USER} -d ${DB_NAME}
```

Set an environment variable `DB_ENV` to provide the location of the DB to Maven 
at runtime:

```bash 
export DB_ENV=" -Dseqware_meta_db_host=${DB_HOST} \
-Dseqware_meta_db_port=${DB_PORT} \
-Dseqware_meta_db_name=${DB_NAME} \
-Dseqware_meta_db_user=${DB_USER} \
-Dseqware_meta_db_password=${DB_PASSWORD}"
```

Make sure your `.seqware/settings` file is set up as described in 
[basic testing](#basic_tests) (particularly the REST_URL).

## Build and run integration tests

Make sure you have PostgreSQL installed and Niassa configured 
as per [the above instructions](#integrated_tests). 

```bash
# ensures that enough memory is allocated for integration tests (presumes 8G RAM is available on your machine)
export MAVEN_OPTS="-Xms8192m -Xmx8192m"
# runs all unit tests and integration tests that only require postgres as a prerequisite
mvn clean install -DskipITs=false ${DB_ENV}
```

## Build and run integration tests and connect using a debugger

If you wish to debug tests, you can connect a debugger on port 5005 after 
starting the tests with:

```bash
# ensures that enough memory is allocated for integration tests (presumes 8G RAM is available on your machine)
export MAVEN_OPTS="-Xms8192m -Xmx8192m"
mvn clean install -Dmaven.surefire.debug -DskipITs=false ${DB_ENV}
```

If you run the full suite of tests, you will have to reconnect the debugger for 
each module.

## Build and run extended integration tests

The extended integration tests profile is used to trigger integration tests that
run our command line utilities. In order to point your command-line tools at 
the web service brought up by the integration tests, you will need to comment 
out your crontab. Make sure you have PostgreSQL installed and Niassa configured 
as per [the above instructions](#integrated_tests).

Run all unit tests and all integration tests including those that require Oozie:

```bash
# ensures that enough memory is allocated for integration tests (presumes 8G RAM is available on your machine)
export MAVEN_OPTS="-Xms8192m -Xmx8192m"
mvn clean install -DskipITs=false -P extITs,embeddedTomcat ${DB_ENV}
```

## Build and run extended integration tests against running Tomcat and PostgreSQL instances

It is possible to disable our embedded Tomcat and run tests against both a 
remote Postgres and Tomcat instance. Note that you will need to deploy the 
seqware-webservice war yourself. 

**WARNING:**  The tests *WILL* clobber your remote database, so be very careful 
running against a remote service.

Set the following variables in your `.seqware/settings` to override these 
settings for the extended integration tests:

```bash
SW_REST_URL=https://otherserver.ca/seqware-webservice
```

Then run with this on the command line
```
export DB_OTHER_ENV="-Dseqware_meta_db_host=otherserver.ca \
-Dseqware_meta_db_port=5433 \
-Dseqware_meta_db_name=other_metadb \
-Dseqware_meta_db_user=other_psql_usr \
-Dseqware_meta_db_password=other_pass"
```

Then run the command:

```bash
# ensures that enough memory is allocated for integration tests (presumes 8G RAM is available on your machine)
export MAVEN_OPTS="-Xms8192m -Xmx8192m"
mvn clean install -DskipITs=false -P extITs,embeddedTomcat ${DB_OTHER_ENV}
```

# Troubleshooting

## Integration tests aren't passing on `develop`!

First, make sure you let the tests run to completion and that they are truly
failing. Niassa prints a number of errors and stack traces as part of testing 
and it's not truly failed until Maven has decided it is. The `develop` branch 
should always be passing.

Check to make sure that Niassa tests are passing at all on develop at Travis-CI: 
[oicr-gsi/niassa](https://travis-ci.org/oicr-gsi/niassa). If they are, check the
[.travis.yml](https://github.com/oicr-gsi/niassa/blob/develop/.travis.yml) file
to see if the integration procedure has changed. We try to keep this document in
sync with the latest, but errors will happen.

If these are fine, try an escalating series of things to fix your integration
tests:

1. Make sure the `DB_` environment variables are set properly 
  (try `env | grep DB_`)
1. Double check that the `$DB_ENV` and `~/.pgpass` files have the correct 
  information for the currently running database (and that `~/.pgpass` only has 
  one entry for the integration test MetaDB).
1. Set up your environment again from scratch as described above. Replace your 
  `~/.seqware/settings`, `~/.m2/toolchains.xml`, and `~/.mavenrc` files.
1. Kill the running Docker MetaDB and start it again using the 
  `.travis/start-postgres.sh` script. Make sure you set your `.pgpass` and `$DB_ENV`
  variables again.
1. Remove your `~/.m2/repository` and `mvn clean install -DskipTests` to start 
  everything again.

Good luck!

