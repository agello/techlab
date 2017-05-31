
# Lab 9: Codechanges by webhook triggering rebuild on OpenShift

In this lab, we'll show you the Docker Build Workflow by example, and you'll learn how to push a build and deploy the application to OpenShift with a push into the Git repository.

## Task: LAB9.1: Preparation of Github Account and Fork

### Github Account

In order to make changes to the source code of our example application, you need a separate GitHub account. If you do not already have one, set up an account at https://github.com/.

### Search for example project

**Sample project:** https://github.com/agello/example-spring-boot-helloworld

Go to the [GitHub Project page] (https://github.com/agello/example-spring-boot-helloworld) and [fork] (https://help.github.com/articles/fork-a-repo/) the project.

You now have
```
https://github.com/[YourGitHubUser]/example-php-docker-helloworld
```

a fork of the Example project that you can expand as you want.

## Deploy your own fork

Create a new project:
```
$ oc new-project [USER]-example4
```

Create a new app for your fork. **Note:** Replace `[YourGithubUser]` with the name of your GitHub account:

```
$ oc new-app https://github.com/agello/example-spring-boot-helloworld.git --strategy=docker --name=agello-java-docker
```
By means of the parameter `--strategy=docker`, we explicitly tell the `oc new-app` command to look for a Dockerfile in the specified Git repository and use it for the build.

Now expose the service with:
```
$ oc expose service agello-java-docker
```

## Task: LAB9.2: Set up web hook on GitHub

When creating the app, BuildConfig (bc) directly defined webhooks. You can do this by using the following command:

```
$ oc describe bc agello-java-docker
Name:		agello-java-docker
Namespace:	example4
Created:	3 minutes ago
Labels:		app=agello-java-docker
Annotations:	openshift.io/generated-by=OpenShiftNewApp
Latest Version:	1

Strategy:	Docker
URL:		https://github.com/agello/example-spring-boot-helloworld.git
From Image:	ImageStreamTag base-centos7:latest
Output to:	ImageStreamTag agello-java-docker:latest

Build Run Policy:	Serial
Triggered by:		Config, ImageChange
Webhook GitHub:
	URL:	https://openshift-master.noutje.nl:443/oapi/v1/namespaces/example4/buildconfigs/agello-java-docker/webhooks/oc3nSoIc1lvKxf8Ck_Sj/github
Webhook Generic:
	URL:		https://openshift-master.noutje.nl:443/oapi/v1/namespaces/example4/buildconfigs/agello-java-docker/webhooks/Afd8HhSQSIgawJQ1GRUS/generic
	AllowEnv:	false

Build			Status		Duration	Creation Time
agello-java-docker-1 	complete 	2m32s 		2017-05-31 10:06:32 +0200 CEST
```

You can also copy the GitHub WebHook from the Web Console. To do this, go to the appropriate build using Builds → Builds and select the Configuration tab:

Copy the GitHub [WebHook](https://developer.github.com/webhooks/) URL and paste it into GitHub.

In your project, click Settings:
![Github Webhook](../images/lab_09_webhook_github1.png)

Click Webhooks & services:
![Github Webhook](../images/lab_09_webhook_github2.png)

Add a WebHook:
![Github Webhook](../images/lab_09_webhook_github3.png)

Insert the appropriate GitHub WebHook URL from your OpenShift project and "disable" the SSL verification. On the Lab platform we have only self-signed certificates.
![Github Webhook](../images/lab_09_webhook_github4.png)

From now, all pushes on your GitHub repository triggers a build and then deploy the code changes directly to the OpenShift platform.

## Task: LAB9.3: Adjust the code

Now edit the code in your forked repo any way you like. The fastest method is totthe code directly in your browser;
For example, edit the index.html of the java app to display something else;

https://github.com/[YOURGITHUBUSERNAME]/example-spring-boot-helloworld/blob/master/src/main/resources/static/index.html

Once you've pasted the changes, OpenShift starts a build of the new source code
```
$ oc get builds
```

And then deployes the change.

## Task: LAB9.4: Rollback

With OpenShift, different software versions can be activated and deactivated by simply starting another version of the image.

The commands `oc rollback` and `oc deploy` are used for this purpose.

To run a rollback, you need the name of the DeploymentConfig:

```
$ oc get dc

NAME                 REVISION   DESIRED   CURRENT   TRIGGERED BY
agello-java-docker   2          1         1         config,image(agello-java-docker:latest)

```

Use the following command to roll back to the previous version:

```
$ oc rollback agello-java-docker
#3 rolled back to agello-java-docker-1
Warning: the following images triggers were disabled: agello-java-docker:latest
  You can re-enable them with: oc deploy agello-java-docker --enable-triggers -n example4
```

**Tip:** The automatic deployments of new versions are now switched off for this application to prevent unintentional changes after the rollback. To turn automatic deployment back on, run the following command:


```
$ oc deploy agello-java-docker --enable-triggers
```

---

**End Lab 9**

<p width = "100px" align = "right"> <a href="10_persistent_storage.md"> Connect and use persistent storage for database → </a> </p>

[← back to overview](../README.md)
