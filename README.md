# jboss-api

Java EE basic REST application that can be deployed on [JBoss EAP7](https://developers.redhat.com/products/eap/download/).  The [pom.xml](https://github.com/mechevarria/jboss-api/blob/master/pom.xml) was copied and edited from the [jboss-eap-quickstarts](https://github.com/jboss-developer/jboss-eap-quickstarts)

The [web.xml](https://github.com/mechevarria/jboss-api/blob/master/src/main/webapp/WEB-INF/web.xml) has a config that can be uncommented to integrate with [Red Hat Single Sign On](https://access.redhat.com/products/red-hat-single-sign-on)

## To get started
Import as a maven project in [JBoss Developer Studio](https://www.redhat.com/en/technologies/jboss-middleware/developer-studio)

Once deployed, you can see a status value at
http://localhost:8080/jboss-api/status

This project works to create a [Angular Resource](https://docs.angularjs.org/api/ngResource/service/$resource) REST api that works with [jboss client](https://github.com/mechevarria/jboss-client) and can do all CRUD operations on the `jboss-api/item` endpoint

You can also import the integration tests as a collection to [Postman](https://www.getpostman.com/)

By default the application is backed by an in-memory **h2** database.  The `assets` directory has scripts to help with an external Postgresql datasource.

**Optional config: **You will need [PostgreSQL](https://www.postgresql.org/) already installed and running.  This configuration allows admins to health check the database connection from the EAP admin console

## Openshift

Run with the **Red Hat JBoss EAP 7.0** xPaas image

Name the service **eap-server**

This will allow seamless integration with [JBoss Client](https://github.com/mechevarria/jboss-client)

## Single Sign On
* The server configuration is in [keycloak.json](https://github.com/mechevarria/jboss-api/blob/sso/src/main/webapp/WEB-INF/keycloak.json)
* The additional elements are added in [web.xml](https://github.com/mechevarria/jboss-api/blob/sso/src/main/webapp/WEB-INF/web.xml)

By default the keycloak config is commented out

The EAP instance requires that the [Java Adapter](https://keycloak.gitbooks.io/documentation/securing_apps/topics/oidc/java/jboss-adapter.html) be installed

## Install the jdbc driver
If you choose to not use the embedded h2 datasource, the following will help setup and source like Postgresql

### Script
```bash
jboss-module/module.sh add
``````
### Manual
~~~bash
~/local/devstudio/runtimes/jboss-eap/bin/jboss-cli.sh

connect
~~~
Adjust the path to the PostgreSQL jar as necessary
~~~bash
module add --name=org.postgresql --resources=/home/redhat/git/jboss-api/jboss-module/postgresql-9.4.1212.jar --dependencies=javax.api,javax.transaction.api

/subsystem=datasources/jdbc-driver=postgresql:add(driver-name="postgresql",driver-module-name="org.postgresql")
~~~

## Configure the jdbc connection using the newly installed driver
In the jboss admin console

`http://localhost:9990/console`

goto **Configuration -> Subsystems -> Datasources -> Non-XA**

Click Add -> PostgreSQL Datasource

Accept the default datasource attributes **PostgresDS** and **java:/PostgresDS**

In the JDBC Driver step select the **postgresql** option in the Detected Driver tab

In the Connection Settings use

jdbc:postgresql://localhost:5432/**jboss**

username: **jboss**

password **jboss**

After finishing the wizard, select **PostgresDS** > **Test Connection** to verify



## Removing the Datasource and Driver

### Script

`jboss-module/module.sh rm`

### Manual
~~~bash
~/local/devstudio/runtimes/jboss-eap/bin/jboss-cli.sh

connect

data-source disable --name=PostgresDS

data-source remove --name=PostgresDS

/:reload

/subsystem=datasources/data-source=postgresql:remove

/subsystem=datasources/jdbc-driver=postgresql:remove

module remove --name=org.postgresql
~~~
