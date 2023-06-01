---
title: "Simplify AWS Cloud Management with Terraform"
datePublished: Tue Mar 14 2023 13:26:36 GMT+0000 (Coordinated Universal Time)
cuid: clf8aesg1000108lgfjez6x6z
slug: simplify-aws-cloud-management-with-terraform
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1678799668513/3a81ae82-1102-4e58-92a1-6566cfa01ed1.jpeg
tags: aws, terraform, terraform-state, terraform-cloud, 90daysofdevops-devops-projectdevelopment-nonitbackground-github-docker-cloudplatforms-ec2-aws-elasticbeanstalk-lambdafunctions-devopspipelines-terraform-jenkins-docker-devsecops-scm-git-gitlab-bitbucket-buildtools-griddle-maven-ant-msbuild-monitoringtools-prometheus-grafana-ansible-ai-chatgpt-valueaddition-realworldproblems

---

# Prerequisites

To deal with AWS using Iac(Infrastructure as Code) we need certain credentials:

* **AWS CLI(Command Line Interface)**: It is a CLI that helps the user to interact with aws management console and several other services using command lines.
    
    **NB:** It doesn't come pre-installed in any instances so, you have to install it explicitly by the command `sudo apt-get install awscli`
    
* **IAM user**: You need an IAM user apart from the root user and the user should have permission to access the services.
    
* **AccessKey & SecrerAccessKey**: Generate the access key for the respective user and keep it handy for configuration with AWS CLI.
    
* **Configuration**: Run the command `aws configure` and give the access key and secret access key as below and the user will be configured.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678782624700/08b44461-1e1f-45f0-95ff-3bcc588b01bc.png align="left")
    
* **Exporting AccessKeys for Terraform**: The access keys will be used while writing the **terraform** file so we need to use these keys as **variables** inside. So first, we have to declare them as environment variables using `export` the as below:
    
    ```bash
    export AWS_ACCESS_KEY_ID=AKIAQIAW77BLMXXDUGO 
    
    export AWS_SECRET_ACCESS_KEY=DT0GTtKoHn4scFYhkETVSpVrVye17mWb1KY9RlF
    ```
    

# Creating EC2 instance & S3 Bucket using Terraform

## main.tf

```bash
terraform {
      required_providers {

        aws =  {
                source = "hashicorp/aws"
                version = ">= 4.16"
                }

        }

        required_version = ">= 1.2.0"

}

provider "aws" {

        region = "us-east-1"
}

resource "aws_instance" "aws_ec2_instance" {

        ami = "ami-0557a15b87f6559cf"
        instance_type = "t2.micro"
        tags = {
                Name = "Terraform_Server2"
                }
}
resource "aws_s3_bucket" "bucket_form_terraform_by_rkn" {

        bucket = "aws-s3-bucket.bucket-form-terraform.ritesh"
        tags = {

                Name = "bucket_from_terraform_by_ritesh1"
                Environment = "DevQa"
                }
}
}
output "ec2_public_ips" {
        value = aws_instance.aws_ec2_instance[*].public_ip
        }
```

# Explanation

### terraform { } block

```bash
terraform {
      required_providers {

        aws =  {
                source = "hashicorp/aws"
                version = "-> 4.16"
                }

        }
```

This block specifies the required provider for the AWS resource type in the Terraform configuration.

Here's what this code block does:

* `terraform`: This is the top-level block in a Terraform configuration file. It specifies the required Terraform version and other settings.
    
* `required_providers`: This is a sub-block under the `terraform` block. It specifies the required providers for the resources in the configuration.
    
* `aws`: This is the name of the provider. In this case, it refers to the AWS provider.
    
* `source`: This specifies the location of the provider's source code. In this case, it's the HashiCorp AWS provider.
    
* `version`: This specifies the required version of the provider. In this case, it's `-> 4.16`, which means any version greater than or equal to 4.16 is acceptable.
    

By including this block in your Terraform configuration file, you are telling Terraform that your configuration requires the AWS provider version 4.16 or later, and that it should download and install the provider if it's not already installed.

### provider { } block

```bash
provider "aws" {

        region = "us-east-1"
}
```

This is a Terraform configuration block for the AWS provider, specifying the region to use for resources created in the Terraform configuration.

Here's what this code block does:

* `provider`: This is a top-level block in a Terraform configuration file that defines a provider to use in the configuration. In this case, it's the AWS provider.
    
* `"aws"`: This specifies the name of the provider to use. Terraform has built-in support for many providers, and `"aws"` specifies the AWS provider.
    
* `region`: This is a setting for the provider that specifies the region to use for resources created in the Terraform configuration. In this case, it's set to `"us-east-1"`, which is the US East (N. Virginia) region.
    

By including this block in your Terraform configuration file, you are telling Terraform to use the AWS provider in the configuration and to create resources in the US East (N. Virginia) region.

You can modify the `region` value to specify a different region to use for resources in your configuration. Note that some resources may not be available in all regions, so you should consult the AWS documentation to determine which regions are suitable for your use case.

### resource { } block

```bash
resource "aws_instance" "aws_ec2_instance" {

        ami = "ami-0557a15b87f6559cf"
        instance_type = "t2.micro"
        tags = {
                Name = "Terraform_Server2"
                }

}
```

Here's what this code block does:

* `resource`: This is a top-level block in a Terraform configuration file that defines a resource to create in the infrastructure. In this case, it's an EC2 instance.
    
* `"aws_instance"`: This specifies the type of resource to create. Terraform has built-in support for many resource types, and `"aws_instance"` specifies an EC2 instance.
    
* `"aws_ec2_instance"`: This is a name for the EC2 instance resource that is being created. It is user-defined and can be any valid Terraform identifier.
    
* `ami`: This specifies the Amazon Machine Image (AMI) to use for the EC2 instance. In this case, it's set to `"ami-0557a15b87f6559cf"`, which is the ID of an Amazon Linux 2 AMI.
    
* `instance_type`: This specifies the instance type to use for the EC2 instance. In this case, it's set to `"t2.micro"`, which is a low-cost, general-purpose instance type.
    
* `tags`: This specifies a set of key-value pairs to tag the EC2 instance with. In this case, it's set to `{ name = "Terraform_Server2" }`, which creates a tag with the key `"name"` and the value `"Terraform_Server2"`.
    

By including this block in your Terraform configuration file, you are telling Terraform to create an EC2 instance using the specified AMI and instance type in the AWS region specified by the provider block. The instance will also be tagged with the specified key-value pair.

You can modify the `ami` and `instance_type` values to specify a different AMI and instance type for your EC2 instance, respectively. You can also add additional tags or modify the existing tag values as needed.

### output{ } block

```bash
output "ec2_public_ips" {
        value = aws_instance.aws_ec2_instance[*].public_ip
        }
```

Here's what this code block does:

* `output`: This is a top-level block in a Terraform configuration file that defines an output that Terraform will generate after applying the configuration. In this case, it's an output for the public IP address of the EC2 instance.
    
* `"ec2_public_ips"`: This specifies the name of the output. It is user-defined and can be any valid Terraform identifier.
    
* `value`: This is a setting for the output that specifies the value of the output. In this case, it's set to `aws_instance.aws_ec2_instance.public_ip`, which references the `public_ip` attribute of the `aws_instance` resource with the name `aws_ec2_instance`.
    
* The `"[*]"` syntax is used to select all instances created by the "aws\_ec2\_instance" resource. This means that if the configuration creates multiple instances, all of their public IP addresses will be returned in a list.
    

By including this block in your Terraform configuration file, you are telling Terraform to generate an output after applying the configuration that displays the public IP address of the EC2 instance created by the configuration.

## `terraform init`

After creating the main.tf file we have to perform `terraform init` so that all the required dependencies will be downloaded.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678787594269/a9cf3365-0b00-4d98-8311-4811be5c2fed.png align="center")

## `terraform plan`

Once dependencies are downloaded we can see an executable blueprint made by terraform using `terraform plan` which will show the entire configuration on the basis of which the ec2 instance will be triggered.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678788013382/b3f423e6-0e8c-4bd7-9456-4ccfb15397f0.png align="center")

## `terraform apply`

Once a blueprint is created we can simply apply it and it will trigger an ec2 instance with our defined instances.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678788348160/fc4eeaa9-5921-4290-a877-47d0b855a0ab.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678788601930/9f7610d1-1d52-4bae-abcf-cfb0b4705ce9.png align="center")

And now you can see above a new instance named Terraform\_Server2 has been created.

## Creating multiple instances

```bash
resource "aws_instance" "aws_ec2_instance" {
        count = 2
        ami = "ami-0557a15b87f6559cf"
        instance_type = "t2.micro"
        tags = {
                Name = "Terraform_Server2"
                }

}
```

To create multiple instances all we need is to add another attribute `count` and assign the value as above and it will lunch the number of instances you have defined.

After defining the count as 2 when we reapplied it again started creating another instance as one instance was already created.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678790324667/c7197107-9568-44e5-a3bb-5fc12eca503c.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678790538631/245c9d8a-6038-4314-9ed8-90439a047647.png align="center")

# S3 bucket creation

```bash
resource "aws_s3_bucket" "bucket_form_terraform_by_rkn" {

        bucket = "aws-s3-bucket.bucket-form-terraform.ritesh"
        tags = {

                Name = "bucket_from_terraform_by_ritesh1"
                Environment = "DevQa"

                }
}
```

The "bucket" parameter specifies the unique name of the S3 bucket that Terraform will create. In this case, the bucket name is "aws-s3-bucket.bucket-form-terraform.ritesh".

The "tags" parameter is an optional map that allows you to add metadata to the bucket in the form of key-value pairs. In this case, the "Name" and "Environment" tags are defined with the values "bucket\_from\_terraform\_by\_ritesh1" and "DevQa", respectively.

**aws-s3-bucket.bucket-form-terraform.ritesh** named bucket has been created.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678792679034/ae19f458-eb78-41ab-afda-35e79d246e38.png align="center")

# `terraform destroy`

`terraform destroy` is a command used to destroy the infrastructure created by a Terraform configuration. It will remove all the resources defined in the Terraform configuration file and any associated state files.

It's important to note that `terraform destroy` is a destructive command and cannot be undone. Before running this command, ensure that you have a backup of any important data and have verified that you want to destroy the infrastructure.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678795761181/2853ea37-1d31-44a6-a080-7465c21ea599.png align="center")

Now you can see 2 instances have been terminated.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678798184246/42faa4ef-58d9-490c-9f47-3e92132ca122.png align="center")

**NB: As you can see the name of the instances is the same for both instances. However, we can give different names for each instance using meta arguments such as count and for each count which we will be discussing going forward.**

# Meta Argument

In Terraform, meta-arguments are special arguments that can be used to control how resources are managed and provisioned. These arguments are used to specify configuration details that are not directly related to the resource being created but instead, provide additional information on how to create or manage the resource.'

There are several meta-arguments available in Terraform, including:

* `depends_on`: This argument is used to specify dependencies between resources. Terraform uses this information to determine the order in which resources should be created or destroyed.
    
* `count`: This argument is used to create multiple instances of a resource, each with a unique set of attributes.
    
* `lifecycle`: This argument is used to specify how Terraform should handle the lifecycle of a resource. For example, you can use this argument to prevent a resource from being destroyed or to ignore changes to certain attributes.
    
* `provider`: This argument is used to specify the provider used to create a resource. Providers are plugins that Terraform uses to interact with specific services, such as AWS or Google Cloud.
    
* `provisioner`: This argument is used to specify how to provision a resource after it has been created. For example, you can use this argument to execute scripts or run configuration management tools like Ansible or Chef.
    

Meta-arguments are an important part of Terraform's flexibility and can be used to customize the behavior of resources to fit your specific needs.

## main.tf with `count`

```bash
terraform {
        required_providers {
                aws = {
                     source = "hashicorp/aws"
                     version = ">= 4.16"
                        }
        }
        require_version = ">=1.2.0"
}

provider "aws" {
        region = "us-east-1"
}
resource "aws_instance" "aws_ec2_test" {
        count = 4
        ami = "ami-0557a15b87f6559cf"
        instance_type = "t2.micro"
        tags = {
                Name = "TerraformServer- ${count.index}"
                }
        }
```

Once you apply the `terraform apply` it will create 4 new instances with different names as per the index below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678893254249/9e956ee2-e4a0-44d6-8c74-17a933cb6e8a.png align="center")

In the above **main.tf** file we have used `${count.index}` which is treated as a variable referring to the index of the count. Here each instance is triggered and named with its index number.

## main.tf with `for-each`

```bash
terraform {
        required_providers {
                aws = {
                     source = "hashicorp/aws"
                     version = ">= 4.16"
                        }
        }
        require_version = ">=1.2.0"
}

provider "aws" {
        region = "us-east-1"
}

locals {
        instanceNames = toset(["Server-Dev","Server-QA","Server-UAT","Server-Staging"])
}

resource "aws_instance" "aws_ec2_test" {
        for_each = local.instanceNames
        ami = "ami-0557a15b87f6559cf"
        instance_type = "t2.micro"
        tags = {
                Name = each.key
                }
        }
```

In the above snippet, you can notice the differences which are :

* We have added a new block as `locals {`toset(....)`}`: Here **toset()** is a method that converts a list to a set because for-each works only on sets as it contains unique values.
    
* Replaced the `count=4` with `for_each = locals.instanceNames` : for\_each work on iteration. It will iterate through the set **instanceNames**
    
* Changed the name to `each.key`: Once the for\_each iteration starts `each.key` will hold one value per iteration and pass it as a name. Here the names of the instances will be named as the names present in the **instanceNames** set.
    

Once you apply these changes terraform will destroy all the previous instances and will recreate new instances with respective names as below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678895958929/dd2dd400-456c-4192-84c8-f9e6686f9532.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678895895385/21587c7f-e56c-49ce-8fe3-35549be35aa9.png align="center")

# Creating instances with Different AMI and Names

**Scenario**: If you have a requirement that says you need 4 instances of different OS and also different instance names respectively.

To resolve the above scenario, we can use **map** instead of **Set**. As you know Map data structure consists of data in a key-value pair manner. Accordingly the **main.tf** code will look be changed as below:

```bash
terraform {
        required_providers {
                aws = {
                     source = "hashicorp/aws"
                     version = ">= 4.16"
                        }
        }
        require_version = ">=1.2.0"
}

provider "aws" {
        region = "us-east-1"
}

locals {
        instanceNames = instanceNames = {"Server-Dev":"ami-005f9685cb30f234b","Server-QA":"ami-0557a15b87f6559cf","Server-UAT":"ami-005f9685cb30f234b","Server-Staging":"ami-0557a15b87f6559cf"}
}

resource "aws_instance" "aws_ec2_test" {

        for_each = local.instanceNames
        ami = each.value
        instance_type = "t2.micro"
        tags = {
                Name = each.key
                }
        }
```

* The `locals{ }` block has been changed with key-value pair that contains Server Name as Key and AMI Id as the value where `ami-005f9685cb30f234b` and `ami-0557a15b87f6559cf` are **AWS-Linux** and **Ubuntu** respectively.
    
* In the resource { } block `ami = each.value` which means the value of the instanceNames map has been changed and assigned to variable ami and in `tags { }` block `Name = each.key` the key is assigned to **Name.**
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678953377680/dc0ef082-85f4-4270-a007-c44648015fa6.png align="center")