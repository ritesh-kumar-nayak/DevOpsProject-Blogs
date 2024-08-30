---
title: "Managing Containers with Azure DevOps"
datePublished: Fri Aug 30 2024 10:35:41 GMT+0000 (Coordinated Universal Time)
cuid: cm0gkvqar000j09jqalg4hvpo
slug: managing-containers-with-azure-devops
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1725013499925/feaa777c-2fa5-4480-9d17-07358bba2f83.png
tags: docker, azure, containerization, azure-pipelines, azure-devops, containerization-vs-virtualization, azuredevops, acr

---

## What's inside?

* Understanding Virtual Machines vs Containers
    
* Challenges with non-containerized applications
    
* Containerize a Webapp
    
* What are Azure Container Instances(ACI)
    
* Azure DevOps CI/CD pipeline to deploy to ACI
    

# VMs vs Containers

### Virtual Machines - VMs

**Architecture**: VMs are hosted on a hypervisor, either Type 1 or Type 2, which resides between the hardware and the operating systems. Each VM contains its own complete operating system in addition to the application, and more importantly, the necessary binaries and libraries.

**Isolation**: VMs offer strong isolation because an OS of its own is present on each VM along with its resources. It also has the potential to incorporate higher, stronger overhead isolation.

**Resource Utilization**: VMs are pretty resource-intensive, as an instance of a full OS has to run for each VM. This can actually consume much more in terms of resources and can also have a long starting time.

**Flexibility**: VMs are excellent for running applications requiring full OS environments, really complex applications, or legacy software.

**Management**: VMs are commonly managed like a full OS with updates, patches, and configurations; therefore, management could be cumbersome.

### **Containers**

1. **Architecture**: For instance, containers run on a shared OS kernel but are isolated from each other. They package an application and its dependencies into a single unit, which is more lightweight than a VM.
    
2. **Isolation**: Containers offer a lighter and faster way of providing isolation: they can offer process-level isolation, not whole-OS isolation. This may be less secure than VMs.
    
3. **Resource Utilization**: Containers do not require a full OS and share a kernel with the host, which generally leads to much more resource effectiveness, decreasing overhead and thus improving rise times.
    
4. **Flexibility**: Containers are particularly amenable to microservices architectures, continuous integration/continuous deployment (CI/CD) pipelines, and high scales of deployment.
    
5. **Management**: Container management holds only orchestration tools on the container plane, such as Kubernetes, and wrangling with container images—this can be easier compared to full VMs.
    

### **When to Use What**

* **VMs**: Use VMs when you need to run applications that require a full OS or when working with legacy systems. They are also useful when strong isolation is necessary.
    
* **Containers**: Use containers for developing and deploying microservices, when you need rapid scaling, or when you want to maximize resource efficiency.
    

# Architecture

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724851696937/0a615ad6-29bc-46f7-92bd-565d0afe960f.png align="center")

Here we'll first dockerize the application by creating the dockerfile followed by a container image out of it.

Then we'll store the image in **Azure Container Registry** which is a private registry for storing your container images, and ACI can pull these images to create and run containers. This integration allows you to easily manage your container lifecycle within the Azure ecosystem.

# Azure DevOps Setup

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724595884768/1af39beb-636e-47d6-a814-8a6b01d92336.png align="center")

In our latest project, "Containerization-CI-CD," we've successfully set up the repository and migrated the codebase to Azure Repos, laying the foundation for streamlined container management and continuous integration and delivery.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724596154233/3f2e1d18-1b16-47d0-8f62-0df4e7329105.png align="center")

# Dockerfile

Before proceeding to pipeline creation for build and release we need to have the Dockerfile ready to be used to build the Docker image and a few other pre-requisites like **Azure Container Registry(ACR)** and **Azure Container Instance(ACI)**

```yaml
FROM node:18-alpine AS Installer
WORKDIR /app
COPY package.json ./**
RUN npm install
COPY . .
RUN npm run build
FROM nginx:latest AS Deployer
COPY --from=Installer /app/build usr/share/nginx/html
```

# Azure Container Registry

Azure Container Registry (ACR) is a fully managed, private container registry service provided by Microsoft Azure. It allows storing and managing container images and other related artifacts like Helm charts and OCI artifacts in a secure and scalable environment.

As of now, the Dockerfile is ready. Once the image is built from the Dockerfile, it will be stored in the Azure Container Registry (ACR).

## ACR Creation

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724854080452/b9123597-1779-4e0b-85b6-fad967505859.png align="center")

Search for Container Registries and click on the container registries as highlighted followed by Clicking on Create. Fill up the details and make sure the **"Registry Name"** has to be unique across the globe as the registry will be accessed with the givenName.azurecr.io i.e. **devopswithritesh.azurecr.io** in our case.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724898141078/19b2a36b-7684-416b-87bb-041d31d2b1be.png align="center")

We have chosen the **Standard Pricing Plan** for this demo hence, private access is not available. And now our registry has been created successfully

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724922687453/3af53930-1013-4d7c-ba00-dab4c42f89d6.png align="center")

Once the registry is created, we need to capture the username and password, which will be required when pushing the image to ACR. Under Settings, find the access key and enable the **Admin User** checkbox to view the password. You can then upload this information to Azure KeyVault or store it securely elsewhere.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724923181193/94800c7f-634d-40a7-a4c9-ab7fe7176006.png align="center")

# Pipeline

With our Azure Container Registry (ACR) set up, we can now proceed to create an Azure DevOps Pipeline to automate the build and publishing of container images. This pipeline will streamline the process, ensuring that every update is efficiently built and securely stored in our registry, ready for deployment.

### Azure Repo Integration

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724925791439/6645c43f-c9a0-480d-a528-46e83efb02b6.png align="center")

After selecting Azure Repos Git as the source, you'll be presented with a list of available Azure Repos that can be integrated into the pipeline. From this list, you can choose the specific repository you'd like to connect, allowing seamless integration for your pipeline setup.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724926864150/d1990312-b645-45f0-8e70-2247c9e7000c.png align="center")

At the configuration step, you can select the '**Docker Build and Push to Container Registry**' option. This choice provides a pre-built template tailored for Docker image builds and pushes to your Azure Container Registry. The template is fully customizable, allowing you to tailor the pipeline to our specific needs as well.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724927050101/33a647df-7eac-47b5-8f34-42a3d198ab22.png align="center")

### Container Registry Integration

After selecting the 'Docker Build and Publish' option, you'll be prompted to choose your Azure subscription and authorize access. Once authorization is complete, you can then select the Container Registry, specify the image name, and define the Dockerfile location as shown below, and finally click on Validate and Configure.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724927617891/677806e7-41db-4631-9d73-f7d34d15ad13.png align="center")

After clicking on 'Validate and Configure,' a template pipeline code is automatically generated. However, we've made several modifications to the pipeline code to better suit our needs. The updated pipeline code is now as follows

### Pipeline Code

```yaml
# Docker
# Build and push an image to Azure Container Registry
# https://docs.microsoft.com/azure/devops/pipelines/languages/docker

trigger:
- main
pool:
  name: Default

resources:
- repo: self

variables:
  # Container registry service connection established during pipeline creation
  dockerRegistryServiceConnection: 'XXXXXXXX-XXXX-XXXX-XXXXXX'
  imageRepository: 'todoapp'
  containerRegistry: 'devopswithritesh.azurecr.io'
  dockerfilePath: '$(Build.SourcesDirectory)/Dockerfile'
  tag: '$(Build.BuildId)'

stages:
- stage: Build
  displayName: Build and push stage
  jobs:
  - job: Build
    displayName: Build
    steps:
    - task: AzureCLI@2
      inputs:
        azureSubscription: 'Pay-As-You-Go(XXXXXXXXX-XXXXXXX)'
        scriptType: 'bash'
        scriptLocation: 'inlineScript'
        inlineScript: 'az acr login --name=$(containerRegistry)'
    # Task to build and push the image to ACR
    - task: Docker@2
      displayName: Build and push an image to container registry
      inputs:
        command: buildAndPush
        repository: $(imageRepository)
        dockerfile: $(dockerfilePath)
        containerRegistry: $(dockerRegistryServiceConnection)
        tags: |
          $(tag)

    #Task to create Container Instance(ACI)
    - task: AzureCLI@2
      inputs:
        azureSubscription: 'Pay-As-You-Go(XXXXX-XXXXXXXXX-XXXXXX)'
        scriptType: 'bash'
        scriptLocation: 'inlineScript'
        inlineScript: |
          az container create \
          --name todo_container \
          --resourse-group Contaiinerization_with_Azdo \
          --image $(containerRegistry)/$(imageRepository):$(tag) \
          --registry-login-server $(containerRegistry) \
          --registry-username devopswithritesh  \
          --registry-password XXXXXXXX \
          --dns-name-label aci-devopswithritesh
```

# Azure Container Instance(ACI)

Azure Container Instances (ACI) is a fully managed service provided by Microsoft Azure that allows users to run containers directly on the Azure cloud without the need to manage any underlying virtual machines or orchestrators like Kubernetes. ACI is ideal for scenarios where you need to quickly deploy and manage containers without the overhead of managing infrastructure.

### Key Features of Azure Container Instances:

1. **Simplicity and Speed**: ACI offers a straightforward way to deploy containers. You can start running containers within seconds, making it an excellent choice for tasks that require fast and temporary computing resources.
    
2. **No Infrastructure Management**: ACI abstracts the underlying infrastructure, allowing you to focus solely on your containers. There's no need to manage or scale virtual machines, patch operating systems, or configure orchestrators.
    
3. **Scalability**: ACI enables easy scaling of containerized applications. You can adjust the CPU and memory resources allocated to your containers as needed, ensuring that your application can handle varying workloads.
    
4. **Cost-Effective**: ACI operates on a pay-as-you-go pricing model, meaning you only pay for the compute resources your containers use. This makes it cost-effective, especially for short-lived or bursty workloads.
    
5. **Seamless Integration with Azure Services**: ACI integrates well with other Azure services, such as Azure Virtual Network, enabling you to deploy containers in a secure and isolated environment. It also integrates with Azure DevOps, allowing for smooth CI/CD pipeline setups.
    
6. **Event-Driven Containers**: ACI can be used for event-driven scenarios, such as processing tasks from an Azure Event Grid, Azure Service Bus, or Azure Functions, allowing for a dynamic response to changes in your environment.
    

```yaml
#Task to create Container Instance(ACI)
    - task: AzureCLI@2
      inputs:
        azureSubscription: 'Pay-As-You-Go(XXXXX-XXXXXXXXX-XXXXXX)'
        scriptType: 'bash'
        scriptLocation: 'inlineScript'
        inlineScript: |
          az container create \
          --name todo-container \
          --resourse-group Contaiinerization_with_Azdo \
          --image $(containerRegistry)/$(imageRepository):$(tag) \
          --registry-login-server $(containerRegistry) \
          --registry-username devopswithritesh  \
          --registry-password XXXXXXXX \
          --dns-name-label aci-devopswithritesh
```

This task creates an Azure Container Instance (ACI) named **"todo-container"** using the specified Docker image. The container is deployed within the resource group **"Containerization\_with\_Azdo"**. The application, packaged within the Docker image, will run in the container and be exposed on the port defined in the Docker file.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725008408588/66101630-38f6-4e62-8775-eeb65183a4e6.png align="center")

Once the container is successfully created, the application will be accessible via the Fully Qualified Domain Name (FQDN) associated with the container. This FQDN can be used to directly access the application from any browser or HTTP client.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1725008517530/bc433d89-612b-4dc3-9ede-38cf0679a4ec.png align="center")

# Conclusion

Containerization with Azure DevOps offers a powerful and streamlined approach to modern application deployment. By leveraging Azure Container Instances (ACI), we can efficiently deploy, manage, and scale containerized applications with ease. The integration of Azure DevOps pipelines automates the entire process, from building and pushing Docker images to creating and configuring container instances. This not only enhances deployment speed and reliability but also ensures that applications are readily accessible via fully qualified domain names (FQDNs). As organizations continue to embrace cloud-native technologies, mastering containerization with Azure DevOps becomes an essential skill for delivering robust, scalable, and efficient applications in today's fast-paced digital landscape.