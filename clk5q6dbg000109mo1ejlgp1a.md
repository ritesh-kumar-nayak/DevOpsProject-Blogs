---
title: "Continuous Integration and Delivery"
datePublished: Sun Jul 16 2023 17:43:10 GMT+0000 (Coordinated Universal Time)
cuid: clk5q6dbg000109mo1ejlgp1a
slug: continuous-integration-and-delivery
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1689128538797/609c4d91-2947-44ce-9446-89c763a471b7.png
tags: devops, jenkins, ci-cd, devops-articles, trainwithshubham

---

# Project Overview

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689529281401/a335edf0-a7f1-4736-a2d1-88714e3b79fb.png align="center")

This project is an extension of [continuous integration](https://hashnode.com/post/cljvfjpu0000709l88g5349tr) in which, we'll move the application until the Continuous Delivery phase.

Developers will make code commits to the source code management tool **Git** which will trigger the build.

The build tool(Maven) starts the code build and performs unit testing automatically and sends the report to the **Slack** channel.

In the next phase, it will perform a code analysis with code analysis tools and a notification will be sent to Slack on passing or failing the **Quality Gates.**

On passing the code analysis phase, a **docker image** will be built that contains the artifact inside it.

The docker image will further be pushed into **Amazon ECR(Elastic Container Registry).**

We'll run the **AWS CLI** command from Jenkins that will fetch the docker image from ECR to **Amazon ECS(Elastic Container Service).**

# Flow of Execution

1. Update GitHub WebHook with the new Jenkins Server's IP
    
2. Copy the Docker file from the VProfile repo to our repository as it is already written
    
3. Prepare **2** different Jenkins files for **Staging and Production** in the source code
    
4. AWS Steps
    
    * Creating IAM user to access ECR from AWS CLI
        
    * Setting up the ECR repository
        
5. Jenkins Plugin Installation
    
    * Amazon ECR plugin
        
    * Docker plugin
        
    * Docker build & Publish plugin
        
    * Pipeline: aws steps
        
6. Install docker engine and aws-CLI from Jenkins
    
7. Write Jenkins file for build and publish the image to ECS(Elastic Container Service)
    
8. ECS Setup
    
    * Cluster, task definition, and service
        
9. Update the Jenkins file with the code to deploy the image to ECS.
    
10. Repeat the **8 and 9th** steps for production ECS Cluster
    
11. Promoting docker image for Prod
    

# Branches and WebHook

We have already set up the Jenkins server from the previous [continuous integration](https://hashnode.com/post/cljvfjpu0000709l88g5349tr) project and we'll use the same servers so, we just need to update the public IP of restarted Jenkins server in our repository webhook.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689400663318/05b7ed5b-724d-4545-ba3d-2e8af2fa9b8b.png align="center")

As we have set up the webhook now, we'll prepare our branch to push the code changes, newly created docker files, and Jenkinsfile for two different environments that are **Staging and Production.**

Currently, we are on **ci-cd-jenkins** branch.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689402174701/fc86c681-e40c-46ab-87f3-1ba51d8dff32.png align="center")

And in that branch, we have newly added Docker files, and 2 new folders named **StagePipeline and ProdPipeline**.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689402312848/c48c44c7-6d73-4fac-8ee2-88054f38666d.png align="center")

The ProdPipeline contains the Jenkinsfile for the Production environment and StagPipeline contains Jenkinsfile for the Staging environment.

Now, we'll push the codes to the remote Git repository:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689402857475/9afcd602-7e03-4e7c-95fe-e04c25f2726e.png align="center")

Now a new branch **jenkins-ci-cd** has been created containing all the files and code changes:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689403032773/8abd0b93-c962-4113-8813-215f2b1b9d04.png align="center")

# IAM and ECR

## Creating Jenkins user for programmatic access

As we have prepared our Git repository with all our essential code changes now we'll create an IAM user and ECR(Elastic Container Registry).

The IAM user will access ECR from AWS CLI for security purposes we won't be accessing ECR or ECS as a root user instead we'll access them with limited permissions.

We need to create the IAM user with the below permissions:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689404565293/013b29b2-2575-480a-a280-bebeacabd69b.png align="center")

Both permissions are required to access ECS and ECR from AWS CLI as we are going to push the container images to ECR and deploy them in ECS.

## Creating ECR repository

As discussed previously we need a container repository just like we have DockerHub, nexus repository, etc, ECR is an AWS-managed container registry. We are going to create a repository in it to store our Docker Images that will be created from the Docker files that we have in our Git repository.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689405131105/624bf271-b551-4855-9040-9257afd4b9d5.png align="center")

# Jenkins Configurations

In this section, we'll configure 3 things on Jenkins.

* Installing necessary plugins
    
* Storing AWS credentials in it
    
* Installing docker-engine in Jenkins
    

### Installing necessary plugins

1. [**Docker PipelineVersion563.vd5d2e5c4007f**](https://plugins.jenkins.io/docker-workflow)
    
2. [**CloudBees Docker Build and PublishVersion1.4.0**](https://plugins.jenkins.io/docker-build-publish)
    
3. [**Amazon ECRVersion1.114.vfd22430621f5**](https://plugins.jenkins.io/amazon-ecr)
    
4. [**Pipeline: AWS StepsVersion1.43**](https://plugins.jenkins.io/pipeline-aws)
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689489673621/a86d89b3-1870-479b-a612-bbb0c907bb6c.png align="center")

These are the plugins we need additionally for continuous delivery apart from the existing plugins.

### Storing AWS credentials in Jenkins

Now, we'll store AWS credentials such as an access key and secret key for authentication to AWS services via AWS CLI that will be used by Jenkins.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689490146126/593064d9-6806-42f2-bd82-ad4dc9ef500a.png align="center")

This will be used in the pipeline code.

### Installing docker-engine & AWS CLI in Jenkins server

We have to do SSH to the Jenkins server and the with the below command we can install all the dependencies.

Before installing docker, we need to install aws cli `sudo apt update && apt install awscli -y` .

With `sudo apt install docker.io -y` we can install docker.

* We are going to write code in Jenkins to build the docker image
    
* The docker image is built by docker commands so we need to have the docker engine installed in the Jenkins server so Jenkins can run the docker command and build the images.
    
* However, docker commands can only be run by the **root** users or by the users present in the Docker Group. Jenkins user by default can't run docker commands
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689491431487/7dd9f978-f992-464d-bdaf-3c059d4fc40c.png align="center")
    
* Now, either we always have to run the command being the root user or we have to add the **Jenkins user to the Docker group** which is more feasible.
    
* To add a Jenkins user to the Docker group we can use:
    
    `$ usermod -aG docker jenkins`
    
* And then reboot the Jenkins server to make it effective.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689491927357/27b3a0e4-1ce4-418a-8985-d6219fb6c87f.png align="left")
    

Now, Jenkins is configured with all the necessary plugins, AWS CLI, docker-engine also Jenkins user is now enabled to run docker commands.

# Docker build in the pipeline

Here we'll edit the Jenkins file that will include the stages for building docker images and uploading them to Elastic Container Registry(ECR). So, the new Jenkins file will have the below configurations:

* We have added 3 new environment variables which are:
    
    ```java
    registryCredential = 'ecr:us-east-1:awscreds'
    appRegistry = '017758101718.dkr.ecr.us-east-1.amazonaws.com/vprofileapping'
    vprofileRegistry = 'https://017758101718.dkr.ecr.us-east-1.amazonaws.com'
    ```
    
    the `registryCredential` will be used to access ECR and push the image to ECR so that, we have written it like `ecr:regionName:awscreds`. The **'**`awscreds`**'** is the credential name in Jenkins where we have stored the access key and secret key of IAM user that'll access ECR.
    
    `appRegistry` variable stores the image name. Here whatever is assigned to the variable is the image name.
    
    The `vprofileRegistry` variable stores the ECR registry URL where the images will be stored and fetched from by ECS.
    

Now our next step is to create 2 stages:

## The stage for building the Docker image

```java
stage("Build app image"){
            steps{
                script{
                    dockerImage = docker.build( appRegistry + ":$BUILD_NUMBER", "./Docker-files/app/multistage/" )
                }
            }
        }
```

This step builds the Docker image for the application using a multistage build.

1. `stage("Build app image")`: This line defines a Jenkins stage with the label "Build app image". A stage is a logical division of work within a pipeline.
    
2. `steps{}`: This block contains the steps to be executed within the stage.
    
3. `script{}`: This block allows the execution of Groovy script code.
    
4. `dockerImage =` [`docker.build`](http://docker.build)`( appRegistry + ":$BUILD_NUMBER", "./Docker-files/app/multistage/" )`: This line builds a Docker image using the [`docker.build`](http://docker.build) method provided by the Jenkins Docker Plugin. It takes two parameters: the image name and the path to the Dockerfile.
    
    * `appRegistry + ":$BUILD_NUMBER"` is the image name with a dynamic tag `$BUILD_NUMBER`. The `appRegistry` variable refers to the registry where the image will be pushed. It is likely defined earlier in the pipeline script.
        
    * `"./Docker-files/app/multistage/"` is the path to the Dockerfile. The Dockerfile is expected to be located in the specified directory.
        
    
    The [`docker.build`](http://docker.build) method builds the Docker image based on the provided Dockerfile and returns a reference to the built image, which is stored in the `dockerImage` variable.
    

Overall, this stage in the Jenkins Pipeline script is responsible for building the Docker image for the application using the specified Dockerfile in the multistage build directory. The image is tagged with the registry name and the Jenkins build number.

## The stage for uploading the Docker image to ECR

```java
stage("Upload app image to ECR"){
            steps{
                script{
                    docker.withRegistry( vprofileRegistry, registryCredential)
                    dockerImage.push("$BUILD_NUMBER")
                    dockerImage.push('latest')
                }
            }
        }
```

This is another stage in a Jenkins Pipeline script that handles the upload of a Docker image to an Amazon Elastic Container Registry (ECR).

1. `stage("Upload app image to ECR")`: This line defines a Jenkins stage with the label "Upload app image to ECR".
    
2. `steps{}`: This block contains the steps to be executed within the stage.
    
3. `script{}`: This block allows the execution of Groovy script code.
    
4. `docker.withRegistry( vprofileRegistry, registryCredential)`: This line configures the Docker daemon to use the specified registry for pushing the image. The `vprofileRegistry` variable contains the URL or name of the ECR registry and `registryCredential` refers to the credentials required to authenticate with the registry.
    
5. `dockerImage.push("$BUILD_NUMBER")`: This line pushes the Docker image with the tag "$BUILD\_NUMBER" to the configured registry. The `$BUILD_NUMBER` is a Jenkins environment variable that represents the current build number. This allows each build to have a unique tag.
    
6. `dockerImage.push('latest')`: This line pushes the same Docker image with the "latest" tag to the configured registry. The "latest" tag is commonly used to refer to the most recent version of an image.
    

Overall, this stage in the Jenkins Pipeline script is responsible for uploading the Docker image, which was built in a previous stage, to the specified ECR registry. The image is pushed with the build number tag and the "latest" tag. This step ensures that the image is available in the ECR registry for deployment or further use in the CI/CD pipeline.

## Creating stage pipeline

As we have prepared the new Jenkins file that contains all the stages to build and push docker images now our goal is to create the pipeline for the staging environment as we have modified the pipeline script of staging. The below image shows the same pipeline configuration for the staging environment:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689499844113/4c0b42ae-2a79-45ed-9b20-b3cc71d5ef30.png align="center")

Here we are referring to the new branch that we have created especially for Delivery and the script path is changed to StagePipeline/Jenkinsfiles which means now Jenkins will build this pipeline based on the Jenkinsfile present in StagePipeline directory.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689499887174/2f3b2e2d-f86f-4287-aa1f-77915b99b4ee.png align="center")

Now, once we commit and push the pipeline code changes it will trigger the Jenkins and build the pipeline.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689500664468/0a6fb87b-48d9-4b41-98d4-be0012fc7d48.png align="center")

Right after committing the build has been started:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689504796980/96412dbd-d98d-461b-b557-fc26ae826fd7.png align="center")

Now our job is successful and the docker image has been uploaded to ECR and it has been tagged to 7 which is the build number and latest:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689504846795/f164931f-c3ee-4efa-9eb3-24d4bd2d19ae.png align="center")

Now our image is ready to be deployed. Our next goal is to deploy the image to ECS.

# AWS ECS Setup

We have already prepared the docker image of our application having the artifact in it and uploaded it to ECR. Now, the next step is to deploy the same to ECS, to achieve that we need to set up the ECS cluster.

We'll be setting up 2 clusters that are **Staging** and **Production**.

**Staging**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689516636356/14b1e5af-f6b5-40fc-80e7-06a990e5ec36.png align="center")

With the above configuration, we are creating the cluster and the cluster is up and running

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689517161904/8f7adc57-ffd0-4f76-b36b-82fa7d808b14.png align="center")

## Creating Task Definition

We'll create the task here and the **service** will use this task definition to create and manage the container.

### Step-1

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689517514861/76c7fc73-272c-40e7-82eb-9114dc413d7b.png align="center")

Here we have defined the name of the task followed by the container name and the registry URL in the Image URL in the Container-1 section.

We have opened the HTTP connection to port number **8080** as the Tomcat Server runs on this port.

### Step-2

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689517812485/1fa9c7a9-cbd3-4458-9511-def5ba18c5de.png align="center")

That's it. Now we have just created the task definition:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689517916432/1137d066-ef78-4ac4-8f1c-04dd04a6ca4a.png align="center")

## Creating Service from Task Definition

Now, we have the task definition ready and we'll create the service on the basis of the taste definition and that service will further manage our container. To achieve that click on Create in the service tab

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689518199801/fa336197-c6d4-456c-94d9-bf78064b1909.png align="center")

Now we have configured the Service with the below configurations:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689518604485/47be6a05-37b2-44d2-a9a8-7181d3933218.png align="center")

### Editing target group

The container listens on port 8080 hence we have to ensure the Target Group VProAppStagingTG also does the health check at port 8080.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689518967998/aa6d0c36-ed53-4c7f-92d7-136868b84fc9.png align="center")

### Editing the Security Group

We also have to edit the security group ***VProAppStageSG*** that is associated with the service above and allow the 8080 port there:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689519136498/d006db39-3910-4fa5-93e2-a9f703604616.png align="center")

Now the service is ready:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689519419420/55344a64-9660-4856-9b58-077e4c706a19.png align="center")

## Accessing the application from ELB URL

Now the container is running with our application and we can verify it by the LoadBalancer URL:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689519650297/0eaec6c5-45e0-460e-98aa-ab502ef9d85e.png align="center")

# Configuring the Pipeline with ECS

Until now we have set it up manually however, now we'll configure the ECS with Jenkins so that whenever there is a new commit or a new Image is built it will automatically deploy the newly created Image to ECS.

This will bring **AWS CLI** into the picture. We have already installed AWS CLI in the Jenkins server and we have also set up the user credentials `awscred` in Jenkins that has permission to interact with AWS ECS from CLI.

From the next code changes, this user will ask ECS to fetch the newly created image via CLI and the **task definition of the cluster** will be updated with a new image and the service will delete the currently running container and replace it with a new one with the latest image.

Now to achieve all these we have to update the **Jenkinsfile code as below:**

We'll add 2 more variables which are that contains **cluster name** and **service name** respectively:

```java
cluster = "VPro-Staging" //this is the cluster name
service = "VProAppStageSVC" //this is the service name
```

## Deployment Stage

Until now we had the stages till the artifact upload to ECR and here, we'll add a new stage that will deploy the artifact from ECR to ECS for which the code looks like below:

```java
stage('Deploy to ECS staging') {
            steps {
                withAWS(credentials: 'awscreds', region: 'us-east-1') {
                    sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
                } 
            }
        }
```

Now with this, our pipeline has been set up and integrated with ECS. We don't have to create the setup manually anymore. Every time the developer makes a new code change it will automatically deploy the Image to the respective environment.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689522202640/734c1328-7dd1-47f8-adcd-94a5e4a142dd.png align="center")

Now in the above image, you can see that 2 new stages have been created automatically and deployed to the **staging** environment.

# Promoting the build to Production

Now our application is running in the Staging environment and ready for testing by QAs.

Once the build is tested the same build will be promoted to Production and will be made available to the end user.

The same steps will be repeated and we'll create another cluster([VProProd](https://us-east-1.console.aws.amazon.com/ecs/v2/clusters/VProProd?region=us-east-1)), task definition(**VProProdTask**) etc.. for **Production.**

## Service configuration

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689523957157/814f6185-4e92-4711-bd22-44c26810d4a9.png align="center")

## Creating a new Pipeline for Production

We have already differentiated the pipeline script for production. Now we'll add the AWS ECS credentials and variables to configure it with ECS.

Before doing that we'll create a new git branch for Production from the working branch and push it to the remote repository.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689525419968/ea885546-d130-4b73-85f6-1e9a0a9d8fec.png align="center")

In real-time we'll have a different branch for production named Master.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689526997811/4b939403-85b7-4a43-bdd1-444828de9fe4.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689527022344/5be15a67-5da9-4442-a4f3-e0fb757a004c.png align="center")

And above we have created a pipeline and configured it with the respective branch name and directory. And next we'll edit the Jenkins file and push it to the remote repository and then the pipeline will be built from the updated Jenkinsfile.

### Editing Jenkinsfile

For production, we do not need all the stages such as building, code analysis, artifact generation, etc but we just need to fetch the Image and deploy so we'll clean up some of the stages and the pipeline code will be:

```java
def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]

pipeline {
    agent any

    environment {
        cluster = "VProProd"
        service = "VProAppProdSVC"
    }

    stages {
        stage('Deploy to Prod ECS') {
          steps {
        withAWS(credentials: 'awscreds', region: 'us-east-1') {
          sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
        }
      }
     }

    }
    post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#jenkinscicd',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info available at: ${env.BUILD_URL}"
        }
    }
}
```

In this pipeline, we just have **one stage** and **two** environment variables. Here we are just deploying the image to the Prod cluster.

And right after pushing the code the pipeline started and deployed:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689527524836/707f2687-471e-4887-a384-3c5bce6b1562.png align="center")

## Accessing the application from ELB URL

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689527732991/c14ba46a-8e2b-4f33-acb5-342c653fd63d.png align="center")

# Conclusion

With this am concluding the Continuous Delivery phase of the deployment process.