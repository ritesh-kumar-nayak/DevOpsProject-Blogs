---
title: "Terraform Advanced(Nginx Deployment Automation) -1"
datePublished: Fri Mar 03 2023 07:33:53 GMT+0000 (Coordinated Universal Time)
cuid: cles7yts7000c0amh1xhj9sgc
slug: terraform-advancednginx-deployment-automation-1
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1677828728259/14829496-01cb-4b72-920c-dc85f5632afc.png
tags: docker, nginx, terraform, containerization, hashicorp

---

# Terraform with Docker

As we all are aware, Docker is a containerization tool that allows developers to easily deploy, run, and manage applications in containers. Here our aim will be :

* Installing docker
    
* Pulling docker image from docker hub
    
* Creating a Container from the image and running it.
    

Previously we used to perform all the above operations manually which was a tedious task. However, today we will be automating all the above steps using terraform with an HCL file.

# Writing Terraform file for Docker

Terraform needs to know the provider, source, version etc. to work on docker. To do that we will be writing a file named **main.tf** which will contain all the configuration.

```bash
terraform{
 required_provider{
  docker={
   source  = "kreuzwerker/docker"
   version = "3.0.1"
}
}
}
provider docker{}
resource "docker_image" "nginx"{

  name = "nginx:latest"
  keep_locally = false

}

resource "docker_container" "nginx"{

 image = docker_image.nginx.latest
 name = "nginx_container"
 ports {

        internal = 80
        external = 80
        }
}
```

Explanation of the Config file

### `terraform{}`

The `terraform{}` block is a top-level block in a Terraform configuration file that is used to specify Terraform-specific configuration options. This block is optional, but it is commonly used to set global configuration options that apply to the entire Terraform configuration.

`required_providers`**:** This option is used to specify the providers that are required by the configuration. The Docker provider is used to interact with Docker containers and images. It uses the Docker API to manage the lifecycle of Docker containers. Because the Docker provider uses the Docker API, it is immediately compatible not only with single server Docker but Swarm and any additional Docker-compatible API hosts.

`source = "kreuzwerker/docker"`: is a reference to a Docker provider that can be used in a Terraform configuration file. In Terraform, a provider is a plugin that defines the resources and data sources that Terraform can use to manage a particular infrastructure platform or service. The provider is a third-party provider that allows Terraform to manage Docker containers and images. This provider can be used to define Docker resources such as containers, images, and networks, and to manage their configuration and lifecycle.

`provider docker{ }` : The `docker` keyword indicates that this is a Docker provider block. Once the provider is configured, you can define Docker resources using resource blocks.

`resource "docker_image" "nginx"{`

`name = "nginx:latest"`

`keep_locally = false}` : `docker_image` keyword indicates that this is a resource block of **docker image** because to create any container we first need the docker image for that. Inside the block, we have defined the name of the image which is `nginx:latest` this will be pulled from the docker hub.

`resource "docker_container" "nginx"{`

`image = docker_image.nginx.latest`

`name = "nginx_container"`

`ports {internal = 80`

`external = 80}`

`} :` **docker\_container** keyword indicates that this is a container resource block, and `nginx` is the name of the container instance. The `image` attribute specifies the Docker image to use for the container, and the `ports` attribute specifies the port mappings for the container.

We have now completed creating the **main.tf** file which will automate the entire enginx deployment using docker. Now, as discussed on the **fundamental blog** earlier let's apply `terraform init` and `terraform plan` to see what are operations to be performed by Terraform:

### `terraform init`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677822430983/41c4d9f6-2e8d-4585-aa09-1683d1cb22d5.png align="center")

### `terraform plan` :

when applying `terraform plan` the below error is thrown which means we have not installed docker yet so terraform is not able to identify and connect.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677823822765/4edf639c-b59a-4a99-af04-f96e88507677.png align="center")

To proceed further we need to install and give permission to docker using `sudo apt install docker.io` and `sudo chown $USER /var/run/docker.sock`

Now, after giving permission terraform has prepared the plan to execute as below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677824259800/c035b7a1-4583-429a-b86d-160fc0dde0b1.png align="center")

### `terraform apply`

Now finally we are applying the conifg which will pull the docker image of nginx, create the image and run the image in a container as below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677824671765/c340f912-b099-460a-8bda-c4b61a094b9c.png align="center")

### Verifying if the container is Up

`docker ps` : using the docker ps command you can verify if the docker container for nginx is up and running as below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677825049515/a29f082b-b815-42f8-bfdc-0a507238f2d1.png align="center")

# Nginx Successfully Up & Running

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677825128699/987d46a2-fd84-45d1-99a4-295ce405f3ce.png align="center")

# Some common options that can be set in the `terraform{}` block:

* `backend`: This option is used to configure the backend that Terraform uses to store its state. For example:
    

```java
terraform {
  backend "s3" {
    bucket = "my-terraform-state-bucket"
    key    = "terraform.tfstate"
    region = "us-west-2"
  }
}
```

* `version`: This option is used to specify the required version of Terraform. For example:
    

```java
terraform {
  required_version = ">= 1.0"
}
```

* `locals`: This option is used to define local values that can be used within the Terraform configuration. For example:
    

```python
terraform {
  locals {
    environment = "prod"
    region      = "us-west-2"
  }
}
```

There are many other configuration options that can be set in the `terraform{}` block. The options that are available depend on the version of Terraform being used, and on the installed providers and modules.