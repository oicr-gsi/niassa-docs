---
layout: page
title: Webservice Installation Guide
---
{% include functions.liquid %}

* toc
{:toc}

# Requirements

The [Niassa Web service]({{version_url}}/web-service) requires:

* Java 8
* Apache Tomcat 8.0+
* An [installed and configured MetaDB]({{version_url}}/installation/metadb) PostgreSQL database accessible from the host where the Web service will be hosted

These instructions have been tested on Debian 8.

# Download

For stable releases, the WAR is available for download from Artifactory:
[seqware-webservice-{{version}}.war](https://artifacts.oicr.on.ca/artifactory/gsi-release/com/github/seqware/seqware-webservice/{{version}}/seqware-webservice-{{version}}.war)

# Building from source

If you want to build an unstable or development version, see the 
[Niassa Build Guide](https://oicr-gsi.github.io/niassa-docs/current/installation/building) 
for detailed build instructions. After the build completes, the WAR file for 
deployment will be in 
`seqware-webservice/target/seqware-webservice-{{version}}.war`

# Installation

This installation guide is used at OICR in order to install and configure the 
Web Service for production. 

## Increase Tomcat memory

As the tomcat8 user, configure your Tomcat instance by modifying the `/etc/default/tomcat8` file.

```bash
# set a standard timezone and memory requirements. Adjust as necessary.
JAVA_OPTS="-Duser.timezone=GMT -Djava.awt.headless=true -Xmx18g -Xms18g"

# gc options and enable string dedup
JAVA_OPTS="${JAVA_OPTS} -XX:+UseG1GC -XX:+UseStringDeduplication -XX:ConcGCThreads=2"

#enable verbose gc
JAVA_OPTS="${JAVA_OPTS} -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps"
```

## Enable Tomcat Promethus metrics

If you are using [Prometheus](https://prometheus.io/) integration, add these 
additional lines to the `/etc/default/tomcat8` file:

```bash
#enable jmx
JAVA_OPTS="${JAVA_OPTS} -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.port=1098"

#enable metrics
JAVA_OPTS="${JAVA_OPTS} -javaagent:"/var/lib/tomcat8/lib/jmx_prometheus_javaagent-0.2.0.jar=9090:/var/lib/tomcat8/lib/prometheus_jmx.yml""

...

test -f /var/lib/tomcat8/lib/jmx_prometheus_javaagent-0.2.0.jar || wget -O /var/lib/tomcat8/lib/jmx_prometheus_javaagent-0.2.0.jar https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/0.2.0/jmx_prometheus_javaagent-0.2.0.jar
test -f /var/lib/tomcat8/lib/prometheus_jmx.yml || wget -O /var/lib/tomcat8/lib/prometheus_jmx.yml https://raw.githubusercontent.com/prometheus/jmx_exporter/master/example_configs/tomcat.yml
```

## Configure Tomcat connections

As the tomcat8 user, edit `/etc/tomcat8/server.xml` to better enable the Web service to operate under 
load:

Change the following line from :
```xml
<Connector port="8080" protocol="HTTP/1.1"
   connectionTimeout="20000"
   URIEncoding="UTF-8"
   redirectPort="8443" />
```
To:
```xml
<Connector port="8080"
   protocol="HTTP/1.1"
   connectionTimeout="120000"
   URIEncoding="UTF-8"
   redirectPort="8443"
   acceptCount="2048"
   maxThreads="32"
   maxConnections="2048"
   maxKeepAliveRequests="1" />
```

## Enable Tomcat Manager, if desired

You can double-check whether these Tomcat settings were successfully set by going to 
[http://localhost:8080/manager/status/all](http://localhost:8080/manager/status/all) 
. You may need to edit your `/etc/tomcat8/tomcat-users.xml` file and add/enable the 
following lines.

```xml
<role rolename="manager"/>
<role rolename="admin"/>
<user username="admin" password="admin" roles="admin,manager"/>
```


<a name="xml"/>
## Configure the webservice XML file

As the tomcat8 user, edit or create 
`/etc/tomcat8/Catalina/localhost/seqware-webservice.xml` and copy the following
into that file:

```
<?xml version="1.0" encoding="UTF-8"?>
<Context antiJARLocking="true" reloadable="true" path="" autodeploy="true">
    <Resource
        auth="Container"
        description="Seqware Hibernate"
        driverClassName="org.postgresql.Driver"
        jdbcInterceptors="org.apache.tomcat.jdbc.pool.interceptor.StatementDecoratorInterceptor"
        name="jdbc/SeqWareMetaDB"
        url="jdbc:postgresql://HOST:5432/DATABASE"
        username="USER"
        password="PASSWORD"
        testOnBorrow="true"
        testOnReturn="true"
        testWhileIdle="true"
        type="javax.sql.DataSource"
        factory="org.apache.commons.dbcp.BasicDataSourceFactory"
        maxTotal="35"
        maxActive="35"
        maxIdle="35"
        minIdle="1"
        initialSize="1"
        timeBetweenEvictionRunsMillis="20000"
        minEvictableIdleTimeMillis="20000"
        numTestsPerEvictionRun="35"
        validationQuery="SELECT 1"/>
</Context>
```
Replace:

* `HOST` : the IP or name of the host where the MetaDB is installed
* `DATABASE` : the name of the MetaDB database 
* `USER` : the Postgres user that can access the MetaDB
* `PASSWORD` :  the password for the above Postgres user


## Deploy the Web Service

In order to deploy the Web service into Tomcat, acquire the WAR file either from
[downloading it](#download) or by [building from source](#building-from-source).

If downloading, you can perform the download and transfer in one command:

```
curl 'https://artifacts.oicr.on.ca/artifactory/gsi-release/com/github/seqware/seqware-webservice/{{version}}/seqware-webservice-{{version}}.war' -o '/var/lib/tomcat8/webapps/seqware-webservice.war'
```

If you built the WAR, drop it in 
`/var/lib/tomcat8/webapps/seqware-webservice.war`. Make sure the name of the WAR 
file matches the [XML file you created earlier](#xml).


## Restart Tomcat

Restarting Tomcat will apply the configuration changes and deploy the WAR.

```
sudo /etc/init.d/tomcat8 restart
```

The Web Service will deploy at 
[http://localhost:8080/seqware-webservice}](http://localhost:8080/seqware-webservice), 
or http://host.address:port/seqware-webservice-x.x.x, where your `host.address` 
is the address of the machine running the Web service, the `port` is the Tomcat 
port, and `seqware-webservice-x.x.x` is the appropriate version number on the 
seqware-webservice WAR. Navigating to this address in your browser should not 
show an error message (you will have to authenticate with an appropriate 
username and password as specified in the MetaDB).

# For Developers 

## Interactive Testing

If you want to startup the embedded Tomcat server for interactive testing, make 
sure that your ~/.seqware/settings file includes the line:

```
SW_REST_URL=http://localhost:8889/seqware-webservice
```

And then run:

```
mvn tomcat7:run
```
The webservice will be available at 
[http://localhost:8889/seqware-webservice](http://localhost:8889/seqware-webservice).

## Configuring the database connection

If you wish to test against a real database, two files need to be edited with 
database information in the source in order to successfully build, test and run 
the Web service.

These files are located at:

* Running: `seqware-webservice/src/main/webapp/META-INF/context.xml` - change 
  all parameters to match your local seqware_meta_db connection.
* Testing: `seqware-webservice/src/main/webapp/WEB-INF/jetty-env.xml]` - leave 
  the DB name as test_seqware_meta_db. This DB will be dropped and re-created at 
  test runtime.

Three variables need to be changed in each file to reflect your local setup:

* url="jdbc:postgresql://localhost:5432/seqware_meta_db"
* username="seqware"
* password="seqware"

The url, username and password need to be changed to reflect the local database. 
The username and password are the PostgreSQL database username and password.

# Troubleshooting

## Tomcat is ignoring my memory settings
In some cases, Tomcat will ignore configuring the `JAVA_OPTS` either on the 
command line or in `/etc/init.d/tomcat8`. In these cases, you can try adding a 
`setenv.sh` file with the single line:

```bash
export JAVA_OPTS="-server -Xss1024K -Xms1G -Xmx2G -XX:MaxPermSize=128M -XX:NewSize=512m"
```