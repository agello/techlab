Google Translate
Original text
Gehen Sie auf die [GitHub Projekt-Seite](https://github.com/appuio/example-php-docker-helloworld) und [forken](https://help.github.com/articles/fork-a-repo/) Sie das Projekt.
Contribute a better translation
# Lab 9: Codechanges by webhook trig rebuild on OpenShift

In this lab, we'll show you the Docker Build Workflow by example, and you'll learn how to push a build and deploy the application to OpenShift with a push into the Git repository.

## Task: LAB9.1: Preparation of Github Account and Fork

### Github Account

In order to make changes to the source code of our example application, you need a separate GitHub account. If you do not already have one, set up an account at https://github.com/.

### Search for example project

**Sample project:** https://github.com/appuio/example-php-docker-helloworld

Go to the [GitHub Project page] (https://github.com/appuio/example-php-docker-helloworld) and [forken] (https://help.github.com/articles/fork-a- Repo /) you the project.

![Fork](../ images / lab_9_fork_example.png)


You have now
```
https://github.com/[YourGitHubUser]/example-php-docker-helloworld
```

A Fork of the Example project that you can expand as you want.

## Deploy your own fork

Create a new project:
```
$ oc new-project [USER] -example4
```

Create a new app for your fork. **Note:** Replace `[YourGithubUser]` with the name of your GitHub account:

```
$ oc new-app https://github.com/[YourGithubUser]/example-php-docker-helloworld.git --strategy=docker --name=appuio-php-docker-ex
```
By means of the parameter `--strategy=docker`, we explicitly tell the `oc new-app` command to look for a Dockerfile in the specified Git repository and use it for the build.

Now expose the service with:
```
$ oc expose service appuio-php-docker-ex
```

## Task: LAB9.2: Set up web hook on GitHub

When creating the app, BuildConfig (bc) directly defined webhooks. You can do this by using the following command:

```
$ oc describe bc appuio-php-docker-ex

Name: appuio-php-docker-ex
Created: 57 seconds ago
Labels: app = appuio-php-docker-ex
Annotations: openshift.io/generated-by=OpenShiftNewApp
Latest version: 1

Strategy: Docker
URL: https://github.com/appuio/example-php-docker-helloworld.git
From Image: ImageStreamTag php-56-centos7: latest
Output to: ImageStreamTag appuio-php-docker-ex: latest
Triggered by: Config, ImageChange
Webhook Generic: https://master.appuio-beta.ch:443/oapi/v1/namespaces/techlab-example4/buildconfigs/appuio-php-docker-ex/webhooks/EqEq18JtxaY3vG2zvPSU/generic
WebHook GitHub: https://master.appuio-beta.ch:443/oapi/v1/namespaces/techlab-example4/buildconfigs/appuio-php-docker-ex/webhooks/hqQ3h1CzUGIXvWqjiV-G/github

Build Status Duration Creation Time
Appuio-php-docker-ex-1 run for 56s 2016-06-17 16:56:34 +0200 CEST


```

You can also copy the GitHub WebHook from the Web Console. To do this, go to the appropriate build using Builds → Builds and select the Configuration tab:

![Webhook](../images/lab_9_webhook_ose3.png)

Copy the GitHub [WebHook](https://developer.github.com/webhooks/) URL and paste it into GitHub.

In your project, click Settings:
![Github Webhook] (../images/lab_09_webhook_github1.png)

Click Webhooks & services:
![Github Webhook] (../images/lab_09_webhook_github2.png)

Add a WebHook:
![Github Webhook] (../images/lab_09_webhook_github3.png)

Insert the appropriate GitHub WebHook URL from your OpenShift project and "disable" the SSL verification. On the Lab platform we have only self-signed certificates.
![Github Webhook] (../images/lab_09_webhook_github4.png)

From now, all pushes on your GitHub repository triggers a build and then deploy the code changes directly to the OpenShift platform.

## Task: LAB9.3: Adjust the code

Clone your Git repository and change to the code directory:
`` `
$ git clone https://github.com/[YourGithubUser]/example-php-docker-helloworld.git
$ cd example-php-docker-helloworld
`` `

For example, adjust the file on line 56 ./app/index.php:

```
$ vim app/index.php
```

![Github Webhook](../images/lab_9_codechange1.png)

```
    <Div class = "container">

      <Div class = "starter-template">
        <H1> Hi <? Php echo 'OpenShift Techlab'?> </ H1>
        <P class = "lead"> APPUiO Example Dockerfile PHP </ p>
      </ Div>

    </ Div>
```

Push your Change:

```
$ git add.
$ git commit -m "updated Hello"
$ git push
```

Alternatively, you can edit the file directly on GitHub:
![Github Webhook](../images/lab_9_edit_on_github.png)

Once you've pasted the changes, OpenShift starts a build of the new source code
```
$ oc get builds
```

And then deployed the change.

## Task: LAB9.4: Rollback

With OpenShift, different software versions can be activated and deactivated by simply starting another version of the image.

The commands `oc rollback` and `oc deploy` are used for this purpose.

To run a rollback, you need the name of the DeploymentConfig:

```
$ oc get dc

NAME TRIGGERS LATEST
Appuio-php-docker-ex ConfigChange, ImageChange 2

```

Use the following command to roll back to the previous version:

```
$ oc rollback appuio-php-docker-ex
# 3 rolled back to appuio-php-docker-ex-1
Warning: the following images were disabled: appuio-php-docker-ex
  You can re-enable them with: oc deploy appuio-php-docker-ex -enable-triggers -n phptest
```

Once the old version has been deployed, you can use its browser to check whether the original header **Hello APPUiO** is displayed again.

**Tip:** The automatic deployments of new versions are now switched off for this application to prevent unintentional changes after the rollback. To turn automatic deployment back on, run the following command:


```
$ oc deploy appuio-php-docker-ex -enable-triggers
```

---

**End Lab 9**

<p width = "100px" align = "right"> <a href="10_persistent_storage.md"> Connect and use persistent storage for database → </a> </p>
[← back to overview] (../README.md)
