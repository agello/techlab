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
total 12
drwxr-xr-x. 7 default root  106 May 29 09:25 .
drwxr-xr-x. 4 default root   57 May 29 09:25 ..
drwxr-xr-x. 6 default root   61 May 29 09:24 .gradle
drwxr-xr-x. 3 default root   19 Feb  1 12:35 .pki
drwxr-xr-x. 9 default root  120 May 29 09:25 build
-rw-r--r--. 1 root    root 1145 May 29 09:22 build.gradle
drwxr-xr-x. 3 root    root   21 May 29 09:24 gradle
-rwxr-xr-x. 1 root    root 4971 May 29 09:22 gradlew
drwxr-xr-x. 4 root    root   30 May 29 09:24 src
```

## Task: LAB7.2

Individual commands within the container can be executed using `oc exec`:

```
$ oc exec [POD] env
```


```
$ oc exec example-spring-boot-4-8mbwe env
PATH=/opt/app-root/src/bin:/opt/app-root/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=example-spring-boot-1-nkhw7
KUBERNETES_PORT_53_TCP_ADDR=172.30.0.1
EXAMPLE_SPRING_BOOT_SERVICE_HOST=172.30.158.17
EXAMPLE_SPRING_BOOT_SERVICE_PORT_8080_TCP=8080
EXAMPLE_SPRING_BOOT_PORT=tcp://172.30.158.17:8080
EXAMPLE_SPRING_BOOT_PORT_8080_TCP_PROTO=tcp
KUBERNETES_SERVICE_HOST=172.30.0.1
KUBERNETES_PORT=tcp://172.30.0.1:443
EXAMPLE_SPRING_BOOT_PORT_8080_TCP_ADDR=172.30.158.17
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_53_UDP_PROTO=udp
KUBERNETES_PORT_53_UDP_PORT=53
EXAMPLE_SPRING_BOOT_SERVICE_PORT=8080
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_SERVICE_PORT_DNS_TCP=53
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_53_UDP=udp://172.30.0.1:53
KUBERNETES_PORT_53_TCP_PROTO=tcp
KUBERNETES_PORT_53_TCP_PORT=53
KUBERNETES_PORT_53_TCP=tcp://172.30.0.1:53
EXAMPLE_SPRING_BOOT_PORT_8080_TCP=tcp://172.30.158.17:8080
EXAMPLE_SPRING_BOOT_PORT_8080_TCP_PORT=8080
KUBERNETES_SERVICE_PORT_DNS=53
KUBERNETES_PORT_443_TCP=tcp://172.30.0.1:443
KUBERNETES_PORT_443_TCP_ADDR=172.30.0.1
KUBERNETES_PORT_53_UDP_ADDR=172.30.0.1
STI_SCRIPTS_URL=image:///usr/libexec/s2i
STI_SCRIPTS_PATH=/usr/libexec/s2i
HOME=/opt/app-root/src
BASH_ENV=/opt/app-root/etc/scl_enable
ENV=/opt/app-root/etc/scl_enable
PROMPT_COMMAND=. /opt/app-root/etc/scl_enable
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

Activity: Access the Springboot Metrics from [Lab 4](04_deploy_dockerimage.md).

```
$ oc get pod 
NAME                          READY     STATUS    RESTARTS   AGE
example-spring-boot-1-nkhw7   1/1       Running   0          1h
$ oc port-forward example-spring-boot-1-nkhw7 9000:9000 
Forwarding from 127.0.0.1:9000 -> 9000
Forwarding from [::1]:9000 -> 9000
```

Do not forget to change the pod name to your own installation. Autocompletion can be used if installed.

The metrics can now be viewed as the following: [http://localhost:9000/metrics/](http://localhost:9000/metrics/) The metrics are displayed as json. With the same concept, you can now connect to a database using your local SQL client.

For more information on port forwarding, please see the following link: https://docs.openshift.com/container-platform/3.3/dev_guide/port_forwarding.html

---

**End Lab 7**

<p width = "100px" align = "right"> <a href="08_database.md"> Deploy and bind the database → </a> </p>

[← back to overview](../README.md)
