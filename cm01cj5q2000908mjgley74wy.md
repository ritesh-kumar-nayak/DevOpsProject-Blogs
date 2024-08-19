---
title: "Azure DevOps and Terraform Integration"
datePublished: Mon Aug 19 2024 18:45:25 GMT+0000 (Coordinated Universal Time)
cuid: cm01cj5q2000908mjgley74wy
slug: azure-devops-and-terraform-integration
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1723401687070/efe0823b-e66d-484a-8608-46ae5a4fde6c.png
tags: azure, terraform, azure-devops, iac, terraform-terraform-cloud-devops-devops-articles-devsecops-cloud-aws-gcp-azure-terraform-aws-infrastructureascode-provisioning-automation-cloudcomputing-infrastructure-as-code-terraform-state-technical-writing-blogging-infrastructure-management

---

Integrating Terraform with Azure DevOps allows organizations to harness the power of Infrastructure as Code (IaC) for streamlined, automated deployments in the cloud. By leveraging Terraform's capabilities within Azure DevOps pipelines, teams can manage infrastructure efficiently, reduce manual errors, and maintain consistent environments across development, staging, and production. This synergy between Terraform and Azure DevOps enables seamless provisioning, management, and scaling of resources, ensuring that infrastructure changes are deployed with the same rigor and reliability as application code, ultimately driving operational excellence and innovation.

In this article, we will focus on integrating Terraform workflows with the Azure DevOps pipeline to enhance infrastructure automation. If you want to learn Terraform from the basics, please refer to [my blog on Terraform fundamentals](https://www.devopswithritesh.in/complete-terraform-fundamentals).

The Terraform workflow typically follows a series of steps designed to manage infrastructure as code effectively. Here's an overview of the key stages:

1. **Write**:  
    In this initial stage, you define your infrastructure using HashiCorp Configuration Language (HCL) in Terraform files. These files describe the resources and configurations needed for your cloud environment, including virtual machines, networks, storage, and more. The infrastructure code is stored in version control systems like Git to ensure collaboration and tracking of changes.
    
2. **Initialize (terraform init)**:  
    Before applying your configurations, you need to initialize your Terraform working directory. This step downloads the necessary provider plugins (e.g., for Azure and AWS) and sets up the environment for Terraform to run. This is typically the first command executed in a Terraform workflow.
    
3. **Plan (terraform plan)**:  
    The `terraform plan` command generates an execution plan, detailing the actions Terraform will take to reach the desired state of your infrastructure. It shows what resources will be created, modified, or destroyed without making any actual changes. This step is crucial for reviewing and validating the changes before applying them.
    
4. **Apply (terraform apply)**:  
    After reviewing the plan, the `terraform apply` the command is used to execute the changes. Terraform interacts with the cloud provider's API to create, update, or delete resources as defined in your configuration files. The applied changes bring your infrastructure to the desired state.
    
5. **Manage and Evolve**:  
    Once the infrastructure is deployed, you can continue to manage and evolve it by modifying the Terraform configurations. Changes are tracked through version control, and the workflow cycles through planning and applying updates. Terraform maintains a state file that records the current state of your infrastructure, enabling it to track changes and ensure consistency.
    
6. **Destroy (terraform destroy)**:  
    When resources are no longer needed, the `terraform destroy` command can be used to tear down the entire infrastructure or specific resources. This command helps clean up and manage costs by removing unused resources.
    

The Terraform workflow can be automated using continuous integration/continuous deployment (CI/CD) pipelines in platforms like Azure DevOps. This automation ensures that infrastructure changes are consistently and reliably deployed, reducing the potential for human error.

# Azure CLI Local Setup

Terraform supports multiple ways of authentication to Azure which are:

* [Authenticating to Azure using the Azure CLI](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/azure_cli)
    
* [Authenticating to Azure using Managed Se](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/azure_cli)[rvic](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/managed_service_identity)[e Identity](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/azure_cli)
    
* [Authenticating to Azure using](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/azure_cli) [a Service Prin](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/managed_service_identity)[cipal and a Client Certificate](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/azure_cli)
    
* [Authentic](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/azure_cli)[ating t](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/managed_service_identity)[o Az](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/service_principal_client_certificate)[ure using a Service Prin](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/managed_service_identity)[cipal and a Client Secret](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/azure_cli)
    
* [Authenticating](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/azure_cli) [to Azure u](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/service_principal_client_certificate)[sing OpenID Connect](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/managed_service_identity)
    

Our demo will use [Authentic](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/azure_cli)[ating t](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/managed_service_identity)[o Az](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/service_principal_client_certificate)[ure using a Service Prin](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/managed_service_identity)[cipal and a Client Secret](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/guides/azure_cli). You can get more information about the authentication process on [azurerm terraform documentation](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs).

Integrating Terraform with Azure CLI is crucial because it simplifies and secures the process of managing Azure resources. By using Azure CLI for authentication, Terraform can seamlessly ***interact with your Azure environment*** without needing to manage separate credentials, reducing the risk of exposure. This integration enables Terraform to leverage existing Azure CLI configurations, such as active subscriptions and managed identities, streamlining the deployment process and ensuring consistent, secure access to Azure resources.

## 1\. **Install Azure CLI**

* Ensure that Azure CLI is installed on your machine. You can install it by following [Azure CLI Installation Guide](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).
    
* Verify the installation by running:
    
    ```bash
    az --version
    ```
    

## 2\. **Install Terraform**

* Install Terraform on your machine. You can download it from the official Terraform website.
    
* Verify the installation by running:
    
    ```bash
    terraform --version
    ```
    

## 3\. **Authenticate Azure CLI**

* Log in to Azure using Azure CLI:
    
    ```yaml
    az login
    ```
    
    After executing the command, a new browser window will open, directing you to the Azure sign-in page. On the sign-in page, select the Azure account you want to use. If you have multiple accounts, you can choose the appropriate one.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723973666050/fd3e7312-a76f-4ef8-8854-53cb9302a353.png align="center")
    
    Once you are logged in successfully it will display the account details on your Local CLI as below
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723973892016/27c6ba10-8403-4344-b248-0e6ff7e914e5.png align="center")
    
* If you work with multiple subscriptions, set the active subscription:
    
    ```yaml
    az account set --subscription "your-subscription-id"
    ```
    

## 4\. **Create** Service Principal

We can now create the Service Principal which will have permission to manage resources in the specified Subscription using the following command:

```bash
az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/0000000-9342-4b5d-bf6b-5456d8fa879d"
```

You will be getting the output having below 4 values

```markdown
{
  "appId": "0000000-20be-41c8-bad0-60299ed476ae",
  "displayName": "azure-cli-2024-08-18-10-13-02",
  "password": "XXXXXXXXXXXXX",
  "tenant": "bbb-xxxx-zzzzz"
}
```

These values map to the Terraform variables like so:

* `appId` is the `client_id` defined above.
    
* `password` is the `client_secret` defined above.
    
* `tenant` is the `tenant_id` defined above.
    

## 5\. **Login using Service Principle**

Now the service principal has the **contributor role** assigned hence we need to log in again using the created service principal

```bash
az login --service-principal -u CLIENT_ID -p CLIENT_SECRET --tenant TENANT_ID
```

Once execute the above command with appropriate values, you will get an output as below where you will be logged in via the service principal

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723976933939/995de9f3-a219-4d70-bf48-d2fc89d5bfae.png align="center")

With the service principal credentials, Terraform can now communicate with Azure to create, modify, and destroy infrastructure as defined in your Terraform scripts.

## **6\. Configuring the Service Principal in Terraform**

As we've obtained the credentials for this Service Principal - it's possible to configure them in a few different ways.

When storing the credentials as Environment Variables, for example:

```shell
# sh
export ARM_CLIENT_ID="00000000-0000-0000-0000-000000000000"
export ARM_CLIENT_SECRET="12345678-0000-0000-0000-000000000000"
export ARM_TENANT_ID="10000000-0000-0000-0000-000000000000"
export ARM_SUBSCRIPTION_ID="20000000-0000-0000-0000-000000000000"
```

These values can also be hard-coded under the provider block in the Terraform manifest. However, it is not recommended to hard-code secret credentials as plain text. Instead, we can use environment variables as mentioned above or store them in Azure Key Vault.

### **Outcome**

From now on, whenever you run `terraform apply` or `terraform destroy`, Terraform will authenticate with Azure using this service principal. This setup ensures secure and automated infrastructure management, aligned with best practices for cloud authentication.

# **Create a Terraform Configuration**

## provider.tf

Start by defining your infrastructure in a `.tf` file:

* ```bash
        # We strongly recommend using the required_providers block to set the
        # Azure Provider source and version being used
        terraform {
          required_providers {
            azurerm = {
              source  = "hashicorp/azurerm"
              version = "=3.0.0"
            }
          }
        }
        
        # Configure the Microsoft Azure Provider
        provider "azurerm" {
          # here you can hard code your azurerm credentials such as tenant_id, subscription_id, client_id, client_secret etc.. however, it is not at all recomended to hardcode secret credentials as plain text. We can use environment variables or store them in azure key-vault.
          features {}
        }
    ```
    
* Initialize Terraform:
    
    ```yaml
    terraform init
    ```
    
* Apply the configuration:
    
    ```yaml
    terraform apply
    ```
    

## **Best Practices**

* **Use Managed Identity**: If running Terraform from within Azure (e.g., Azure DevOps), consider using a Managed Identity to handle authentication automatically without needing service principals.
    
* **State Management**: Use remote state management (e.g., Azure Storage) to securely store your Terraform state files.
    
* **Environment Variables**: Terraform can also use Azure credentials stored in environment variables (`ARM_CLIENT_ID`, `ARM_CLIENT_SECRET`, `ARM_TENANT_ID`, `ARM_SUBSCRIPTION_ID`), which are useful in CI/CD pipelines.
    

***NB: You can access the terraform manifests created as part of this article in the below repository***

Click here to access [Terraform Manifests](https://github.com/ritesh-kumar-nayak/azuredevops-integration-terraform)

# Azure DevOps Integration

So far, all steps in our infrastructure provisioning process have been performed locally using the Azure CLI and a local Terraform installation. While this approach is effective for initial testing and development, fully automating the process is crucial for consistent, repeatable, and scalable infrastructure deployments.

To achieve full automation, we will leverage **Azure DevOps** pipelines to handle all Terraform operations, including initialization, planning, application, and destruction of infrastructure. This approach ensures that infrastructure provisioning is integrated into our CI/CD processes, providing version control, automated testing, and streamlined deployment.

## Project Creation and setup

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724064357913/d46cc52e-0d79-4b44-adbe-470f1c2c0a95.png align="center")

### **Integrating GitHub with Azure DevOps for Pipeline Automation**

To streamline our infrastructure provisioning process, we’ve created a new Azure DevOps project. In this project, we will integrate our Terraform code hosted on GitHub and proceed with creating an automated pipeline.

### **Steps to Integrate GitHub and Set Up the Pipeline:**

1. **Access Project Settings:**
    
    * Navigate to the newly created Azure DevOps project.
        
    * In the project, click on **Project settings** located in the bottom-left corner of the Azure DevOps interface.
        
2. **Configure GitHub Connection:**
    
    * Under **Pipelines**, select **GitHub connections**.
        
    * You should see your GitHub repository already listed since it's been added beforehand.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724064622695/b548d5dd-cf4d-4161-a3c7-10432aeaf55a.png align="center")
        
3. **Connect to the GitHub Repository:**
    
    * If the repository is not already connected, click **Add Connection**, then select your GitHub repository from the list.
        
    * Authenticate with GitHub if prompted, and grant Azure DevOps the necessary permissions to access your repository.
        
4. **Set Up the Pipeline:**
    
    * Now that your GitHub repository is connected, go back to the **Pipelines** section in Azure DevOps.
        
    * Click on **Pipelines** &gt; **New pipeline**.
        
    * Select **GitHub** as the repository source.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724064799505/22398d65-83f8-4be3-8e86-12c905933865.png align="center")
        
    * Choose the appropriate repository where your Terraform code is stored.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724064729232/291673cb-7949-4825-b658-8af50a93bd95.png align="center")
        
    * Follow the prompts to set up your pipeline, starting with a basic YAML pipeline or importing an existing one.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724064897450/52c1d13b-7a2a-4536-a1c5-b4812dd637c6.png align="center")
        
5. **Configure the Pipeline for Terraform Operations:**
    
    * Install Terraform Extention
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724065137197/98b98e8e-9e05-4327-9525-5c86f9a6401c.png align="center")
        
        At the **Organization setting** click on the extension and then browse the extension. Search for Terraform and install below two extensions below following the further prompts:
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724065268676/a06accd4-d1b0-484b-b21a-2a421fc929c6.png align="left")
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724065416451/3d7821ff-49a0-43a4-b60f-b7609a26e295.png align="center")
        
        Once installed you will be able to see the below assistance while writing the pipeline code
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724065622811/821d5c05-e28e-40a7-9f96-ccc597008aff.png align="center")
        
    * In the pipeline configuration, define the steps for Terraform initialization, planning, and applying.
        
    * Ensure the pipeline includes the correct service connections and variables required for Terraform to authenticate with Azure and manage resources.
        

# Pipeline Creation

## Build Pipeline

In our Terraform build pipeline, the primary objective is to automate the essential steps of infrastructure provisioning, ensuring consistency, compliance, and repeatability. The pipeline is designed to carry out key Terraform operations, leading up to the creation and storage of the `tfstate` file as an artifact. This artifact will then be utilized in the release pipeline for further stages of infrastructure deployment.

### **Pipeline Stages**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724067092037/e9663d15-74b3-41e0-841f-01d15b58467d.png align="left")

1. **Terraform Initialization (**`terraform init`):
    
    * The first stage in our pipeline is initializing Terraform. This step configures the backend and prepares the environment for Terraform operations. It ensures that Terraform has the necessary plugins and access to the remote state file.
        
        ```yaml
        trigger:
         - main
        pool: 
          name: Default
        
        stages:
          - stage: Terraform
            jobs:
              - job: Build
                steps:
                - task: TerraformTaskV4@4
                  displayName: Terraform Init
                  inputs:
                    provider: 'azurerm'
                    command: 'init'
                    backendServiceArm: 'Pay-As-You-Go(4accce4f-9342-4b5d-bf6b-5456d8fa879d)'
                    backendAzureRmResourceGroupName: 'storage-rg'
                    backendAzureRmStorageAccountName: 'storetfaccritesh'
                    backendAzureRmContainerName: 'statefilestore'
                    backendAzureRmKey: 'prod.terraform.tfstate'
        ```
        
2. **Terraform Validation (**`terraform validate`):
    
    * The pipeline then validates the Terraform configuration files to ensure they are syntactically correct and consistent with the defined standards. This step is crucial to catch any errors before they propagate further in the pipeline.
        
        ```yaml
        - task: TerraformTaskV4@4
          displayName: Terraform Validate
          inputs:
                    provider: 'azurerm'
                    command: 'validate'
        ```
        
3. **Terraform Formatting (**`terraform fmt`):
    
    * Formatting is an important practice to maintain a consistent code style across the team. This step automatically formats the Terraform configuration files according to the standard convention, improving readability and collaboration.
        
        ```yaml
        - task: TerraformTaskV4@4
          displayName: Terraform Format
          inputs:
                    provider: 'azurerm'
                    command: 'custom'
                    outputTo: 'console'
                    customCommand: 'fmt'
                    environmentServiceNameAzureRM: 'Pay-As-You-Go(4accce4f-9342-4b5d-bf6b-5456d8fa879d)'
        ```
        
4. **Terraform Plan (**`terraform plan`):
    
    * This stage generates an execution plan, outlining the changes Terraform will make to the infrastructure. The plan is saved to a file, providing a preview of the modifications before any resources are applied.
        
        ```yaml
        - task: TerraformTaskV4@4
          displayName: Terraform Plan
          inputs:
                    commandOptions: '-out $(Build.SourcesDirectory)/tfplanfile' # this will save the plan to a file name tfplanfile
                    provider: 'azurerm'
                    command: 'plan'
                    environmentServiceNameAzureRM: 'Pay-As-You-Go(4accce4f-9342-4b5d-bf6b-5456d8fa879d)'
        ```
        
5. **Archiving the** `tfstate` File:
    
    * Once the Terraform plan is created, the pipeline archives the Terraform state (`tfstate`) file, which contains the latest state of the infrastructure. This file is crucial for managing the lifecycle of the resources and is stored as an artifact.
        
        ```yaml
        - task: Bash@3
          displayName: Install zip utility
          inputs:
                    targetType: 'inline'
                    script: 'sudo apt-get update && sudo apt-get install -y zip'
        - task: ArchiveFiles@2
          displayName: Archive Files
          inputs:
                    rootFolderOrFile: '$(Build.SourcesDirectory)'
                    includeRootFolder: true
                    archiveType: 'zip'
                    archiveFile: '$(Build.ArtifactStagingDirectory)/$(Build.BuildId).zip'
                    replaceExistingArchive: true
        ```
        
6. **Publishing the** `tfstate` Artifact:
    
    * Finally, the archived `tfstate` file is published as an artifact, making it available for the release pipeline. This ensures that the release pipeline has access to the accurate and up-to-date state of the infrastructure for further deployment stages.
        
        ```yaml
        - task: PublishBuildArtifacts@1
          displayName: Publish Artifact
          inputs:
                    PathtoPublish: '$(Build.ArtifactStagingDirectory)'
                    ArtifactName: '$(Build.BuildId)-build'
                    publishLocation: 'Container'
        ```
        

### **End Goal**

The build pipeline culminates in the creation and preservation of the `tfstate` file, a critical component for managing infrastructure as code (IaC) in Terraform. By archiving and publishing this state file as an artifact, we enable a seamless transition to the release pipeline, where the actual deployment and management of cloud resources will take place.

This approach not only ensures a structured and automated workflow for infrastructure provisioning but also enhances collaboration and reliability by maintaining the integrity of the Terraform state throughout the CI/CD process.

## Release Pipeline

The release pipeline is designed to take the output generated from the build pipeline—specifically, the Terraform plan encapsulated within the `tfstate` file—and proceed to the deployment phase. This process ensures that the infrastructure changes are thoroughly reviewed and approved before being applied to the environment.

### **Pipeline Stages:**

1. **Fetch Artifact from Build Pipeline:**
    
    * The release pipeline begins by retrieving the artifact generated in the build pipeline. This artifact contains the Terraform plan and the `tfstate` file, which captures the desired state of the infrastructure. In the screenshot below, you can see that the build has been configured along with a continuous release trigger.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724086060020/091f78bd-fce2-4506-b0c9-854747984793.png align="center")
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724086508876/541bac43-6d74-428f-996f-1a6c2d605aeb.png align="center")
        
2. **Agent Configuration**
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724086651560/cbb4c901-d049-49a4-817e-f730171b52eb.png align="center")
    
    This task will acquire our self-hosted agent on which it will execute the further terraform operations.
    
3. **Unarchiving Build Artifact**
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724087257105/c68edbc5-ac09-4373-86d6-dd092b899a43.png align="center")
    
    At this stage, we are unarchiving the zipped artifacts downloaded from the build pipeline.
    
4. **Terraform Init**
    
    The `Terraform Init` stage in the release pipeline is crucial for ensuring that Terraform is properly configured and ready to apply the infrastructure changes. This stage is necessary because the release pipeline may be executed on a different machine, container, or pod than the build pipeline. In such cases, Terraform needs to be initialized in the new environment before it can apply the generated artifact.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724087558905/2f759d86-2975-4756-8d1f-62aa9ee8ba6d.png align="center")
    
    The rest of the configuration will be the same as the build pipeline.
    
5. **Apply Terraform Apply:**
    
    * After the artifact is fetched, the pipeline is set to run the `terraform apply` command using the `tfstate` file. This step will execute the planned changes, provisioning or updating the infrastructure according to the specifications defined in the Terraform code.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724087873982/b6691b61-98b4-46d5-ba46-c15af7c8449a.png align="center")
        
        After the successful initialization of Terraform, the next crucial step in the release pipeline is to apply the infrastructure changes. Since the Terraform plan (`tfstate` file) has already been downloaded as an artifact from the build pipeline, we can proceed directly to the `apply` task. This task will apply the changes defined in the plan to the target environment. To streamline the process, we include the `--auto-approve` flag in the apply command.
        
6. **Post-Approval Execution:**
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724089047332/05fd10f3-0e84-494c-8571-900d0b59fc54.png align="center")
    
    Before applying the changes, the pipeline includes an approval gate. This requires a manual review and approval from the designated stakeholders, ensuring that the infrastructure changes are scrutinized and verified before execution. Once the approval is granted, the pipeline proceeds with the application of the Terraform plan.
    

### **Outcome:**

The release pipeline ensures a controlled and automated deployment process. By fetching the artifact from the build pipeline and running the `terraform apply` post-approval, we maintain a high level of governance and security over infrastructure changes. This setup allows for a smooth and reliable transition from planning to execution, adhering to best practices in continuous deployment and infrastructure as code (IaC).

### Destroy Stage

After the infrastructure has been successfully deployed, there may be scenarios where we need to clean up the resources or recreate them from scratch. To facilitate this, we’re incorporating a `Terraform Destroy` stage in the release pipeline. This stage ensures that any unnecessary or outdated infrastructure can be safely and efficiently decommissioned.

### **Purpose of the Destroy Stage:**

* **Resource Cleanup:** The `destroy` stage is crucial for cleaning up resources that are no longer needed, helping to minimize costs and maintain a clean environment.
    
* **Infrastructure Rebuild:** In cases where you need to recreate the infrastructure, the destroyed stage allows for a complete teardown before the infrastructure is rebuilt, ensuring no residual configurations or resources are left behind.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724088471593/47e4da01-e8a2-4abe-a40f-d2e88d48687e.png align="center")

All the stages will remain the same as the Deployment stage, but the apply task will be replaced by Destroy in the Destroy Stage as shown below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724089944710/d559936c-206d-467a-8f53-a463ebcee25c.png align="center")

### Pre-Destroy Approval

An approval stage has been added before destroying the infrastructure so that the approved can review what resources are going to be impacted as part of this destruction.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724088869754/e3b829b1-0cbf-4f9f-a731-8a748040e5a4.png align="center")

# End-to-End Execution

I have triggered an end-to-end run that will build the latest artifact, publish the artifact to the release pipeline, and then apply the infrastructure based on the generated plan artifact. Once approved, it can also destroy the same.

* The plan has been completed after triggering.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724090201230/d9469f04-91bf-43ce-b50c-5cc072bde755.png align="center")

* Post completion, a release has been triggered and waiting for approval
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724090287171/9f87bef0-dde4-47aa-86c9-c61f4763a041.png align="center")

* Post Approval the deployment has been started
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724090353352/706c46da-4916-4712-af94-6ea5b697373b.png align="center")

* And now the infrastructure has been deployed with Terraform apply task completion
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724091548528/0b368ba3-90ca-4149-8a96-06fe8b8ed253.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724091593740/983f56ae-dfc0-4f8a-850f-899b63ccace1.png align="center")

* You can see that 5 resources have been added which are being displayed on Azure Portal
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724091815405/4c829a2d-09e7-4c1d-b47d-1e0154e19562.png align="center")

* After completion, it is now awaiting approval for destruction.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724091694078/18600961-9c07-4131-a5d3-b939000ab7d2.png align="center")

* Once destruction is approved it will again proceed with destruction.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724091758178/146be488-46b3-4329-9818-b46742a55677.png align="center")

* Now you can see below destruction has started
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724091890330/16e409bc-db48-4eee-b202-edb3dc7ca586.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724092140584/15663049-8b9b-4353-bf8c-ed8baabf37a7.png align="center")

# Conclusion

And finally, after 3.37 minutes, the destruction was completed successfully. This marks the successful completion of our fully automated, end-to-end infrastructure management process, integrated seamlessly with Azure DevOps.

Throughout this journey, we’ve demonstrated how to build, publish, and apply infrastructure artifacts using Terraform within an Azure DevOps pipeline. From initializing Terraform in various environments to handling complex tasks such as artifact archiving, approval workflows, and automated cleanup, we've covered every step necessary to manage infrastructure efficiently and reliably. This process not only ensures that infrastructure changes are executed in a controlled and repeatable manner but also empowers teams to maintain agility and scalability in their cloud environments.

The ability to automate everything from provisioning to destruction, all within a unified pipeline, underscores the power of combining Terraform's infrastructure as code capabilities with the robust CI/CD features of Azure DevOps. This integration enables us to manage our cloud resources with precision, ensuring that deployments are consistent, auditable, and aligned with best practices.

# Repository

**Explore the code** and configurations used in this setup on GitHub: [Azure DevOps and Terraform Integration](https://github.com/ritesh-kumar-nayak/azuredevops-integration-terraform)