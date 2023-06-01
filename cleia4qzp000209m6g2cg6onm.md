---
title: "Must Have AWS Services For DevOps"
datePublished: Fri Feb 24 2023 08:36:47 GMT+0000 (Coordinated Universal Time)
cuid: cleia4qzp000209m6g2cg6onm
slug: must-have-aws-services-for-devops
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1677227482904/c18a6af6-bbc0-4988-b4f8-c760620878d6.png
tags: lambda, ec2, aws, devops, aws-fargate

---

# Why and How Much of AWS for DevOps?

AWS(Amazon Web Service) is a public cloud provider that provides 200+ cloud services covering infra, storage services, compute services, application services and many more that come under PaaS(Platform as a Service), IaaS(Infrastructure as a Service), SaaS(Software as a Service).

However from a DevOps perspective, we do not really need to master all the 200 services so, we will be picking our services depending on our responsibility. In this article, we will be covering the minimum required AWS services to get started with DevOps.

Moving forward, categorizing the required basic services into **3** parts which are:

* Authentication & Networking (IAM, VPC, KMS)
    
* Storage (S3, EBS, RDS, DynamoDB)
    
* Compute (EC2, ECS, Lambda, Fragate)
    

# Authentication & Networking (IAM, VPC, KMS)

## 1- IAM(Identity Access Management)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677140007457/beb5643b-04cd-41c1-a57b-6754045f3ffc.png align="center")

IAM stands for Identity Access Management. It is a web service that is used to securely control access to your AWS resources.

**NB: IAM** is a **Global** service.

Whenever someone logs in to AWS via aws console or through CLI or API, IAM principals must be authenticated to send requests(with few exceptions)

A principal is a person or application that can make a request for an action or operation on an AWS resource. So we have **users**, **roles**, **federated users** and **applications**.

We have a couple of policies available so that we can assign permissions to the users and then we can use policies to control access to the resources. These policies determine what the people in a group or individually are allowed to do.

### Root User

**The root user** has access to all the services on AWS, root user is when you initially create your AWS account and the email that you use to log into the AWS console for the first time.

The **root** user sets permissions with IAM policies, **granting only the permissions required to perform a task**. You do this by defining the actions that can be taken on specific resources under specific **conditions**, also known as **least-privilege permissions.**

## Creating a User

Go to access management and choose the user below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677140826421/438634fa-7f32-4249-9d69-17ac32aeb9f2.png align="left")

Click on **add users :**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677140904131/da3fc166-7d6e-4f08-aba1-a5cd941d6601.png align="center")

And going forward you can create users being the root user.

### What is Group and Why Group?

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677141308039/2657aea4-9477-4d52-bb1e-d13f71f753d9.png align="left")

An IAM user group is **a collection of IAM users**. User groups let you specify permissions for multiple users, which can make it easier to manage the permissions for those users.

Once the root user d defines a user name, he/she will be prompted with the above page. Where the root can attach policies to a user individually through **In-Line Policy** **or** can copy permission from an existing user/group **or** can create a group by attaching required policies and then include the user to the group it belongs.

So, creating a group and gathering users who need similar permissions under that group is the most **convenient** approach and is used across industries.

Once the group name is defined the root user can attach policies while creating a group, you can create custom policies or can assign AWS-managed policies. Below are some samples of AWS policies.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677141888180/b10bb7cb-887f-4980-972a-45badd8cbc04.png align="center")

Once the user is created, it can access **only allowed** services that are permitted through **Permissions Policies**. If the user tries to access a service that is not being allowed by the root then the user will be getting the below error as we follow the least-privileged principle:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677142952224/37894fc5-c326-4c46-8eae-550e72757755.png align="center")

### Policies and Types

**POLICIES** are the way to assign **PERMISSIONS**.

A policy is **an object in AWS that, when associated with an entity or resource, defines its permissions**. AWS evaluates these policies when a principal, such as a user, makes a request. Permissions in the policies determine whether the request is allowed or denied. Most policies are stored in AWS as JSON documents.

There are 2 types of policies :

1- **Identity-based policy**: These policies can be applied to users, groups or roles.

2- **Resource based policy**: These policies can be applied to resources such as S3 buckets which is a containers for storing data or DynamoDB which is a type of database.

## IAM Roles

An IAM role is an IAM identity that has some specific permissions. The roles are assumed by users, applications and services. Once roles are assumed, the IDENTITY of the user becomes the ROLE and they have the permissions assigned to the ROLE.

## IAM Roles vs Policies

**IAM Roles manage who has access to your AWS resources, whereas IAM policies control their permissions**. A Role with no Policy attached to it won't have to access any AWS resources.

## Service Control Policy

The Service Control Policy(SCP) controls the maximum available permissions. This policy can be applied from the root level. After root, management accounts are created which are not restricted and can do anything.

Tag Policy is applied to enforce **Standardization**. SCPs are a way to say that **"You are allowed to use specific resources up to a specific extent"**. It also enables the Management Access or Root account to override the permissions when needed.

**NB:** SCPs don't grant permissions however they control the permissions you are allowed to use.

**In-Line Policy**: Assigning policies directly to a user.

## Programmatic Access to AWS Console

Generally, we have been using the AWS console from the browser which is the GUI version. But when we will be forming infrastructures from **CloudFormation** or want to make any API or programmatic call for services like **code commit, git etc** we'll be needing **Access Key** and **Secret Keys** because for programmatic access we'll be using a terminal or code.

In the below image, you can see that am making a programmatic call to **AWS Console** from the terminal. Before making any call you need to configure **AWS CLI** in you system by downloading from [https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) and installing the CLI accordingly.

> aws configure : helps to configure AWS CLI in your system using the Access key and the Secret access keys as below highlighted.
> 
> aws --version : shows the version of AWS CLI
> 
> aws s3 ls : shows the buckets available in S3
> 
> aws s3 ls s3://bucketName : shows the list of items inside a bucket

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677148589482/014c2ae7-bf75-4de3-8dfd-f7a5368e5802.png align="left")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677149349894/02b194ab-3b92-4b3a-b3be-b937cea77468.png align="left")

We have been accessing the AWS console from GUI but above we are accessing the console from CLI where. **AccessKey and Secret Access Keys** play a vital role.

# Some IAM Best Practices

* Lock away your AWS account's access keys.
    
* Create individual IAM users or use AWS Organization.
    
* Use groups to assign permissions to users.
    
* Grant the least privileges.
    
* Get started with AWS-managed policies.
    
* Use customer-managed policies instead of in-line policies.
    
* Use access levels to review IAM permissions.
    
* Configure a strong password policy.
    
* Enable MFA(Multi-Factor Authentication)
    
* Use roles for applications that run on Amazon EC2 instances.
    
* Use roles to delegate permissions.
    
* Do not share access keys
    
* Change the passwords and access keys rotationally.
    
* Use policy conditions for extra security.
    

## 2- VPC

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677150970306/b1e953e3-2a91-4ccb-aebc-ef0611936d5b.png align="center")

A VPC is a virtual network dedicated to your AWS account. VPC is a logically isolated portion of the AWS cloud within a region. Within a region we can create multiple VPCs Each VPC has a different IP address

Each VPC is region-specific, so the VPCs in each region are separate and not connected whatsoever. It is possible to connect multiple VPCs in different regions using VPN or Inter-region VPC peering.

Provides complete control over the Virtual networking environment including a selection of IP ranges, creation of subnets and configuration of route tables and gateways. You can lunch your AWS resources such as EC2 instances into your VPC.

Within each VPC we have a big block of address which is called CIDR(Classless Interdomain Routing) within each CIDR we have multiple subnets which can be private or public subnets and subnets have different respective unique IP addresses.

When we create VPC we must specify a range of IPv4 addresses for the VPC in the form of a Classes Inter-domain Routing(CIDR) block, for example, 10.0.0.0/16

A VPC spans all the AZ in the region. You have full control over who has access to AWS resources inside your VPC

By default, you can create up to 5 VPCs per region however you can increase the limit. A default VPC is created in each region with a subnet in each AZ.

### Components Of VPC

**Subnet** - A segment of VPC's IP address range where you can place groups of isolated resources. Internet Gateway/Egress-only Internet Gateway - The amazon VPC side of the connection to the public internet for IPv4/IPv6

**Router** - Routers interconnect subnets and direct traffic between internet gateways, virtual private gateways, NAT gateways and subnets

**Peering connection** - Direct connection between two VPCs VPC endpoints - Private connection to public AWS services.

**NAT Instance** - Enables internet access for the EC2 instances in a private subnet managed by you. NAT gateways - Enables internet access for the EC2 instances in a private subnet managed by AWS.

**Virtual Private Gateway** - The amazon VPC side of a Virtual Private Network(VPN) connection. Customer Gateway - Customer side of VPN connection.

**AWS direct connect** - High speed, high bandwidth, private network connection from the customer to AWS. Security group - Instance-level firewall.

**Network ACL** - Subnet-level firewall

## Security Groups and Network ALC(Access Control List)

A firewall is essentially a security device that scans the incoming and outgoing traffic to the instance and checks whether they gonna be allowed or disallowed according to the rules we set.

### **Stateful VS Stateless**

1- A stateful firewall allows the return traffic automatically A stateless firewall checks for an allow rule for both the connection that is client and the web server

2- Security Group is stateful Network ACL(NACL) is Stateless and it operates at the Subnet level. It has either ALLOW rule or DENY rule. In NACL rules are processed in the order.

Network ACL is applied at the subnet level so that each subnet will be attached to respective ACL NACLs apply only to traffic entering/exiting the subnet.

Security Groups are applied at the Instance level. Security groups can be applied to instances in any subnet however when the traffic is going from one instance to another instance inside the same subnet then NACLs are not involved during the traffic routing. NACLs are involved only when there is inbound or outbound happens between 2 different VPCs.

Inbound Rule: Inbound means the traffics that is allowed to communicate Security group support allows rules only. A source can be an IP address or security group ID.

### VPC Peering

VPC peering is a way to connect VPCs privately using a private IP. VPC peering enables routing using private IPv4 or IPv6 Each VPC must have separate CIDR blocks that should not overlap.

VPC peering is not transitive which means, one VPC can not connect with another VPC keeping one other VPC in between they would connect directly to each other without any middleware.

VPCs can be in different accounts and regions. So bottom line is this way we can connect VPCs from different regions as well as different accounts through a private IP

VPC peering is not transitive which means, one VPC can not connect with another VPC keeping one other VPC in between they would connect directly to each other without any middleware.

VPCs can be in different accounts and regions. So bottom line is this way we can connect VPCs from different regions as well as different accounts through a private IP.

## 3- KMS

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677215393233/5b7ac2ae-ae4c-476f-9027-60cd40e065db.png align="left")

**AWS Key Management Service** (KMS) gives you centralized control over the cryptographic keys used to protect your data. The service is integrated with other AWS services making it easier to encrypt data you store in these services and control access to the keys that decrypt it.

### Advantages of KMS

* **Protect your data at rest:** Activate server-side encryption with AWS KMS using KMS keys that you control and manage.
    
* **Encrypt and decrypt data:** Use AWS Encryption SDK to securely handle cryptographic operations in your applications.
    
* **Sign and verify the digital signature:** Protect signing operations with AWS KMS using asymmetric KMS keys.
    
* **Validate JSON web token using HMAC:** Generate HMAC using AWS KMS to verify message integrity and authentication.
    

# Storage (S3, EBS, RDS, DynamoDB)

# 1- EBS(Elastic Block Store)

EBS is a storage service by Amazon that is used by EC2. Whenever we spin a server up it comes up with some EBS attached to it. EBS is where the OS and files of EC2 are stored persistently One EC2 instance can have one or more EBS volumes attached to it but we can't attach multiple instances to a single EBS volume. We can not attach an EBS volume of a different availability zone. The availability zone of the EC2 instance and its EBS volume must be in the same availability zone.

EBS volume data persists independently of the life of the instance. EBS volumes don't need to be attached to an instance You can attach multiple EBS volumes to an instance You can use multi-attach to attach a volume to multiple instances but with some constraints, Root EBS volumes are deleted on termination by default. Extra non-boot volumes are not deleted by default.

We have 2 types of EBS storage :

* EBS SSD(Solid State Drive)
    
* EBS HDD
    

## 2- S3(Simple Storage Service)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677216581170/f287f004-336f-4dac-9407-5fd97d0ec7ba.png align="left")

S3 is an Object-based storage service that stands for Simple Storage Service. We can connect to the S3 service through our browser using an HTTP request.

In S3 we create a container called Bucket to which we need to upload our files or objects.

we can connect in two ways:

[http://bucket.s3.aws-region.amazonaws.com](http://bucket.s3.aws-region.amazonaws.com)

[https://s3.aws-region.amazonaws.com/bucket](https://s3.aws-region.amazonaws.com/bucket)

**An object/file consists of :**

* Key(File Name)
    
* Version Id
    
* Value(actual data)
    
* Metadata
    
* Subresources
    
* Access control information
    

### Connection thru VPC

When we connect to S3 thru a VPC, the connection goes through an Internet Gateway. There is another way through which we can establish a private connection with S3 which is called S3 Gateway Endpoint.

### Features of S3

You can store any type of data in S3 Files can be anywhere from 0 bytes to 5TB There is unlimited storage available Your S3 bucket should be within a region but until and unless you configure it for different regions explicitly S3 is a universal Name Space so your bucket name should be unique globally Its a best practice to create buckets in regions that are physically closer to your users to reduce latency.

### Additional Features of S3

Transfer Acceleration: Speed up data uploads using CloudFront in reverse

Requestor pays: The requestor rather than the bucket owner pays for the requests and data transfer. Here requestor would be a different account.

Events: Triggers notifications to SNS, SQS, or Lambda when certain events happen in the S3 bucket.

Static web hosting: Simple and massively scalable static web hosting.

Versioning of application: Retain versions of Objects and replicate objects within and across AWS regions.

### S3 Versioning, Replication and Lifecycle Rules

* Versioning: It is a means of keeping multiple variants of an object in the same bucket. Use versioning to preserve, retrieve and restore every version of the object stored in your S3 bucket. Versioning enables us to recover objects from accidental deletion or overwrite
    
* Replication: S3 replication enables us to replicate objects from one bucket to another There are two types of replication-:
    
    1. Cross-region replication: This replication is done when there are buckets in two different regions.
        
    
    2)Same-region replication: Buckets can be in the same region Replication can be done between different accounts as well. Replication requires Versioning so, make sure you enable versioning before enabling replication.
    
    When we enable replication, new objects after replication enabled will be replicated to the destination, previously existing ones won't.
    

## 3- RDS(Relational Database)

RDS is a managed relational database service by AWS. RDS runs on EC2 instances so you must choose an instance type. RDS is an Online Transaction Processing type of database. Easy to set up, high availability, fault tolerance Common use cases are banking systems, online shopping transactions etc We can encrypt RDS instances and snapshots at rest by enabling the encryption option for RDS during creation The encryption uses the AWS key management service.

RDS supports the following database engines:

* Amazon Aurora
    
* MySQL
    
* Maria DB
    
* Oracle
    
* Microsoft SQL Server
    
* PostgreSQL
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677218938706/d51cb6f5-0fe0-4da0-ae5b-ac63960799dc.png align="center")

If the database we need is not in the above engine list then we need to install it in EC2 explicitly With RDS we can scale up (vertically )which means we can change the EC2 instance size for better storage and performance

In RDS we can also scale out(horizontally) and implement Disaster Recovery(DR).

In RDS we have RDS Master DB and an RDS Standby DB in another AZ. This Standby DB is used for Disaster Recovery. Standby DB always stays in sync with RDS Master and whenever any issue occurs the standby DB takes over. Standby DB is a Multi-AZ and primarily used for DR.

We also have another component called RDS Replica which is similar to Stand by DB but it stays in the same AZ as the master DB. It stays in sync but with a certain delay. Replica DB is used for scaling database queries(Reads) which is known as Horizontal or Scale-out

All the write operations happen to the master DB, Read-Only operations happen to the replica.

### Amazon Aurora(Relational DB)

**High Performance & Scalability:** Offers high performance, self-healing storage that scales up to 64TB, point-in-time recovery and continuous backup to S3.

**DB Compatibility:** It is compatible with existing MySQL and PostgreSQL open-source databases

**Aurora Replica:** In-region read scaling and failover target upto 15(Can use auto-scaling)

**MySQL Read Replicas:** Cross-region cluster with read scaling(Fast rereplication/low latency). Can remove secondary and promote

**Multi-Master:** Scales out writes within a region.

**Serverless:** On-demand,auto-scaling configuration for Amazon Aurora doesn't support read replicas or public IPs. Can only access it through VPC or Direct-connect.

## 4-Amazon DynamoDB(Non-relational)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677220151204/5c0ff282-4893-4dae-8a93-90688370ef61.png align="center")

DynamoDB is a non-relational and NoSQL database service from Amazon. DynamoDB is created as tables with different attributes. Examples Structure:

`UserID Name Location BestScore`

`38833 Ritesh Bhubaneswar 900`

**Serverless**: DynamoDB is a fully managed, fault tolerance.

**High availability**: 99.99 % availability SLA for global tables. NoSQL Type of the database with Name/Value structure: Flexible and good to use when data is unstructured or unpredictable. Horizontal Scaling: Seamless scaling with a push button scaling or auto-scaling.

**DynamoDB Accelerator**: Fully managed in-memory cache for DynamoDB that increases the performance Backup: Point-in-time recovery down to the second in the last 35 days; On-demand backup and restore Global Tables: Fully managed and multi-region, multi-master solution.

# Compute (EC2, ECS, Lambda, Fragate)

## 1- EC2

EC2 has several physical hardware available at the amazon data centers. On top of that physical hardware, ec2 hosts are managed by amazon. ec2 instance can run Windows, Linux, Mac etc. An ec2 instance is a virtual server. EC2 hots is managed by amazon where as ec2 instances are managed by the customer because the customer needs to take care of applications, patches, updates etc.

We have Region and inside Region, we have BPC(Virtual Private Cloud) which contains all the resources. In VPC the resources are private but the accessibility can be private. Inside Region, we have Availablity Zone and inside Availablity Zone, we have Public Subnet to which we can connect from the internet. But we also have a Private Subnet to which we can not connect from the outside.

Data is stored in EBS(Elastic Block Storage). A Security Group controls inbound and outbound traffic and EC2 is built from **AMI(Amazon Machine Image).**

The Internet Gateway enables access to and from the internet. The Admin connects to the instance from the internet through the Internet Gateway.

Security Group: It is a firewall that determines the protocol we can use to connect our instance MSWindows use a different protocol apart from SSH which is Remote Desktop Protocol.

## 2- ECS(Elastic Container Service)

AWS ECS(Elastic Container Services) is the aws service that allows us to run docker containers

### What is Container?

Previously we had web apps above operating systems above virtual processors and memories above Hypervisors above Physical Servers.

**BUT IN THE CASE OF ECS**

The Hypervisor is eliminated. Here we have Applications above Docker Engine or any other containers above Operating System above Physical Server The Docker engine gives us the ability to create something called Container.

A container gives us the ability to run an application but it doesn't have it's own Operating System. It is sharing the underlying host OS of the physical server. A container contains all the codes, dependencies and everything required to run an application/ There can be multiple containers and they all are isolated from each other. Containers start up very quickly and they are very lightweight

### AWS ECS

Amazon ECS can run across multiple availability zones.

ECS Cluster: an ECS cluster is a logical grouping of tasks or services.

Inside ECS Cluster we have the tasks

An EBS Task is a running Docker Container Each task has a Task Definition which describes, what the task does Each ECS Task is created from a task definition

### There are two different types of ECS(Elastic Container Services):

**1- EC2 Lunch Type**

You explicitly provision the EC2 instances You are responsible for managing EC2 instances and charged per running EC2 instances EFS and EBS integrations You handle cluster optimization and more granular control over the infrastructure.

**2- Fargate Lunch Type**

Farget automatically provisions resources Charged for running tasks No EFS and EBS integrations Limited control.

# Serverless Services

Serverless services are where we have a service that runs but doesn't have to manage the underlying server to manage.

For example, we don't have EC2 instances to manage patches, they automatically run, and scale.

S3 is an example of a Serverless Service as we do not need to manage any server to use S3 we simply upload files to it

Advantages of Serverless :

With serverless services there is no instance to manage You don't need to provision hardware There is no management of OS or software Capacity provisioning and patching are handled automatically Provides automatic scaling and high availability can be very cheap.

## ECR(AWS Elastic Container Registry)

ECR is nothing but AWS-customized **DockerHub .** The application of ECR is exactly the same as DockerHub.

Each task contains a registry. Docker images can be stored in Amazon ECR. All the images of a task are stored in the registry. When a task runs, it pulls the image from the registry.

**NB**: We will be doing the complete image creation and Containerization with AWS ECS in a separate hands-on blog.

**Step-1: We need to create a docker file and build platform independent image.**

**Step-2: After creating the image we need to push it to ECR which will be further pulled to ECS.**

## 2- **AWS Fargate**

AWS Fargate is a serverless, pay-as-you-go compute engine that lets you focus on building applications without managing servers. AWS Fargate is compatible with both ECS and EKS.

### How does Fargate Work?

To utilize Fargate to manage the deployment of your application container, you will need to have a container stored in a container registry like ECR or DockerHub and set up a task and cluster via ECS or EKS.

The steps in the deployment cycle are:

**Step-1:** Build a container image

**Step-2:** Host in a registry ie Amazon ECR or DockerHub

**Step-3:** Choose an orchestration service - either Amazon ECS or EKS

**Step-4:** Create a Cluster taking the AWS Fargate option

Your container Image is a read-only template that can be built from a docker file that contains your code, system libraries, tools, runtime and other dependencies required by your application to deploy. The docker file is a plain text file that details all of the required components that generate the container Image for storage in a container registry like DockerHub or Amazon Elastic Container Registry.

Once in the registry, your container can be pulled into a cluster instance to be run whenever required.

To set up the container to run using ECS you will then need to set up a task definition. Typically a task definition is a JSON file containing a blueprint of the requirements and settings needed to run the container. Container definitions like the image type, memory, CPU & network mode.

## 3- AWS Lambda

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1677227576140/12a9af51-94a6-4476-a3a0-0e8f269e2d52.png align="center")

AWS Lambda is a serverless service that enables you to run compute functions which means you are able to process code and it does so in response to triggers.

### How does it work?

When a developer uploads some code to lambda functions and an event triggers that can be from CLI, API, SDK or any scheduled triggers then the code gets executed and automatically scale if needed.

We only have to pay for the time code is executed not for storing the code which is cost-effective. We only pay for the compute time, and the amount of memory it consumed during the execution.

### The primary usage of Lambda:

Data processing. Real-time file processing. Stream processing. To build a serverless backend for web, mobile, IoT, and 3rd party API requests

AWS Lambda is a serverless service by AWS. You won't need to worry about infrastructure

### Lambda-Supported Languages:

Java, GO, java, python, ruby, NodeJS, C# and all

### Project :

I will be uploading a complete project and will link to it ASAP..!!

Happy Learning :)