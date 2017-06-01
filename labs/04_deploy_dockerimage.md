# Lab 4: Deploying a Docker Image

In this lab, we will jointly deploy the first "pre-built" Docker image and take a closer look at the OpenShift concepts Pod, Service, DeploymentConfig and ImageStream.


## Task: LAB4.1

After deploying the source-to-image workflow to deploy an application to OpenShift, we are now ready to deploy a pre-built docker image from Docker Hub or another Docker hub, Registry too.

> [Further documentation](https://docs.openshift.com/container-platform/3.3/dev_guide/new_app.html#specifying-an-image)

The first step is to create a new project. A project is a grouping of resources (containers and docker images, pods, services, routes, configuration, quotas, limits, and more). Authorized users can manage these resources. Within an OpenShift V3 cluster, the name of a project must be unique.

Therefore, create a new project called `[USER]-dockerimage`:

``oc new-project [USER]-dockerimage``

`oc new-project` will automatically switch to the newly created project. The `oc get` command can display resources of a particular type.

use

``oc get project``

To view all the projects to which you are authorized.

Once the new project is created, we can deploy the Docker Image in OpenShift using the following command:

``oc new-app tomcc/example-spring-boot``

Output:
```
--> Found Docker image 0573eff (25 hours old) from Docker Hub for "tomcc/example-spring-boot"

    Agello Spring Boot App 
    ---------------------- 
    Example Spring Boot App

    Tags: builder, springboot

    * An image stream will be created as "example-spring-boot:latest" that will track this image
    * This image will be deployed in deployment config "example-spring-boot"
    * Port 8080/tcp will be load balanced by service "example-spring-boot"
      * Other containers can access this service through the hostname "example-spring-boot"

--> Creating resources with label app=example-spring-boot ...
    imagestream "example-spring-boot" created
    deploymentconfig "example-spring-boot" created
    service "example-spring-boot" created
--> Success
    Run 'oc status' to view your app.
```

For our Lab we use an Agello example (Java SpringBoot application):

- Docker Hub: https://hub.docker.com/r/tomcc/example-spring-boot/
- GitHub (Source): https://github.com/agello/example-spring-boot-helloworld

OpenShift provides the necessary resources, in this case Docker Hub downloads Docker Hub and then deployes the appropriate pod.

**Tip:** Use `oc status` to get an overview of the project.

Or use the `oc get` command with the` -w` parameter to continually display changes to the resources of type Pod (abort with ctrl + c):

``oc get pods -w``

Depending on the Internet connection or whether the image has already been downloaded on your OpenShift Node, this can take a while. Please check the current status of the deployment in the Web Console:

1. Log in to the Web Console
2. Select your project `[USER] -dockerimage`
3. Click Applications
4. Select Pods


**Tip** To create your own docker images for OpenShift, you should follow these best practices: https://docs.openshift.com/container-platform/3.3/creating_images/guidelines.html


## Viewing the created resources

When we first executed `oc new-app tomcc/example-spring-boot`, OpenShift created some resources for us in the background. These are required to deploy this docker image:

- [Service](https://docs.openshift.com/container-platform/3.3/architecture/core_concepts/pods_and_services.html#services)
- [ImageStream](https://docs.openshift.com/container-platform/3.3/architecture/core_concepts/builds_and_image_streams.html#image-streams)
- [DeploymentConfig](https://docs.openshift.com/container-platform/3.3/dev_guide/deployments/how_deployments_work.html)

### Service

[Services](https://docs.openshift.com/container-platform/3.3/architecture/core_concepts/pods_and_services.html#services) serve as an abstraction player, entry point and proxy / loadbalancer on the pods behind it within OpenShift. The service makes it possible to find and address a group of pods of the same type within OpenShift.

As an example, if an application instance of our example can no longer process the load alone, we can scale the application to, for example, three pods. OpenShift maps these as endpoints automatically to the service. Once the pods are ready, requests are automatically distributed to all three pods.

**Note:** The application can currently not be reached from the outside, the service is an OpenShift internal concept. In the following lab we will make the application publicly available.

Now let's take a closer look at our service:

``oc get services``

```
NAME                  CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
example-spring-boot   172.30.158.17   <none>        8080/TCP   6m
```


As you can see on the output, our service (example-spring-boot) is accessible via an IP and port (172.30.124.20:8080) **Note:** Your IP may be different.

**Note:** Service IPs remain the same for their lifetime.

Use the following command to read additional information about the service:

``oc get service example-spring-boot -o json``

```
{
    "kind": "Service",
    "apiVersion": "v1",
    "metadata": {
        "name": "example-spring-boot",
        "namespace": "tomcc-dockerimage",
        "selfLink": "/api/v1/namespaces/tomcc-dockerimage/services/example-spring-boot",
        "uid": "5fa5fd32-452a-11e7-a461-06aef59db650",
        "resourceVersion": "17467",
        "creationTimestamp": "2017-05-30T11:23:15Z",
        "labels": {
            "app": "example-spring-boot"
        },
        "annotations": {
            "openshift.io/generated-by": "OpenShiftNewApp"
        }
    },
    "spec": {
        "ports": [
            {
                "name": "8080-tcp",
                "protocol": "TCP",
                "port": 8080,
                "targetPort": 8080
            }
        ],
        "selector": {
            "app": "example-spring-boot",
            "deploymentconfig": "example-spring-boot"
        },
        "portalIP": "172.30.158.17",
        "clusterIP": "172.30.158.17",
        "type": "ClusterIP",
        "sessionAffinity": "None"
    },
    "status": {
        "loadBalancer": {}
    }
}
```

You can also use the corresponding command to display the details of a pod:

``oc get pod example-spring-boot-3-nwzku -o json``

**Note:** First, query the pod name from your project (`oc get pods`) and replace it in the upper command.
```
{
    "kind": "Pod",
    "apiVersion": "v1",
    "metadata": {
        "name": "example-spring-boot-1-nkhw7",
        "generateName": "example-spring-boot-1-",
        "namespace": "tomcc-dockerimage",
        "selfLink": "/api/v1/namespaces/tomcc-dockerimage/pods/example-spring-boot-1-nkhw7",
        "uid": "6178cce3-452a-11e7-a461-06aef59db650",
        "resourceVersion": "17540",
        "creationTimestamp": "2017-05-30T11:23:18Z",
        "labels": {
            "app": "example-spring-boot",
            "deployment": "example-spring-boot-1",
            "deploymentconfig": "example-spring-boot"
        },
        "annotations": {
            "kubernetes.io/created-by": "{\"kind\":\"SerializedReference\",\"apiVersion\":\"v1\",\"reference\":{\"kind\":\"ReplicationController\",\"namespace\":\"tomcc-dockerimage\",\"name\":\"example-spring-boot-1\",\"uid\":\"6008b742-452a-11e7-a461-06aef59db650\",\"apiVersion\":\"v1\",\"resourceVersion\":\"17488\"}}\n",
            "openshift.io/container.example-spring-boot.image.entrypoint": "[\"container-entrypoint\",\"/bin/sh\",\"-c\",\"java -Xmx64m -Xss1024k -jar /opt/app-root/springboots2idemo.jar\"]",
            "openshift.io/deployment-config.latest-version": "1",
            "openshift.io/deployment-config.name": "example-spring-boot",
            "openshift.io/deployment.name": "example-spring-boot-1",
            "openshift.io/generated-by": "OpenShiftNewApp",
            "openshift.io/scc": "restricted"
        }
    },
    "spec": {
        "volumes": [
            {
                "name": "default-token-XXXXX",
                "secret": {
                    "secretName": "default-token-XXXXX"
                }
            }
        ],
        "containers": [
            {
                "name": "example-spring-boot",
                "image": "tomcc/example-spring-boot@sha256:74e0b8a6347258d8523b14ca02bdd737cd0e11521d3ae26106cb11061cbb7c32",
                "ports": [
                    {
                        "containerPort": 8080,
                        "protocol": "TCP"
                    }
                ],
                "resources": {},
                "volumeMounts": [
                    {
                        "name": "default-token-XXXXXX",
                        "readOnly": true,
                        "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount"
                    }
                ],
                "terminationMessagePath": "/dev/termination-log",
                "imagePullPolicy": "Always",
                "securityContext": {
                    "capabilities": {
                        "drop": [
                            "KILL",
                            "MKNOD",
                            "SETGID",
                            "SETUID",
                            "SYS_CHROOT"
                        ]
                    },
                    "privileged": false,
                    "seLinuxOptions": {
                        "level": "s0:c8,c2"
                    },
                    "runAsUser": 1000060000
                }
            }
        ],
        "restartPolicy": "Always",
        "terminationGracePeriodSeconds": 30,
        "dnsPolicy": "ClusterFirst",
        "nodeSelector": {
            "role": "app"
        },
        "host": "ip-10-20-4-132.eu-west-1.compute.internal",
        "serviceAccountName": "default",
        "serviceAccount": "default",
        "nodeName": "ip-10-20-4-132.eu-west-1.compute.internal",
        "securityContext": {
            "seLinuxOptions": {
                "level": "s0:c8,c2"
            },
            "fsGroup": 1000060000
        },
        "imagePullSecrets": [
            {
                "name": "default-dockercfg-5mbjf"
            }
        ]
    },
    "status": {
        "phase": "Running",
        "conditions": [
            {
                "type": "Initialized",
                "status": "True",
                "lastProbeTime": null,
                "lastTransitionTime": "2017-05-30T11:23:18Z"
            },
            {
                "type": "Ready",
                "status": "True",
                "lastProbeTime": null,
                "lastTransitionTime": "2017-05-30T11:23:48Z"
            },
            {
                "type": "PodScheduled",
                "status": "True",
                "lastProbeTime": null,
                "lastTransitionTime": "2017-05-30T11:23:18Z"
            }
        ],
        "hostIP": "10.20.4.132",
        "podIP": "172.16.10.3",
        "startTime": "2017-05-30T11:23:18Z",
        "containerStatuses": [
            {
                "name": "example-spring-boot",
                "state": {
                    "running": {
                        "startedAt": "2017-05-30T11:23:48Z"
                    }
                },
                "lastState": {},
                "ready": true,
                "restartCount": 0,
                "image": "tomcc/example-spring-boot@sha256:74e0b8a6347258d8523b14ca02bdd737cd0e11521d3ae26106cb11061cbb7c32",
                "imageID": "docker-pullable://docker.io/tomcc/example-spring-boot@sha256:74e0b8a6347258d8523b14ca02bdd737cd0e11521d3ae26106cb11061cbb7c32",
                "containerID": "docker://e6fd1c792dc28f5f1aa817ec53a3185c17ea6f6231b00249553ae4bcbc3dc60e"
            }
        ]
    }
}
```

The `selector` area in the service defines which pods (`labels`) are used as endpoints. To do so, consider the corresponding configurations of the service and pod together.

```
Service:
--------
"Selector": {
    "App": "example-spring-boot",
    "Deploymentconfig": "example-spring-boot"
},
Pod:
"Labels": {
    "App": "example-spring-boot",
    "Deployment": "example-spring-boot-1",
    "Deploymentconfig": "example-spring-boot"
},
```

This link can be viewed using the `oc describe` command:

``oc describe service example-spring-boot``

```
Name:			example-spring-boot
Namespace:		tomcc-dockerimage
Labels:			app=example-spring-boot
Selector:		app=example-spring-boot,deploymentconfig=example-spring-boot
Type:			ClusterIP
IP:		     	172.30.158.17
Port:			8080-tcp	8080/TCP
Endpoints:		172.16.10.3:8080
Session Affinity:	None
No events.
```

Under Endpoints, you will now find the currently running pod.


### ImageStream
[ImageStreams](https://docs.openshift.com/container-platform/3.3/architecture/core_concepts/builds_and_image_streams.html#image-streams) are used to perform automatic tasks such as updating a deployment when a new Version of the image or base image is available.

Builds and deployments can monitor image streams and respond to changes appropriately. In our example, the ImageStream is used to trigger a deployment when something changes to the image.

Use the following command to read additional information about the Image Stream:

``oc get imagestream example-spring-boot -o json``

```
{
    "kind": "ImageStream",
    "apiVersion": "v1",
    "metadata": {
        "name": "example-spring-boot",
        "namespace": "tomcc-dockerimage",
        "selfLink": "/oapi/v1/namespaces/tomcc-dockerimage/imagestreams/example-spring-boot",
        "uid": "5f91c925-452a-11e7-a461-06aef59db650",
        "resourceVersion": "17470",
        "generation": 2,
        "creationTimestamp": "2017-05-30T11:23:15Z",
        "labels": {
            "app": "example-spring-boot"
        },
        "annotations": {
            "openshift.io/generated-by": "OpenShiftNewApp",
            "openshift.io/image.dockerRepositoryCheck": "2017-05-30T11:23:15Z"
        }
    },
    "spec": {
        "tags": [
            {
                "name": "latest",
                "annotations": {
                    "openshift.io/imported-from": "tomcc/example-spring-boot"
                },
                "from": {
                    "kind": "DockerImage",
                    "name": "tomcc/example-spring-boot"
                },
                "generation": 2,
                "importPolicy": {}
            }
        ]
    },
    "status": {
        "dockerImageRepository": "172.30.87.171:5000/tomcc-dockerimage/example-spring-boot",
        "tags": [
            {
                "tag": "latest",
                "items": [
                    {
                        "created": "2017-05-30T11:23:15Z",
                        "dockerImageReference": "tomcc/example-spring-boot@sha256:74e0b8a6347258d8523b14ca02bdd737cd0e11521d3ae26106cb11061cbb7c32",
                        "image": "sha256:74e0b8a6347258d8523b14ca02bdd737cd0e11521d3ae26106cb11061cbb7c32",
                        "generation": 2
                    }
                ]
            }
        ]
    }
}
```

### DeploymentConfig

The following points are defined in [DeploymentConfig](https://docs.openshift.com/container-platform/3.3/dev_guide/deployments/how_deployments_work.html):

- UpdateStrategy: How are application updates executed, how is the container replaced?
- Triggers: Which triggers lead to a deployment? In our example, ImageChange
- Container
  - Which image should be deployed?
  - Environment Configuration for the pods
  - ImagePullPolicy
- Replicas, number of pods to be deployed

The following command can be used to read additional information about DeploymentConfig:

``oc get deploymentConfig example-spring-boot -o json``

In contrast to DeploymentConfig, which tells OpenShift how an application is to be deployed, the ReplicationController defines how the application should look during the runtime (for example, 3 replicas should always run).

**Tip:** for each resource type, there is also a short form. For example, you can simply write ``oc get deploymentconfig`` as ``oc get dc``.

---

## Additional tasks

Look at the created resources with `oc get [ResourceType] [name] -o json` and `oc describe [ResourceType] [name]` from the first project `[USER]-example1`.

---

**End Lab 4**

<p width = "100px" align = "right"> <a href="05_create_route.md"> Create routes → </a> </p>

[← back to overview](../README.md)
