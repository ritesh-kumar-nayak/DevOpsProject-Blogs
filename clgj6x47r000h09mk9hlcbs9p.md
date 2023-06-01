---
title: "AWS-Serverless Project-1(Node API Deployment)"
datePublished: Sun Apr 16 2023 09:14:03 GMT+0000 (Coordinated Universal Time)
cuid: clgj6x47r000h09mk9hlcbs9p
slug: aws-serverless-project-1node-api-deployment
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1681635646042/58a9a4ca-a99b-4ffc-b122-d13094e90a6d.png
tags: aws, serverless, aws-lambda, serverless-architecture, serverless-framework

---

# Serverless Framework

Serverless Framework is an open-source framework for building and deploying serverless applications. It provides a unified experience for managing AWS Lambda functions, along with other serverless services, such as Amazon API Gateway, Amazon DynamoDB, and Amazon S3.

With Serverless Framework, developers can write and deploy serverless applications without having to worry about the underlying infrastructure. The framework allows developers to define their application as a set of functions, events, and resources in a YAML or JSON file, which can then be deployed to the cloud provider of their choice.

The Serverless Framework supports multiple programming languages, including Node.js, Python, Java, and Go. It also provides a plugin architecture, allowing developers to extend its functionality and integrate it with other services.

Overall, the Serverless Framework is a powerful tool for simplifying the development and deployment of serverless applications.

Go to the [serverless framework](https://www.serverless.com/) and create an account by signing up.

# How does Serverless work?

As explained above **Serverless** is nothing but a tool that internally integrates several cloud services of defined cloud providers(AWS in our case) and automates the infrastructure formation. Going forward in our project this Serverless tool will work with several AWS services such as **CloudFormation,** **aws lambda, S3, API gate way** etc and provide the resources for our deployment.

# Serverless Framework Installation

### Step-1(Installation of NodeJS in Ubuntu)

Serverless Framework is made on NodeJS hence, before installing Serverless Framework we need to have NodeJS installed on our system to support it.

After the system update, install Node.js 14 on Ubuntu 22.04|20.04|18.04 by first installing the required repository:

```bash
curl -sL https://deb.nodesource.com/setup_14.x | sudo bash -
```

Once the repository is added, you can begin the installation of Node.js 14 on Ubuntu Linux:

```bash
sudo apt -y install nodejs
```

Now the installation has been completed and you can check it by:

```bash
node -v
npm -v
```

### Step-2(Installing ServerlessFramework)

After installing NodeJS we are proceeding to install **serverless**.

Here, **npm** which is the NodeJS package manager and it will be used to install serverless globally as below:

```bash
sudo npm install -g serverless
```

# Creating and Deploying an API with Serverless

Once all the above installation processes are completed we can deploy a project using the serverless. To do so, we have to check what all kinds of projects are we allowed to deploy using Serverless Framework. To do so, simply command `serverless` which will list all the projects it supports to be deployed as below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681623722036/7924f5a6-90e9-450f-b23c-b941ca49f434.png align="center")

Now once you decided on the project that needs to be created it will download all the templated required for that particular project as below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681625445141/9505e3b8-4b7a-4d24-8a59-7934cded2286.png align="center")

Here I have chosen **AWS - Node.js - HTTP API** as my project so, the Serverless has downloaded all its predefined templates and files required for deployment as below where we have **index.html** and **serverless.yml** files. Here **first-serverless-hello-api** is the name of the project.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681625754418/68e44614-769d-420f-8340-bb021ad08f87.png align="center")

## index.js

Here you can see **index.js** file has been downloaded as part of our project. This index.js is an API function written in javascript that looks like the below:

```javascript
module.exports.handler = async (event) => {
  return {
    statusCode: 200,
    body: JSON.stringify(
      {
        message: "Go Serverless v3.0! Your function executed successfully!",
        input: event,
      },
      null,
      2
    ),
  };
};
```

**NB:** This is a demo file created by serverless developers. In real-world projects this file will be given by the Developers to us(DevOps) engineers.

## serverless.yaml

`serverless.yaml` is a configuration file used by the Serverless Framework to define the structure and resources of a serverless application. This file is written in YAML format and contains information such as the AWS **Lambda functions, events, and resources** that make up the application.

`serverless.yaml` is a powerful tool for defining the structure and resources of a serverless application and enables easy deployment and management using the Serverless Framework.

Below is the YAML file that we need to create for deployment if we use lambda from the AWS console. However, here we are using the **Serverless Framework** that's why this has been created by the framework itself.

```javascript
service: first-serverless-hello-api
frameworkVersion: '3'

provider:
  name: aws
  runtime: nodejs18.x
  region: us-east-1
functions:
  api:
    handler: index.handler
    events:
      - httpApi:
          path: /
          method: get
```

This is a YAML configuration file for **an AWS Lambda function** using the **HTTP API event type**. However, this configuration includes an additional section under `provider` that specifies the AWS region where the function will be deployed. In this case, the region is set to `us-east-1`.

Setting the region is important because it determines the location where the function will be executed and where the data associated with the function will be stored. It's recommended to choose a region that is geographically close to where the majority of your users or resources are located to minimize latency.

The `handler` property specifies the entry point of the Lambda function, which in this case is `index.handler`. This suggests that the code for the function is in a file named `index.js` which we have shown in **index.js** section above and that the `handler` function within that file is exported as the entry point for the Lambda function.

The `events` property specifies the events that trigger the Lambda function. In this case, the event is an HTTP API event that is triggered when an HTTP GET request is made to the root path `/`. The `path` property specifies the endpoint that the API will be listening on, while the `method` property specifies the HTTP method that the API will be listening for.

Overall, this configuration sets up a serverless API that responds to HTTP GET requests at the root path, using a Node.js 18.x runtime, and deploys to the `us-east-1` region.

## API Gateway

API Gateway is a fully managed service provided by Amazon Web Services (AWS) that allows developers to create, deploy, and manage APIs for their applications. It acts as a front door for applications to access data, business logic, or functionality from backend services, such as AWS Lambda, AWS Elastic Beanstalk, or EC2 instances, and makes it easy to expose those services through a public or private API.

API Gateway supports RESTful APIs and WebSocket APIs and provides features like request/response transformation, caching, throttling, security, and monitoring. It also integrates with other AWS services like AWS Identity and Access Management (IAM), AWS Lambda, Amazon S3, and Amazon DynamoDB, making it easier to build and scale serverless applications.

```yaml
    events:
      - httpApi:
          path: /
          method: get
```

The above snippet from the main `serverless.yml` file will create an API using AWS API Gateway.

## Deployment of the API(`serverless deploy` / `sls deploy` )

```bash
serverless deploy
```

`serverless deploy` is a command used in the Serverless Framework to deploy a serverless application to a cloud provider like AWS, Azure, or Google Cloud Platform.

When you run `serverless deploy`, the Serverless Framework packages your application code and any necessary dependencies, uploads them to the specified cloud provider, and sets up the necessary resources to run the application, such as AWS Lambda functions, API Gateway endpoints, and IAM roles.

The `deploy` command also creates a CloudFormation stack that manages the resources created by the Serverless Framework. This stack includes all the necessary configurations and permissions needed to run your application on the cloud provider.

The Serverless Framework uses a YAML configuration file called `serverless.yml` to define your serverless application, including the functions, events, and resources needed to run your application. This configuration file is used by the `deploy` command to deploy your application to the cloud provider.

Overall, `serverless deploy` simplifies the process of deploying and managing serverless applications, allowing developers to focus on building and testing their code without worrying about the underlying infrastructure.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681631409897/ca573a4a-b9a8-41c2-bab1-0e91bd7722c5.png align="center")

Now you can see in the above image that right after commanding `serverless deploy` the **Serverless Framework** started its magic. It has started creating resources on AWS. Here **9** resources will be created to deploy the `first-serverless-hello-api-dev-api` API as shown below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681631696551/885fe7a7-b86a-4156-839c-b48d6e617c00.png align="center")

## On Deployment Completion

After the successful completion of the deployment, a **GET** API has been generated by AWS **API Gateway** which is :

[https://1rdjr8uy82.execute-api.us-east-1.amazonaws.com/](https://1rdjr8uy82.execute-api.us-east-1.amazonaws.com/)

In the below image, you can see that our API has been registered via API Gateway without even touching it manually. It is all made possible by the Serverless Framework.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681632797935/94c01d4e-1f6a-4d01-aade-a58b8df85b20.png align="center")

Now when we are sending the request to the above endpoint we are getting the response as below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681631957016/e8f97500-7514-4412-a309-2d86074ef355.png align="center")

## Updating the API response

Now we have made some changes to the **index.js** file and eliminated the complete event logging as below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681632419205/19a9bac7-4ac2-40a3-ba91-1673bbba6943.png align="center")

Now we will redeploy the application and see the changes. Here it is just updating the stack with the latest changes we have made to our API code.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681632502425/2d1801ca-0000-44e5-8958-877b3fa5401f.png align="center")

Now we are getting a much simpler message as below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681632587421/0d47e22a-24be-4891-803d-3977c609b249.png align="center")

# Resources used for this Deployment

As we have discussed, the Serverless Framework has implemented AWS **CloudFormation** to deploy the entire API. Here are the resources that have been created throughout this deployment which are:

* AWS Cloud Formation( To provision the Infra)
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681633805252/5b20cc31-aaf3-44fd-b7ca-8bd3fdf530df.png align="center")
    
* IAM Role( For getting access to AWS resources)
    
* Lambda( For serverless deployment)
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681633758000/7b1e9493-d0f3-4a0a-b233-3260f30f6821.png align="center")
    
* API Gateway(To generate the API which enables us to trigger the event)
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681632797935/94c01d4e-1f6a-4d01-aade-a58b8df85b20.png align="left")
    
* S3 Bucket( To store the project resources )
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681633676010/04745c9b-2378-457f-9648-f28bff6ebeb8.png align="center")
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681633302267/e0668691-cf17-4fc0-9e2e-34aa02af0ac1.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681633325329/bacb836c-e61a-441a-b726-dc8af882018c.png align="center")

# Deployment Diagram

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681634219786/e8fee60c-04d9-4911-aa49-8dd8117b0369.png align="center")

# Serverless Dashboard

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681634946598/ea5736e4-50bf-4022-9a8b-1ff8b8c26457.png align="center")

Now we can also see our application on Serverless Dashboard using the command

`serverless login` and choosing **login to the dashboard.**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681635056225/15d0c9b9-c20a-4a66-ba85-1abd9f586616.png align="center")

In the above image, we have deployed our project to Serverless Dashboard and now we can monitor this from the serverless dashboard itself as below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681635577947/c14ce481-9b7c-48e2-af80-180cfea0c93b.png align="center")

This Dashboard can further be used as a replacement for Grafana.

# Conclusion

Here we have used a tool called Serverless to provision our infrastructure by integrating it with our Cloud Provider AWS.

This tool uses several AWS-managed services and helps us to deploy our Application.