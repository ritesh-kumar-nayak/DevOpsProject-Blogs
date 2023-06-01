---
title: "Terraform Advanced-2"
datePublished: Mon Mar 13 2023 13:01:23 GMT+0000 (Coordinated Universal Time)
cuid: clf6u2iib000a09mh3npw1zfi
slug: terraform-advanced-2
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1678709874397/316be54b-e163-487d-a2b4-f34407d0f689.png
tags: cloudformation, devops, terraform, iac, terraformstate

---

# Variables

Terraform variables are a way to parameterize your Terraform configurations. Variables allow you to define values that can be used across your Terraform codebase, making it easier to create reusable modules and manage configuration values for multiple environments.

Variables can be defined in several ways, including in-line in your Terraform code, in a separate file, or as environment variables. You can also define variable types, such as strings, numbers, and booleans, to ensure that the values passed to your Terraform code are of the correct type.

Using variables can help you avoid hard-coding values in your Terraform code, making it more flexible and easier to maintain.

You can set variable values when you run Terraform using the command line, or by using a Terraform input variable file. Input variable files allow you to define values for all of the variables used in your configuration in one place, making it easier to manage your configuration across multiple environments.

## Usecase

* You might define a variable for the AWS region where your resources will be deployed so that you can easily change the region in the future without having to modify your code.
    
* You can keep **a file path** in a variable and use it anywhere, instead of hardcoding it which may give rise to mistakes and execution failure.
    

## Creating a variable

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678669902546/7a7f5575-387d-42d0-a2f5-095beee23a72.png align="center")

In the above image, you can see that the file path has been assigned to the variable name **filename.** You can create variables in the **main.tf** and edit **as below** however, this is **NOT** the recommended way:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678670069360/e5c67ee8-0e2f-43f5-a088-e35744bc29ac.png align="center")

Considering the coding standard we are always recommended to create and edit variables from **outside** the **main.tf** file which helps in writing clean, easily readable and less error-prone code.

So, to create any variable it is recommended to keep a separate terraform file that will contain all the variables that will be used further. Here are variable file name will be **variables.tf as below:**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678671081928/d3dc9124-cc57-4896-bfc7-fdacb6e46e6d.png align="center")

### variables.tf explanation

**variable:** To create a variable file there is **NO** resource type needed, you can simply create a block named **variable** with a name such as **filename** as shown in the above image.

**default:** it is an argument that holds the value you want to pass to the variable.

In the above image, we have declared **2** variable blocks which means two variables will be declared holding their value as defined in the **content.**

### Accessing variable from variables.tf to main.tf

After declaring the variables we need to implement them in the required place. To do so we need to call the variables from the **variables.tf** file to **main.tf** file.

**NB: variables.tf file and main.tf file should be in the same field as terraform.tfstate**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678672403054/d38c8593-9a4f-4a56-ac04-83b86c7323e0.png align="center")

In the above image, you can see we have added another resource to create a file using variables called the variable **filename** from the **variables.tf** file using **var** object.

`var.filename`**:** It will fetch the file path that was defined

`var.content`**:** It will fetch the content defined for the variable filename in variables.tf file.

By commanding `terraform plan` we can see the differences made below where the content and file name have been fetched from the variables.tf file.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678672942960/293dfe5c-be15-4493-b346-51f226300e3a.png align="center")

After doing `terraform apply` we can see the file has been created as devops\_automated with content:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678673394512/c7b16e0a-f6fc-4329-a5cb-ea725e76d42f.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678673576208/b044e0a6-5091-4d1f-bfa8-b14ac3045751.png align="center")

### Declaring variables and Initializing value at Run-Time

We can also create an empty variable in **variables.tf** file and assign it's value as environmental value/run-time.

To do so:

**Step-1: Creating an empty variable in variables.tf**

In the below picture you can see we have created a new empty variable as **region\_name :**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678674379449/d2918970-b3d0-417d-b0d0-d2e953b0f445.png align="center")

**Step-2: Catching the output in main.tf**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678675049899/f1a2a349-1aeb-465e-87a2-1e0e38f12c60.png align="center")

We need to create a block as **output** to catch the variable that will be defined in during run-time as shown above.

**Step-3: Defining the environmental value**

Syntax: `export TF_VAR_variablename`

`export TF_VAR_region_name = mumbai` will define the value of region\_name as **mumbai** which was previously empty.

**Step-4: terraform apply**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678675965324/214ed876-988e-4976-b57a-81b7bfc8ffae.png align="center")

Once all the steps are done, you can apply the changes and see the region name has been caught as Mumbai.

# Data Types in Terraform

Like other programming languages, Terraform supports several data types that can be used in resource definitions, input variables, and output values. These data types include:

1. String: A sequence of characters.
    
2. Number: A numerical value, either an integer or a floating-point number.
    
3. Boolean: A value that is either true or false.
    
4. List: A sequence of values of the same type.
    
5. Map: A set of key-value pairs, where each key is unique.
    
6. Object: A complex data type that can contain multiple attributes of different types.
    
7. Tuple: An ordered sequence of values of different types.
    

In addition to these built-in data types, Terraform also allows users to define their own custom data types using the `type` keyword. This can be useful for creating reusable modules or defining complex data structures.

## Map data type

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678678615576/3ff6e0c7-ab89-47d0-8196-05a592b20a4c.png align="center")

The map data type is like a dictionary in python which is used to store data in **key-value** pairs. We can declare a map as above where the **type** will be **map.**

### Accessing map data in main.tf

Syntax: `var.mapName[keyName]`

the above syntax can be used to call any value from a map using it's key name

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678678235162/947042e8-487f-430e-a4c0-2d6dbc88ce94.png align="center")

In the above image, we have called the value for the variable **content** from the map **content\_map** using the syntax.

### Viewing the changes using terraform plan

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678678816572/9543a92e-4e83-4946-a865-b8a4db2d87cb.png align="center")

### terraform apply

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678678940222/2c93e2b7-35fe-41c1-af2f-84877b3bffd5.png align="center")

After verifying the changes we applied the changes using terraform apply which says **2 added and 2 destroyed** which means the 2 content values that we have replaced in main.tf by fetching from content\_map have been reflected so that the previously created file has been deleted as below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678679144125/bad20235-663f-4a65-8bea-62647786bd88.png align="center")

## List data type

In Terraform, the `list` data type is used to represent a sequence of values of the same type. The syntax for defining a list in Terraform is as follows:

```java
variable "example_list" { 
type = list(string) 
default = ["value1", "value2", "value3"] 
}
```

**Syntax Explanation:** `example_list` variable is defined as a list of strings with a default value of `["value1", "value2", "value3"]`.

### Example:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678698387193/81e4ad5c-cdec-4013-add2-99f3d82d9703.png align="center")

### Accessing the List from variables.tf to main.tf

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678698829833/ab4040a7-a931-498a-bdb3-9d4b6944dab9.png align="center")

As you can see we are calling the data present inside the list **file\_list** using **var object** and assigning it to **filename.**

### Viewing the list changes using `terraform plan`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678699228434/c753913a-c874-4db8-ae9c-2a3cd38914a6.png align="center")

Here you can see that the file names are being replaced.

### Applying the changes

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678700120376/a3220816-5f48-401b-aa9c-6ebe616cc2c0.png align="center")

After applying the changes you can see two files has been replaced as file1.txt and file2.txt

## Object Data Type

In Terraform, the `object` data type is used to represent a collection of key-value pairs where the keys are strings and the values can be of any type. Objects are useful for grouping related values together and passing them around as a single entity.

In the case of Object data type, we have full control of defining the type of data we want to declare.

Below you can see that first we have to declare the variable names and define it's type then

**Syntax:**

```java
variable "example_object" {
  type = object({
    name  = string
    age   = number
    email = string
  })
  default = {
    name  = "Ritesh"
    age   = 24
    email = "ritesh@example.com"
  }
}
```

### Example with a use case:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678704458462/72771e21-8531-4955-96a4-a7477406d54d.png align="center")

Here we have defined the variables for launching AWS instances. In the future, we will be automating the instance lunches in this way.

### Accessing the values from variables.tf in main.tf

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678704857294/a2d6a154-66aa-4025-af53-0db1c364c85b.png align="center")

Here we are fetching the number of **instances** to the argument **value** by `var.aws_ec2_object.instances` which means `var` object is referring to `aws_ec2_object` which is a variable that further calls `instances` which is an argument containing value 4.

### Viewing the plan

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678705338034/27f17667-7cde-4e6c-8e0e-9737b7857bee.png align="center")

Here, the value of `aws_ec2_instances` has been fetched from the variable.tf .

### Applying the changes

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678705501854/bcf33f0d-4df1-4f22-9f84-415a3e6148cb.png align="center")

# Terraform State

The Terraform state is a file that keeps track of the current state of your infrastructure. It stores information such as the IDs of resources created by Terraform, the current status of those resources, and any metadata associated with them. Terraform uses this state to know what resources it needs to create, update, or delete when you run a command like `terraform apply` or `terraform destroy`.

The state file is typically named `terraform.tfstate` or `terraform.tfstate.d` and is stored locally on your machine. However, for larger projects or team collaboration, it is often recommended to use a remote state backend to store the state file centrally.

It's important to manage the Terraform state carefully, as it is critical to ensuring that Terraform is aware of the current state of your infrastructure. Any changes made to the infrastructure outside of Terraform (i.e., manually) can result in the state becoming out of sync, which can cause issues and errors when running Terraform commands.

In summary, the Terraform state is an essential aspect of Terraform that helps keep track of the infrastructure resources you have provisioned and their current state. It's important to manage the state file carefully and keep it in sync with your infrastructure to avoid issues and errors.

### Can we use the terraform commands if we do not have a state?

Yes, we can. The state is just a blueprint. Even if we delete the terraform.tfstate file we can still use apply init all the dependencies will be downloaded.

### terraform.tfstate file:

The `terraform.tfstate` file includes the details of the resources you've provisioned, such as their configuration, metadata, and current status. Terraform uses this file to determine the current state of your infrastructure and to make any necessary changes to bring it into the desired state defined in your Terraform code.

It's important to manage the `terraform.tfstate` file carefully and keep it in sync with your infrastructure to avoid issues and errors. Any changes made to the infrastructure resources outside of Terraform (such as manually or through another tool) can result in the state becoming out of sync, which can cause problems when running Terraform commands.

### terraform.tfstate file for entire exercises we have done yet looks as below:

```java
{
  "version": 4,
  "terraform_version": "1.3.9",
  "serial": 3,
  "lineage": "09d471e5-0e5b-efd0-bd2d-77b9ad2a207b",
  "outputs": {
    "random_string": {
      "value": "0{AIMcc!31XO=\u003e%}=Ye?",
      "type": "string"
    }
  },
  "resources": [
    {
      "mode": "managed",
      "type": "local_file",
      "name": "devops",
      "provider": "provider[\"registry.terraform.io/hashicorp/local\"]",
      "instances": [
        {
          "schema_version": 0,
          "attributes": {
            "content": "DevOps With terraform",
            "content_base64": null,
            "directory_permission": "0777",
            "file_permission": "0777",
            "filename": "/home/ubuntu/terraform-local/devopsAutomated.txt",
            "id": "f8cfed07696e224611fa3975cc513eae7a93d8f4",
            "sensitive_content": null,
            "source": null
          },
          "sensitive_attributes": []
        }
      ]
    },
    {
      "mode": "managed",
      "type": "random_string",
      "name": "str",
      "provider": "provider[\"registry.terraform.io/hashicorp/random\"]",
      "instances": [
        {
          "schema_version": 2,
          "attributes": {
            "id": "0{AIMcc!31XO=\u003e%}=Ye?",
            "keepers": null,
            "length": 20,
            "lower": true,
            "min_lower": 0,
            "min_numeric": 0,
            "min_special": 0,
            "min_upper": 0,
            "number": true,
            "numeric": true,
            "override_special": null,
            "result": "0{AIMcc!31XO=\u003e%}=Ye?",
            "special": true,
            "upper": true
          },
          "sensitive_attributes": []
        }
      ]
    }
  ],
  "check_results": null
}
```

# Local terraform.tfstate vs Remote terraform.tfstate

Local state files are stored on the machine running Terraform. This can be convenient because everything is in one place and Terraform can access the state file quickly. However, there are some downsides to using local state files:

* If the machine running Terraform is lost or the state file is deleted, there is no backup.
    
* It can be difficult to share state files between team members.
    
* There is no built-in way to lock the state file, so two people could try to make changes at the same time and overwrite each other's changes.
    

Remote state files are stored in a storage backend like Amazon S3, Azure Blob Storage, or HashiCorp Consul. This can be more secure and reliable because the state file is stored in a centralized location. There are several advantages to using remote state files:

* It is easy to share state files between team members.
    
* There is a built-in locking mechanism to prevent multiple people from making changes at the same time.
    
* Remote state files can be backed up and versioned, making it easier to recover from mistakes or rollback changes.
    

However, there are also some disadvantages to using remote state files:

* Access to the storage backend is required for Terraform to work, which can introduce additional complexity and potential points of failure.
    
* Remote state files can be slower to access than local state files, depending on the storage backend used.
    

Overall, the decision of whether to use local or remote state files depends on the specific needs of the project and team. In general, remote state files are recommended for teams working together on infrastructure projects, while local state files may be sufficient for individuals or small teams working on simple projects.