# Lab 2: Installing OpenShift CLI

In this lab, we will install and configure the oc client so that we can then take the first steps on the OpenShift Techlab platform.

## Command Line Interface

The ** oc client ** provides an interface to OpenShift V3.

The client is programmed in Go and comes as a single binary for the following operating systems:

- Microsoft Windows
- macOS
- Linux


## oc Download and install client

The oc client is downloaded from the [GitHub Repository of OpenShift Origin] (https://github.com/openshift/origin/releases/tag/v1.3.2):

* [Windows] (https://github.com/openshift/origin/releases/download/v1.3.2/openshift-origin-client-tools-v1.3.2-ac1d579-windows.zip)
* [Mac] (https://github.com/openshift/origin/releases/download/v1.3.2/openshift-origin-client-tools-v1.3.2-ac1d579-mac.zip)
* [Linux 64bit] (https://github.com/openshift/origin/releases/download/v1.3.2/openshift-origin-client-tools-v1.3.2-ac1d579-linux-64bit.tar.gz)
* [Linux 32bit] (https://github.com/openshift/origin/releases/download/v1.3.2/openshift-origin-client-tools-v1.3.2-ac1d579-linux-32bit.tar.gz)
* [CHECKSUM] (https://github.com/openshift/origin/releases/download/v1.3.2/CHECKSUM)

Once the client has been downloaded, it must be placed on the system in a directory accessible via the ** PATH **.

** Linux **

``
~/Bin
``

** macOS **

``
~/Bin
``

** Windows **

``
C:\OpenShift\
``

## Proper authorization on Linux and macOS

The oc client must be able to run.

``
cd ~/bin

chmod +x oc
``

## register the oc client in the PATH

Under ** Linux ** and ** macOS ** the directory ~ / bin is already in the PATH, so nothing has to be done here.

If the oc client has been placed in a different directory, the PATH can be set as follows:
``
$ export PATH = $PATH: [path to oc client]
``

### Windows

On Windows, the PATH can be configured in the advanced system settings. This is dependent on the corresponding Windows version:

- [Windows 7] (http://geekswithblogs.net/renso/archive/2009/10/21/how-to-set-the-windows-path-in-windows-7.aspx)
- [Windows 8] (http://www.itechtics.com/customize-windows-environment-variables/)
- [Windows 10] (http://techmixx.de/windows-10-umgebungsvariablen-bearbeiten/)

** Windows Quickhack **

Place the oc client directly in the directory * C:\Windows *.


## Verify the installation

The oc client should now be installed correctly. The best way to do this is to run the following command:
``
$ oc version
``
The following output should be displayed:
``
Oc v1.3.1
Kubernetes v1.3.0+52492b4
``

If this is not the case, the PATH variable may not be set correctly.

---

## bash / zsh completion

With Linux and Mac, the bashkompletion can be temporarily set with the following command:

``
source <(oc completion bash)
``

Or for zsh:
``
source <(oc completion zsh)
``

---



<P width = "100px" align = "right"> <a href="03_first_steps.md"> Getting Started on the Lab Platform → </a> </ p>
[← back to overview] (../README.md)
