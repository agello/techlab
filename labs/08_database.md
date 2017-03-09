# Lab 8: Connect the database

Most applications are in some way stateful and save data persistently. Be this in a database or as files on a file system or ObjectStore. In this lab, we will create a MySQL service in our project and connect it to our application so that several applicationpods can access the same database.

For this example, the SpringBoot sample is taken from [LAB 4](04_deploy_dockerimage.md), `[USER]-dockerimage`

## Task: LAB8.1: Create MySQL service

For our example, we use an OpenShift template in this lab, which creates a MySQL database with EmptyDir Data Storage. This is to be used only for test environments, since all data will be lost when restarting the MySQL Pod. In a later lab, we will show how to attach a Persistent Volume (mysql-persistent) to the MySQL database. As a result, the data remain at Restarts and is thus suitable for productive operation.

We can create the MySQL service via the web console as well as via the CLI.

To get the result, you just have to set the database name, Username, Password, and DatabaseServiceName, regardless of which variant is used:

- MYSQL_USER appuio
- MYSQL_PASSWORD appuio
- MYSQL_DATABASE appuio
- DATABASE_SERVICE_NAME mysql

### CLI

Using the CLI, the MySQL service can be created as follows:

```
$ oc new-app mysql-ephemeral \
     -pMEMORY_LIMIT=128Mi \
     -pMYSQL_USER=appuio -pMYSQL_PASSWORD=appuio \
     -pMYSQL_DATABASE=appuio -pDATABASE_SERVICE_NAME=mysql
```

### Web Console

In the web console, the MySQL service can be added to the project via "Add to Project".
![MySQLService](../images/lab_8_addmysql_service.png)


## Task: LAB8.2: Connect the application to the database

By default, a H2 memory database is used for our example-spring-boot application. This can be changed to our new MySQL service by setting the following environment variables:

- SPRING_DATASOURCE_USERNAME appuio
- SPRING_DATASOURCE_PASSWORD appuio
- SPRING_DATASOURCE_DRIVER_CLASS_NAME com.mysql.jdbc.Driver
- SPRING_DATASOURCE_URL jdbc: mysql://[MySQL service address]/appuio?AutoReconnect=true

For the address of the MySQL service, we can use either its cluster IP (`oc get service`) or its DNS name (` <service> `). All services and pods within a project can be resolved via DNS.

For example, the value for the variable SPRING_DATASOURCE_URL is:
```
Name of service: mysql

Jdbc: mysql://mysql/appuio?AutoReconnect=true
```

We can now set these environment variables in the DeploymentConfig example-spring-boot. After the **ConfigChange** (ConfigChange is registered as a trigger in the DeploymentConfig), the application is automatically deployed again. Because of the new environment variables the application connects to the MySQL DB and [Liquibase](http://www.liquibase.org/) creates the schema and imports the test data.

**Note:** Liquibase is open source. It is a database independent library to manage database changes and to apply them to the database. Liquibase recognizes when starting the application, whether DB changes have to be applied to the database or not. See Logs.


```
SPRING_DATASOURCE_URL = jdbc: mysql://mysql/appuio?AutoReconnect=true
```

**Note:** mysql solves the cluster IP of the MySQL service within its project via DNS query. The MySQL database is only available within the project. The service is also available by the following name:

```
Projectname = techlab-dockerimage

Mysql.techlab-dockerimage.svc.cluster.local
```

Command for setting the environment variables:
```
 $ oc env dc example-spring-boot \
      -e SPRING_DATASOURCE_URL=jdbc: mysql://mysql/appuio?AutoReconnect=true \
      -e SPRING_DATASOURCE_USERNAME=appuio -e SPRING_DATASOURCE_PASSWORD=appuio \
      -e SPRING_DATASOURCE_DRIVER_CLASS_NAME=com.mysql.jdbc.Driver
```

You can use the following command to view DeploymentConfig as JSON. New, the Config also contains the set environment variables:

```
 $ oc get dc example-spring-boot -o json
```

```
...
 "Env": [
	        {
	            "Name": "SPRING_DATASOURCE_USERNAME",
	            "Value": "appuio"
	        },
	        {
	            "Name": "SPRING_DATASOURCE_PASSWORD",
	            "Value": "appuio"
	        },
	        {
	            "Name": "SPRING_DATASOURCE_DRIVER_CLASS_NAME",
	            "Value": "com.mysql.jdbc.driver"
	        },
	        {
	            "Name": "SPRING_DATASOURCE_URL",
	            "Value": "jdbc: mysql://mysql/appuio"
	        }
	    ],
...
```

The configuration can also be viewed and changed in the web console:

(Applications → Deployments → example-spring-boot, Actions, Edit YAML)

## Task: LAB8.3: Log into MySQL Service Pod and connect to DB manually

As described in the Lab [07](07_troubleshooting_ops.md), you can log into a pod using `oc rsh [POD]`:
```
$ oc get pods
NAME READY STATUS RESTARTS AGE
Example-spring-boot-8-wkros 1/1 Running 0 10m
Mysql-1-diccy 1/1 Running 0 50m

```

Then log into the MySQL Pod:
```
$ oc rsh mysql-1-diccy
```

Now you can connect to the database using mysql tool and display the tables:
```
$ mysql -u $ MYSQL_USER -p $ MYSQL_PASSWORD -h $ MYSQL_SERVICE_HOST appuio
Welcome to the MySQL monitor. Commands end with; Or \g.
Your MySQL connection id is 54
Server version: 5.6.26 MySQL Community Server (GPL)

Copyright (c) 2000, 2015, Oracle and / or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and / or its affiliates
Affiliates. Other names may be trademarks of their respective companies
Links.

Type 'help;' Or '\h' for help. Type '\c' to clear the current input statement.

Mysql>
```

Then you can use

```
showtables;
```

Display all tables.


## Task: LAB8.4: Import dump to MySQL DB

The task is to put the [dump](https://raw.githubusercontent.com/appuio/techlab/lab-3.3/labs/data/08_dump/dump.sql) into the MySQL pod.


**Tip:** Use `oc rsync` to copy local files to a pod. **Attention:** Note that the rsync command of the operating system is used. On UNIX systems, rsync can be installed with the package manager, on Windows, for example, [cwRsync](https://www.itefix.net/cwrsync). If an installation of rsync is not possible, you can, for example, log into the pod and download the dump via `curl -O <URL>`.

**Tip:** Use the mysql tool to load the dump.

**Tip** The existing database must be empty beforehand. It can also be deleted and re-created.


---

## Solution: LAB8.4

Sync an entire directory (dump). This includes the file `dump.sql`. Also note the rsync command above.

```
oc rsync ./labs/data/08_dump mysql-1-diccy:/tmp/
```

Log in to the MySQL Pod:

```
$ oc rsh mysql-1-diccy
```

Delete existing database:

```
$ mysql -u $ MYSQL_USER -p $ MYSQL_PASSWORD -h $ MYSQL_SERVICE_HOST appuio
...
Mysql> drop database appuio;
Mysql> create database appuio;
```

Insert dump:

```
$ mysql -u $ MYSQL_USER -p $ MYSQL_PASSWORD -h $ MYSQL_SERVICE_HOST appuio </tmp/08_dump/dump.sql
```

**Note:** The dump can be created as follows:

```
mysqldump --user=$MYSQL_USER --password=$MYSQL_PASSWORD --host=$MYSQL_SERVICE_HOST appuio > /tmp/dump.sql
```


---

**End Lab 8**

<p width = "100px" align = "right"> <a href="09_dockerbuild_webhook.md"> Directly integrate code changes via Webhook → </a> </p>
[← back to overview] (../README.md)
