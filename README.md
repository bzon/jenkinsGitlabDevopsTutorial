# Jenkins Tutorial

## Table of Contents

- [Jenkins Installation](#jenkins-installation)
    * [Using Docker](#using-docker)
- [Jenkins Plugins](#jenkins-plugins)
    * [Plugins to Install](#plugins-to-install)
        * [Required Plugins to Install](#required-plugins-to-install)
        * [Installing plugins from the initial setup](#installing-default-plugins-from-the-initial-setup)
        * [Installing plugins manually](#installing-plugins-manually)
- [Jenkins Tools](#jenkins-tools)
    * [Installing Oracle JDK](#installing-oracle-jdk)
    * [Installing Apache Maven](#installing-apache-maven)
- [Creating a FreeStyle Jenkins Build](#creating-a-freestyle-jenkins-build)

## Jenkins Installation

### Using Docker

```bash
docker run --name=jenkins -v jenkins_data:/var/jenkins_home -p 8080:8080 -p 50000:50000 -d jenkins/jenkins:lts
```

Get the initial administrator password by view the jenkins logs.   
```bash
docker logs jenkins
```
And you should see something like the below. Save it.  
```bash
Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

11233260a6414ce59870ae52dbf0a173

This may also be found at: /var/jenkins_home/secrets/initialAdminPassword
```

## Jenkins Plugins

Jenkins has thousands of plugins that you can use in your software development project. In this tutorial, we will install plugins that are required to build my forked Java maven project [spring-petclinic](https://github.com/bzon/spring-petclinic).

### Required Plugins to Install

- Rebuilder
- SSH Agent Plugin
- HTML Publisher Plugin
- JUnit Plugin
- Gitlab Plugin
- OWASP Dependency Check Plugin
- Slack Notification

#### Installing default plugins from the initial setup  
From the initial setup you can choose to `Select plugins to install` plugins.  

![](./img/jenkins_plugin_install_1.png)
![](./img/jenkins_plugin_install_2.png)

#### Installing plugins manually

![](./img/jenkins_plugin_install_3.gif)

## Jenkins Tools

Your goal is to install Oracle JDK and Apache maven for building our [spring-petclinic](https://github.com/bzon/spring-petclinic). Please take note of the Tool name that you will apply.  

You have to go to `Manage Jenkins` -> `Global Tool Configuration`.  

### Installing Oracle JDK

An account in oracle.com is required to be able to install Oracle JDK from Jenkins configuration.  

![](./img/install_oracle_jdk.gif)

### Installing Apache Maven

![](./img/install_maven.gif)

## Creating a FreeStyle Jenkins Build

__Create a new Job__

Click `New Item` and select `Freestyle`.  

Name the job name as `PetClinicBuild`

![](./img/create_freestyle_job.gif)

__Configuring Git SCM Source__

Select `Git` under `Source Code Management` and put https://github.com/bzon/spring-petclinic.  

![](./img/freestyle_git_config.gif)

__Configuring Build Step__

The spring petclinic repository has a pre configured pom for this tutorial. To use it in this build, add an `Execute shell` Step and put the following.  

```bash
mv devops_docs/pom.xml.completePlugins pom.xml
```

![](./img/freestyle_exec_shell.gif)

Next, add an `Invoke top-level Maven targets` Step to build our petclinic app. Ensure that you use the `Maven Tool` that you named during [Installing Apache Maven](#installing-apache-maven) step.  

![](./img/freestyle_maven_config.gif)  


