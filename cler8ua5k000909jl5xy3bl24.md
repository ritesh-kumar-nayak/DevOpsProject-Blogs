---
title: "Complete Terraform Fundamentals"
datePublished: Thu Mar 02 2023 15:10:35 GMT+0000 (Coordinated Universal Time)
cuid: cler8ua5k000909jl5xy3bl24
slug: complete-terraform-fundamentals
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1677769435355/b3df3a15-dd25-4414-b742-d93fcae88008.png
tags: devops, terraform, iac, terraform-state, terraform-cloud

---

# What is Terraform?

Terraform is an open-source infrastructure as code (IAC) tool that is used to create, manage, and provision infrastructure resources across multiple cloud platforms and services.

It allows users to define infrastructure resources and their dependencies in a declarative language, and then automatically creates and manages those resources in a consistent and repeatable way. Terraform supports a wide range of cloud providers such as Amazon Web Services, Microsoft Azure, Google Cloud Platform, and many others, as well as on-premises infrastructure.

Terraform also provides a powerful workflow for managing infrastructure resources, allowing users to apply changes to infrastructure in a controlled and auditable way. It also supports version control and allows teams to collaborate on infrastructure changes through a centralized configuration.

Overall, Terraform is a powerful tool for managing infrastructure resources at scale and is widely used by DevOps teams and infrastructure engineers.

# IaC(Infrastructure as Code)

IaaC stands for Infrastructure as Code, which is an approach to managing IT infrastructure by writing and managing code, rather than using manual processes or graphical user interfaces. With IaaC, you define your infrastructure as code in a declarative language, and use a tool like Terraform to manage the deployment, configuration, and maintenance of your infrastructure.

Using IaaC has several benefits, including:

1. Consistency: Infrastructure is defined and managed using code, so it is easy to ensure that all resources are configured in the same way, reducing the risk of configuration drift.
    
2. Version Control: Infrastructure code can be stored in version control systems like Git, making it easy to track changes, roll back to previous versions, and collaborate with others.
    
3. Automation: IaaC tools can automate the provisioning and configuration of infrastructure, making it easy to scale resources up or down as needed, and to perform routine maintenance tasks.
    
4. Reusability: Infrastructure code can be reused across different environments, making it easy to create development, staging, and production environments with the same configuration.
    
5. Predictability: Infrastructure code can be tested and validated before it is deployed, ensuring that it will work as expected when it is deployed.
    

Overall, IaaC allows organizations to manage their infrastructure more efficiently, with fewer errors, and with greater flexibility and control. Terraform is one of the most popular IaaC tools, and is widely used to manage cloud infrastructure on platforms like AWS, Azure, and Google Cloud.

# Terraform VS AWS CloudFormation

Terraform and CloudFormation both are IaaC tools used to manage resources on the cloud. However, Terraform is from HashiCrop's IaaC tool and CloudFormation is from AWS. Even though the aim of both tools is the same, the way they function is different. Below are some differences between both tools:

1. Multi-cloud support: Terraform supports multiple cloud providers, including AWS, Azure, Google Cloud Platform, and many others. CloudFormation, on the other hand, is specific to AWS.
    
2. Declarative vs. Imperative: Terraform uses a **declarative language** to define infrastructure resources and their dependencies, whereas CloudFormation uses an **imperative language**. This means that Terraform focuses on **describing the desired end-state**, while **CloudFormation** focuses **on the steps to get there.**
    
3. Ecosystem: Terraform has a larger ecosystem of community-created modules and integrations with third-party tools. CloudFormation has a more limited set of integrations and modules.
    
4. Granularity: Terraform provides more granular control over resources and allows for more complex infrastructure deployments. CloudFormation is more straightforward for smaller deployments.
    
5. State management: Terraform has a separate state management system that allows for greater control over the lifecycle of resources. CloudFormation manages the state within the service, which can sometimes result in unexpected behavior.
    

Ultimately, the choice between Terraform and CloudFormation depends on your specific needs and preferences. If you're working solely in AWS and prefer an imperative approach, CloudFormation may be the better choice. If you need to manage resources across multiple cloud providers or prefer a declarative approach, Terraform may be a better fit.

## Terraform Installation

**Getting the properties** : `sudo apt-get update && sudo apt-get install -y gnupg software-properties-common`  
**GPG Key:** `wget -O-` [`https://apt.releases.hashicorp.com/gpg`](https://apt.releases.hashicorp.com/gpg) `| gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg`

**Verifying Key Fingerprint**: `gpg --no-default-keyring --keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg --fingerprint`

**Adding Repository to the system**: `echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg]` [`https://apt.releases.hashicorp.com`](https://apt.releases.hashicorp.com) `$(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list`

**Updating system**: `sudo apt update`

**Installing Terraform**: `sudo apt-get install terraform`

# HCL(HashiCorp Configuration Language)

HCL (HashiCorp Configuration Language) is the configuration language used by Terraform. It is a simple, easy-to-read language that is used to define infrastructure resources and their dependencies.

## Writing your First Terraform file

As described above, terraform supports HCL and to manage infra we will be writing the infrastructure configuration file in HCL.

**NB:** The extention for any terraform file is **.tf** .

Example: demo.tf, local.tf

### HCL Syntax

For a resource in HCL, you use the following syntax:

```java
resource "type" "name" {
  attribute1 = value1
  attribute2 = value2
}
```

* `type`: The type of resource you want to create, such as `aws_instance` .
    
* `name`: A unique name for the resource.
    
* `attribute`: The configuration options for the resource, such as `ami` or `instance_type` for an EC2 instance.
    

## A first Sample file for Illustration

```java
resource "local_file" "devops" {
              |          |
        (ResourceType) (ResourceName)

    filename = "/home/ubuntu/terraform-local/devopsAutomated.txt"
    content = "DevOps With terraform"

}
```

### **What is** `local_file` ?

In Terraform, the `local_file` resource allows you to create a local file on the machine running Terraform. This resource is useful for generating configuration files or other files that are used by your infrastructure. To use the `local_file` resource, you define a new resource block with the type `local_file`. You also specify the content of the file using the `content` argument, and the file path using the `filename` argument.

### Explanation:

The above HCL config is made to automate the creation of a simple text file locally where the resource block is named **devops** and the type of the resource is **local\_file. filename** field represents the file name with its complete path where it needs to be created and the **content** represents the information this text file will hold.

All the key values inside the curly braces are called **attributes** and all the attributes combined to form a **block** that is enclosed by curly braces.

# Commands to Interact with Terraform

### `terraform init`

So, after writing the terraform configuration you need to validate the file if it is in the proper format or not. To do that we use `terraform validate` however, before doing the validation you need to install the dependencies required up to this point so that terraform can identify the file and run the validation on it. To do that we use `terraform init` as below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677760014654/b2b5c6d5-8ef2-4abd-ade2-91fdfa70b74b.png align="center")

### `terraform validate`

It is a command in Terraform that is used to check the validity of the syntax and format of your Terraform code. This command performs a syntax check of your Terraform configuration files and ensures that all required variables are defined.

When you run `terraform validate`, Terraform reads your configuration files and checks for any syntax errors or other issues that could prevent your configuration from being applied. If any issues are found, `terraform validate` will report them along with an explanation of the error.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677761080978/e5311adb-9c6e-48c8-9d12-549d97edf39b.png align="center")

### `terraform plan`

It is a Terraform command that creates an execution plan for your infrastructure. This command reads your Terraform configuration files and generates a detailed plan of the changes that Terraform will make to your infrastructure.

When you run `terraform plan`, Terraform compares your desired state (as defined in your Terraform configuration files) to your current state (as stored in the state file). Terraform then generates a report that shows the changes that will be made to your infrastructure to bring it into the desired state.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677763651243/338856f0-2ff9-41c7-b664-ba4e61e756ef.png align="center")

### `terraform apply`

It is a Terraform command that applies the changes described in your Terraform configuration files to your infrastructure. This command reads your configuration files and generates an execution plan using `terraform plan`, and then applies that plan to your infrastructure.

When you run `terraform apply`, Terraform compares your desired state (as defined in your Terraform configuration files) to your current state (as stored in the state file). Terraform then makes any necessary changes to your infrastructure to bring it into the desired state. If you are using the `terraform apply` for the first time on a config file it simply creates the resources defined in the config as shown below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677764055248/6dba8ea7-2a98-4b97-a4fc-8116ac735286.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677764075940/71a17d64-ba96-4880-a353-ee2cd726faff.png align="center")

Let's check if the file we wanted terraform to create is created or not.

Below using simple `ls` command we can see that a new file **devopsAutomated.txt** has been created and the content is also visible when we apply `cat` on it.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677764384723/8b70c87d-48a2-48fb-a20f-4d296fe45ebf.png align="center")

**What is terraform.tfstate ?**

It is like a snapshot taken after every terraform apply .

The `terraform.tfstate` file is generated by Terraform when you run `terraform apply` or `terraform import` commands. This file is typically stored in the same directory as your Terraform configuration files, but you can also specify a different location using the `backend` configuration block. `terraform.tfstate` is a file that Terraform uses to store the current state of your infrastructure. This file contains information about the resources that Terraform has created, modified, or destroyed, and allows Terraform to track changes to your infrastructure over time.

Even if you make changes outside of Terraform. Terraform can also use this file to detect any changes made to your infrastructure and make any necessary updates to keep it in the desired state.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677764704557/98894c06-47fc-4f81-8bde-2190e3449fe2.png align="center")

# Adding multiple resources in a single terraform config file

### Editing the existing config file

We can add multiple resources and automate them in a single HCL config file.

Here, we will create another resource which will create a random string. We'll update the same config file and add the new resource as below:

```java
resource "local_file" "devops" {

    filename = "/home/ubuntu/terraform-local/devopsAutomated.txt"
    content = "DevOps With terraform"

}

resource "random_string" "str" {

length = 20
special = true

}

output random_string{

value = random_string.str.result

}
```

`random_string` resource to generate a random string in Terraform HCL.

In this example, the `random_string` resource generates a random string of length 10 that includes special characters. The `output` block is used to print the generated string to the console.

When you run `terraform apply`, Terraform will generate a new random string and print it to the console which will look like below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677767964120/a9afc76a-899d-43fa-a04f-064de30e4f2b.png align="center")

### Validating and checking the Plan

Once the newly edited config is validated, check the plan where it shows the actions going to be performed.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677767584074/cc044af9-d681-4de7-a60a-ea62e41949d4.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677767752428/6121db59-3662-4e81-9396-e591ff09b0dd.png align="center")

**NB: Every time you make changes or add a new resource type, you need to command** `terraform init` **to download the provider else you will be getting the below error :**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677767438687/5cba8a04-e891-4a55-8efa-a4cac7f5db48.png align="center")

### Applying the changes to recreate the resources

Once you command `terraform apply` it will give the output and recreate the resources by running the new config as below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677768166427/1e6ec657-806d-4c6d-b5e7-55d22183be92.png align="center")

## Use case of `random_string` in S3 bucket:

You can use the generated random string in your Terraform configuration by referencing the `result` attribute of the `random_string` resource and use them as unique name for S3 buckets :

```java
resource "aws_s3_bucket" "buckets" {
  bucket = "buckets-${random_string.str.result}"

}
```

# Terraform State

`terraform state` is a command in the Terraform CLI that allows you to view and modify the state of your infrastructure as managed by Terraform.

## Commands for Terraform State

The `terraform state` command provides several subcommands that allow you to perform various operations on your Terraform state, including:

* `terraform state list`: Lists all resources currently tracked in the state.
    
* `terraform state show`: Displays the attributes of a resource in the state.
    
* `terraform state pull`: Pulls the current state from a remote backend and saves it to a local file.
    
* `terraform state push`: Pushes a local state file to a remote backend.
    
* `terraform state rm`: Removes a resource from the state.
    
* `terraform state mv`: Moves a resource to a new address in the state.
    

These commands can be useful for debugging issues with your Terraform infrastructure, or for making manual modifications to your state.

For example, if you wanted to view the attributes of an AWS EC2 instance that Terraform is managing, you could use the following command:

```bash
terraform state show aws_instance.example
```

In this example, `aws_instance.example` is the address of the resource in the Terraform state. The `terraform state show` command will display the attributes of the instance, including its ID, IP address, and other properties.

It's important to use caution when manually modifying your Terraform state, as this can cause issues with your infrastructure. In general, you should try to use Terraform to manage your infrastructure as much as possible, and only use `terraform state` commands when necessary.