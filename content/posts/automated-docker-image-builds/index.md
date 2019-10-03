---
title: "Automated docker image builds in Azure DevOps"
date: 2019-09-21T20:37:20+01:00
draft: false
author: Aideen
description: How to setup automated docker image.
cover:
tags:
  - Docker
  - AzureDevOps
  - AzurePipelines
  - DockerHub
  - containers
---

As part the initial steps of my project to get BWBBLE deployed on Kubernetes, I created two Docker images - one for the aligner part of the program and another for building the multi-reference genome.

We decided to use Azure DevOps to automate our docker image builds. Why would you do this? Setting up "autobuilds" means that Azure pipelines re-builds your docker images everytime you make commits to your source code and pushes the built image/images to your Docker repositories, whether that be the DockerHub Registry, Google Container Registry or the Azure Container Registry. In my case, I used the DockerHub registry so my guideline will be written from the perspective of using Azure pipelines to push images to the DockerHub registry.

When you set up autobuilds, you create a list of branches and tags that you want to build into Docker images. When you push code to a source code branch (for example in GitHub) for one of those listed image tags, the push uses a webhook to trigger a new build, which produces a Docker image. The built image is then pushed to the Docker Hub registry.

So how can you do this?

#### 1. Have a repository containing a DockerFile

If you don't have a repository with a DockerFile, you can fork this repository containing a sample application and a DockerFile.
<code data-author-content="https://github.com/MicrosoftDocs/pipelines-javascript-docker
">https://github.com/MicrosoftDocs/pipelines-javascript-docker
</code>

If you want to use the DockerHub registry to store and distribute your images, you should make sure you have an account with them at this stage.

#### 2. Create a pipeline in Azure Devops with build step.

1. Sign in to your Azure DevOps organization and navigate to your project.

2. Go to Pipelines, and then select New Pipeline.

3. Select GitHub as the location of your source code and select your repository.
4. Select Starter pipeline. In the Review tab, replace the contents of azure-pipelines.yml with the following snippet

<pre tabindex="0" class="has-inner-focus"><code class="lang-YAML" data-author-content="trigger:
- master

pool:
vmImage: 'Ubuntu-16.04'

variables:
imageName: 'pipelines-javascript-docker'

steps:

- task: Docker@2
  displayName: BuildAndPush
  inputs:
  containerRegistry: |
      $(dockerHub)
    repository: $(imageName)
    command: push
    tags: |
      test1
      test2
  "><span class="hljs-attr">trigger:</span>
  <span class="hljs-bullet">-</span> <span class="hljs-string">master</span>

<span class="hljs-attr">pool:</span>
<span class="hljs-attr"> vmImage:</span> <span class="hljs-string">'Ubuntu-16.04'</span>

<span class="hljs-attr">variables:</span>
<span class="hljs-attr"> imageName:</span> <span class="hljs-string">'pipelines-javascript-docker'</span>
<span class="hljs-attr"> tag:</span> <span class="hljs-string">'$(Build.BuildId)'</span>

<span class="hljs-attr">steps:</span>
<span class="hljs-attr">- task:</span> <span class="hljs-string">Docker@2</span>
<span class="hljs-attr">  displayName:</span> <span class="hljs-string">Push</span> <span class="hljs-string">image</span>
<span class="hljs-attr">  inputs:</span>
<span class="hljs-attr">    containerRegistry:</span> <span class="hljs-string">|
      $(dockerHub)
</span><span class="hljs-attr">    repository:</span> <span class="hljs-string">$(imageName)</span>
<span class="hljs-attr">    command:</span> <span class="hljs-string">buildAndPush</span>
<span class="hljs-attr">    tags:</span> <span class="hljs-string">|
     $(tag)
</span></code></pre>

##### Information about the azure-pipelines.yaml schema

<code>trigger</code> specifies what branches will cause a continuous integration build to run.

<code>pool</code> specifies which pool to use for a job of the pipeline. It also holds information about the job's strategy for running. If your pipelines are in Azure Pipelines, then you can run your jobs using a Microsoft-hosted agent. With Microsoft-hosted agents, each time you run a pipeline, you get a fresh virtual machine. Instead of managing each agent individually, you organize agents into agent pools.

<code>variables</code>are hardcoded values that can be added directly or variable groups that can be referenced.

<code>steps</code> A set of steps can be defined in one file and used multiple places in another file.

#### 3. Create a new service connection to DockerHub

You will need to create a Docker registry service connection to serve as secure options for storing credentials needed to login to the container registry before pushing the image. These service connections can be directly referenced in Docker task to login to the registry without the need to add a script task for docker login and setting up of secret variables for username and password. To create this:

1. In Azure DevOps, open the Service connections page from the project settings page. In TFS, open the Services page from the "settings" icon in the top menu bar.

2. Choose + New service connection and select the type of service connection you need.

3. Fill in the parameters for the service connection. The list of parameters differs for each type of service connection - see the following list. For example, this is the default Azure Resource Manager connection dialog:

4. Decide if you want the service connection to be accessible for any pipeline by setting the Allow all pipelines to use this connection option. This option allows pipelines defined in YAML, which are not automatically authorized for service connections, to use this service connection. Then press OK to create the connection and your good to go.

5. Go back to the azure-pipelines.yaml file to make sure you have updated the Docker task section to include the name of your containerRegistry and repository.

#### 4. Save and Run

Select Save and run, after which you're prompted for a commit message as Azure Pipelines adds the azure-pipelines.yml file to your repository. After editing the message, select Save and run again to see the pipeline in action.

I hope this helps encapsualte the steps you can follow to setup your own automated docker image builds.
