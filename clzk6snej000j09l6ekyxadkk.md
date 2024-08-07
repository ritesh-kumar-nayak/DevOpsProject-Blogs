---
title: "Azure DevOps-Release Pipeline | Project"
datePublished: Wed Aug 07 2024 18:32:45 GMT+0000 (Coordinated Universal Time)
cuid: clzk6snej000j09l6ekyxadkk
slug: azure-devops-release-pipeline
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1722743642010/53001c56-a410-4103-a7b7-bbcf30ab9cba.png
tags: aws, azure, devops, azure-pipelines, azure-devops, ci-cd, devops-articles

---

Azure DevOps offers a complete set of tools for managing the entire software development lifecycle. Two key parts of this lifecycle in Azure DevOps are Build Pipelines and Release Pipelines. We have already discussed the [Build Pipeline here](https://www.devopswithritesh.in/azure-devops-with-project-build-pipeline). In this article, we'll be discussing the Azure-DevOps Release Pipeline.

# Content

* Difference between Build and Release pipelines.
    
* Automating Deployments Using Multi-Stage Release Pipeline
    
* Creating a Release Pipeline
    

* Continuous Deployment Triggers
    
* Pre-Deployment Conditions
    
* Deployment Slots
    
* Integrate Deployment Slots with Stages
    
* Pre-Deployment Approval
    

* Reconfigure Build Pipeline with the Release Pipeline
    

* Blue-Green Deployment Using Swap in Azure App Service
    

# Build Pipeline vs Release Pipeline

In Azure DevOps, the Release Pipeline is seldom utilized due to its lack of flexibility in creating pipelines as code. The Release Pipeline primarily relies on the **classic editor** for pipeline creation, which does not align with the modern practices of Infrastructure as Code (IaC). Consequently, most organizations prefer to manage both Continuous Integration (CI) and Continuous Deployment (CD) using the Build Pipeline, leveraging YAML to define and version their pipelines. This approach provides greater control, consistency, and the ability to integrate with source control systems, enhancing the overall DevOps workflow.

### Build Pipeline

1. **Purpose**: The primary purpose of a build pipeline is to compile source code, run tests, and produce build artifacts. This is the "build" part of Continuous Integration (CI).
    
2. **Process**:
    
    * **Compilation**: Converts source code into executable code.
        
    * **Testing**: Runs unit tests to ensure the code is functional and meets quality standards.
        
    * **Artifact Creation**: Generates and builds artifacts (e.g., binaries, libraries, packages) that can be used in subsequent stages.
        
3. **Triggers**: Build pipelines are typically triggered by code changes (commits or pull requests) in the source repository.
    
4. **Output**: The primary output is a set of build artifacts that are stored in a specified location (e.g., Azure Artifacts, a file share).
    

### Release Pipeline

1. **Purpose**: The main objective of a release pipeline is to deploy the build artifacts to various environments (e.g., development, staging, production). This is the "release" part of Continuous Deployment (CD).
    
2. **Process**:
    
    * **Artifact Retrieval**: Retrieves build artifacts from the build pipeline or artifact repository.
        
    * **Deployment**: Deploys the artifacts to different environments. This may include running scripts, configuring infrastructure, and installing software.
        
    * **Testing**: This may include additional tests such as integration, performance, or user acceptance tests.
        
3. **Triggers**: Release pipelines can be triggered manually, on a schedule, or automatically based on the completion of a build pipeline or other criteria.
    
4. **Output**: The primary output is a deployed application or service in the target environment.
    

### Key Differences

1. **Focus**:
    
    * **Build Pipeline**: Focuses on building and validating code.
        
    * **Release Pipeline**: Focuses on deploying and verifying applications.
        
2. **Artifacts**:
    
    * **Build Pipeline**: Produces build artifacts.
        
    * **Release Pipeline**: Consumes build artifacts and deploys them.
        
3. **Environments**:
    
    * **Build Pipeline**: Typically runs in a single, controlled environment (e.g., build server).
        
    * **Release Pipeline**: Can deploy to multiple environments (e.g., development, staging, production).
        
4. **Stages**:
    
    * **Build Pipeline**: Generally has fewer stages (e.g., compile, test).
        
    * **Release Pipeline**: This can have multiple stages corresponding to different deployment environments.
        
5. **Automation**:
    
    * **Build Pipeline**: Often fully automated and runs frequently with code changes.
        
    * **Release Pipeline**: This can be automated but may require manual approval steps, especially for production deployments.
        
6. **Tools**:
    
    * **Build Pipeline**: Uses tools and tasks related to building and testing code.
        
    * **Release Pipeline**: Uses tools and tasks related to deployment and configuration management.
        

# Creating a Release Pipeline

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723035241929/addc5a7f-2c69-4a3b-88a7-727574092f0f.png align="center")

Move to your project inside the organization and under the **Pipeline** section click on the **Releases** and then click on New Pipeline which will bring you to the below classic editor-like page

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723035905334/8820e07b-8ac0-4d77-9259-15c8b25778f1.png align="center")

Just like the build pipeline, we can find a variety of templates that we can use directly. For this demo, I'll use the Azure App Service deployment. However, in my current organization, we deploy to a Kubernetes Cluster, and we have a template available called Deploy to a Kubernetes Cluster for the same.

You can apply the template and modify the template accordingly as below

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723036186293/dd73fcd2-0a3e-4f72-853a-9950229007cc.png align="center")

Once a stage is added, you can see the artifact section

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723036300424/acb2241b-0b31-495a-9b07-217f3b8ae705.png align="center")

The artifact section allows us to add the build artifact from the upstream system, artifact repositories, or directly from the build pipeline. When you click on "Add artifact," it will give you several options to choose from where you want to pull the artifacts.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723036689572/07450b31-3358-4b64-88a4-697d38ccee3c.png align="center")

Here, we'll proceed with **Build**. We have our build pipeline ready from the [previous demo](https://www.devopswithritesh.in/azure-devops-with-project-build-pipeline). Then, you can choose the project details, source, default version, etc., to configure. By default, we'll always take the latest build artifact.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723037443175/ab60b0d3-ba10-4d32-8b51-b7ed1bc982db.png align="center")

# **Continuous Deployment Triggers**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723037656937/23fd673e-115c-410c-8fdb-476ec97234c6.png align="center")

Once the artifact is set, the highlighted Thunder symbol represents a continuous deployment trigger. Here, we can configure how we want the deployment pipeline to be triggered. By default, it is disabled.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723038306648/190c6eb9-fe46-44a0-b75d-4b7349b79dcb.png align="center")

Once enabled you can set the filters based on which the deployment pipeline will be triggered.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723038486148/7abd867b-ff98-4ecc-91fa-dbe178087bbf.png align="center")

There are two options ***Branch filter and default branch filter.* The branch filter** allows you to select multiple branches and when changes are made to those particular branches deployment will be triggered. You can **Include and Exclude** multiple branches

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723038752887/b83a88ce-ae37-4abb-a862-27f6fc90d456.png align="center")

However, we'll choose Build **Pipeline's Default Branch as of now**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723038874631/37f6937b-1bcb-4456-b764-6a497f8341cd.png align="center")

### Pull Request Trigger

You can also choose **Pull Request Trigger** which allows us to trigger the pipeline when a PR is merged

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723039195090/88252076-0bf6-4a63-9dd2-c409be320bc8.png align="center")

### Scheduled Trigger

You can also **Schedule** a trigger which will allow you to trigger a release at a particular time

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723039341566/1a90155e-1087-4e70-b5b4-dadceb7a50bf.png align="center")

# Pre Deployment Conditions

In Azure DevOps, pre-deployment conditions are used to control when a deployment to a specific environment can proceed in a release pipeline. These conditions help ensure that deployments occur under the right circumstances and meet predefined criteria before moving to the next stage. Here are the main pre-deployment conditions you can configure:

### Types of Pre-Deployment Conditions

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723039945224/d1d6236e-21d1-4d72-8ef2-7d7f107764bf.png align="center")

1. **Approvals**:
    
    * **Manual Approvals**: Require one or more users to manually approve the deployment before it proceeds. This is useful for ensuring that a human reviews the deployment details and signs off before the deployment can continue.
        
    * **Automated Approvals**: Use automated processes or systems to grant approval based on specific criteria or rules.
        
2. **Gates**:
    
    * **Query-Based Gates**: Evaluate queries against external systems or databases to ensure certain conditions are met (e.g., checking the status of work items, and monitoring systems).
        
    * **Time-Based Gates**: Delay the deployment for a specified period to ensure other processes or dependencies have had time to complete.
        
3. **Checks**:
    
    * **Health Checks**: Integrate with monitoring tools to check the health status of the environment before deployment.
        
    * **Policy Checks**: Validate compliance with organizational policies or security standards before proceeding.
        
4. **Artifacts**:
    
    * Ensure that the required build artifacts are available and meet specific criteria before deployment.
        

### Configuring Pre-Deployment Conditions

To configure pre-deployment conditions in Azure DevOps, follow these steps:

Click on the Thunder icon left to stages, once the preferred trigger is selected you can configure the conditions based on multiple factors like Pre-deployment approvals, Gates, and Deployment queue settings.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723040215817/3613014c-2b92-48b9-9a67-1a980420dd37.png align="center")

Here we have added only one condition that says deployment should be triggered when the artifact is from the main branch. You can specify more conditions based on your preference.

### Example Use Cases

* **Manual Approval for Production Deployments**: Before deploying to production, require approval from a manager or a senior engineer to ensure all necessary reviews have been completed.
    
* **Health Check Gate**: Implement a health check that verifies the staging environment is stable and all services are running correctly before deploying a new release.
    
* **Policy Check**: Ensure that the deployment meets specific security policies, such as passing all security scans or ensuring that no critical vulnerabilities are present.
    

### Benefits

* **Control**: Pre-deployment conditions provide control over when and how deployments proceed, ensuring they meet predefined standards and criteria.
    
* **Compliance**: Helps maintain compliance with organizational policies, security standards, and industry regulations.
    
* **Risk Mitigation**: Reduces the risk of deployment failures or issues by ensuring that necessary checks and balances are in place before a deployment proceeds.
    

By leveraging pre-deployment conditions, organizations can enhance their deployment processes, improve reliability, and ensure that releases meet all requirements before reaching production.

# Deployment Slots

In Azure DevOps, deployment slots are a feature used primarily in conjunction with Azure App Services. Deployment slots allow you to create different environments (e.g., staging, production) for your web apps, API apps, and mobile app backends, facilitating safer and more controlled releases.

## Creating a Deployment Slot

To use deployment slots with Azure DevOps, you typically follow these steps:

* In the **Azure portal**, navigate to your App Service under the "Deployment" section, and select "Deployment slots".
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723042144059/c29dc90c-2c9c-40ee-aab1-29ddbf835c39.png align="center")
    
* We have created a slot for staging, and a DNS has been generated where staging deployments can be accessed. As you can see below, 100% of the traffic is currently being sent to **production** instead of **staging**.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723042750779/c0ee93d3-ca70-4447-a5df-e5a78f17c048.png align="center")
    

# Integrate Deployment Slots with Stages

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723041144228/db7a3ec4-dad9-489b-9ece-48a750d79078.png align="center")

This is similar to the classic build pipeline where you can add multiple stages and jobs using a template. Currently, we have only one stage for deploying to the Dev Environment. Once you click on the stage it will land you on the job configuration page.

## Agent Configuration Step

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723041515812/ec17a30a-10e4-4780-ae06-d8d4fb193270.png align="center")

The first job here will be initializing the agent where this Job will be executed. Here we have taken the agent pool as **Default** because I have added my self-hosted agent to the default. You can choose from multiple options available in the **agent pool**.

## Deploy to App Service

Once the agent is set, we can add the step for deploying the application to Azure App Service using the template as shown below

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723043756685/c2efb5a5-b981-4493-bbb7-cfa08b265713.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723044051848/a7def6ae-70ef-4daa-a6b9-2f8cc20bf1ec.png align="center")

Once you select the checkbox for **"Deploy to Slot or App Service Environment,"** you can choose the Resource Group and the desired Slot from the dropdown list. Since I have already created a slot named **staging**, I selected that one.

Finally, you can save your release pipeline to the Dev environment is ready with a stage named **Dev Deployment**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723044510381/95d64c96-2c58-4b72-bb95-293d07e1b050.png align="center")

# Production Stage

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723044826807/c41e9dff-a5f5-4527-a60b-94b0bb696e3a.png align="center")

You can click on the **Add** or **Clone** options highlighted respectively to create a different stage for Production and then you can make necessary changes like below

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723044965120/194b4630-7918-4f6d-b12a-eba2ccfba9d3.png align="center")

# Configuring Prod Deployment

## Pre-Deployment Condition in Prod

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723045082212/2e5aeaa2-25db-42a0-9cb7-8a06d0015733.png align="center")

By default, the conditions are set to **After Stage**, which means the production deployment stage will only be executed after the **Dev Deployment** is completed.

## Pre-Deployment Approval

Pre-deployment approval in Azure DevOps is a feature that allows you to require one or more people to review and approve a deployment to a specific environment, such as production before it proceeds. This is especially important for production deployments, as it ensures that all necessary checks are in place to prevent potential issues.

**Enable Pre-Deployment Approvals**:

* Toggle the "Pre-deployment approvals" switch to enable it.
    
* Add one or more approvers by typing their names or selecting them from the list. You can specify individual users, groups, or service accounts.
    
* Optionally, configure the approval settings such as approval timeout and comments required.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723045418588/1512b70f-5fd5-4c31-bde9-c37d9acc4c78.png align="center")

Approval policies in Azure DevOps are essential for ensuring that deployments to critical environments are thoroughly reviewed and meet organizational standards before proceeding. These policies help maintain control, compliance, and accountability within the deployment process.

## Configuring Stage

We need to configure the production deployment to happen to the production slot instead of the staging slot. To achieve that the Prod Deployment stage configuration will look like the below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723045919659/c3915e8d-4159-4a29-88a5-fc14442d8eee.png align="center")

In the Azure portal, we do not create a specific slot for production; it should be deployed directly to the main slot with the default DNS. Therefore, we simply uncheck the **Deploy to Slot or App Service Environment** option, allowing the deployment to happen directly in production, as highlighted below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723046056508/955803bb-13ba-4088-a7bf-d1a5d21c66ba.png align="center")

And now save the pipeline.

# Reconfigure Build Pipeline with the Release Pipeline

Earlier, our Build Pipeline code handled both the build and release processes, as shown below:

```yaml
---
trigger:
 - main
pool: 
  name: Default

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
           - task: PublishBuildArtifacts@1
             inputs:
              PathtoPublish: build
              ArtifactName: 'drop'

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

In the Deploy stage, the build artifact was downloaded and deployed to Azure App Service. This task will now be handled by the Release Pipeline.

Now, the stage can be removed and reconfigured as shown below. The build pipeline will be completed once the artifact is published.

```yaml
---
trigger:
 - main
pool: 
  name: Default

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
           - task: PublishBuildArtifacts@1
             inputs:
              PathtoPublish: build
              ArtifactName: 'drop'
```

Once pipeline code is saved and pushed, it auto-triggered the build pipeline which is completed as below

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723052012540/f0f2f8de-8134-4d07-a0f7-918be2659bed.png align="center")

You can see the build completed by producing 1 Artifact.

# Deployment via Release Pipeline

Now right after the build pipeline succeeds, the release pipeline was triggered and was able to fetch the published artifact.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723052351914/134801e6-703b-4469-8676-e5ea264c6082.png align="center")

As shown above, the deployment to the Dev Environment has begun and succeeded as below

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723052587275/6e15898e-86ce-4fcc-b317-c4bd5378868a.png align="center")

It has been deployed to the Azure App Service ***Staging*** *deployment slot* as configured earlier

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723052696625/b18167bd-1bcf-4bcd-b6b4-efb4043dc477.png align="center")

Now we are successfully able to access the application in Staging slot as shown and highlighted below

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723052894734/36d5b935-cc40-4134-bf0b-2017599d6986.png align="center")

## Deployment Approval for Production

After the Staging deployment is successful, the deployment to the Production environment enters a pending state, awaiting my approval.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723053052738/d0995620-74e4-4e61-88bb-e663fc1b5f26.png align="center")

A mail notification has been triggered for the same

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723053314683/c5528814-69a2-4a43-a9a1-d2a37b906ac8.png align="center")

Once you click on approve it will ask for a comment and then you can accept or reject the deployment.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723053386765/b7c5d708-c968-4953-9494-f1e745bdb749.png align="center")

Post approval the deployment has been started

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723053435706/158ed1ad-9f96-4cb3-8b7e-c64919081ebc.png align="center")

And now the deployment has been completed to prod

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723053642130/584fc9c3-55b1-42b7-b8fd-15f61ae2054a.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723053559934/03b74de0-a136-41be-bbf3-c933f385ccb3.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723053618182/c8c99829-943c-465b-bdf5-0f20f4abb824.png align="center")

Now instead of Staging, it has been deployed to default Prod DNS.

# Blue-Green Deployment Using Swap in Azure App Service

Blue-green deployment is a technique for minimizing downtime and reducing risk by running two identical production environments, referred to as "Blue" and "Green." Azure App Service makes it easy to implement blue-green deployments using its deployment slots feature.

Since we have already set up two deployment slots named **Staging** and **Production**, we can now swap them as shown below.

* Once you are confident that the new version is ready for production, navigate to the "Deployment slots" section of your Azure App Service.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723054416167/764f7db8-f4cb-4f07-9eed-baf46cea658f.png align="center")
    
* Click on the "Swap" button to start the swap operation between the staging slot and the production slot.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723054480734/d5eafa5c-ff69-4b97-ad0b-3f346c097239.png align="center")
    
    * The swap process exchanges the environment variables and settings between the staging and production slots, making the new version live without any downtime.
        
    * Monitor the production environment closely after the swap to ensure everything is functioning correctly.