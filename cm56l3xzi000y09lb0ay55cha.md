---
title: "Improving Infrastructure as Code (IAC) Using DevOps and CI/CD | Multi-Environment Deployment"
datePublished: Fri Dec 27 2024 10:02:54 GMT+0000 (Coordinated Universal Time)
cuid: cm56l3xzi000y09lb0ay55cha
slug: improving-infrastructure-as-code-iac-using-devops-and-cicd-multi-environment-deployment
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1735293593167/ce7ba03a-6145-40f3-a54b-d457614fdd8c.png
tags: azure, devops, terraform, infrastructure-as-code, azure-pipelines, azure-devops, iac, devops-articles, infrastructure-management, devops-journey, iac-terraform-devops-aws, iac-infrastructure-as-code, terraformwithazure

---

In this project, we’ll leverage the Azure DevOps pipeline for automating the infrastructure deployment. We are also going to create multiple environment such as Dev, QA, Staging, and Prod having identical resources in each environment via Terraform.

# Creating Service Connection

This build and release pipeline will plan and apply Terraform manifests, as part of which it will generate the state files and create resources that are on Azure. To enable this communication between the pipeline and Azure Cloud, we need to establish a Service Connection which can be done using the following steps.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734623792178/75a54998-c4bc-43ec-8a07-218265074018.png align="center")

# CI-Build Pipeline

## Task 1

In this step, the Terraform manifests, which are essential for infrastructure provisioning, are copied from the system's default directory to the build artifact directory. This ensures that the configuration files are organized and accessible for downstream pipeline stages.

The Continuous Integration (CI) pipeline is a critical part of the DevOps lifecycle, enabling seamless integration and testing of code changes. In this pipeline, we handle the preparation of Terraform manifests for deployment and ensure they are readily available for subsequent release pipelines.

```yaml
# Starter pipeline
# Start with a minimal pipeline that you can customize to build and deploy your code.
# Add steps that build, run tests, deploy, and more:
# https://aka.ms/yaml

trigger:
- main

pool: Default
  #vmImage: ubuntu-latest
stages:
  - stage: GetTerraformManifests
    displayName: Build Stage
    jobs:
      - job: FetchTerraformManifests
        steps:
          - bash: echo "contents in working directoy"; ls -lrth $(System.DefaultWorkingDirectory)

          - task: CopyFiles@2
            inputs:          #/home/ubuntu/myagent/_work/1/s/16-Azure-IAC-DevOps/terraform-manifests
              SourceFolder: '$(System.DefaultWorkingDirectory)/16-Azure-IAC-DevOps/'
              Contents: '**'
              TargetFolder: '$(System.DefaultWorkingDirectory)'

          - bash: echo "contents in working directoy"; ls -lrth $(System.DefaultWorkingDirectory)
            displayName: List Contents post copying

          # Copy Terraform files to the Artifact Staging Directory
          - task: CopyFiles@2
            displayName: Copy Terraform Manifests to Staging Directory
            inputs:
              SourceFolder: '$(System.DefaultWorkingDirectory)/terraform-manifests'
              Contents: '**/*'  # Copy all files and subdirectories
              TargetFolder: '$(Build.ArtifactStagingDirectory)'
              
          - task: PublishBuildArtifacts@1
            displayName: Publish Manifests to Release pipeline
            inputs:
              PathtoPublish: '$(Build.ArtifactStagingDirectory)'
              ArtifactName: 'terraform-manifests'
              publishLocation: 'Container'
```

This pipeline is a foundational example that demonstrates how to fetch, process, and publish Terraform manifests for use in subsequent stages like deployment. Below is a detailed breakdown of each section of the YAML pipeline.

---

#### **Trigger Section**

```yaml
trigger:
- main
```

* **Purpose:**  
    The pipeline is configured to trigger automatically whenever there is a commit to the **main** branch.
    

---

#### **Pool Section**

```yaml
pool: Default
```

* **Purpose:**  
    The pipeline runs on the default agent pool. I have added my self-hosted agent to the default pool.
    

---

#### **Stages**

The pipeline contains a single stage, `GetTerraformManifests`, designed to build and prepare Terraform manifests.

---

##### **Stage: GetTerraformManifests**

```yaml
stages:
  - stage: GetTerraformManifests
    displayName: Build Stage
```

* **Purpose:**  
    The primary stage to gather and publish Terraform manifests required for deployment.
    

---

##### **Job: FetchTerraformManifests**

```yaml
jobs:
  - job: FetchTerraformManifests
```

* **Purpose:**  
    A single job under the stage that defines the steps to fetch and publish the Terraform manifests.
    

---

#### **Steps Breakdown**

1. **List Initial Directory Contents**
    
    ```yaml
    - bash: echo "contents in working directory"; ls -lrth $(System.DefaultWorkingDirectory)
    ```
    
    * **Explanation:**  
        Outputs the contents of the working directory before any processing. This helps to verify the initial state and debug potential issues.
        

---

2. **Copy Terraform Manifests from Source to Working Directory**
    
    ```yaml
    - task: CopyFiles@2
      inputs:
        SourceFolder: '$(System.DefaultWorkingDirectory)/16-Azure-IAC-DevOps/'
        Contents: '**'
        TargetFolder: '$(System.DefaultWorkingDirectory)'
    ```
    
    * **Purpose:**
        
        * Copies all files from the specified source folder (`16-Azure-IAC-DevOps`) to the working directory.
            
        * Ensures Terraform files are available for further processing.
            

---

3. **List Directory Contents Post-Copying**
    
    ```yaml
    - bash: echo "contents in working directory"; ls -lrth $(System.DefaultWorkingDirectory)
      displayName: List Contents post copying
    ```
    
    * **Purpose:**  
        Outputs the contents of the directory after copying to verify the successful transfer of files.
        

---

4. **Copy Terraform Files to Staging Directory**
    
    ```yaml
    - task: CopyFiles@2
      displayName: Copy Terraform Manifests to Staging Directory
      inputs:
        SourceFolder: '$(System.DefaultWorkingDirectory)/terraform-manifests'
        Contents: '**/*'  # Copy all files and subdirectories
        TargetFolder: '$(Build.ArtifactStagingDirectory)'
    ```
    
    * **Purpose:**
        
        * Moves all Terraform manifests to the `Artifact Staging Directory`.
            
        * Prepares the files for publishing as build artifacts.
            

---

5. **Publish Terraform Manifests as Build Artifacts**
    
    ```yaml
    - task: PublishBuildArtifacts@1
      displayName: Publish Manifests to Release pipeline
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)'
        ArtifactName: 'terraform-manifests'
        publishLocation: 'Container'
    ```
    
    * **Purpose:**
        
        * Publishes the Terraform manifests from the staging directory as build artifacts.
            
        * The artifact is named `terraform-manifests` and is made available in the container.
            
        * These artifacts can be used in subsequent **Release Pipelines** for deploying infrastructure.
            

---

### **Key Benefits of This Pipeline**

1. **Streamlined Artifact Management:**  
    Automates the process of gathering and preparing Terraform manifests for deployment.
    
2. **Modularity:**  
    Artifacts published here can be reused across multiple release pipelines, enabling consistent and efficient deployment workflows.
    
3. **Traceability:**  
    Each step is logged and auditable, ensuring that the process is transparent and easy to troubleshoot.
    
4. **Scalability:**  
    Provides a base pipeline that can be extended to include additional stages, such as testing or multi-environment deployments.
    

This pipeline is a crucial first step in integrating Terraform workflows into your CI/CD processes, ensuring that the infrastructure-as-code artifacts are always ready for deployment.

# Release Pipeline

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734490288202/0721b925-4847-4864-b13f-c0bcbd73f665.png align="center")

The release pipeline leverages the artifacts created during the CI process to deploy identical resources in multiple environments—**Dev**, **QA**, **Staging**, and **Production**. Each environment is isolated and configured with unique settings to reflect the appropriate stage of the application lifecycle.

**NOTE:** by default, the creation of a **Release Pipeline** is disabled as it is considered as legacy. To create the release pipeline you have to

1. Go to your **Project Settings**.
    
2. Under the **Pipelines** section, select **Settings**.
    
3. Scroll down to the **Classic Release Pipelines** option and enable it.
    

## Configure Artifact Source

In the release pipeline, we have first to configure the source of artifacts i.e. from our build pipeline

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734762720806/92757741-b899-4eb9-b547-3e4b17288f5e.png align="center")

Then enable the continuous deployment trigger as shown below

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734763014385/2d6929a3-f502-4ab7-81be-2f9356f7f6ec.png align="center")

## Dev Environment

### Release Pipeline for Dev

1. Configure the agent job where you define the pool and other configurations
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734763675999/6b0ed081-2406-41e1-a80b-188a12640f9f.png align="center")

2. Terraform Installation Task where you define the supported terraform version that needs to be installed in your server and perform the execution
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734765311940/51f6d757-fb51-40e7-aae3-1659774d233d.png align="center")
    
3. **Terraform Init task**
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734765431178/2cf6ebcb-3de7-4627-b9b5-18fee91b4b06.png align="center")
    
    Here we configure the backend which eliminates the need to explicitly define details in the terraform configuration ( versions.tf ) file.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734765493889/a2bba903-a0d6-4ece-904e-7b2ce1949b39.png align="center")
    
4. **Terraform Validate Task**
    
    This task will validate the terraform configuration present inside the path given in the configuration directory
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1735181168111/c66edbe6-1842-41a2-b91b-dbc6b2280231.png align="center")
    
5. **Terraform Plan Task**
    
    This task will run the terraform plan command and display the resource creation plan for the same **dev** environment. Here along with `terraform plan`, we are providing an additional argument as `-var-file=dev.tfvars` which will utilize the variables dedicated to the dev environment for resource creation
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1735181568458/62c5e902-87e5-4b61-a846-156a5036f52f.png align="center")
    

**Terraform Apply Task**

This task will run the terraform apply command and perform the resource creation in the **dev** environment. Here along with `terraform apply`, we are providing an additional argument as `-var-file=dev.tfvars` & `-auto-approve` which will utilize the variables dedicated to the dev environment for resource creation without asking for manual intervention

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1735182053301/db1d5d48-4bdf-4afb-abac-7456efe49892.png align="center")

### Dev Environment Completion

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1735286402151/5ae7489d-37d6-4abb-a8da-66682c6b60b8.png align="center")

With this the dev deployment has been completed

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1735286685018/7d1d6d76-d976-412f-bc0b-bb7a9f72d8ca.png align="center")

### Terraform manifest Modification for Dev Environment

The Dev environment serves as the first stage for deploying and testing infrastructure. The configuration (`tfvars`) for the Dev environment includes:

* **Virtual Network (VNet):** `10.1.0.0/16`
    
* **Subnets:**
    
    * Web Subnet: `10.1.1.0/24`
        
    * App Subnet: `10.1.11.0/24`
        
    * DB Subnet: `10.1.21.0/24`
        
    * Bastion Subnet: `10.1.100.0/24`
        

This environment is automatically deployed without manual intervention.

### tfvars for Dev environment

```yaml
environment = "dev"
vnet_address_space = ["10.1.0.0/16"]

web_subnet_name    = "websubnet"
web_subnet_address = ["10.1.1.0/24"]

app_subnet_name    = "appsubnet"
app_subnet_address = ["10.1.11.0/24"]

db_subnet_name    = "dbsubnet"
db_subnet_address = ["10.1.21.0/24"]

bastion_subnet_name    = "bastionsubnet"
bastion_subnet_address = ["10.1.100.0/24"]
```

## QA Environment

### Release Pipeline for QA Environment with Pre-Deployment Approval

The QA environment introduces **Pre-Deployment Approvals**, ensuring changes are reviewed before deployment. This step adds an extra layer of validation to maintain infrastructure consistency.

Most of the pipeline configuration will remain same with minor modifications, moving forward we’ll highlight only modified configurations for QA environment here. As most of the configuration will remain unchanged we can directly clone the dev stage by **clicking on the clone** has highlighteded below and then make the necessary modification to it.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1735287349441/26f05549-21af-4a59-82d1-e3d168ff5fcd.png align="center")

Once cloned we can make the necessary modifications as shown

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1735287559184/8d16baef-5c59-424e-9198-ccf87e3f5bd3.png align="center")

1. **Configure Pre-Deployment Approver**
    
    Here in QA deployment, we are required to verify the deployment before proceeding for which setting up the deployment approval is necessary and that can be configured as shown below
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1735288004379/e66850c6-6536-464d-a8a8-41f97025274e.png align="center")
    
2. **Terraform Init Task - QA**
    
    In the QA environment, the `terraform init` command must be configured with a backend block pointing to the QA-specific container key.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1735288459174/aa0d39ee-e9d2-444b-a304-a0453a589994.png align="center")
    
3. **Terraform Plan Task - QA**
    
    There will be no change to the validation task. At the plan task the terraform plan command has to proceed with dev.tfvars file which requires the following change and then only the command will act as `terraform plan -var-file=qa.tfvars`
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1735288777397/9189f26a-3d48-4c07-a50d-cf83f6bf7eed.png align="center")
    
4. **Terraform Apply Task - QA**
    
    Similarly, for the terraform apply task the `qa.tfvars` var file has to be picked up as shown below
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1735289056809/a2508610-5fc6-4bb1-85c8-a9c7610690da.png align="center")
    

### QA Environment Completion

As per the configuration, the QA deployment is now pending approval and once it is approved, it will deploy the identical environment.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1735290777846/8343c142-73b8-4eea-b18a-c562278926df.png align="center")

Once approved it has now started the deployment

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1735290939973/418367cc-400d-41c2-92c8-875668d25e57.png align="center")

QA Deployment is now completed with all respective resources

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1735291158797/45fea3a1-5a6f-4864-9191-33e893cd4bae.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1735292781480/c41a7762-c66e-43c6-bf8f-23de6b59e195.png align="center")

### Terraform Manifest Modification for QA Environment

Configuration for QA includes:

* **Virtual Network (VNet):** `10.2.0.0/16`
    
* **Subnets:**
    
    * Web Subnet: `10.2.1.0/24`
        
    * App Subnet: `10.2.11.0/24`
        
    * DB Subnet: `10.2.21.0/24`
        
    * Bastion Subnet: `10.2.100.0/24`
        

### tfvars for QA environment

```yaml
environment = "qa"

vnet_address_space = ["10.2.0.0/16"]

web_subnet_name    = "websubnet"
web_subnet_address = ["10.2.1.0/24"]

app_subnet_name    = "appsubnet"
app_subnet_address = ["10.2.11.0/24"]

db_subnet_name    = "dbsubnet"
db_subnet_address = ["10.2.21.0/24"]

bastion_subnet_name    = "bastionsubnet"
bastion_subnet_address = ["10.2.100.0/24"]
```

## Staging Environment with Pre & Post Deployment Approval

Just like the QA environment similar modification to each task also has to be done

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1735290382737/ddaf2efd-0fea-4d13-804c-2c9046089985.png align="center")

### Staging Environment Completion

For staging as well the deployment is completed post approval

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1735293062279/4820bc67-5c21-4e3b-a341-412b7159a768.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1735293101102/717627fb-c609-4b50-aa45-9d470b7d6fac.png align="center")

The Staging environment represents a near-production setup and includes both **Pre-Deployment** and **Post-Deployment Approvals**. Pre-deployment approval ensures changes are authorized before execution, while post-deployment approval validates successful deployment and functionality.

Configuration for Staging includes:

* **Virtual Network (VNet):** `10.3.0.0/16`
    
* **Subnets:**
    
    * Web Subnet: `10.3.1.0/24`
        
    * App Subnet: `10.3.11.0/24`
        
    * DB Subnet: `10.3.21.0/24`
        
    * Bastion Subnet: `10.3.100.0/24`
        

### tfvars for Staging environment

```yaml
environment = "staging"

vnet_address_space = ["10.3.0.0/16"]

web_subnet_name    = "websubnet"
web_subnet_address = ["10.3.1.0/24"]

app_subnet_name    = "appsubnet"
app_subnet_address = ["10.3.11.0/24"]

db_subnet_name    = "dbsubnet"
db_subnet_address = ["10.3.21.0/24"]

bastion_subnet_name    = "bastionsubnet"
bastion_subnet_address = ["10.3.100.0/24"]
```

## Prod Environment with Pre-Deployment Approval

Like the Stage environment, similar changes must also be made for Prod.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1735290482230/bcb7ae09-ad70-4a72-b15d-01879c4a96df.png align="center")

### Prod Deployment Completion

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1735293234465/f863cb0d-5de7-4c96-8ed4-b49b45474e12.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1735293265964/a7f2fd18-07b3-42f9-b2e4-5f32baf46f50.png align="center")

The Production (Prod) environment is the final deployment stage, ensuring the infrastructure is ready for live traffic. This environment requires **Pre-Deployment Approvals** to prevent unintended changes and maintain high reliability.

Configuration for Prod includes:

* **Virtual Network (VNet):** `10.4.0.0/16`
    
* **Subnets:**
    
    * Web Subnet: `10.4.1.0/24`
        
    * App Subnet: `10.4.11.0/24`
        
    * DB Subnet: `10.4.21.0/24`
        
    * Bastion Subnet: `10.4.100.0/24`
        

### tfvars for Prod Environment

```yaml
environment = "prod"

vnet_address_space = ["10.4.0.0/16"]

web_subnet_name    = "websubnet"
web_subnet_address = ["10.4.1.0/24"]

app_subnet_name    = "appsubnet"
app_subnet_address = ["10.4.11.0/24"]

db_subnet_name    = "dbsubnet"
db_subnet_address = ["10.4.21.0/24"]

bastion_subnet_name    = "bastionsubnet"
bastion_subnet_address = ["10.4.100.0/24"]
```

# Remote Backend

To ensure environment-specific isolation and consistency in Terraform state management, we configure a separate state storage container for each environment. This approach allows for safe, concurrent deployments while avoiding conflicts in state files. Here's how the Terraform settings are structured:

```yaml
terraform {
  required_version = "~>1.5.6" # Minor version upgrades are allowed
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~>4.3.0"
    }

    random = {
      source  = "hashicorp/random"
      version = ">=3.6.0"
    }
  }
  # Nothing should be configured here in backend block as the detils will be passed via Azure DevOps pipeline
  backend "azurerm" {
    
  }

}

provider "azurerm" {
  features {}
  subscription_id = "XXXXXX-9342-XXXXXXX-XXXXXXXX"
}
```

#### **Terraform Block**

The `terraform` block defines the required Terraform version, providers, and the backend configuration:

* **Required Version:** Locked to `~>1.5.6` allowing minor upgrades.
    
* **Providers:**
    
    * `azurerm`: Azure Resource Manager provider, pinned to `~>4.3.0` for stability.
        
    * `random`: Used for generating random values, with a flexible version constraint of `>=3.6.0`.
        
* **Backend Block:**  
    The backend block is intentionally left blank as the configuration will be dynamically provided through the Azure DevOps pipeline, ensuring secure and consistent handling of state files across environments.
    

#### **Provider Configuration**

The `provider` block configures the Azure Resource Manager (AzureRM) provider. It includes:

* **Features Block:** Enables additional provider features.
    
* **Subscription ID:** Specifies the Azure subscription for provisioning resources.
    

This setup ensures that the Terraform backend remains decoupled from the static code and is dynamically configured during pipeline execution. It allows for:

1. Environment-specific state management.
    
2. Enhanced security by avoiding hardcoding backend credentials.
    
3. Flexibility in scaling across multiple environments.
    

# Dependency Lock File

The `.terraform.lock.hcl` file ensures consistency and reproducibility of Terraform runs by locking provider dependencies to specific versions. When multiple team members or CI/CD pipelines work on the same Terraform configuration, this file ensures that all environments use the exact same provider versions. Provider updates may introduce breaking changes or unexpected behaviors. Locking versions prevents Terraform from automatically upgrading to potentially incompatible versions.

You can use the below command to generate a platform-independent lock file which will support all types of operating systems.

```yaml
terraform providers lock -platform=windows_amd64 -platform=darwin_amd64 -platform=linux_amd64
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1733541132346/a3b2097e-b36a-4337-bbdd-1a947f9b70ea.png align="center")