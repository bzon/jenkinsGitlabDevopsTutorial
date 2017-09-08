# Table of Contents

- [Jenkins](#jenkins)
    * [Jenkins Installation](#jenkins-installation)
    * [Jenkins Plugins](#jenkins-plugins)
        * [Plugins to Install](#plugins-to-install)
            * [Required Plugins to install](#required-plugins-to-install)
            * [Installing plugins from the initial setup](#installing-default-plugins-from-the-initial-setup)
            * [Installing plugins manually](#installing-plugins-manually)
    * [Jenkins Tools](#jenkins-tools)
        * [Installing Oracle JDK](#installing-oracle-jdk)
        * [Installing Apache Maven](#installing-apache-maven)
    * [Jenkins Slave Setup](#jenkins-slave-setup)
    * [Creating a FreeStyle Jenkins Build for Petclinic](#creating-a-freestyle-jenkins-build)
        * [Create a new Job](#create-a-new-job)
        * [Configuring Git SCM Source](#configuring-git-scm-source)
        * [Configuring Build Step](#configuring-build-step)
        * [Configuring Post Build Actions](#configuring-post-build-actions)
    * [TODO - Creating a Pipeline Jenkins Build for Petclinic](#creating-a-pipeline-jenkins-build-for-petclinic)
- [Gitlab](#gitlab)
    * [Gitlab Installation](#gitlab-installation)
    * [Import Petclinic Project from Github to Gitlab](#import-petclinic-project-from-github-to-gitlab)
    * [Create Jenkins Gitlab clone Credentials](#create-jenkins-gitlab-clone-credentials)
    * [Configure Gitlab and Jenkins Integration](#configure-gitlab-and-jenkins-integration)
        * [Create a Gitlab user API Token](#create-a-gitlab-user-api-token)
        * [Create a Gitlab API Token Credentials in Jenkins](#create-a-gitlab-api-token-credentials-in-jenkins)
        * [Configure Gitlab connection in Jenkins](#configure-gitlab-connection-in-jenkins)
        * [OPTIONAL - Configure Jenkins System Logs](#configure-jenkins-system-logs)
    * [Disable Prevent Cross Site Request Forgery exploit in Jenkins](#disable-prevent-cross-site-request-forgery-exploit-in-jenkins)
    * [Update the PetClinicBuild Jenkins Job for Gitlab Integration](#update-the-petclinic-build-jenkins-job-for-gitlab-integration)
        * [Update Source Code Management](#update-source-code-management)
        * [Add Build Triggers for Push Events](#add-build-triggers-for-push-events)
    * [TODO - Update the PetClinicBuild Pipeline Jenkins Job for Gitlab Integration](#update-the-petclinicpipeline-jenkins-job-for-gitlab-integration)
    * [Add Webhook Integration in the Petclinic Repository for Push events](#add-webhook-integration-in-the-petclinic-repository-for-push-events)
    * [Testing the Push Event webhook](#testing-the-push-event-webhook)
- [SonarQube](#sonarqube)
    * [SonarQube Installation](#sonarqube-installation)
    * [Update the PetClinicBuild Jenkins Job to run SonarQube Maven](#update-the-petclinicbuild-jenkins-job-to-run-sonarqube-maven)
    * [TODO - Update the PetClinicBuild Pipeline Jenkins Job to run SonarQube Maven](#update-the-petclinicbuild-pipeline-jenkins-job-to-run-sonarqube-maven)
- [Nexus](#nexus)
    * [Nexus Installation](#nexus-installation)
    * [Update the PetClinicBuild Jenkins Job to run Maven Deploy to Nexus](#update-the-petclinicbuild-jenkins-job-to-run-maven-deploy-to-nexus)
    * [TODO - Update the PetClinicBuild Pipeline Jenkins Job to run Maven Deploy to Nexus](#update-the-petclinicbuild-pipeline-jenkins-job-to-run-maven-deploy-to-nexus)
- [Deployments using Docker](#deployments-using-docker)
    * [Update the PetClinicBuild Pipeline Jenkins Job to Build and Deploy Petclinic](#update-the-petclinicbuild-pipeline-jenkins-job-to-build-and-deploy-petclinic)
    * [Update your PetClinic project Dockerfile](#update-your-petclinic-project-dockerfile)
- [TODO - FitNesse and Selenium](#fitnesse-and-selenium)
- [Slack Notifications](#slack-notifications)
----

# Jenkins

## Jenkins Installation

```bash
docker run --name=jenkins \
--volume jenkins_data:/var/jenkins_home \
--publish 8080:8080 --publish 50000:50000 \
-d jenkins/jenkins:lts
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
- Parameterized Remote Trigger Plugin
- Environment Injector Plugin
- SonarQube Scanner Plugin
- OWASP Dependency Check Plugin
- Self-Organizing Swarm Plug-in Modules
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

## Jenkins Slave Setup

In this guide, we are going to create a Jenkins slave to run our build instead of running builds in the `Master` server. We are going to use a Linux server and bootsrap it via `SSH` protocol. The requirement is an ssh `private key` to be added as a Jenkins credential.  

Create a `Credential` with Kind `SSH Username with private key`.  

![](./img/jenkins_create_ssh_cred.png)

Get your Linux server private ip address.  
```bash
hostname -i
```

Go to `Manage Jenkins` -> `Manage Nodes` -> Then `Add` a new `Permanent` node. 

- `Remote root directory`, is where a Jenkins job will create its workspace.  
- `Labels`, is an identifier for your Jenkins slave. Add more by separating labels with spaces.  
- `Usage`, is `Use this node as much as possible` to make it the default server to run your builds.  
- `Launch Method`, is `Launch slave agents via SSH` because we are connecting to a Linux server using SSH key.  
    * `Host`, is the server's `private ip address`.  
    * `Credential`, is the Jenkins Credential you created from the above. 
    * `Host Key Verification Strategy`, ensure that this is `Non verifiying Verification Strategy`  

Follow the example below.  

![](./img/jenkins_linux_slave_config.gif)

Click `Save`.  

## Creating a FreeStyle Jenkins Build for Petclinic

### Create a new Job

Click `New Item` and select `Freestyle`.  

Name the job name as `PetClinicBuild`

![](./img/create_freestyle_job.gif)

### Configuring Git SCM Source

Select `Git` under `Source Code Management` and put https://github.com/bzon/spring-petclinic.  

![](./img/freestyle_git_config.gif)

### Configuring Build Step

The spring petclinic repository has a pre configured pom for this tutorial. To use it in this build, add an `Execute shell` Step and put the following.  

```bash
mv devops_docs/pom.xml.completePlugins pom.xml
```

![](./img/freestyle_exec_shell.gif)

Next, add an `Invoke top-level Maven targets` Step to build our petclinic app. Ensure that you use the `Maven Tool` that you named during [Installing Apache Maven](#installing-apache-maven) step.  

![](./img/freestyle_maven_config.gif)  


### Configuring Post Build Actions

__Add post-build action__

- `Publish JUnit test result report`
- `Publish HTML reports`, to get and diplay the Unit Tests Jacoco reports.  
- `Publish OWASP Dependency-Check results`

![](./img/freestyle_post_build_config.png)

## Creating a Pipeline Jenkins Build for Petclinic

__TODO__

---
# Gitlab  

From here, you will have to use your own hosted Gitlab server instead of Github. 

## Gitlab Installation

__Using Docker__

```bash
docker run --hostname localhost \
--publish 443:443 --publish 80:80 \
--publish 9022:22 --name gitlab \
--restart always \
--volume gitlab_config:/etc/gitlab \
--volume gitlab_logs:/var/log/gitlab \
--volume gitlab_data:/var/opt/gitlab \
-d gitlab/gitlab-ce
```

## Import Petclinic Project from Github to Gitlab

![](./img/gitlab_import_petclinic.gif)

## Create Jenkins Gitlab clone Credentials

The Credential's `Kind` should be a `Username and Password`.  

![](./img/jenkins_create_gitlab_creds.gif)

## Configure Gitlab and Jenkins Integration

### Create a Gitlab user API Token

- Go to your personal `Settings` -> `Access Tokens`.  
- Name your token and ensure that `api Access your API` and `read_user` is ticked.  
- __Take note of this token__  

![](./img/gitlab_create_user_token.gif)

### Create a Gitlab API Token Credentials in Jenkins

The Credential's `Kind` should be a `Gitlab API Token`.  

![](./img/jenkins_create_gitlab_api_cred.gif)

### Configure Gitlab connection in Jenkins

- Go to `Manage Jenkins` -> `Configure System`, and find `Gitlab` configuration.  
- As of this writing, ticking off `Enable authentication for '/project' end-point` does not harm anything.  
- `Gitlab host url` is your Gitlab http host url. 
- `Credentials` is your Gitlab API token credentials.  

![](./img/jenkins_gitlab_connection_config.gif)

### Configure Jenkins System Logs 

__This is OPTIONAL__

Murphy's Law.

To debug webhook calls accepted by the [gitlab plugin](https://github.com/jenkinsci/gitlab-plugin), add these configuration in `Manage Jenkins` -> `System Logs`.

Click `Add` then `Save`.  

![](./img/jenkins_system_log_gitlab_config.png)

## Disable Prevent Cross Site Request Forgery exploit in Jenkins

Ensure that this is disabled to open up Jenkins API and enable webhook calls from Gitlab.  

![](./img/disable_prevent_x_site.gif)

## Update the PetClinicBuild Jenkins Job for Gitlab Integration  

Some old example of what you are trying to achieve here can be found in [gitlab plugin wiki](https://github.com/jenkinsci/gitlab-plugin/wiki/Setup-Example).  

### Update Source Code Management 

__Name__ should be:  
`origin`

__Refspec__ should be:  
`+refs/heads/*:refs/remotes/origin/* +refs/merge-requests/*/head:refs/remotes/origin/merge-requests/*`

__Branch specifier__ is:  
`origin/${gitlabSourceBranch}`

![](./img/jenkins_gitlab_src_config.png)

### Add Build Triggers for Push Events

- Ensure that `Push Events` is the only `Event` that is ticked.  
- Under `Advanced`, in the `Secret token` parameter, click `Generate` and __take note of this token__, in the screenshot below it is `1cc736aa787ab0f3fef63682545c1564`.  
- __Take note of the Jenkins url__, in the screenshot below it is `http://52.15.197.136:8080/project/PetClinicBuild`.  
- Leave everything else to their default settings but ensure that you bookmark the [plugin documentation page](https://github.com/jenkinsci/gitlab-plugin) for future reference.  

![](./img/jenkins_gitlab_triggers_config.png)

## Update the PetClinicBuild Pipeline Jenkins Job for Gitlab Integration

__TODO__

## Add Webhook Integration in the Petclinic Repository for Push events

- In your Petclinic project repository page, go to `Settings` -> `Integration` then `Add Webhook`. 
- Ensure that `Push Events` is the only one that just ticked.  
- Paste the `noted` parameters from the setup [above](#add-build-triggers-for-push-events). 
- The example below shows an example `Webhook` configuration.  

![](./img/gitlab_webhook_settings.gif)

- Click `Test` -> `Push Events` to test a webhook call to Jenkins and it should trigger the Jenkins job.  

__NOTE__: 

As of this writing, I'm using `gitlab-ce` latest `9.5` released version and there is currently a [webhook bug #37288](https://gitlab.com/gitlab-org/gitlab-ce/issues/37288) introduced in it. 

The impact of this is I wasn't be able to successfully do a `Test` -> `Push Events` or any events from the webhook configuration page.  

## Testing the Push Event webhook

Try to use this command in your terminal while inside the petclinic repository to quickly do a push.  
```bash
echo " " >> pom.xml && git add -A && git commit -m "hook test" && git push origin master
```

![](./img/gitlab_test_push.gif)  

Which results to the Jenkins `PetClinicBuild` job getting triggered.  

![](./img/gitlab_jenkins_triggered.gif)

----
# SonarQube

## SonarQube Installation

__Using Docker__:  pre configured with plugins.  
```bash
docker run --name sonarqube \
--volume sonar_data:/opt/sonarqube \
--publish 9000:9000 --publish 9092:9092 \
--env ADOP_LDAP_ENABLED=false \
--env SONARQUBE_WEB_CONTEXT=/ \
-d bzon/adop-sonar:6.4
```
Docker project: https://github.com/bzon/adop-sonar/  

## Update the PetClinicBuild Jenkins Job to run SonarQube Maven 

Add a `Build Step` of `Invoke top-level Maven targets` with the following configuration  

`sonar.host.url`, the sonarqube server host url.  
`sonar.login`, the administrator username.  
`sonar.password`, the administrator password.  

![](./img/freestyle_maven_sonar_config.png)

## Update the PetClinicBuild Pipeline Jenkins Job to run run SonarQube Maven 

__TODO__

----
# Nexus

## Nexus Installation

__Using Docker__:  
```bash
docker run --name nexus \
--publish 8081:8081 \
--volume nexus_data:/sonatype-work \
-d sonatype/nexus:oss
```

## Update the PetClinicBuild Jenkins Job to run Maven Deploy to Nexus 

[Reference guide](https://github.com/bzon/spring-petclinic/blob/master/devops_docs/NEXUS_DEPLOY.md)

__Deploy to a snapshot repository in Nexus__:  

__Maven properties__:

`maven.skip.test`, should be `true`  
`dependencyCheck.skip`, should be `true`  
`altDeploymentRepository`, should be `nexus-snapshot::default::http://admin:admin123@52.15.197.136:8081/nexus/content/repositories/snapshots/` for snapshot. Change the ip address, username and password (admin/admin123 is Nexus default) accordingly.  

![](./img/freestyle_maven_deploy_config.png)

We are using a snapshot repository because we are building a snapshot version. See your root pom.xml and the version is configured as `<version>1.0-SNAPSHOT</version>` and a release version is something like `<version>1.0.1</version>`.  

For release versions, the `altDeploymentRepository` property should be something like, `nexus-release::default::http://52.15.197.136:8081/nexus/content/repositories/releases/`  

__Downloading binaries from Nexus__:
  
The result of the above configuration is the snapshot will be deployed in Nexus Snapshot repository and can be browse via url http://52.15.197.136:8081/nexus/content/repositories/snapshots/.  

## Update the PetClinicBuild Pipeline Jenkins Job to run Maven Deploy to Nexus 

__TODO__

---
# Deployments using Docker

__Requirements__:  
- Configure your PetclinicBuild Job to run or restrict its execution in a Linux node agent.  
- Ensure that `docker` is installed in the Linux node.    
- Ensure that `git` is installed in the Linux node.  

## Update the PetClinicBuild Pipeline Jenkins Job to Build and Deploy Petclinic

Add a `Build Step` of `Execute shell commands` with the following.  

```bash
sudo docker rm -f spring-petclinic || echo "petclinic container is not running.."

echo "Running docker build.."
sudo docker build -t spring-petclinic:latest .

echo "Deploying the latest build of petclinic container"
sudo docker run -d --name=spring-petclinic -p 8085:8080 spring-petclinic:latest
```  

![](./img/jenkins_deploy_docker_config.png)  

## Update your PetClinic project Dockerfile

Modify the `Dockerfile` in your petclinic git repository with the following and then `commit` it to the repository.  

```dockerfile
FROM marceldekoster/alpine-oracle-jdk-8

MAINTAINER John Bryan Sazon <john.bryan.j.sazon@accenture.com>

ARG IMAGE_VERSION

LABEL version=${IMAGE_VERSION}

EXPOSE 8080

ADD target/spring-petclinic-*.jar /spring-petclinic.jar

CMD ["java","-jar","/spring-petclinic.jar"]
```

---
# FitNesse and Selenium

----
# Slack Notifications

Configure your PetClinic Jenkins Build to send slack notifications in a channel.  

Follow the guide here:  https://github.com/jenkinsci/slack-plugin  