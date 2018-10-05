---
layout: page
title: Add new Niassa users
---
{% include functions.liquid %}

Depending on the Niassa setup, several systems need to be configured for new
users. This guide is for Niassa administrators.

* TOC
{:toc}

# All Systems

Add a new Niassa user by adding a new user to the [MetaDB]({{version_url}}/metadb) `registration` table.
The new user will be able to access the
[Web Service]({{version_url}}/web-service) by setting `SW_REST_USER` and
`SW_REST_PASS` properties in their
[User Settings]({{version_url}}/environment/user-configuration) file. Once this
step is done, the user will be able to connect using the
[CLI]({{version_url}}/CLI), but may not be able to run workflows.

## Adding an ordinary user

Niassa web service passwords are salted and encrypted. Note that from a user
point of view, their password will remain the same. Salting means that
their passwords will be stored more securely on the server-side in the database.

You will need direct access to the Niassa PostgreSQL
[MetaDB]({{version_url}}/metadb).

1. Use the `psql` client to log into the Niassa MetaDB (here known as
  `seqware_meta_db`)
  ```
$ psql seqware_meta_db
  ```
2. Insert a row into the `registration` table with the user's information (and
  unsalted, unencrypted password--we will fix this in the next step).
  ```
seqware_meta_db=# insert into registration(email, password, password_hint, first_name, last_name, create_tstmp, last_update_tstmp)
    VALUES('testing@test.com',
           'test-password',
           'a test password hint',
           'firstname',
           'lastname',
           now(),
           now());
INSERT 0 1
  ```
3. Run the salting-encrypting utility
  [RegistrationMigrationPlugin](https://github.com/oicr-gsi/niassa/blob/develop/seqware-pipeline/src/main/java/net/sourceforge/seqware/pipeline/plugins/RegistrationMigrationPlugin.java)
  to encrypt their password.
  ```
java -jar ~/.seqware/self-installs/seqware-distribution-{{version}}-full.jar --plugin net.sourceforge.seqware.pipeline.plugins.RegistrationMigrationPlugin
  ```

## Adding an admin user

Follow the same steps as for [adding an ordinary user](#adding-an-ordinary-user), and additionally:

4. Set the `lims_admin` column to be `true` for the admin user.
  ```
seqware_meta_db=# update registration set lims_admin=true where email='admin@admin.com';
  ```

## Allow the user to log on to \*-user

The user will log on to the \*-user host to schedule workflows (see
[for Niassa admins]({{version_url}}/admins) for details on the multi-server
architecture). Follow the established practices for your institution to allow
the user to log on to the \*-user server.

(For OICR, send an email to Helpdesk IT requesting access to the appropriate
host, e.g. `nsdev-user.hpc.oicr.on.ca`)

# Oozie and Oozie-SGE workflow engines

These steps only need to be taken if the user requires use of one of the
Oozie workflow engines in Niassa [Pipeline]({{version_url}}/pipeline). The
instructions below assume that there is a separate \*-node host with Oozie
installed. Below, we use `nsdev-node.hpc.oicr.on.ca`.

## Create user's HDFS directory (OOZIE_APP_PATH)

Once created, this directory can be used in the
[User Configuration]({{version_url}}/environment/user-configuration) file under
`OOZIE_APP_PATH`.

1. Connect to the "oozie node" or get an administrator to do the following
  ```
ssh nsdev-node.hpc.oicr.on.ca
  ```
2. Create the hdfs directory
  ```
export NEW_USER='mlaszloffy'
HADOOP_USER_NAME=hdfs hadoop fs -mkdir /user/${NEW_USER}
  ```
3. Change the directory ownership
  ```
HADOOP_USER_NAME=hdfs hadoop fs -chown ${NEW_USER}:supergroup /user/${NEW_USER}
  ```
4. Check the permissions (they should be 755)
  ```
hadoop fs -ls /user/
...
drwxr-xr-x   - mlaszloffy   supergroup          0 2016-10-14 13:40 /user/mlaszloffy
  ```

5. Instruct the user to update their `~/.seqware/settings` file with the
  following:
```
...
OOZIE_APP_PATH=hdfs://nsdev-node.hpc.oicr.on.ca:8020/user/${USER}/
...
```
