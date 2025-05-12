---
title: "DevOps Interview Questions-2025"
datePublished: Mon May 05 2025 05:23:56 GMT+0000 (Coordinated Universal Time)
cuid: cmaamy2c3000e09l264p93nbu
slug: devops-interview-questions-2025
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1745390599614/df1b15a3-44b1-48c7-b1b9-8b0ca2a147bd.jpeg
tags: devops, interview-questions, devops-interview-questions-and-answers

---

# *Git Questions*

1. **How to initialize Git locally?**
    
    **Ans**: We can initialize git using the `git init` command\*\*.\*\* Once git is initialized, a .git directory will be created which is responsible for handling all git operations.
    
2. **What is a Pre-Commit Hook?**
    
    **Ans**: A pre-commit hook is a script  that executes before a commit is made. These hooks are part of *git’s native hook framework* and are located in the `.git/hooks` directory.
    
    Pre-commit hooks are extremely helpful in preventing sensitive information like passwords, secrets, access tokens, etc, from being committed to git. When a user runs git commit, the pre-commit is triggered before the commit is finalized, and if the hook fails, then the commit is aborted.
    
3. **Which command is used to compare the changes in your working directory with the last committed version in the repository?**
    
    **Ans**: `git diff` command is used to compare changes in your working directory with the last committed version in the repository. It also allows you to see the difference between two commits.
    
    *E.g.:* `git diff <commit_id1> <commit_id2>`
    
4. **What is git fork?**
    
    **Ans**: `git fork` is used to create a copy of the repository to your account, where you can modify it and create your own version of the repository.
    
5. **Can you explain the usage of git cherry-pick?**
    
    **Ans**: git cherry-pick is the command that is used to pick a particular commit and merge it into the main. It is useful when you want to pick up a specific commit only. You have to be on the main branch before cherry-picking.
    
    E.g.: `git cherry-pick <commitid>`
    
6. **What is the difference between** `git merge` **and** `git rebase`**?**
    
    **Ans**: Both git merge and git rebase serve the same purpose, that is, merging your code from a feature branch to the main branch, but in 2 different ways as follows:
    
    When you use git rebase, you get a linear commit history, but when you use git merge, it creates a commit at the top, disturbing the sequence.
    
7. **What is the difference between** `git pull` **and** `git fetch`**?**
    
    **Ans**: `git fetch` command only informs you about the changes that are there on the remote but not available on your local working copy, but it does not bring the changes to your local code base, whereas git pull directly updates your local code base with the new changes from the remote repository.
    
    `git pull == git fetch + git merge`
    
8. **What are pre-commit hooks and post-commit hooks?**
    
    **Ans**: A hook is something that you want to run before and after the occurrence of an action. So, pre-commit hooks are actions taken before you do the commit:
    
    ***Example***\*:\* when you have a public key, private key, or password that you do not wish to be pushed to git then you can configure a pre-commit hook to execute a script that checks a certain pattern that will scan the code before every commit you make.
    
    Similarly, post-commit hooks are nothing but actions configured to be run post a commit is made:
    
    ***Example***\*:\* running static analysis tools, linters, or code formatters to ensure committed code adheres to defined quality standards.
    
9. **What is** `git stash`**, and what are its use cases?**
    
    **Ans:** git stash is the command that is used to temporarily save the changes made to your working copy.
    
    ***Use-Case:***
    
    * ***Temporarily saving the work:*** *When you need to switch branches to work on something urgent but don’t want to commit the incomplete changes, then you can do* `git stash` *—&gt;* `git checkout another_branch` *—&gt; work on the other branch —&gt;* `git checkout original_branch` *—&gt;* `git stash pop`*.*
        
    * ***Pulling Updates without losing the work:*** *if you need to pull the latest changes from the remote repository, but you have some uncommitted changes, then you can do* `git stash` *—&gt;* `git pull` *—&gt;* `git stash pop`*.*
        
    * ***Resolving merge conflicts:*** *if merge or rebase fails due to uncommitted changes, then you can do* `git stash` *—&gt;* `git merge target_branch` *—&gt; resolve the conflicts—&gt;* `git stash pop`
        
10. **How to amend a commit in git?**
    
    **Ans:** When you have made a commit with some mistake and pushed it to the remote repository, and now you want to rectify that particular commit, then you can use `git commit --amend,` It allows you to modify the most recent commit
    
11. **How do you revert a commit that has already been pushed and made public?**
    
    **Ans:** You can undo the changes by making another commit using the command `git revert <commit_id>`
    
12. **What is the difference between** `git stash apply` **and** `git stash pop`**?**
    
    **Ans:** The key difference between git stash pop and git stash apply lies in how they handle the stash after applying the changes in your working directory:
    
    * `git stash pop` Applies the stash and removes it from the stash stack permanently. After running git stash pop, the stashed changes are restored to your working directory and stash id is deleted from the stash stack. It should be used when you are sure that you do not need that stash after applying it because once your working copy is restored, the stash will be removed.
        
    * `git stash apply` Applies the stash but keeps it in the stash stack. After running the git stash apply, the stashed changes are restored to your working directory but the stash remains in the stack for future use. It is useful when you want to keep the stash for future use.
        
13. **How to find a list of files that are changed during a commit?**
    
    **Ans:** `git diff-tree -r <commit_id>`
    
14. **How to check whether a branch has already been merged to the master or not?**
    
    **Ans:** `git branch –merged` gives the list of merged branches, and `git branch –not-merged` gives the list of unmerged branches.
    
15. **How to remove a file from your git without removing it from your local filesystem?**
    
    **Ans:** using `git reset <filename>` which is exactly opposite to the git add command.
    
16. **What is the difference between git revert and git reset?**
    
    **Ans:** git reset is used to return the entire working tree to the last committed state; it also resets the file from the staging area, whereas the git revert command adds a new history to the project and it does not modify the existing history.
    
17. **What is** `git bisect`, **and how do you use it to determine the source of a bug?**
    
    **Ans:** git bisect is used to find a commit that introduced a bug by using a binary search algorithm. You can use it by giving it a bad commit ID, which you think could be a potential reason for the bug being introduced, and a good commit up to which everything was working fine. Then this command picks the commits between these 2 endpoints and asks you whether the selected commit is good or bad. It keeps narrowing down the commits until it finds the buggy commit.
    
18. **What is** `git reflog`**?**
    
    **Ans:** It keeps track of every change made in a repository's reference. It shows deleted, renamed, and all other actions when executed.
    

# *Docker Questions*

1. **What is Docker?**
    
    **Ans:** Docker is an open-source containerization platform. It enables developers to package applications into containers. We have used Docker to build Docker images out of Docker files for lightweight application packaging.
    
2. **How are containers different from Virtual Machines?**
    
    **Ans:** Containers are very lightweight as they do not have a complete OS and its utilities. Containers have very minimal OS functionality and only require dependencies.
    
    Docker provides *<mark>process-level isolation,</mark>* whereas Virtual Machines provide stronger isolation.
    
3. **What is Docker Lifecycle?**
    
    **Ans:** The general flow of the Docker lifecycle is as follows:
    
    * Step 1: Create a Dockerfile with a set of instructions
        
    * Step 2: Building the Docker image from the Docker file
        
    * Step 3: Push the image to image registry like Docker Hub, ACR, and ECR
        
    * Step 4: Create the container out of the Docker image.
        
    
    A Docker image acts as a set of instructions to build a container, and it can be compared to a Snapshot in a VM.
    
4. **What are the different Docker components?**
    
    **Ans:** Docker consists of several key components, such as:
    
    * **Docker Engine** which includes the Docker daemon for managing resources and the Docker CLI for user interaction.
        
    * **Docker Images** serve as a blueprint for containers.
        
    * **Docker Containers** are lightweight and portable instances for images.
        
    * **Docker Registry** for storing images and sharing them.
        
    * **Docker Volume** enables data persistence.
        
    * **Docker Network** provides connectivity and isolation for containers.
        
    * **Docker Compose** simplifies the management of multiple containers.
        
5. **What is the difference between Docker** `COPY` **and** `ADD`**?**
    
    **Ans:** `docker ADD` can copy files from a URL unlike `Docker COPY` can only copy files from host system into the container.
    
6. **What is the difference between** `CMD` **and** `Entrypoint` in Docker?
    

**Ans:**

| **CMD** | **ENTRYPOINT** |
| --- | --- |
| Specifies the <mark>default</mark> command to execute when a container starts. | Specifies the main command that <mark>always executes </mark> when the container starts. |
| It can be <mark>overridden </mark> at runtime by providing an additional argument during docker run. | It is less likely to be overridden as it is treated as the container’s <mark>primary process</mark>. |
| CMD is best for providing default arguments that can be changed at runtime. | ENTRYPOINT is ideal for primary non-negotiable processes like a service or a script. |

We mostly use ENTRYPOINT and CMD together for better flexibility.

7. **What is the default networking in Docker, and explain networking types?**
    
    **Ans:** Docker supports several networking types; however **<mark>bridge network </mark>** <mark>is the default</mark> networking in Docker. The following is a list of networking types that Docker supports:
    
    * ***Bridge Network (Default):***
        
        A private network where containers can communicate with each other through their IP addresses. It is ideal for standalone containers needing limited communication with the host or external network. Here, containers get a private IP address, and you can use -p or —publish to expose their services.
        
        `docker network ls` Can show you the available networks
        
    * ***Host Network:***
        
        A host network <mark>removes the network’s isolation between the container and the host.</mark> The container directly uses the host’s network stack. It is ideal for performance-sensitive applications where networking overhead needs to be minimized. No additional port mapping required, anyone having access to the host IP can access the application with `hostip:port` .
        
        `docker run —network host my-container` The command can be used to run a container with the host network.
        
    * ***Overlay Network:***
        
        It allows communication between containers across multiple Docker hosts in a swarm or K8S cluster. It is ideal for <mark>distributed systems or multi-host environments</mark>.
        
        `docker network create -d overlay my-overlay-network` Command is used to create an overlay network.
        
    * ***MacVLAN Network:***
        
        It assigns containers their own <mark>MAC address </mark> and allows them to <mark>appear as physical devices</mark> on the network. It is ideal for a legacy application requiring direct access to the physical network.
        
        `docker network create -d macvlan —subnet=192.168.10/24 my-macvlan-network` This command can be used to create a MacVLAN network.
        
    * ***Custom-Bridge Network:***
        
        Similar to the default bridge network, but allows user-defined configuration for better control. Containers on the same custom bridge network can resolve each other by container names.
        
        `docker network create my-bridgenetwork` This command can be used to create a custom network.
        
    * ***None-Network:***
        
        It disables networking for the container completely and is useful for security purposes or a container that doesn't require networking
        
        `docker run —network none mycontainer`
        
8. **Can you explain how to isolate networking between containers?**
    
    **Ans:** By default, all the containers get deployed on the bridge network, which is Docker eth0. To isolate and secure a container, you can create a customer bridge network and assign it to the targeted container.
    
9. **What is a multistage build in Docker?**
    
    **Ans:** Multistage build in Docker allows you to build your Docker container in multiple stages, allowing you to copy only necessary artifacts from one stage to another. The major advantage is that it helps build lightweight containers.
    
    A multistage build contains multiple `FORM` instructions in a single Docker file, for example: you can compile the application in one of the stages (development stage) and copy only the compiled binary to a lightweight runtime image in the final stage. This reduces image size, improves security, and simplifies the build process.
    
10. **What are distro-less images in Docker?**
    
    **Ans:** Distro-less images are minimalist Docker images that include only the application and it required runtime by excluding the OS package managers, shell, and other unnecessary utilities.
    
    They offer smaller image sizes, enhance security by reducing attack surface, and ensure consistency in the production environment.
    
    **NOTE:** Since distro-less images lack system native tools and utilities, the debugging should be done at the application or build pipeline level.
    
    ```yaml
    #Stage1: build the java application
    FROM maven:3.8.7-openjdk-17 as Builder
    WORKDIR /app
    COPY pom.xml
    COPY src ./src
    RUN mvn package
    #Stage2: Use distro-less for runtime
    FROM gcr.io/distroless/java:17
    WORKDIR /app
    COPY --form=builder /app/target/myapp.jar
    CMD ["myapp.jar"]
    ```
    
11. **What are some real-time challenges with Docker?**
    
    **Ans:** Some challenges and disadvantages of Docker are stated as follows:
    
    * Docker is a single daemon process, which can cause a single point of failure. In case the Docker daemon goes down, then all the applications will face downtime.
        
    * Docker daemon runs as the root user, which is a security threat. Any process running with root privileges can have an adverse effect when it is compromised, as it can affect other applications and containers on the host.
        
    * If you are running too many containers on a single host, then you may experience issues with resource constraints. This can result in slow performance or crashes.
        
12. **What step would you take to secure containers?**
    
    **Ans:** We can take the following safety measures to secure containers:
    
    * Use distro-less images so there are fewer chances of CVE or security issues
        
    * Ensure that the networking is configured properly. This is one of the common reasons for security issues. If required, configure a custom bridge network and assign it to isolate containers.
        
    * Use utilities like Docker Scout to scan your container images.
        
13. **You have noticed many stopped containers and unused networks taking up space. Describe how** you would **clean up these resources effectively.**
    
    **Ans:** The `docker prune` command can be used to remove unused resources.
    
    * `docker image prune`: to remove unused images
        
    * `docker container prune`: to remove containers
        
    * `docker volume prune`: to remove volumes
        
    * `docker network prune`: to remove networks
        
    
    Running `docker system prune` combines all the above commands and cleans up all resources that are <mark>not associated with any running containers</mark>.
    
14. **You are working on a project that requires Docker containers to persistently store data. How would you handle persistent storage in Docker?**
    
    **Ans:** Storing data persistently means saving, accessing and reusing the data post the containers are destroyed. This can be achieved in the following 2 ways
    
    ***Docker Volume:***
    
    Docker volume can be used for persistent storage. They are managed by Docker and can be attached to one or more containers. Docker volumes are more reliable and efficient for persisting data <mark>across the container lifecycle</mark>. These are easy to back up and can be maintained independently, even if a container is stopped or removed.
    
    Example: in a production environment, you might create a named volume to persist database data like this
    
    —&gt; Create Docker Volume: `docker volume create db_data`
    
    —&gt; Attach the volume to the container whose data needs to be persisted: `docker run -d —name MySQL -v db_data: /var/lib/mysql mysql:latest`
    
    This ensures the database data remains intact even if the container is stopped.
    
    ***Bind Mounts:***
    
    In some cases, we can use bind mounts where a directory of the host machine is mapped directly to a directory in the container. It is useful in a development environment where you need immediate synchronization between host changes and containers.
    
15. **A company wants to run thousands of containers. Is there any limit on how many containers you can run in Docker?**
    
    **Ans:** There is no limit when it comes to running containers. The only limit is the hardware or machine limit. If the machine does not have enough CPU, it will not be able to launch the containers.
    
16. **You are managing a Docker environment and need to ensure that each container operates within defined CPU and memory limits. How do you limit the CPU and memory of a Docker container?**
    
    **Ans:** Docker allows you to limit the CPU and memory usage for a container using resource constraints. You can set the CPU limit using the `--CPU` and `--memory` options when running the container using the docker run command.
    
    **Example**: `docker run --cpu 2 --memory 1g mycontainer`
    
17. **Can you define the resource limit in the Docker file? Or are there any other ways you can limit resource usage?**
    
    **Ans:** Resource constraints, such as limiting memory and CPU, <mark>cannot be defined </mark> directly in a Docker file. A Docker file is meant for defining build instructions of the image, not runtime behaviors. Instead, the constraints can be applied in the configuration file of orchestrators like Docker Compose and Kubernetes.
    
    `docker run --cpu=”15” --memory=”512m” my image`
    
    Alternatively, using a K8S pod definition, can set a limit like below:
    
    ```yaml
    resources:
        limit:
            memory: "512m"
            cpu: "1500m"
    ```
    
18. **What is the difference between a Docker container and a Kubernetes pod?**
    
    **Ans:** A docker container is a lightweight and <mark>single instance</mark> of an application. It is managed by docker and <mark>provides process level isolation</mark>.
    
    On the other hand, a Kubernetes pod is a high-level abstraction that can contain one or more Docker containers(or other container runtimes).
    
19. **How would you debug issues in a Docker container?**
    
    **Ans:** There are several techniques to debug issues in a Docker container:
    
    * **Logging:** Docker captures the standard output and error logs of a container making it easy to inspect using the docker logs command. So initially, we’ll check the logs of the container.
        
    * **Shell Access:** We can access a running container’s shell using the `docker exec` command, which allows you to investigate and troubleshoot issues interactively.
        
    * **Image Inspection:** You can inspect the Docker image content and configuration using `docker image inspect` to check if there is any misconfiguration.
        
    * **Health Check:** Docker supports defining health checks for containers, allowing you to monitor health status and automatically restart or take actions using pre-defined conditions.
        
    * **Tools:** Additionally, if containers are configured with ELK or Prometheus, then we can make use of these tools to troubleshoot better.
        
20. **Can you describe a situation where you optimized a Dockerfile for faster build times or smaller image size?**
    
    **Ans:** Optimizing Dockerfile could involve various stages like using smaller base images (Alpine images or distro-less images), reducing the number of layers by combining commands, or using multistage builds to exclude unnecessary files from the final image.
    
21. **How do you create a multi-stage build in Docker?**
    
    **Ans:** Multi-stage Docker builds allow you to create optimized images by leveraging multiple build stages. To create a multi-stage build, you define multiple `FORM` instructions in the Docker file, each representing a different build stage and the subsequent stage copies only required artifacts from the previous build using `COPY --from` the instruction. This technique reduces image size by excluding build tools and dependencies from the final image. Each stage can use a different base image.
    
22. **How do you create a custom Docker network?**
    
    **Ans:** To create a custom Docker network, you can use `docker network create` command.
    
23. **An update needs to be applied without any data loss. How would you update a Docker container without losing data?**
    
    **Ans:** Steps to update Docker container without losing data are as follows:
    
    * Check where the application stores the data (i.e.,/var/lib/mysql)
        
    * use `docker inspect <container_name>` to identify mounted volumes or bind mounts. Look for the <mark>mount</mark> <mark>section</mark> in the output to locate the data directory.
        
    * If it is a bind-mount method, then use the docker cp command to copy data from the container to the host machine `docker cp <container name>:<path in container> <path on host>` .
        
    * If it is the Docker volume method, then you can create a tarball for the volume contents by running a <mark>temporary container to access the volume and then tar it</mark>.
        
    * For databases, we can use application-specific backup tools like <mark>mysql dump for mysql or pgdump for PostgreSQL.</mark>
        
    * Then, stop the container using the docker stop command.
        
    * Pull the latest version of the container using docker pull.
        
    * Start a new container with the latest image, making sure to map any volumes or bind mounts and make sure the container is working fine.
        
24. **How do you secure Docker containers?**
    
    **Ans: A** Few ways to secure containers are as follows:
    
    * Use distro-less images having fewer packages and dependencies.
        
    * Use custom networking instead of the default bridge network.
        
    * Use image scanning tools like Docker Scout.
        
25. **Have you ever used Docker in combination with CI/CD?**
    
    **Ans:** Yes, in all the projects we use Docker with Azure DevOps CI/CD. For example, in the current project, we have integrated Docker with Azure DevOps in the build stage. We have used `Docker@2` task in a build job in Azure DevOps and push it to ACR.
    
    We have further made it pull to AKS for deployment. We have also configured Azure App services with the container.
    
26. **How do you monitor Docker containers?**
    
    **Ans:** There are various ways to monitor containers, such as:
    
    * Using Docker built-in commands such as docker stats and docker container stats
        
    * In our project set Docker container monitoring is primarily handled by a dedicated monitoring team using the ELK stack.
        
    * For instance, I ensured all containers had proper log routing to the monitoring teams’ ELK stack by configuring log drivers and standardizing log formats
        
    * The monitoring team created Kibana to visualize container performance, and I ensured that each container is configured properly for logging by setting up log drivers or editing the `/etc/docker.daemon.json` configuration to make it effective globally. This configuration was performed via Ansible during host VM configuration.
        
    * I also work closely with the monitoring team to define relevant metrics such as resource usage, errors, and access logs.
        

# Terraform Questions

1. **What is Terraform, and how does it work?**
    
    **Ans:** Terraform is an IAC(Infrastructure as Code) tool that lets you write code to define and manage your cloud and on-prem infrastructure. With Terraform, you describe the desired state of your infrastructure using Terraform manifests, and Terraform figures out how to achieve that state, and then it interacts with cloud providers to achieve this state. As part of this, Terraform also generates a state file.
    
2. **A DevOps Engineer manually created infrastructure on Azure, and there is a requirement to manage that resource using Terraform. How would you import it into Terraform code?**
    
    **Ans:** This situation is called configuration drift, and yes, we can manage manually created resources via Terraform following the steps below:
    
    * Write the Terraform configuration code for each resource you want to import.
        
    * Run the terraform import command for each resource, specifying the resource type and its unique identifier.
        
        `terraform import azurerm_resource_group.example /subscriptions/<SUBSCRIPTION_ID>/resourceGroups/rg-dev-apps` .
        
    * Verify the import using `terraform show` command will show the latest state.
        
    * Run `terraform plan` to see if there are any discrepancies between the imported state and your configuration.
        
3. **You have multiple environments, such as dev, qa, staging, and prod, for your application, and you want to use the same code for all these environments. How would you do that?**
    
    **Ans:** There are multiple ways we can achieve this, and a few of them will be discussed here:
    
    * ***Using different*** `.tfvars` ***files:*** You can define a separate `.tfvars` file for each environment containing environment-specific variables, and <mark>during the </mark> `terraform apply` <mark>time</mark> you can refer to the respective variable file with the following command:
        
        `terraform apply -var-file=”dev.tfvars”`
        
        `terraform apply -var-file=”qa.tfvars”`
        
    * ***Using Workspace:*** The Terraform workspace allows you to manage state files within the same configuration. Each workspace represents a separate environment with its respective state file. We can create workspaces using `terraform workspace new <workspacename>` the command and switch to each workspace using `terraform workspace select <workspacename>`. And we can use `terraform. workspace` it as a variable like `terraform. Workspace` this in the Terraform manifest for referring to the environments.
        
    * ***Using*** *a* ***dedicated directory structure for each environment:*** *we can create different directory structures for each environment, such as dev, qa, and uat, which will have their own manifests file, but we do not mostly use this as it might be redundant and require more maintenance.*
        
4. **What is a Terraform state file, and why is it important?**
    
    **Ans:** Terraform state file is a JSON or binary file that stores the current state of the managed infrastructure. It is the heart of Terraform. **It is like a blueprint that stores information about the infrastructure you manage.**
    
    It is crucial because it helps Terraform to understand what is already set up and what changes need to be made to the existing infrastructure by comparing the current state and desired state.
    
5. **A DevOps Engineer accidentally deleted the state file. What step should be taken to resolve this?**
    
    **Ans: The** following steps can be taken to resolve this issue:
    
    * ***Recover Backup:*** If available, restore the state file from the recent backup. When the terraform state is managed locally, a `terraform.tfstate.backup` file is created every time someone performs terraform apply. You can simply rename `terraform.tfstate.backup` to `terraform.tfstate` The latest state file has been recovered this way.
        
    * ***Recreate the State file:*** If no backup is available, then we have to manually reconstruct the stateful by inspecting existing infrastructure and using terraform import for each missing resource. This is not the ideal way.
        
6. **What are the best practices for managing the Terraform state file?**
    
    **Ans:** The following are some best practices that we can follow while managing the Terraform state file:
    
    * **Remote State Store:** stores the state file remotely, which uses Azure Blob store and AWS S3 with Dynamo DB.
        
    * **State Locking:** Enable state locking so that the state file will remain intact when multiple people try to make changes simultaneously.
        
    * **Backup:** Enable automated backup for state file safety in case of accidental deletion.
        
    * **Access Control:** Limit access to the state file to authorize users and services.
        
    * **Distinguished Environments:** Finally, create a separate state file for each environment for better management and isolation.
        

**Your team is adopting a multi-cloud strategy, and you need to manage resources on both AWS and Azure using Terraform. How do you structure Terraform code to handle this?**

**Ans:** Terraform is cloud agnostic, which means it supports multi-cloud setups.

***1\. Use Separate Provider Blocks:***

Define providers for **both clouds** in your root [`main.tf`](http://main.tf) or relevant modules:

```yaml
# Azure provider
provider "azurerm" {
  alias   = "azure"
  features {}
}

# AWS provider
provider "aws" {
  alias  = "aws"
  region = "us-east-1"
}
Isolate State Files Per Cloud
```

***2\. Keep separate*** `backend` ***configurations (e.g., S3 for AWS, Azure Blob for Azure):***

```yaml
terraform {
  backend "azurerm" {
    # for Azure
  }
}
```

For AWS setup:

```yaml
terraform {
  backend "s3" {
    # for AWS
  }
}
```

You can also use **workspaces** or **separate pipelines** to manage them.

***3\. Use Workspaces or CI/CD to Manage Environments:***

For `dev`, `qa`, `prod` across clouds, use:

* Named workspaces
    
* Separate `*.tfvars` per environment
    
* Environment-specific modules
    

***4\. Separate Azure and AWS infrastructure into modules:***

```yaml
terraform/
├── main.tf
├── providers.tf
├── aws/
│   └── ec2-instance/
│       └── main.tf
├── azure/
│   └── storage-account/
│       └── main.tf
├── variables.tf
└── terraform.tfvars
```

Then call them like this in the root module:

```yaml
module "azure_storage" {
  source   = "./azure/storage-account"
  providers = {
    azurerm = azurerm.azure
  }
  # pass variables
}

module "aws_ec2" {
  source   = "./aws/ec2-instance"
  providers = {
    aws = aws.aws
  }
  # pass variables
}
```

7. **You want to run some shell scripts after creating your resources with Terraform, so how would you achieve this?**
    
    **Ans:** You can achieve this using provisioners. There are 3 types of provisions in Terraform:
    
    * ***Local Exec Provisioners:*** This provisioner executes commands and scripts locally on your local machine.
        
    * ***Remote Exec Provisioners:*** It executes commands and scripts on a provisioned remote machine.
        
    * ***File Provisioners:*** It moves files to the provisioned resources.
        
    
    In this given scenario, we can use File Provisioners and Remote Exec Provisioners combined, which will copy the script file to the remote server and execute the same, respectively.
    
8. **Your company is looking to enable High Availability. How can you perform blue-green deployment using Terraform?**
    
    **Ans:** To implement a blue-green deployment using Terraform, you can provision two identical environments—commonly referred to as **blue** and **green**—using infrastructure components like **Virtual Machine Scale Sets (VMSS)** in Azure or **Auto Scaling Groups (ASG)** in AWS.
    
    Once the new (green) environment is deployed and thoroughly tested, you can switch traffic from the existing (blue) environment to the green one by updating the **Load Balancer backend pool** or modifying **DNS records**. This approach minimizes downtime and allows easy rollback if issues are detected.
    
9. **Your company wants to automate Terraform through CI/CD pipeline. How can you integrate Terraform with CI/CD pipelines?**
    

**Ans:** Integrating Terraform with a CI/CD pipeline involves setting up an automated workflow that handles infrastructure provisioning and management consistently. Here's a step-by-step guide to achieve this:

***Step 1: Store Terraform Code in a Version Control System (VCS):***

* Push your Terraform configuration files (`.tf` files) to a source code repository such as **GitHub**, **Azure Repos**, or **GitLab**.
    
* Follow a branching strategy (e.g., feature, staging, production branches) to manage infrastructure changes.
    
    ***Step 2: Set Up Remote Backend for State Management:***
    
* Use a remote backend like **Azure Storage Account**, **AWS S3 with DynamoDB**, or **Terraform Cloud** to manage the Terraform state file securely and support collaboration.
    
    ***Step 3: Define CI/CD Pipeline Configuration:***
    
* Create a pipeline file (e.g., `azure-pipelines.yml`, `.gitlab-ci.yml`, or GitHub Actions workflow) with Terraform stages like:
    
    * **Terraform Init**
        
    * **Terraform Validate**
        
    * **Terraform Plan**
        
    * **Terraform Apply** (manual approval for production)
        
    
    ***Step 4: Configure Pipeline Agent and Environment:***
    
    * Use a **self-hosted agent** or **cloud-hosted agent** with Terraform installed.
        
    * Set up environment variables or secrets to securely pass cloud provider credentials (e.g., Azure service principal, AWS access keys).
        
    
    ***Step 5: Implement Pipeline Steps:***
    
    Typical pipeline stages:
    
    1. **Terraform Init** – Initialize the working directory.
        
    2. **Terraform Validate** – Check for syntax or configuration errors.
        
    3. **Terraform Plan** – Show the execution plan of proposed changes.
        
    4. **Terraform Apply** – Apply the changes (with manual approval if needed).
        
    5. (Optional) **Terraform Destroy** – Clean up resources after testing.
        

***Step 6: Add Approval Gates and Environment Controls:***

* Use **manual approvals** or **release gates** to control changes to production.
    
* Configure role-based access to ensure only authorized personnel can approve or apply changes.
    

***Step 7: Monitor and Audit:***

* Enable logging and notification integrations (e.g., Slack, Microsoft Teams).
    
* Track infrastructure changes via version control and pipeline run history.