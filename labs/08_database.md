# Lab 8: Connect the database

Most applications are in some way stateful and save data persistently. Be this in a database or as files on a file system or ObjectStore. In this lab, we will create a MySQL service in our project and connect it to our application so that several applicationpods can access the same database.

For this example, the SpringBoot sample is taken from [LAB 4](04_deploy_dockerimage.md), `[USER]-dockerimage`

## Task: LAB8.1: Create MySQL service

For our example, we use an OpenShift template in this lab, which creates a MySQL database with EmptyDir Data Storage. This is to be used only for test environments, since all data will be lost when restarting the MySQL Pod. In production, we will attach a Persistent Volume (mysql-persistent) to the MySQL database. As a result, the data remains between restarts and is thus suitable for productive operation.

We can create the Mariadb service via the web console as well as via the CLI.

To get the result, you just have to set the database name, Username, Password, and DatabaseServiceName, regardless of which variant is used:

- MYSQL_USER agello
- MYSQL_PASSWORD agello
- MYSQL_DATABASE agello
- DATABASE_SERVICE_NAME mysql

### CLI

Using the CLI, the Mariadb service can be created as follows:

```
$ oc new-app mariadb-ephemeral \
     -pMEMORY_LIMIT=256Mi \
     -pMYSQL_USER=agello -pMYSQL_PASSWORD=agello \
     -pMYSQL_DATABASE=agello -pDATABASE_SERVICE_NAME=mariadb
```

### Web Console

In the web console, the Mariadb service can be added to the project via "Add to Project".
![MariadbService](../images/lab_8_addmysql_service.png)


## Task: LAB8.2: Connect the application to the database

By default, a H2 memory database is used for our example-spring-boot application. This can be changed to our new MySQL service by setting the following environment variables:

- SPRING_DATASOURCE_USERNAME agello
- SPRING_DATASOURCE_PASSWORD agello
- SPRING_DATASOURCE_DRIVER_CLASS_NAME com.mysql.jdbc.Driver
- SPRING_DATASOURCE_URL jdbc: mysql://[MySQL service address]/agello?AutoReconnect=true

For the address of the MySQL service, we can use either its cluster IP (`oc get service`) or its DNS name (`<service>`). All services and pods within a project can be resolved via DNS.

For example, the value for the variable SPRING_DATASOURCE_URL is:
```
Name of service: mysql

Jdbc: mysql://mariadb/agello?AutoReconnect=true
```

We can now set these environment variables in the DeploymentConfig example-spring-boot. After the **ConfigChange** (ConfigChange is registered as a trigger in the DeploymentConfig), the application is automatically deployed again. Because of the new environment variables the application connects to the MySQL DB and [Liquibase](http://www.liquibase.org/) creates the schema and imports the test data.

**Note:** Liquibase is open source. It is a database independent library to manage database changes and to apply them to the database. Liquibase recognizes when starting the application, whether DB changes have to be applied to the database or not. See Logs.


```
SPRING_DATASOURCE_URL = jdbc:mysql://mysql/agello?AutoReconnect=true
```

**Note:** mysql solves the cluster IP of the MySQL service within its project via DNS query. The MySQL database is only available within the project. The service is also available by the following name:

```
Projectname = techlab-dockerimage

Mysql.techlab-dockerimage.svc.cluster.local
```

Command for setting the environment variables:
```
 $ oc env dc example-spring-boot \
      -e SPRING_DATASOURCE_URL=jdbc:mysql://mariadb/agello?AutoReconnect=true \
      -e SPRING_DATASOURCE_USERNAME=agello -e SPRING_DATASOURCE_PASSWORD=agello \
      -e SPRING_DATASOURCE_DRIVER_CLASS_NAME=com.mysql.jdbc.Driver
```

You can use the following command to view DeploymentConfig as JSON. New, the Config also contains the set environment variables:

```
 $ oc get dc example-spring-boot -o json
```

```
...
                        "env": [
                            {
                                "name": "SPRING_DATASOURCE_URL",
                                "value": "jdbc:mysql://mariadb/agello?AutoReconnect=true"
                            },
                            {
                                "name": "SPRING_DATASOURCE_USERNAME",
                                "value": "agello"
                            },
                            {
                                "name": "SPRING_DATASOURCE_PASSWORD",
                                "value": "agello"
                            },
                            {
                                "name": "SPRING_DATASOURCE_DRIVER_CLASS_NAME",
                                "value": "com.mysql.jdbc.Driver"
                            }
```

The configuration can also be viewed and changed in the web console:

(Applications → Deployments → example-spring-boot, Actions, Edit YAML)

## Task: LAB8.3: Log into MySQL Service Pod and connect to DB manually

As described in the Lab [07](07_troubleshooting_ops.md), you can log into a pod using `oc rsh [POD]`:
```
$ oc get pods
NAME READY STATUS RESTARTS AGE
Example-spring-boot-8-wkros 1/1 Running 0 10m
mariadb-1-b6rgb             1/1 Running 0 7m

```

Then log into the MySQL Pod:
```
$ oc rsh mariadb-1-b6rgb
```

Now you can connect to the database using mysql tool and display the tables:
```
sh-4.2$ mysql -u$MYSQL_USER -p$MYSQL_PASSWORD -h$MYSQL_SERVICE_HOST agello
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 226
Server version: 10.1.19-MariaDB MariaDB Server

Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 

```

Then you can use

```
show tables;
```

Display all tables.


## Task: LAB8.4: Import dump to MySQL DB

The task is to put the [dump](https://raw.githubusercontent.com/agello/techlab/lab-3.3/labs/data/08_dump/dump.sql) into the MySQL pod.


**Tip:** Use `oc rsync` to copy local files to a pod. **Attention:** Note that the rsync command of the operating system is used. On UNIX systems, rsync can be installed with the package manager, on Windows, for example, [cwRsync](https://www.itefix.net/cwrsync). If an installation of rsync is not possible, you can, for example, log into the pod and download the dump via `curl -O <URL>`.

**Tip:** Use the mysql tool to load the dump.

**Tip** The existing database must be empty beforehand. It can also be deleted and re-created.


---

## Solution: LAB8.4

Sync an entire directory (dump). This includes the file `dump.sql`. Also note the rsync command above.

```
oc rsync ./labs/data/08_dump mariadb-1-b6rgb:/tmp/
```

Log in to the MySQL Pod:

```
$ oc rsh mariadb-1-b6rgb
```

Insert dump:

```
$ mysql -u$MYSQL_USER -p$MYSQL_PASSWORD -h$mariadb agello </tmp/08_dump/dump.sql
```

**Note:** The dump can be created as follows:

```
mysqldump --user=$MYSQL_USER --password=$MYSQL_PASSWORD --host=mariadb agello > /tmp/dump.sql
```


---

**End Lab 8**

<p width = "100px" align = "right"> <a href="09_dockerbuild_webhook.md"> Directly integrate code changes via Webhook → </a> </p>


[← back to overview](../README.md)
