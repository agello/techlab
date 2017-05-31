# Lab 6: Pod Scaling, Readiness Probe and Self Healing

In this lab we will show you how to scale applications in OpenShift. We also show how OpenShift ensures that the number of expected pods is started and how an application can report back to the platform that it is ready for requests.

## Example High-scale application

We are creating a new project

```
$ oc new-project [USER]-scale
```

And add an application to the project

```
$ oc new-app https://github.com/agello/example-php-sti-helloworld.git --name=agello-php-sti-example
```

And provide the service (expose)

```
$ oc expose svc agello-php-sti-example
```

If we want to scale our Example application, we must tell our ReplicationController (rc) that we want to have 3 replicas of the image running at the same time.

Let's take a closer look at the ReplicationController (rc):

```
$ oc get rc
NAME                       DESIRED   CURRENT   AGE
agello-php-sti-example     1         1         1m
```

For more details:

```
$ oc get rc agello-php-sti-example   -o json
```

The rc tells us how many pods we expect (spec) and how many are currently deployed (status).

## Task: LAB6.1 scale our sample application
Now we scale our Example application on 3 replicas:

```
$ oc scale --replicas=3 dc agello-php-sti-example
```

Let us check the number of replicas on the ReplicationController:

```
$ oc get rc
NAME                       DESIRED   CURRENT   AGE
agello-php-sti-example   3         3         7m
```

And accordingly indicate the pods:

```
$ oc get pods
NAME                             READY     STATUS      RESTARTS   AGE
agello-php-sti-example-1-0hj9l   1/1       Running     0          59s
agello-php-sti-example-1-build   0/1       Completed   0          9m
agello-php-sti-example-1-c82f3   1/1       Running     0          59s
agello-php-sti-example-1-dlkjs   1/1       Running     0          8m
```

Finally, look at the service. This should now reference all three endpoints:

```
$ oc describe svc agello-php-sti-example
Name:			agello-php-sti-example
Namespace:		tomcc-scale
Labels:			app=agello-php-sti-example
Selector:		app=agello-php-sti-example,deploymentconfig=agello-php-sti-example
Type:			ClusterIP
IP:			172.30.226.111
Port:			8080-tcp	8080/TCP
Endpoints:		172.16.10.6:8080,172.16.8.6:8080,172.16.8.9:8080
Session Affinity:	None
No events.
```

Scaling pods within a service is very fast as OpenShift simply starts a new instance of the docker image as a container.

**Tip:** OpenShift V3 also supports autocaling, the documentation can be found at the following link: https://docs.openshift.com/container-platform/3.3/dev_guide/pod_autoscaling.html

## Task: LAB6.2 scaled app in the web console

Look at the scaled application in the Web Console.

## Check interruption-free scaling

Use the browser refresh on your application to check if your service is available while you scale up and down from the web console.

The requests are routed to the different pods, once you are scaled down to 1 pod, then only pod gives a response

What happens if we start a new deployment 

In our example, we use a very lightweight pod, so redeployment does not impact the service. Interruptions due to deployments are more pronounced if the container takes longer until it can process requests. For example, Java application of LAB 4: **Startup: 30 seconds**

```
pod: example-spring-boot-2-73aln TIME: 16: 48: 25,251
pod: example-spring-boot-2-73aln TIME: 16: 48: 26.305
pod: example-spring-boot-2-73aln TIME: 16: 48: 27.400
pod: example-spring-boot-2-73aln TIME: 16: 48: 28.463
pod: example-spring-boot-2-73aln TIME: 16: 48: 29.507
<html> <body> <h1> 503 Service Unavailable </ h1>
No server is available to handle this request.
</ Body> </ html>
 TIME: 16: 48: 33.562
<html> <body> <h1> 503 Service Unavailable </ h1>
No server is available to handle this request.
</ Body> </ html>
 TIME: 16: 48: 34.601
 ...
pod: example-spring-boot-3-tjdkj TIME: 16: 49: 20,114
pod: example-spring-boot-3-tjdkj TIME: 16: 49: 21.181
pod: example-spring-boot-3-tjdkj TIME: 16: 49: 22.231

```

It may even be that the service is no longer online and the routing layer returns a **503 Error**.

The following chapter describes how to configure your services to enable uninterrupted deployments.


## Interruption-free deployment using Readiness Probe and Rolling Update

The update strategy [Rolling](https://docs.openshift.com/container-platform/3.3/dev_guide/deployments/deployment_strategies.html#rolling-strategy) allows interruption-free deployments. This will start the new version of the application as soon as the application is ready, Request will be routed to the new pod, and the old version will be undeployed.

In addition, using [Container Health Checks](https://docs.openshift.com/container-platform/3.3/dev_guide/application_health.html), the deployed application of the platform can provide detailed feedback on its current state.

Basically, there are two checks that can be implemented:

- Liveness Probe, says whether a running container is still running clean.
- Readiness Probe, provides feedback on whether an application is ready to receive requests. Is especially relevant in the Rolling Update.

These two checks can be implemented as HTTP Check, Container Execution Check (Shell Script in Container) or TCP Socket Check.

In our example, the platform application is to say whether it is ready for requests. For this, we use the Readiness Probe. Our example application returns a status code 200 on port 9000 (management port of the spring application) as soon as the application is ready.

```
http://[route]/health/
```

## Task: LAB6.3

In the Deployment Config (dc), in the Rolling Update Strategy section, you define that the app should always be available during an update: ``maxUnavailable: 0%``

This can be configured in the Deployment Config (dc):

**YAML:**
```
...
Spec:
  Strategy:
    Type: Rolling
    RollingParams:
      UpdatePeriodSeconds: 1
      IntervalSeconds: 1
      TimeoutSeconds: 600
      MaxUnavailable: 0%
      MaxSurge: 25%
    Resources: {}
...
```

The Deployment Config can be edited via Web Console (Applications → Deployments → example-php-docker-helloworld, edit) or directly via `oc`.

```
$ oc edit dc agello-php-sti-example
```

Or edit in JSON format:

```
$ oc edit dc agello-php-sti-example -o json
```

** json **
```
"Strategy": {
    "Type": "rolling",
    "RollingParams": {
          "UpdatePeriodSeconds": 1,
          "IntervalSeconds": 1,
          "TimeoutSeconds": 600,
          "MaxUnavailable": "0%",
          "MaxSurge": "25%"
    },
    "Resources": {}
}

```

The Readiness Probe must be added to the Deployment Config (dc) at:

Spec->template->spec->containers under *resources: {}* 

**YAML:**

```
      - image: 172.30.87.171:5000/tomcc-scale/agello-php-sti-example@sha256:5e7d2b47e6cab2cf30be06fe8ee0ca72520a9d3be3a81c64835fb9bb5996c792
        imagePullPolicy: Always
        name: agello-php-sti-example
        ports:
        - containerPort: 8080
          protocol: TCP
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
```



Adjust this accordingly as above.

Verify during a deployment of the application whether an update of the application now runs without interruption through the web console:


Starting Deployment:

```
$ oc deploy agello-php-sti-example --latest
```


## Self Healing

Through the Replication Controller, we have now told the platform that **n** replicas are to run. What happens if we delete a pod?

Use ``oc get pods`` to find a pod in the status "running", which you can *kill*.

Start the following command in a separate terminal (display the changes to pods)

```
oc get pods -w
```

In the other terminal, delete a pod with the following command

```
oc delete pod agello-php-sti-example-3-q4vz9
```

OpenShift ensures that again **n** replicas of the mentioned pod are running.


---

**End Lab 6**

<p width = "100px" align = "right"> <a href="07_troubleshooting_ops.md"> Troubleshooting what is in the pod? → </a> </p>

[← back to overview](../README.md)
