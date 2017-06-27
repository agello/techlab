Deploying scala in openshift
===================

For this excercise we will deploy our own version of a scala webapp using the [play framework](https://www.playframework.com/)

We specifically use a web-framework because openshift is geared towards web-apps and api's both of which are strong suites of play.

> **Note:**

> In order to successfully complete this exercise you need to be familiar with the core concepts of openshift. More specifically the following prerequisites must be met:
> 
> - Your own github account
> - A new, empty openshift project
> - A functional oc client installation that is logged in to the correct environment.

## Create a new scala project 
Create a project for this exercise, use the following naming strategy:
**[USERNAME]**-scala

For example:
``$ oc new-project tomcc-scala
Now using project "tomcc-scala" on server "https://openshift-master.lab.``

## Make a fork of the sample application

Now clone the agello repository containing a sample play application. The repository that you need to clone can be found [here](https://github.com/agello/s2i-scala). 

>More information on how to clone a git repository to your own account can be found [here](https://help.github.com/articles/fork-a-repo/#fork-an-example-repository). Only a fork is needed, no need to set up syncing.

## Start building the sample app 

Now it is time to deploy the sample app. Do this with the following command:

oc new-app https://github/**[GITHUBUSERNAME]**/s2i-scala --contextdir=java8 --strategy=docker --name=scala

```$ oc new-app https://github.com/agello/s2i-scala/ --context-dir=java8  --strategy=docker --name=scala
--> Found Docker image bb81a09 (4 months old) from Docker Hub for "openshift/base-centos7"
    * An image stream will be created as "base-centos7:latest" that will track the source image
    * A Docker build using source code from https://github.com/agello/s2i-scala/ will be created
      * The resulting image will be pushed to image stream "scala:latest"
      * Every time "base-centos7:latest" changes a new build will be triggered
    * This image will be deployed in deployment config "scala"
    * Ports 8778, 9000 will be load balanced by service "scala"
      * Other containers can access this service through the hostname "scala"
    * WARNING: Image "openshift/base-centos7" runs as the 'root' user which may not be permitted by your cluster administrator
--> Creating resources with label app=scala ...
    imagestream "base-centos7" created
    imagestream "scala" created
    buildconfig "scala" created
    deploymentconfig "scala" created
    service "scala" created
--> Success
    Build scheduled, use 'oc logs -f bc/scala' to track its progress.
    Run 'oc status' to view your app.```

As indicated in the output you can folow the build with ``oc logs -f bc/scala``, much easier is to log into the openshift web portal and follow the build there from the Build menu. 

## Create a route
Now use the web interface to expose port 9000 to the internet. You should be greeted by a message from the webapp. [Relevant Documentation](https://docs.openshift.com/enterprise/3.2/dev_guide/routes.html)

> **Do not skip this step!**

From this moment on the application is live an reachable from the internet. Check this by going to the url mentioned in the openshift overview page.

## Create a health check
The application is running now. We have a singel container running without any checks. If the container crashes it is restarted. However, we have no functional check to see if the application is performing well. In order te be able to scale properly and have confidence in the uptime of our application we need a health check.

In the overview page the following warning should be displayed:

```scala has containers without health checks, which ensure your application is running correctly. Add Health Checks```

Add a health check by clicking on **Add Health Checks**

>Use the following parameters:
> - Add Readiness Probe
> - Type : HTTP-GET
> - Path: /
> - Port: 9000
> - Initial Delay: 30
> - Timeout: 2
> - Do not add a liveness probe

This will make sure the application will be available during redeployments and that crashed containers will be removed and redeployed.

## Baby steps!
Now make a change in your code to change the default message.

The following file contains the message:

https://github.com/**[GITHUBUSERNAME]**/s2i-scala/blob/master/java8/test/test-app/app/controllers/HomeController.scala

Since you made a fork to this repo you have the same repo in your account. Now change the message in the HomeController.scala and redeploy the app.

> **Tip**
> Github has an excellent online editor which lets you edit files directly. After editing the file you need to manually start a new build to download the changed code. Do this by navigating to builds->scala and click on **start build**

## Automate building the app
We don't want to manually start the build each time we make a change.

Use the documentation from [lab 9](https://github.com/agello/techlab/blob/lab-3.3/labs/09_dockerbuild_webhook.md) from the techlab to automate the builds.

Now make a change to the message and watch the build kicking off automatically!

## Start hacking scala!

At this point we've made a web app that automatically redeploys when new code is available and tries to keep itself up and running. See what you can build!

> Tip: https://www.playframework.com/documentation/2.5.x/Tutorials









