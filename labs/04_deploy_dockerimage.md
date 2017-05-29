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

``oc new-app appuio/example-spring-boot``

Output:

``
-> Found Docker image d790313 (3 weeks old) from Docker for "appuio/example-spring-boot"

    APPUiO Springboot App
    As shown in Fig.
    Example Springboot App

    Tags: builder, springboot

    * An image stream will be created as "example-spring-boot: latest" that will track this image
    * This image will be deployed in deployment config "example-spring-boot"
    * Port 8080 / tcp will be load balanced by service "example-spring-boot"
      * Other containers can access this service by the hostname "example-spring-boot"

-> Creating resources with label app = example-spring-boot ...
    Imagestream "example-spring-boot" created
    Deploymentconfig "example-spring-boot" created
    Service "example-spring-boot" created
-> Success
    Run 'oc status' to view your app.
``

For our Lab we use an APPUiO example (Java SpringBoot application):

- Docker Hub: https://hub.docker.com/r/appuio/example-spring-boot/
- GitHub (Source): https://github.com/appuio/example-spring-boot-helloworld

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

When we first executed `oc new-app appuio/example-spring-boot`, OpenShift created some resources for us in the background. These are required to deploy this docker image:

- [Service](https://docs.openshift.com/container-platform/3.3/architecture/core_concepts/pods_and_services.html#services)
- [ImageStream](https://docs.openshift.com/container-platform/3.3/architecture/core_concepts/builds_and_image_streams.html#image-streams)
- [DeploymentConfig](https://docs.openshift.com/container-platform/3.3/dev_guide/deployments/how_deployments_work.html)

### Service

[Services](https://docs.openshift.com/container-platform/3.3/architecture/core_concepts/pods_and_services.html#services) serve as an abstraction player, entry point and proxy / loadbalancer on the pods behind it within OpenShift. The service makes it possible to find and address a group of pods of the same type within OpenShift.

As an example, if an application instance of our example can no longer process the load alone, we can scale the application to, for example, three pods. OpenShift maps these as endpoints automatically to the service. Once the pods are ready, requests are automatically distributed to all three pods.

**Note:** The application can currently not be reached from the outside, the service is an OpenShift internal concept. In the following lab we will make the application publicly available.

Now let's take a closer look at our service:

``oc get services``

``
NAME CLUSTER-IP EXTERNAL-IP PORT (S) AGE
Example-spring-boot 172.30.124.20 <none> 8080 / TCP 2m
``

As you can see on the output, our service (example-spring-boot) is accessible via an IP and port (172.30.124.20:8080) **Note:** Your IP may be different.

**Note:** Service IPs remain the same for their lifetime.

Use the following command to read additional information about the service:

``oc get service example-spring-boot -o json``

```
{
    "Child": "service",
    "ApiVersion": "v1",
    "Metadata": {
        "Name": "example-spring-boot",
        "Namespace": "techlab",
        "SelfLink": "/api/v1/namespaces/techlab/services/example-spring-boot",
        "Uid": "b32d0197-347e-11e6-a2cd-525400f6ccbc",
        "ResourceVersion": "17247237",
        "CreationTimestamp": "2016-06-17T11: 29: 05Z",
        "Labels": {
            "App": "example-spring-boot"
        },
        "Annotations": {
            "Openshift.io/generated-by": "OpenShiftNewApp"
        }
    },
    "Spec": {
        "Ports": [
            {
                "Name": "8080-tcp",
                "Protocol": "TCP",
                "Port": 8080,
                "TargetPort": 8080
            }
        ],
        "Selector": {
            "App": "example-spring-boot",
            "Deploymentconfig": "example-spring-boot"
        },
        "PortalIP": "172.30.124.20",
        "ClusterIP": "172.30.124.20",
        "Type": "ClusterIP",
        "SessionAffinity": "None"
    },
    "Status": {
        "LoadBalancer": {}
    }
}
```

You can also use the corresponding command to display the details of a pod:

``oc get pod example-spring-boot-3-nwzku -o json``

**Note:** First, query the pod name from your project (`oc get pods`) and replace it in the upper command.

The `selector` area in the service defines which pods (`labels`) are used as endpoints. To do so, consider the corresponding configurations of the service and pod together.

``
Service:
--------
...
"Selector": {
    "App": "example-spring-boot",
    "Deploymentconfig": "example-spring-boot"
},

...

Pod:
Reply with quote
...
"Labels": {
    "App": "example-spring-boot",
    "Deployment": "example-spring-boot-1",
    "Deploymentconfig": "example-spring-boot"
},
...

``

This link can be viewed using the `oc describe` command:

``oc describe service example-spring-boot``

``
Name: example-spring-boot
Namespace: techlab
Labels: app = example-spring-boot
Selector: app = example-spring-boot, deploymentconfig = example-spring-boot
Type: ClusterIP
IP: 172.30.124.20
Port: 8080-tcp 8080 / TCP
Endpoints: 10.1.3.20:8080
Session Affinity: None
No events.
``

Under Endpoints, you will now find the currently running pod.


### ImageStream
[ImageStreams](https://docs.openshift.com/container-platform/3.3/architecture/core_concepts/builds_and_image_streams.html#image-streams) are used to perform automatic tasks such as updating a deployment when a new Version of the image or base image is available.

Builds and deployments can monitor image streams and respond to changes appropriately. In our example, the ImageStream is used to trigger a deployment when something changes to the image.

Use the following command to read additional information about the Image Stream:

``oc get imagestream example-spring-boot -o json``

### DeploymentConfig

The following points are defined in [DeploymentConfig] (https://docs.openshift.com/container-platform/3.3/dev_guide/deployments/how_deployments_work.html):

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

**Tip:** for each resource type, there is also a short form. For example, you can simply write `oc get deploymentconfig` as` oc get dc`.

---

## Additional tasks

Look at the created resources with `oc get [ResourceType] [name] -o json` and `oc describe [ResourceType] [name]` from the first project `[USER]-example1`.

---

**End Lab 4**

<p width = "100px" align = "right"> <a href="05_create_route.md"> Create routes → </a> </p>

[← back to overview](../README.md)
