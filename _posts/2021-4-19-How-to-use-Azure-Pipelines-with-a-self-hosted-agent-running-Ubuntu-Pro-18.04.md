---
layout: post
title: How to use Azure Pipelines with a self-hosted agent running Ubuntu Pro 18.04
---

[Azure Pipelines](https://docs.microsoft.com/en-us/azure/devops/pipelines/?view=azure-devops), part of Azure DevOps Services, automatically builds and tests code projects to make them available to others. Azure Pipelines combines CI/CD to constantly and consistently test and build your code and ship it to any target including virtual machines and containers both  on-premises and on cloud platforms. This article shows you how to use Azure Pipelines with a self-hosted agent running Ubuntu Pro 18.04. 

[Ubuntu Pro](https://ubuntu.com/azure/pro) is a premium image designed by Canonical to provide additional coverage for production environments running in the cloud. It includes security and compliance services, enabled by default, suitable for small to largescale Linux enterprise operations — no contract needed. Key features include live kernel patching, which provides instant security and longer uptimes, broad security patching of major open source workloads for production use, and certified components for FedRAMP, HIPAA, PCI and ISO use cases. Ubuntu Pro (18.04 and above) is backed by a 10-year maintenance commitment by Canonical.

With Azure Pipelines, you have a convenient option to run your jobs using a Microsoft-hosted agent. With Microsoft-hosted agents, maintenance and upgrades are taken care of for you. Each time you run a pipeline, you get a fresh virtual machine. The virtual machine is discarded after one use. Microsoft-hosted agents can run jobs directly on the VM or in a container. Azure Pipelines provides a pre-defined agent pool named Azure Pipelines with Microsoft-hosted agents. For many teams this is the simplest way to run your jobs.

For Microsoft-hosted agents, you can use Ubuntu 16.04, 18.04 and 20.04 by just specifying it in your pipeline yaml:

```yaml
jobs:
- job: Linux
  pool:
    vmImage: 'ubuntu-18.04'   # or ubuntu-16.04, ubuntu-20.04 or ubuntu-latest
  steps:
  - script: echo hello from Linux
 ```
There may be times though when you require the extra security and cloud integrations of Ubuntu Pro. In that case, you need to use a self-hosted agent. Self-hosted agents give you more control to install dependent software needed for your builds and deployments. Also, machine-level caches and configuration persist from run to run, which can boost speed. This example will show you how to do that by building a Docker container image with Ubuntu Pro 18.04.

This guide assumes you have an Azure Devops organisation and project created already. You can do that by following [these instructions](https://docs.microsoft.com/en-us/azure/devops/pipelines/get-started/pipelines-sign-up?view=azure-devops) in the Microsoft documentation.

For this example, I forked the [Firefox Send](https://github.com/mozilla/send) repository on Github. 

### Create the Self-hosted Linux agent on Ubuntu Pro 18.04

1.  Get a personal access token by following the instructions in the [Prepare permissions](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/v2-linux?view=azure-devops#permissions) section of the Microsoft documentation.
2. [Create an Ubuntu Pro 18.04 instance](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/canonical.0001-com-ubuntu-pro-bionic?tab=Overview) on Azure and SSH to it.
3. We're creating a Docker container in this example scenario, so we need to install Docker on the VM:

```console
$ sudo apt install docker.io
$ sudo usermod -aG docker $USER
$ logout   # and log back in (to activate your new docker group)
```

4. Install the agent on your Ubuntu Pro 18.04 instance by following the instructions in the [Download and configure the agent](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/v2-linux?view=azure-devops#download-and-configure-the-agent)   section of the Microsoft documentation. You will create a new agent pool and get a download link for the agent tarball. Take a note of the name of the agent pool and copy the download link.  This is what the terminal part looks like on the Ubuntu Pro instance:

```console
$ wget https://vstsagentpackage.azureedge.net/agent/2.185.1/vsts-agent-linux-x64-2.185.1.tar.gz
$ tar zxvf vsts-agent-linux-x64-2.185.1.tar.gz

$ ./config.sh 

  ___                      ______ _            _ _
 / _ \                     | ___ (_)          | (_)
/ /_\ \_____   _ _ __ ___  | |_/ /_ _ __   ___| |_ _ __   ___  ___
|  _  |_  / | | | '__/ _ \ |  __/| | '_ \ / _ \ | | '_ \ / _ \/ __|
| | | |/ /| |_| | | |  __/ | |   | | |_) |  __/ | | | | |  __/\__ \
\_| |_/___|\__,_|_|  \___| \_|   |_| .__/ \___|_|_|_| |_|\___||___/
                                   | |
        agent v2.185.1             |_|          (commit 45c52d0)


>> End User License Agreements:

Building sources from a TFVC repository requires accepting the Team Explorer Everywhere End User License Agreement. This step is not required for building sources from Git repositories.

A copy of the Team Explorer Everywhere license agreement can be found at:
  /home/ubuntu/externals/tee/license.html

Enter (Y/N) Accept the Team Explorer Everywhere license agreement now? (press enter for N) > y

>> Connect:

Enter server URL > https://dev.azure.com/<your devops organisation name>
Enter authentication type (press enter for PAT) > 
Enter personal access token > ****************************************************
Connecting to server ...

>> Register Agent:

Enter agent pool (press enter for default) > <the name of your pool>
Enter agent name (press enter for <the name of your VM>) > 
Scanning for tool capabilities.
Connecting to the server.
Successfully added the agent
Testing agent connection.
Enter work folder (press enter for _work) > 
Enter Perform an unzip for tasks for each step. (press enter for N) > 
2021-04-19 14:29:35Z: Settings Saved.
```

For this example, we'll run the agent interactively. There are [ways to automate](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/v2-linux?view=azure-devops#run-as-a-systemd-service) running the agent on the Microsoft documentation. For now, just run the agent:

```console
$ ./run.sh 
Scanning for tool capabilities.
Connecting to the server.
2021-04-19 14:34:50Z: Listening for Jobs
```


### Create the pipeline

1.  Sign in to your Azure DevOps organization and navigate to your project.
    
2.  Go to  **Pipelines**, and then select  **New Pipeline**.
    
3.  Walk through the steps of the wizard by first selecting  **GitHub**  as the location of your source code.
    
4.  You might be redirected to GitHub to sign in. If so, enter your GitHub credentials.
    
5.  When the list of repositories appears, select your repository. 
    
6.  You might be redirected to GitHub to install the Azure Pipelines app. If so, select  **Approve & install**.
    

> When the  **Configure**  tab appears, select  **Docker**. This will create a Docker image.
> When the **Dockerfile** blade appears, use `$(Build.SourcesDirectory)/Dockerfile`

7.  When your new pipeline appears, take a look at the YAML to see what it does. Change the `pool:` option to your new pool:

```diff
-    pool:
-      vmImage: ubuntu-latest
+    pool: <your self-hosted agent pool name here>
```


8. This is what the full azure-pipelines.yml file should look like:

```yaml
# Docker
# Build a Docker image
# https://docs.microsoft.com/azure/devops/pipelines/languages/docker

trigger:
- master

resources:
- repo: self

variables:
  tag: '$(Build.BuildId)'

stages:
- stage: Build
  displayName: Build image
  jobs:
  - job: Build
    displayName: Build
    pool: <enter the name of your new agent pool here>
    steps:
    - task: Docker@2
      displayName: Build an image
      inputs:
        command: build
        dockerfile: '$(Build.SourcesDirectory)/Dockerfile'
        tags: |
          $(tag)
```

9. When you're ready, select  **Save and run**. 
    
10.  You're prompted to commit a new  _azure-pipelines.yml_  file to your repository. After you're happy with the message, select  **Save and run**  again.
    
If you want to watch your pipeline in action, select the build job. Here is what the "Build an image" in this example step should look like:
```
Starting: Build an image
==============================================================================
Task         : Docker
Description  : Build or push Docker images, login or logout, start or stop containers, or run a Docker command
Version      : 2.185.0
Author       : Microsoft Corporation
Help         : https://aka.ms/azpipes-docker-tsg
==============================================================================
/usr/bin/docker build -f /home/ubuntu/_work/2/s/Dockerfile [...]
Sending build context to Docker daemon  73.92MB

Step 1/33 : FROM node:12 AS builder
 ---> 29be39bd6917
Step 2/33 : RUN set -x     && addgroup --gid 10001 app     && adduser --disabled-password         --gecos ''         --gid 10001         --home /app         --uid 10001         app
 ---> Using cache
 ---> 9f0c1ada819c
Step 3/33 : RUN npm i -g npm
 ---> Using cache
 ---> ab045fb48b2a
Step 4/33 : COPY --chown=app:app . /app
 ---> dd33d87d93a2
Step 5/33 : USER app
 ---> Running in 4f3200538f8f
Removing intermediate container 4f3200538f8f
 ---> 2291f68ee2ed
Step 6/33 : WORKDIR /app
 ---> Running in df8911a6eaf5
Removing intermediate container df8911a6eaf5
 ---> 6322e6bcb9c1
Step 7/33 : RUN set -x     && npm ci     && npm run build
 ---> Running in 849da4e3f6c5
+ npm ci

added 1854 packages, and audited 1921 packages in 1m
[...]
```

The output from your agent in the SSH session should look like this:
```
2021-04-19 14:35:11Z: Running job: Build
2021-04-19 14:40:52Z: Job Build completed with result: Succeeded
```

And that's it! This example showed how to create and run a Docker image build pipeline with Azure Pipelines on an Ubuntu Pro 18.04 self-hosted agent. The key takeaway is that you can prepare your Ubuntu Pro host in the self-hosted agent pool to fit your scenario, ie. installing Docker, installing multiple versions of Python, etc.

