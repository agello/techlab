# Lab 6: Pod Scaling, Readiness Probe and Self Healing

In this lab we will show you how to scale applications in OpenShift. We also show how OpenShift ensures that the number of expected pods is started and how an application can report back to the platform that it is ready for requests.

## Example High-scale application

We are creating a new project

`` `
$ Oc new-project [USER] -scale
`` `

And add an application to the project

`` `
$ Oc new-app appuio / example-php-docker-helloworld --name = appuio-php-docker
`` `

And provide the service (expose)

`` `
$ Oc expose service appuio php docker
`` `

If we want to scale our Example application, we must tell our ReplicationController (rc) that we want to have 3 replicas of the image running at the same time.

Let's take a closer look at the ReplicationController (rc):

`` `
$ Oc get rc

NAME DESIRED CURRENT AGE
Appuio-php-docker-1 1 1 33s
`` `

For more details:

`` `
$ Oc get rc appuio-php-docker-1 -o json
`` `

The rc tells us how many pods we expect (spec) and how many are currently deployed (status).

## Task: LAB6.1 scale our sample application
Now we scale our Example application on 3 replicas:

`` `
$ Oc scale --replicas = 3 rc appuio-php-docker-1
`` `

Let us check the number of replicas on the ReplicationController:

`` `
$ Oc get rc

NAME DESIRED CURRENT AGE
Appuio-php-docker-1 3 3 1m

`` `

And accordingly indicate the pods:

`` `
$ Oc get pods
NAME READY STATUS RESTARTS AGE
Appuio-php-docker-1-2uc89 1/1 Running 0 21s
Appuio-php-docker-1-evcre 1/1 Running 0 21s
Appuio-php-docker-1-tolpx 1/1 Running 0 2m

`` `

Finally, look at the service. This should now reference all three endpoints:
`` `
$ Oc describe svc appuio php docker
Name: appuio-php-docker
Namespace: techlab-scale
Labels: app = appuio-php-docker
Selector: app = appuio-php-docker, deploymentconfig = appuio-php-docker
Type: ClusterIP
IP: 172.30.166.88
Port: 8080-tcp 8080 / TCP
Endpoints: 10.1.3.23:8080,10.1.4.13:8080,10.1.5.15:8080
Session Affinity: None
No events.

`` `

Scaling pods within a service is very fast as OpenShift simply starts a new instance of the docker image as a container.

** Tip: ** OpenShift V3 also supports autocaling, the documentation can be found at the following link: https://docs.openshift.com/container-platform/3.3/dev_guide/pod_autoscaling.html

## Task: LAB6.2 scaled app in the web console

Look at the scaled application in the Web Console.

## Check interruption-free scaling

Use the following command to check if your service is available while you scale up and down.
To do this, replace `[route]` with your defined route:

** Tip: ** oc get route

`` `
While true; Do sleep 1; Curl -s http: // [route] / pod /; Date "+ TIME:% H:% M:% S,% 3N"; Done
`` `

Or in PowerShell (Attention: only from PowerShell version 3.0!):

`` `
While (1) {
	Start-Sleep -s 1
	Invoke-RestMethod http: // [route] / pod /
	Get-Date-Format "+ TIME:% H:% M:% S,% 3N"
}
`` `

And scale from ** 3 ** replicas to ** 1 **.
The output shows the pod that processed the request:

`` `
POD: appuio-php-docker-6-9w9t4 TIME: 16: 40: 04.991
POD: appuio-php-docker-6-9w9t4 TIME: 16: 40: 06,053
POD: appuio-php-docker-6-6xg2b TIME: 16: 40: 07,091
POD: appuio-php-docker-6-6xg2b TIME: 16: 40: 08.128
POD: appuio-php-docker-6-ctbrs TIME: 16: 40: 09.175
POD: appuio-php-docker-6-ctbrs TIME: 16: 40: 10.212
POD: appuio-php-docker-6-9w9t4 TIME: 16: 40: 11.279
POD: appuio-php-docker-6-9w9t4 TIME: 16: 40: 12.332
POD: appuio-php-docker-6-6xg2b TIME: 16: 40: 13.369
POD: appuio-php-docker-6-6xg2b TIME: 16: 40: 14.407
POD: appuio-php-docker-6-6xg2b TIME: 16: 40: 15.441
POD: appuio-php-docker-6-6xg2b TIME: 16: 40: 16.493
POD: appuio-php-docker-6-6xg2b TIME: 16: 40: 17.543
POD: appuio-php-docker-6-6xg2b TIME: 16: 40: 18.591
`` `
The requests are routed to the different pods, once you are runters scaled to a pod, then only one response

What happens if we start a new deployment while the While command is running:

`` `
$ Oc deploy appuio-php-docker --latest
`` `
In a short time, the public route does not answer
`` `
POD: appuio-php-docker-6-6xg2b TIME: 16: 42: 17.743
POD: appuio-php-docker-6-6xg2b TIME: 16: 42: 18.776
POD: appuio-php-docker-6-6xg2b TIME: 16: 42: 19.813
POD: appuio-php-docker-6-6xg2b TIME: 16: 42: 20.853
POD: appuio-php-docker-6-6xg2b TIME: 16: 42: 21.891
POD: appuio-php-docker-6-6xg2b TIME: 16: 42: 22.943
POD: appuio-php-docker-6-6xg2b TIME: 16: 42: 23.980
# No Answer
POD: appuio-php-docker-7-pxnr3 TIME: 16: 42: 42.134
POD: appuio-php-docker-7-pxnr3 TIME: 16: 42: 43.181
POD: appuio-php-docker-7-pxnr3 TIME: 16: 42: 44.226
POD: appuio-php-docker-7-pxnr3 TIME: 16: 42: 45.259
POD: appuio-php-docker-7-pxnr3 TIME: 16: 42: 46.297
POD: appuio-php-docker-7-pxnr3 TIME: 16: 42: 47.571
POD: appuio-php-docker-7-pxnr3 TIME: 16: 42: 48.606
POD: appuio-php-docker-7-pxnr3 TIME: 16: 42: 49.645
POD: appuio-php-docker-7-pxnr3 TIME: 16: 42: 50.684
`` `

In our example, we use a very lightweight pod. The behavior is more pronounced if the container takes longer until it can process requests. For example, Java application of LAB 4: ** Startup: 30 seconds **

`` `
Pod: example-spring-boot-2-73aln TIME: 16: 48: 25,251
Pod: example-spring-boot-2-73aln TIME: 16: 48: 26.305
Pod: example-spring-boot-2-73aln TIME: 16: 48: 27.400
Pod: example-spring-boot-2-73aln TIME: 16: 48: 28.463
Pod: example-spring-boot-2-73aln TIME: 16: 48: 29.507
<Html> <body> <h1> 503 Service Unavailable </ h1>
No server is available to handle this request.
</ Body> </ html>
 TIME: 16: 48: 33.562
<Html> <body> <h1> 503 Service Unavailable </ h1>
No server is available to handle this request.
</ Body> </ html>
 TIME: 16: 48: 34.601
 ...
Pod: example-spring-boot-3-tjdkj TIME: 16: 49: 20,114
Pod: example-spring-boot-3-tjdkj TIME: 16: 49: 21.181
Pod: example-spring-boot-3-tjdkj TIME: 16: 49: 22.231

`` `

It may even be that the service is no longer online and the routing layer returns a ** 503 Error **.

The following chapter describes how to configure your services to enable uninterrupted deployments.


## Interruption-free deployment using Readiness Probe and Rolling Update

The update strategy [Rolling] (https://docs.openshift.com/container-platform/3.3/dev_guide/deployments/deployment_strategies.html#rolling-strategy) allows interruption-free deployments. This will start the new version of the application as soon as the application is ready, Request will be routed to the new pod, and the old version will be undeployed.

In addition, using [Container Health Checks] (https://docs.openshift.com/container-platform/3.3/dev_guide/application_health.html), the deployed application of the platform can provide detailed feedback on its current state.

Basically, there are two checks that can be implemented:

- Liveness Probe, says whether a running container is still running clean.
- Readiness Probe, provides feedback on whether an application is ready to receive requests. Is especially relevant in the Rolling Update.

These two checks can be implemented as HTTP Check, Container Execution Check (Shell Script in Container) or TCP Socket Check.

In our example, the platform application is to say whether it is ready for requests. For this, we use the Readiness Probe. Our example application returns a status code 200 on port 9000 (management port of the spring application) as soon as the application is ready.
`` `
Http: // [route] / health /
`` `

## Task: LAB6.3

In the Deployment Config (dc), in the Rolling Update Strategy section, you define that the app should always be available during an update: `maxUnavailable: 0%`

This can be configured in the Deployment Config (dc):

** YAML: **
`` `
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
`` `

The Deployment Config can be edited via Web Console (Applications → Deployments → example-php-docker-helloworld, edit) or directly via `oc`.
`` `
$ Oc edit dc appuio-php-docker
`` `

Or edit in JSON format:
`` `
$ Oc edit dc appuio-php-docker -o json
`` `
** json **
`` `
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

`` `

The Readiness Probe must be added to the Deployment Config (dc) at:

Spec -> template -> spec -> containers under 'resources: {} `

** YAML: **

`` `
...
          Resources: {}
          ReadinessProbe:
            HttpGet:
              Path: / health /
              Port: 8080
              Scheme: HTTP
            InitialDelaySeconds: 10
            TimeoutSeconds: 1
...
`` `

** json: **
`` `
...
                        "Resources": {},
                        "ReadinessProbe": {
                            "HttpGet": {
                                "Path": "/ health /",
                                "Port": 8080,
                                "Scheme": "HTTP"
                            },
                            "InitialDelaySeconds": 10,
                            "TimeoutSeconds": 1
                        },
...
`` `

Adjust this accordingly as above.

The configuration under Container must then look as follows:
** YAML: **
`` `
      Containers:
        -
          Name: example-php-docker-helloworld
          Image: 'appuio / example-php-docker-helloworld @ sha256: 6a19d4a1d868163a402709c02af548c80635797f77f25c0c391b9ce8cf9a56cf'
          Ports:
            -
              ContainerPort: 8080
              Protocol: TCP
          Resources: {}
          ReadinessProbe:
            HttpGet:
              Path: / health /
              Port: 8080
              Scheme: HTTP
            InitialDelaySeconds: 10
            TimeoutSeconds: 1
          TerminationMessagePath: / dev / termination-log
          ImagePullPolicy: IfNotPresent
`` `

** json: **
`` `
                "Containers": [
                    {
                        "Name": "appuio-php-docker",
                        "Image": "appuio / example-php-docker-helloworld @ sha256: 9e927f9d6b453f6c58292cbe79f08f5e3db06ac8f0420e22bfd50c750898c455",
                        "Ports": [
                            {
                                "ContainerPort": 8080,
                                "Protocol": "TCP"
                            }
                        ],
                        "Resources": {},
                        "ReadinessProbe": {
                            "HttpGet": {
                                "Path": "/ health /",
                                "Port": 8080,
                                "Scheme": "HTTP"
                            },
                            "InitialDelaySeconds": 10,
                            "TimeoutSeconds": 1
                        },
                        "TerminationMessagePath": "/ dev / termination-log",
                        "ImagePullPolicy": "Always"
                    }
                ],
`` `


Verify during a deployment of the application whether an update of the application now runs without interruption:

Once per second a request:
`` `
While true; Do sleep 1; Curl -s http: // [route] / pod /; Date "+ TIME:% H:% M:% S,% 3N"; Done
`` `

Starting Deployment:
`` `
$ Oc deploy appuio-php-docker --latest
`` `


## Self Healing

Through the Replication Controller, we have now told the platform that ** ** replicas are to run. What happens if we delete a pod?

Use `oc get pods` to find a pod in the status" running ", which you can * kill *.

Start the following command in a separate terminal (display the changes to pods)
`` `
Oc get pods -w
`` `
In the other terminal, delete a pod with the following command
`` `
Oc delete pod appuio-php-docker-3-788j5
`` `


OpenShift ensures that again ** n ** replicas of the mentioned pod are running.


---

** End Lab 6 **

<P width = "100px" align = "right"> <a href="07_troubleshooting_ops.md"> Troubleshooting what is in the pod? → </a> </ p>
[← back to overview] (../README.md)
