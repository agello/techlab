# Lab 7: Troubleshooting, what is in the pod?

This lab will show you how to deal with errors and troubleshooting, and which tools are available to you.

## Log in to container

We use the project from [Lab 4](04_deploy_dockerimage.md) `[USER]-dockerimage`

Running containers are treated as unalterable infrastructure and should not be modified in general. However, there are usecases where you have to log into the containers. For example, for debugging and analysis.

## Task: LAB7.1

With OpenShift, remote shells can be opened into the pods without having to install SSH beforehand. For this, the command `oc rsh` is available.

Select a pod using `oc get pods` and execute the following command:

```
$ oc rsh [POD]
```

You can now perform analyzes on the shell in the container:

```
bash-4.2 $ ls -la
Total 16
Drwxr-xr-x. 7 default root 99 May 16 13:35.
Drwxr-xr-x. 4 default root 54 May 16 13:36 ..
Drwxr-xr-x. 6 default root 57 May 16 13:35 .gradle
Drwxr-xr-x. 3 default root 18 May 16 12:26 .pki
Drwxr-xr-x. 9 default root 4096 May 16 13:35 build
-rw-r - r--. 1 root root 1145 May 16 13:33 build.gradle
Drwxr-xr-x. 3 root root 20 May 16 13:34 gradle
-rwxr-xr-x. 1 root root 4971 May 16 13:33 gradlew
Drwxr-xr-x. 4 root root 28 May 16 13:34 src
```

## Task: LAB7.2

Individual commands within the container can be executed using `oc exec`:

```
$ oc exec [POD] env
```


```
$ oc exec example-spring-boot-4-8mbwe env
PATH: /usr/local/bin/usr/sbin:/usr/bin:/sbin:/usr/bin
HOSTNAME = example-spring-boot-4-8mbwe
KUBERNETES_SERVICE_PORT_DNS_TCP = 53
KUBERNETES_PORT_443_TCP_PROTO = tcp
KUBERNETES_PORT_443_TCP_ADDR = 172.30.0.1
KUBERNETES_PORT_53_UDP_PROTO = udp
KUBERNETES_PORT_53_TCP = tcp://172.30.0.1: 53
...
```

## View log files

The log files for a pod can be displayed in the web console as well as in the CLI.

```
$ oc logs [POD]
```

The `-f` parameter has the same behavior as `tail -f`

If a pod is in the status **CrashLoopBackOff**, this means that it could not be started successfully even after repeated restarts. The logfiles can be displayed even if the pod is not running with the following command.

```
$ oc logs -p [POD]
```

With OpenShift an EFK (Elasticsearch, Fluentd, Kibana) stack is delivered, which collects, rotates and aggregates all log files. Kibana allows logs to be searched, filtered and graphically edited. For more information and an optional LAB, please see [here](../additional-labs/logging_efk_stack.md).


## Task: LAB7.3 Port forwarding

OpenShift 3 allows you to forward any ports from the development workstation to a pod. This is useful, for example, for accessing administrative consoles, databases, etc., which are not exposed to the Internet and which are otherwise unavailable. In contrast to OpenShift 2, the port forwarding is tunnelled via the same HTTPS connection that the OpenShift Client (oc) also uses. This also allows OpenShift 3 platforms to access when there are restrictive firewalls and / or proxies between the workstation and OpenShift.

Activity: Access the Springboot Metrics from [Lab 4] (04_deploy_dockerimage.md).

```
oc get po -namespace="[USER]-dockerimage"
oc port-forward example-spring-boot-1-xj1df 9000:9000 --namespace="[USER]-dockerimage"
```

Do not forget to change the pod name to your own installation. Autocompletion can be used if installed.

The metrics can now be viewed as the following: [http://localhost:9000/metrics/](http://localhost:9000/metrics/) The metrics are displayed as json. With the same concept, you can now connect to a database using your local SQL client.

For more information on port forwarding, please see the following link: https://docs.openshift.com/container-platform/3.3/dev_guide/port_forwarding.html

---

**End Lab 7**

<p width = "100px" align = "right"> <a href="08_database.md"> Deploy and bind the database → </a> </p>
[← back to overview] (../README.md)
