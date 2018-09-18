---
layout: page
title: Web service - Add a new user
---
{% include functions.liquid %}

Add a new Niassa user by adding a new user to the [MetaDB]({{version_url}}/metadb) `registration` table.
The new user will be able to access the 
[Web Service]({{version_url}}/web-service) by setting `SW_REST_USER` and 
`SW_REST_PASS` properties in their 
[User Settings]({{version_url}}/environment/user-configuration) file.

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
