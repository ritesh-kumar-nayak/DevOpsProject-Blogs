---
title: "Azure DevOps with Project-Build Pipeline"
datePublished: Mon Jul 29 2024 17:52:56 GMT+0000 (Coordinated Universal Time)
cuid: clz7aerkd000d09l9hn873oro
slug: azure-devops-with-project-build-pipeline
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1722275331714/e156166e-7f20-49e9-b148-8487b80286da.png
tags: aws, azure, devops, pipeline, azure-devops, azure-certified

---

Azure DevOps is a suite of development tools Microsoft provides to support the complete software development lifecycle. It encompasses various tools for planning, developing, delivering, and maintaining software projects, enabling teams to collaborate more effectively and deliver high-quality software faster.

# Contents

1. Why Azure DevOps
    
2. Services Provided by Azure DevOps
    
3. Getting Started with Azure DevOps
    
4. Azure Repos
    
5. Azure DevOps Build Pipeline
    
6. Classic Editor Pipeline
    
7. YAML Pipeline
    
8. Self-Hosted Agent
    

# Why Azure DevOps?

* **All in One:** Azure DevOps provides a complete suite of tools for the entire software development lifecycle, including source control, CI/CD pipelines, project management, testing, and artifact management.
    
* **Seamless Integration:** All the tools are seamlessly integrated, reducing the need to switch between different tools and ensuring a smooth workflow.
    
* **Third-Party Integration:** With a vast extensions marketplace, Azure DevOps can integrate with many third-party tools and services, enhancing its functionality and adaptability.
    
* **All-in-one Place:** A single platform for development, project management, and operations enhances team collaboration and reduces communication gaps.
    
* **Multi-Cloud Deployments:** While optimized for Azure, it supports deployments to other cloud providers like AWS and Google Cloud.
    

# Services Provided by Azure DevOps

* **Azure Repos**: Git repositories for source control and version management.
    
* **Azure Pipelines**: CI/CD pipelines for automated build, test, and deployment.
    
* **Azure Boards**: Agile tools for planning, tracking, and project management.
    
* **Azure Artifacts**: Package management for Maven, npm, NuGet, and more.
    
* **Azure Test Plans**: Tools for manual and automated testing.
    

# Setting up Azure DevOps

Before setting up your Azure devops, you need to have an account with Microsoft, and with this article, I assume you already have an account, so let's move on to the Azure DevOps: [Azure DevOps](https://dev.azure.com/) and sign in with your MS account.

## Creating an Organization and Project

After signing up you need to create an **Organization**, an organization is nothing but a top-level fundamental container that serves as a namespace for managing and organizing your DevOps resources. The organization also facilitates collaboration by allowing multiple teams to work on different projects within the same organizational framework.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722181628752/da05bf1e-08be-45db-94bf-12ba64793ad1.png align="center")

Once a captcha authentication is completed, you'll be able to name your organization and create it which further lands you on the project creation page where you can give a name for your project and get started:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722182334498/703925a2-0c29-4c10-becf-3c3bc0fd44e1.png align="center")

Now, the project has been created under the organization called Hashnode Demo

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722182438261/b42f9247-d606-4a2e-b83c-4e658b1ba555.png align="center")

Within an organization, you can create **multiple projects**. Each project is a container for source code, builds, releases, test plans, and other resources.

# Azure Repos

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722188372469/490ed990-531f-4194-9cda-f85e53dc3935.png align="left")

Azure Repos is nothing but a set of version control tools provided by Microsoft as part of the Azure DevOps suite which supports both **Git repositories** and **Team Foundation Version Control(TFVC)** to manage our code base. Azure repo is designed to support any size of the team and enterprise application. The **TFVC** is now obsoleted and **Git-based** repos have been used widely across the industries.

### Git Repositories

* Fully distributed Version control system.
    
* Supports branching, merging, pull requests, and more.
    
* Ideal for teams using modern version control workflows.
    

### TFVC(Team Foundation Version Control)

* Centralized version control system.
    
* Suitable for teams that prefer a more traditional version control approach.
    
* Supports large codebases and binary files.
    

## Features of Azure Repos

1. **Pull Requests**:
    
    * Facilitates code reviews by enabling team members to comment on and review code changes.
        
    * Allows for automated builds and tests to be triggered as part of the pull request process.
        
2. **Branch Policies**:
    
    * Enforce policies on branches to ensure code quality and compliance.
        
    * Require pull request reviews, successful builds, and code coverage before merging.
        
3. **Code Search**:
    
    * Powerful search capabilities to find code across repositories.
        
    * Helps in quickly locating definitions, references, and changes.
        
4. **Web-Based Code Editing**:
    
    * Edit code directly in the browser without needing a local development environment.
        
    * Useful for quick changes and fixes.
        
5. **Integration with CI/CD Pipelines**:
    
    * Seamlessly integrates with Azure Pipelines for continuous integration and continuous deployment.
        
    * Automate builds, tests, and deployments based on repository changes.
        
6. **Support for Git Hooks**:
    
    * Implement custom scripts to run at different points in the Git workflow.
        
    * Automate and enforce development workflows and practices.
        

## Setting up Azure Repos

Setting up Azure Repos involves creating a new project, initializing a repository, and configuring your development environment to work with it. It also allows you to integrate your existing repositories from Git-Hub or similar platforms. In this article I will be demonstrating 2 ways:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722189012519/c9cf0a89-3213-4cb5-95d1-14b19faa9c3e.png align="center")

1. ### Clone to your Computer( New Repository )
    

Cloning a new repository to your computer from Azure Repos is a simple process. It allows you to create an empty repository, which you can start using immediately without the need for **explicit initialization**. This process is similar to cloning a repository from GitHub, and it provides a straightforward way to set up your local development environment.

Simply copy the given URL and run the below command

`git clone`[`https://Org-Hashnode@dev.azure.com/Org-Hashnode/Hashnode%20Demo/_git/Hashnode%20Demo`](https://Org-Hashnode@dev.azure.com/Org-Hashnode/Hashnode%20Demo/_git/Hashnode%20Demo)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722189427783/954a1a1b-de12-46d4-80fd-2881ad5dca53.png align="center")

Once you run the command it will prompt you to authorize via your Microsoft account

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722189532271/685f7ac0-3935-4c7f-98e7-d84a2d2d17eb.png align="left")

2. ### **Import a repository ( Existing Repository )**
    

Importing your existing repository from a different platform, such as GitHub, to Azure Repos is a seamless process. This is particularly useful when you want to **migrate** a large codebase and continue development within Azure DevOps.

Click on **Import**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722190043950/b91f5e8c-4fca-4cc0-9921-c517266b4fbc.png align="center")

This will further prompt you to choose the repository type out of Git or TFVC and paste your existing repository clone URL

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722190199527/481fcbc8-aa5c-4f60-b5e5-c35e5b368f2c.png align="left")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722190305664/434286e6-754b-4d6d-9579-8e3d6558c4b1.png align="left")

Now you can copy your existing repository HTTPS clone URL and paste it in the Clone URL box:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722190423947/5ccecaab-6f58-4334-8d83-ed57674de6df.png align="left")

Here we're using a YouTube Clone application based on NodeJS for demonstration purposes that has been forked from [Adrianhajdin](https://github.com/adrianhajdin/project_youtube_clone)'s repo.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722190882087/b2abadda-ad29-492e-8b98-74dc6c48286f.png align="left")

Now, click on **Import**. This will migrate your repository to Azure Repos, eliminating the dependency on GitHub. By importing your repository, you ensure that all your code, branches, and commit history are now managed within Azure DevOps, providing a seamless and integrated development environment.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722191074149/75e42fd1-5ca3-4be0-813b-7dedac83ef2b.png align="center")

# Azure Pipelines

**Azure Pipelines** is a powerful and versatile service provided by Microsoft Azure DevOps. It enables you to build, test, and deploy your code automatically and continuously, ensuring high-quality software delivery. Azure Pipelines supports a wide range of languages, frameworks, and platforms, making it a comprehensive solution for continuous integration (CI) and continuous deployment (CD). Azure Pipelines provides the flexibility, scalability, and reliability needed to support modern DevOps practices.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722191512476/96fd8efb-de50-4ae8-9215-c5e27f37eb3a.png align="left")

Azure Pipelines can be created using different methods, each catering to various preferences and project requirements. There are two primary ways to create pipelines in Azure DevOps:

* **Classic Editor (Graphical Interface)**
    

* **Pipeline as Code (YAML Pipelines)**
    

In this article, we'll be covering both ways of creating pipelines.

## Classic Editor (Graphical Interface)

The Classic Editor in Azure DevOps Pipelines provides a graphical user interface (GUI) for creating and managing build and release pipelines. This method is particularly useful for users who prefer a visual approach or are new to pipeline configuration. The Classic Editor offers an intuitive way to define the steps and stages of your CI/CD process without writing YAML code.

### Key Features of the Classic Editor

* **Pre-Built Tasks**:
    
    A library of pre-built tasks and templates to quickly configure common build and deployment scenarios.
    
* **Pipeline Stages and Jobs**:
    
    Visual representation of stages and jobs, making it easier to understand and manage complex workflows.
    
* **Variable Management**:
    
    Easy management of pipeline variables and secrets through the GUI.
    
* **Trigger Configuration**:
    
    Simple setup for continuous integration (CI) and continuous delivery (CD) triggers.
    
* **Integrations**:
    
    Seamless integration with various source control systems, build agents, and deployment environments.
    

### Create Pipeline with Classic Editor

As we have already configured the Azure Repos now it's time to build and deploy the application using the classic editor which will help us understand the workflow better.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722192845189/1d60f2bc-283a-49b8-9e7e-99434f754a56.png align="center")

**NB:** By default, the creation of classic build and release pipelines is disabled in Azure DevOps. To enable the Classic Editor option at the project level, you need to make changes at the organization level as shown below

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722193625615/31a57b18-2428-4a4b-8055-8b0a1db9cae4.png align="center")

Make sure both these options are toggled off to enable the classic editor option at the project level. you will allow the use of the Classic Editor for both build and release pipelines at the project level. This setting ensures that you have access to the Classic Editor’s graphical interface for creating and managing your pipelines.

Now you can see that **Use the classic editor** option is enabled at the project level below

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722193968405/3410e47b-2034-4abf-88fe-89ec78f7a07b.png align="center")

Now once click/ on the hyperlink it will land you on the page on which the source of your code base needs to be chosen

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722194302436/9f7c22dc-dfcd-495e-8880-0bbabcc0a522.png align="center")

We have chosen **Azure Repos Git** as we have already imported our code to Azure Repos. You can also choose other options such as GitHub, Bitbucket Cloud, etc as displayed. The underlying functionality is the same as all of the source options. Click on continue.

Now you can select a template or proceed with an Empty job.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722194382188/7db4fe2a-5adb-4224-93f8-ea696d693320.png align="center")

We'll proceed with the **Empty job** as we are going to configure our custom steps. Once clicked, you will be enabled to add **Steps** for your **Job** as shown below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722194960727/bb0675dc-573f-4828-a225-543fb468da9c.png align="center")

In this demonstration, the pipeline job is named **"Build and Deploy App"** and encompasses multiple steps. For this basic pipeline example, we handle both the build and release processes within a single pipeline.

### Configuring the Agent

The first step in creating any pipeline is configuring the agent where the pipeline jobs will be executed. An **agent** is essentially a server that performs the tasks required to build the application and generate the necessary artifacts.

**Agent Definition**: The agent is a crucial component of the pipeline infrastructure. It executes the tasks defined in the pipeline, such as building the code, running tests, and deploying artifacts.

**Agent Pools**: You can use either Microsoft-hosted agents or configure your own self-hosted agents. Microsoft-hosted agents are managed by Azure DevOps and come pre-installed with commonly used tools. Self-hosted agents are machines you set up and configure yourself, offering greater control and customization.

**Selecting an Agent**: During pipeline creation, you'll specify which agent or agent pool will be used to run the jobs. This selection determines the environment in which your pipeline tasks will be executed.

**NB:** As part of this demonstration, we'll be using self-hosted agents. Creating and configuring self-hosted agents will be demonstrated in later parts of this article.

### Tasks of the Pipelines

To define tasks in Classic Editor you can take the help of existing templates by searching them in the search bar and choosing the required template that will be added to the left as shown in further steps

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722195753824/eff61f11-00c0-4c9f-8a3d-2526e23ed576.png align="center")

1. **npm install**
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722195641196/920bc24e-6215-4b31-8ca7-2a00c7adf59d.png align="center")
    
    Once the template is chosen it will be added to the left and in the right section you can fill in the necessary fields such as command, working folder, variables, etc.. for that particular task as shown above.
    
2. **npm build**
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722196002848/65661cc5-ecfa-49e2-9fcc-3e0d51b224ba.png align="center")
    
    The subsequent task is the **build** task where the application will be built by the command specified as **run build**. These are NodeJS-based command and not related to AZDO
    
3. **Publish Artifacts**
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722196142850/fb69ccf6-ddbe-4d99-939d-6b62f2d0b399.png align="center")
    
    Once the application code is built, it generates the deployable artifacts that need to be published to the hosting server.
    
4. **Deploy to App Service**
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722196292388/82ae126f-1c98-40bc-b45b-84c2c6327c83.png align="center")
    
    As our artifacts are ready, now using the **Azure App Service Deploy** template, we'll be deploying the artifacts/deployable application to Azure App Service.
    

### Run Classic Editor Pipeline

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722196583168/e0b9ddfe-4bc9-4f3c-84a1-0584cfbb9fac.png align="center")

Now click on **Queue or Save & queue** to trigger the pipeline which will further ask you for the **Agent pool** and **Branch/tag**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722196815464/f118551d-5722-411f-8240-3e915bf386df.png align="left")

As we have taken **self-hosted** agents, I have placed them in the default agent pool hence, **Default** has been chosen.

Once **Run** is clicked the pipeline execution will be queued

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722197009389/cd1834b8-e8e8-4f68-8fc3-804cf7da87c3.png align="center")

The execution will begin as below

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722197292264/46321a9e-ca7b-4b61-b1a3-95282d256396.png align="center")

### Classic Editor Pipeline Completion

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722197429975/90e6e855-c566-45b5-82e6-c518b1b7b269.png align="center")

As shown above, once the pipeline execution is completed, the application has been deployed to **Azure App Service** and we can access the application via the red-bordered URL as shown below

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722197624002/de027fcb-6771-4f3b-8b61-5371c34ce8a6.png align="center")

## **Pipeline as Code (YAML Pipelines)**

A YAML pipeline in Azure DevOps is defined using a YAML (YAML Ain't Markup Language) file. This file contains the pipeline configuration as code, making it easy to version, share, and reuse. YAML pipelines provide a way to automate the build, test, and deployment processes in a structured, text-based format.

## Why YAML Over the Classic Editor?

**Version Control**:

* YAML pipeline definitions are stored in the same repository as your code. This means the pipeline configuration can be versioned, tracked, and reviewed along with the application code.
    

**Flexibility and Customization**:

* YAML offers greater flexibility for complex workflows, allowing for more granular control over the build and release processes.
    

**Infrastructure as Code**:

* Embraces the "Infrastructure as Code" (IaC) principle, making managing and automating infrastructure alongside application code easier.
    

**Collaboration**:

* Developers can collaborate on pipeline configurations using pull requests and code reviews, improving the quality and maintainability of the CI/CD processes.
    

## Setting up Pipeline As Code(YAML)

Click on the **Create Pipeline** on the project pipeline section which will land you on the below page asking you to choose where exactly your pipeline code resides

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722262710661/2a14e18d-4a9b-41b8-a689-fbbc48592fd1.png align="center")

As we are using Azure Repos for this demonstration, the same needs to be selected.

Once source is selected it will as you to configure the pipeline using pipelin-code. Here we have multiple options that can assist us in creating the pipeline with templates, it also allows you to import pipeline code if it already exists.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722262846955/942d2d6d-4a21-4ecd-9372-fb158aa75dff.png align="center")

In our case, we'll choose the **Starter pipeline** as we are going to create a pipeline from scratch so starter pipeline will provide a basic format to get started with like below

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722263089815/ac7237f0-f235-4200-8cfe-fcdc07a66427.png align="center")

## Pipeline as Code Architecture

![key concepts graphic](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/media/key-concepts-overview.svg?view=azure-devops align="left")

Its architecture is hierarchical, consisting of triggers, stages, jobs, steps, and tasks, each playing a distinct role in the CI/CD process.

### Key Components of a Pipeline

1. **Trigger**
    
    Triggers define when the pipeline should be run automatically. They can be configured based on various events, such as code commits, pull requests, or on a schedule.
    
    **Types**:
    
    * **CI (Continuous Integration) Triggers**: Automatically run the pipeline when code is committed to a branch.
        
    * **PR (Pull Request) Triggers**: Run the pipeline when a pull request is created or updated.
        
    * **Scheduled Triggers**: Run the pipeline at specified times.
        
    * **Resource Triggers**: Run the pipeline based on changes in external resources like container images.
        
    
    Example:
    
    ```yaml
    ---
    trigger:
     - main
    pool: 
      name: Default
    ```
    
2. **Stage**
    
    Stages are the major phases of the pipeline, **grouping jobs** that logically belong together. Stages can run sequentially or in parallel.**Features**:
    
    * **Isolation**: Each stage can be executed in a different environment.
        
    * **Dependencies**: Stages can have dependencies, ensuring they run in a specific order.
        
    
    ```yaml
    stages:
      - stage: Build
        jobs:
          - job: Build
            pool:
             name: Default
            steps:
               - task: Npm@1
                 displayName: NPM Install
                 inputs:
                  command: 'install'
                  verbose: true
               - task: Npm@1
                 inputs:
                   command: 'custom'
                   customCommand: 'run build'
          - job: Publish_Artifact
            displayName: Publish Artifact
            pool:
              name: Default
            steps:
              - task: PublishBuildArtifacts@1
                inputs:
                  PathtoPublish: 'build'
                  ArtifactName: 'drop'
        - stage: Deploy
        jobs:
          - job: Deploy
            pool:
              name: Default
            steps:
              - task: AzureRmWebAppDeployment@4
                inputs:
                  ConnectionType: 'AzureRM'
                  azureSubscription: 'Pay-As-You-Go(4accce4f-9342-4b5d-bf6b-5456d8fa879d)'
                  appType: 'webAppLinux'
                  WebAppName: 'youtube-devopswithritesh'
                  packageForLinux: $(System.DefaultWorkingDirectory)/build
                  RuntimeStack: 'STATICSITE|1.0'
    ```
    
3. **Job**
    
    Jobs are units of work that run on an agent. Each job consists of a series of steps and can run independently or in sequence with other jobs.
    
    **Features**:
    
    * **Parallelism**: Multiple jobs can run in parallel.
        
    * **Agent Specification**: Jobs specify the agent or pool on which they should run.
        
    
    Example:
    
    ```yaml
    - job: Build
            pool:
             name: Default
            steps:
               - task: Npm@1
                 displayName: NPM Install
                 inputs:
                  command: 'install'
                  verbose: true
               - task: Npm@1
                 inputs:
                   command: 'custom'
                   customCommand: 'run build'
    ```
    
4. **Step**
    
    Steps are the individual actions performed in a job. Each step represents a single task, such as running a script or executing a command.
    
    **Features**:
    
    * **Sequential Execution**: Steps within a job run sequentially.
        
    * **Conditionals**: Steps can have conditions that determine their execution.
        
    
    Example:
    
    ```yaml
    steps:
               - task: Npm@1
                 displayName: NPM Install
                 inputs:
                  command: 'install'
                  verbose: true
               - task: Npm@1
                 inputs:
                   command: 'custom'
                   customCommand: 'run build'
    ```
    
5. **Task**
    
    Tasks are predefined actions or operations that can be executed within a step. Azure DevOps provides a library of built-in tasks, and you can also define custom tasks.
    
    **Types**:
    
    * **Built-in Tasks**: Provided by Azure DevOps for common operations like building code, running tests, and deploying applications.
        
    * **Custom Tasks**: Custom scripts or actions defined by the user.
        
    
    Example:
    
    ```yaml
    steps:
               - task: Npm@1
                 displayName: NPM Install
                 inputs:
                  command: 'install'
                  verbose: true
               - task: Npm@1
                 inputs:
                   command: 'custom'
                   customCommand: 'run build'
    ```
    

While creating pipeline code you can also take the help of pipeline assistance by clicking on Show Assistant which will assist you with pre-built templates for each task.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722266889374/5ccf4c1d-942d-4110-9d89-0e7ebbb21cdc.png align="center")

You can also customize the template by putting custom values

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722267023872/811ea72a-eebb-4c17-98d6-f12556c6cf55.png align="left")

## Pipeline Execution

Once the pipeline is ready, you can run it by clicking on the **Run** button. If you are running the pipeline for the first time, you might need to authorize the pipeline to access the agent pool, especially if you are using **self-hosted** agents. This authorization ensures that the pipeline has the necessary permissions to utilize the resources provided by the agent pool.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722267849724/fd616dd2-5f7e-4a3a-8ba5-99ce6407615d.png align="center")

Now the execution has been started and we have 2 stages that are **Build** and **Deploy** stages respectively.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722272394943/6c02843d-ff1c-44ab-8096-f14b86164b31.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722272622417/d3f85aaf-7278-4c48-bc88-d1f3f43a75fc.png align="center")

In the above picture, you can see that the **Build** stage has been completed. However, it is not proceeding to the **Deploy** stage, causing the **Deploy** stage to remain in a pending state indefinitely. This issue occurs because artifacts generated in the **Build** stage are not automatically passed to the **Deploy** stage.

To resolve this, you need to ensure that the artifacts produced in the **Build** stage are available to the **Deploy** stage. This can be achieved by downloading the artifacts from the **Build** stage in the **Deploy** stage.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722273259403/df67ae40-bb55-4cd5-9d10-b64373e98d18.png align="center")

Now you can see that another task named **DownloadBuildArtifacts** has been added to the pipeline, which we have implemented with the help of the built-in assistance. The **DownloadBuildArtifacts** task is a template provided by Azure DevOps that simplifies the process of transferring artifacts between stages.

Using the **DownloadBuildArtifacts** task ensures that the artifacts generated in the **Build** stage are available in the **Deploy** stage, facilitating a smooth transition between stages.

```yaml
- stage: Deploy
    jobs:
      - job: Deploy
        pool:
          name: Default
        steps:
          - task: DownloadBuildArtifacts@1
            inputs:
              buildType: 'current'
              downloadType: 'single'
              artifactName: 'drop'
              downloadPath: '$(System.ArtifactsDirectory)'
          - task: AzureRmWebAppDeployment@4
            inputs:
              ConnectionType: 'AzureRM'
              azureSubscription: 'Pay-As-You-Go(4accce4f-9342-4b5d-bf6b-5456d8fa879d)'
              appType: 'webAppLinux'
              WebAppName: 'youtube-devopswithritesh'
              packageForLinux: '$(System.ArtifactsDirectory)/drop'
              RuntimeStack: 'STATICSITE|1.0'
```

Now, after adding the **DownloadBuildArtifacts** task, the **Deploy** stage can access the artifacts generated in the **Build** stage. As a result, the deployment has been successfully completed.

By using the **DownloadBuildArtifacts** task, we ensured that the artifacts produced in the **Build** stage were properly transferred to the **Deploy** stage, enabling the deployment process to proceed without any interruptions.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722273786669/4f46bb3f-9403-4c29-9ea4-84368d7322ad.png align="center")

Finally, the application has been deployed and we can access it.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722273914708/d046f79c-4562-475c-832d-8d800e3170bf.png align="center")

# Self-Hosted Agent

A self-hosted agent is a machine that you manage to run your Azure DevOps pipeline jobs. Unlike Microsoft-hosted agents, which are managed by Azure DevOps, self-hosted agents provide greater control over the environment, tools, and resources used during the build and deployment process.

## Configuring Self-Hosted Agent

**Prepare Your Machine**:

* Ensure your machine meets the prerequisites. It should have an operating system supported by Azure DevOps (Windows, Linux, macOS).
    
* Install necessary dependencies (e.g., .NET Core, Node.js, Docker, etc.) that satisfy your project needs. As our application is based on NodeJS all the Node dependencies have been installed on my machine.
    
* Here I am using an EC2 instance hosted in the AWS cloud.
    

**Create an Agent Pool**:

* Navigate to your Azure DevOps organization.
    
* Click on "Organization settings" (gear icon) in the lower-left corner.
    
* Select "Agent pools" under "Pipelines".
    
* Click "Add pool" to create a new agent pool and give it a name.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722274778198/d2034807-6beb-41d4-bc76-13c3371984ee.png align="center")
    
    * Here I have added my machine to the **Default** pool
        
        **Download and Configure the Agent**:
        
        * Within the agent pool, click on the "New agent" button.
            
        * Choose the operating system of your machine and download the agent package.
            
        * Extract the downloaded package to a directory on your machine.
            
        * Open a command prompt or terminal and navigate to the directory where you extracted the agent.
            

* **Run the Configuration Script**:
    
    * Execute the [`config.sh`](http://config.sh) script (for Linux/macOS) or `config.cmd` script (for Windows) and follow the prompts:
        
        ```yaml
        ./config.sh
        ```
        
    * Provide the server URL (Azure DevOps organization URL).
        
    * Enter a Personal Access Token (PAT) with sufficient permissions to register the agent.
        
    * Follow the prompts to configure the agent, including setting the agent pool name and agent name.
        
    
    **Run the Agent**:
    
    * After configuration, start the agent using the provided command:
        
        ```yaml
        ./run.sh
        ```
        
    * For Windows, use `config.cmd` and `svc.cmd` accordingly.
        

**Verify the Agent**:

* Go back to Azure DevOps and verify that the new agent appears in the agent pool and is online.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722275160757/2019e4fb-a0e6-4773-af06-fc6ceb08dfed.png align="center")