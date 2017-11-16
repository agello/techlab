# OpenShift development environment

This page shows different possibilities for self-developed Dockercontainer or OpenShift Templates etc. can be tested without having access to a complete, productive OpenShift platform such as APPUiO.

## oc cluster up

Since version 1.3 of the OpenShift client "oc" exists the possibility to start an OpenShift locally on its own laptop. To do this, a docker container containing an OpenShift installation is downloaded and then started.

Requirements:
* Oc 1.3+
* Docker 1.10

If the prerequisite is fulfilled and Docker is started, the OpenShift environment can be started with the following command:
`` `
$ Oc cluster up
`` `

### Documentation and Troubleshooting

#### iptables
A common source of error is the local firewall. Docker uses iptables to allow the containers to access the Internet. It can happen that certain rules get in the way. Often, the iptables rulechains are flushed after the OpenShift instance has been down-loaded with an `oc cluster down`:
`` `
$ Iptables -F
`` `
Then you can try again a `oc cluster up`.

#### Documentation

The complete documentation is available at https://github.com/openshift/origin/blob/master/docs/cluster_up_down.md.

## CentOS 7

1. Download and install CentOS 7 (Minimal ISO is fine)
2. Install docker

      **Note**
      CentOS 7 ships an older docker: 1.12. However the oc cluster up has some issues with the latest version of Docker

   - Enable IPv4 forwarding: ``sysctl net.ipv4.ip_forward=1`` 
   - Install and enable Docker: 
      ```
      yum install -y wget bzip2 git yum-utils device-mapper-persistent-data lvm2
      yum install docker
      systemctl enable docker
      ```

3. Enable insecure registry: ``vi /etc/sysconfig/docker``

     ```
     INSECURE_REGISTRY='--insecure-registry 172.30.0.0/16'
     ```

4. Start docker: ``systemctl start docker``

5. Setup firewall
    ```
    firewall-cmd --permanent --new-zone dockerc
    firewall-cmd --permanent --zone dockerc --add-source 172.17.0.0/16
    firewall-cmd --permanent --zone dockerc --add-port 8443/tcp
    firewall-cmd --permanent --zone dockerc --add-port 53/udp
    firewall-cmd --permanent --zone dockerc --add-port 8053/udp
    firewall-cmd --reload
    ```

6. Install oc

    ```
    wget https://github.com/openshift/origin/releases/download/v1.5.1/openshift-origin-client-tools-v1.5.1-7b451fc-linux-64bit.tar.gz
    tar -xvzf openshift-origin-client-tools*.tar.gz
    mv openshift-origin-client-tools*/oc /usr/local/bin/
    ```

7. Start oc cluster:  ``oc cluster up``

8. Check status

    ```
    oc login -u system:admin
    oc project default
    oc get pods
    NAME                            READY     STATUS      RESTARTS   AGE
    docker-registry-1-8hxn4         1/1       Running     0          11m
    persistent-volume-setup-fn98b   0/1       Completed   0          11m
    router-1-100hm                  1/1       Running     0          11m
    ```  

## Ubuntu 16.04

The setup for Ubuntu 16.04 is a little different than Fedora, CentOS or RHEL, because the registry access has to be configured differently.

1. Install Docker.
2. Configure Docker Daemon for an insecure Docker Registry.
   - To do this, create the file `/ etc / docker / daemon.json` with the following content:
     `` `
     {
       "Insecure-registries": ["172.30.0.0/16"]
     }
     `` `

   - Restart the docker daemon after creating the configuration.
     `` `
     $ Sudo systemctl restart docker
     `` `

3. Install oc

   Instructions in Lab [Install OpenShift CLI] (labs / 02_cli.md).

4. Open the terminal and execute this command with a user authorized on Docker:
   `` `
   $ Oc cluster up
   `` `

Stop Cluster:
`` `
$ Oc cluster down
`` `

## Vagrant

The [Puppet Module for OpenShift 3] (https://github.com/puzzle/puppet-openshift3/tree/dev) automates the installation of the platform in Vagrant. This Puppet Module is used to install and update productive instances.

## More options

A [Blogpost of Red Hat] (https://developers.redhat.com/blog/2016/10/11/four-creative-ways-to-create-an-openshiftkubernetes-dev-environment/) describes next to `oc cluster Up 'further variants of how a local development environment can be set up.

---

**The End **

[<< back to overview] (../README.md)
