---
title: "Multi-tier WebApp Deployment Migration to AWS"
datePublished: Fri Jun 16 2023 08:55:14 GMT+0000 (Coordinated Universal Time)
cuid: cliyc3vsm000209l6dh535oci
slug: multi-tier-webapp-deployment-migration-to-aws
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1686905570005/3ec0e8ee-d811-430b-b320-b8705be620be.png
tags: paas, deployment-automation, devops, developer-tools, awsdevops

---

# Project Overview & Architecture

In this project, we'll re-architect the [multi-tier application deployment stack](https://hashnode.com/post/clihac37j000309l758si1lee) with **AWS Services**.

All the services such as EC2, MySQL, Memcached, RabbitMQ and Tomcat, etc.. will be replaced by respective **AWS-Managed** Services.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686568655393/f1bb3329-4c22-4421-b4c9-822759573818.png align="center")

* Here, the user will hit the URL which will be routed to and from the **Route53** DNS service. This **Route53** will be configured with the **Amazon CloudFront URL** and when the user request comes to Route53 DNS it will redirect it to CloudFront(the CDN service)
    
* The request will further be redirected to the **Application Load Balancer** which is managed by **Beanstalk.**
    
* The load balancer will further forward the traffic to the instances in **AutoScaling Group** which are also managed by Beanstalk itself.
    
* For backend services, the request will further access **Amazon MQ** as queuing service which will further take the request to **Elastic Cache** and **RDS.**
    
* All these will be monitored by the **Amazon CloudWatch**
    

## Self-Managed vs AWS Managed Services

Here we'll see a brief explanation of our self-managed services that we implemented previously and their respective AWS-Managed alternatives :

| Self Managed &lt;-- | \--&gt; AWS Managed |
| --- | --- |
| EC2 | Beanstalk |
| MySQL(MariaDB) | RDS Instances |
| Memcached | Elastic Cache |
| RabbitMQ | ActiveMQ |
| Route53 | Route53 |

* **Beanstalk:** Previously we launched EC2 instances to install our services like Tomcat server, MySQL, etc. as well as set up the Autoscaling and Load Balancing(NGINX replacement) for them manually. Now, beanstalk will automate all these tasks. It'll also set up **S3** buckets automatically.
    
* **MySQL DB:** Previously we installed MySQL MariaDB as our database service now, instead we'll use **RDS Instances** which is a platform as a service. It allows us to choose the database, scaling will be easy, and regular backups are taken.
    
* **ActiveMQ:** RabbitMQ will be replaced by ActiveMQ as queuing service.
    
* **Route 53:** It will be used as DNS. we are not replacing it as it is already an AWS-managed service.
    
* **CloudFront:** It is an AWS-managed content delivery service for a global audience.
    

# Flow of Execution

1. Create key-pair for Beanstalk Instance login
    
2. Create a Security Group for backend services(Elasticache, RDS, Active MQ)
    
3. Create RDS
    
4. Create Amazon Elastic Cache
    
5. Create Amazon Active MQ
    
6. Create Elastic Beanstalk Environment
    
7. Update the Security Group of backend services to allow traffic from **Beabstalk** Security Group.
    
8. Update SG of the backend for internal traffic.
    
9. Lunch an EC2 instance for DB initialization by SSH to RDS DB
    
10. Change **Healthcheck** on Beanstalk to "**/login**"
    
11. Add **443** Https listener to our ELB(Elastic Load Balancer)
    
    **NB:** ELB will be created by Beanstalk automatically
    
12. Build **Artifact** by providing backend information in `application.properties` file
    
13. Deploy the Artifact to Beanstalk
    
14. Create **CDN(Content Delivery Network)** with SSL Certificate using **Amazon CloudFront**
    
15. Update entry in Godaddy DNS zone
    
16. Test the URL.
    

# Step-1(Creating Key-Pair)

Going forward we'll be using **Elastic Beanstalk** which will take care of launching EC2 instances, ELB etc. However, in some cases, we might need to log in to a particular EC2 instance separately so to be on safer since we are creating Key-pair.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685462364450/8eda12b7-8754-4c06-8c72-fb1247c62283.png?auto=compress,format&format=webp align="left")

**Click on Key Pairs--&gt; Create Key Pair--&gt; Fill in the configuration--&gt; Click on Create Key Pair at the bottom**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686580736225/1b5517f4-3da6-4f48-8e4e-46cb93b2a2de.png align="center")

Here I have chosen **.pem** type as the Key format as I'll be connecting to the instance through **GitBash**. If you are connecting with Putty you can go with **.ppk.**

# Step-2(Creating SG for Backend Service)

Here we'll be creating a security group named `VProfile-backend-SG` which will first have access to **itself** for internal communication. Below you can see that this security group has inbound access only to itself:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686581459026/10d51824-f04e-4f98-8a64-40a75e6d8408.png align="center")

# Step-3(Setting up RDS)

RDS is an AWS-managed Relational Database Service that provides ample customization, auto-scaling, snapshots, mult AZ for high availability and many more.

## Creating Subnet Group

Before creating the database we need to create a Subnet Group.

**Subnet Group:** An RDS Subnet Group is a collection of subnets that you can use **to designate for your RDS database instance in a VPC**. Your VPC must have at least two subnets. These subnets must be in two different Availability Zones in the AWS Region where you want to deploy your DB instance.

To create the Subnet Group we have made the below configuration where the name of the Subnet Group is `vprofile-rds-subnetgroup` , **VPC** is set to `default`, for **Availability Zones** we have chosen all the AZs for high availability, our DB instance will be placed in any of these AZs, and also chosen all the **Subnets** out of all the AZs.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686635265820/0594d042-75ef-4014-96df-7ad0b9a58808.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686635287589/115dde7c-d58c-4d90-bf65-e1d588c44540.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686635566343/bd1410d9-1de6-46c0-9e84-c13af3e9ad72.png align="center")

## Creating Parameter Groups

Parameter Groups are nothing but the settings or configurations of our database. You **manage your database configuration by associating your DB instances and Multi-AZ DB clusters** with parameter groups. Amazon RDS defines parameter groups with default settings. It allows us to define our own parameter groups with customized settings.

We have created the **Parameter Group** with the below configurations where we have used **mysql5.7** as per our project requirement and the name of the group is `VProfile-RDS-ParameterGroup` :

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686635958297/b4990696-efa5-4b51-8f41-bb6dece227e1.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686636095928/4cd5ccae-7c31-4bcf-9b13-e25b47872d9c.png align="center")

## Creating Database

Now all the prerequisites have been completed and we can create our database.

Here, we will go for Standard Create which allows us to customize. And we have chosen **MySQL** as our **DB Engine** with **MySQL 5.7.37** as per our project

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686636226363/de47f2b6-f1a5-4fb9-9ceb-2e05ec4bfd55.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686636398514/133e3723-0804-4249-8845-c1a0b05ae1a6.png align="center")

For templates, we have chosen **Dev/Test** with **Multi-AZ DB Instance** for high availability. Production template is very expensive and required for large-scale projects.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686636581930/672804a1-1e09-4b19-90ea-9eb5778ae669.png align="center")

The setting is as below where the name of the DB is `VProfile-MySQL-DB` :

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686636713734/0509407f-60f4-46ee-9a75-bdd2ae2aa500.png align="center")

Instance configuration is similar to our regular EC2 configuration we have taken **db.t2.micro** as it provides **2 CPUs** and it is also storage and network optimized:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686636936472/54aaa802-ba3a-42c1-bf51-baea829c1c5f.png align="center")

For storage we have Autoscaling enabled:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686637049945/0d1ba375-630d-4003-aa52-ebbc24677d15.png align="center")

For connectivity, we have selected the **default VPC** and our `vprofile-rds-subnetgroup` that we created especially for our DB and this DB is assigned to the existing security group `VProfile-backend-SG` that we created earlier:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686637443359/a66c9c07-6fbf-488f-b3f4-a3403e733f74.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686637573211/154f1fb9-6ab6-40c5-a95c-a5ece8652f2d.png align="center")

For additional configuration:

Here we are creating an initial database as **accounts** for our project, the database will be configured according to the DB parameter group `vprofile-rds-parametergroup` that we created:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686637894895/d816e62d-1dac-4b82-a74f-5c0f02e5464a.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686637832482/719f0113-da1d-4ec2-9746-f98fe04af500.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686637845943/aad1642c-4e89-489c-98d2-44f844d2f58c.png align="center")

And finally, with the above configuration, create the DB:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686640845023/950978c9-fd7e-46cc-bfc7-526f4ba0cc13.png align="center")

# Step-4(Setting up ElastiCache)

As discussed earlier, it is one of our backend services that will replace our previous self-managed service **Memcached.**

For Elastic Cache also, we have to create a **Subnet Group** and **Parameter Group**.

## Creating Subnet Group

So we have created the Subnet Group named `vprofile-elasticache-SubnetGroup` as below with subnets of all the AZs exactly the same way we did for RDS:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686643782748/4a9baa27-5058-4b7e-b353-6f4dfd6bed8b.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686643831605/218a7219-e7a9-4974-bb07-4905ff477ea2.png align="center")

## Creating Parameter Group

Here we have created a Parameter Group named `vprofile-elasticache-ParameterGroup` which has the same functionality as **memcached1.4:**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686644068347/e9888052-cbc0-4c71-aaf4-bfcef0955a6d.png align="center")

## Creating Elastic Cache Cluster

In the Elastic Cache dashboard we have 2 options to choose from and as we have chosen Memcached in the Parameter Group we'll go with Memcached Cluster:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686644482514/504cff19-cb2f-4689-98ee-fe7967fafcb3.png align="center")

**Step-1**

Here we have specified the cluster name as `vprofile-elasticache-svc` , attached to the respective Parameter Group that we created earlier and **Node Type** is t2.micro which is the instance type that will be used for Elastic Cache:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686644750113/be4454fa-9994-4d5c-b256-92d94ce47c53.png align="center")

**Step-2**

Here we are attaching the elastic cache to the security group `VProfile-backend-SG` that we prepared earlier and also setting up the cloud watch alarm:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686645046677/fd9fe258-0671-4a88-94cb-0a3eeef01448.png align="center")

And with this, we have created our AWS-managed caching service for our application.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686645297798/a482e42f-9775-4f0c-bd8b-e69fd979ee25.png align="center")

# Step-5(Setting up Amazon MQ)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686645432244/ed863841-b0f3-47d6-a047-e31ba4e98759.png align="center")

Now we'll be creating the AWS-managed broker service with will be a replacement for the RabbitMQ that we previously used as one of our backend services.

However, Amazon MQ will internally use the Rabbit MQ engine as per our requirement.

First of all, we are selecting the Broker engine type which is RabbitMQ for our project.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686645865510/f26dd94d-8c7a-4f7f-a39f-2f62fd5278a9.png align="center")

For our project, the deployment mode will be a Single-instance broker as we are deploying on a small scale. For large-scale production-level deployments, we should use Cluster Deployment:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686645948463/d557aac6-f64a-4073-a59c-be92454e57d0.png align="center")

And for the configuration we have given the name as `vprofile-rmq` , attached it to the **backend security group** and the **access type is private** as it will be privately accessed by **Beanstalk.**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686646855430/ce053fad-e45f-43f1-8961-a9cf8f94e02a.png align="center")

# Step-6(DB Initialization)

We have set up all our backend services and now we just need to initialize the DB which will be the last step for our Backend Setup.

As we know, RDS is a PaaS service by AWS. It has provided the platform and required element to construct our DB however, it's our responsibility to make it workable according to our project requirement for which we have to initialize it by ourselves. To do so, we'll launch an **EC2 instance and install the MySQL client(** `sudo apt install mysql-client` **)** that will help us to connect to the RDS endpoint.

### logging in to RDS

`mysql -h` [`vprofile-mysql-db.cezpawefwtx5.us-east-1.rds.amazonaws.com`](http://vprofile-mysql-db.cezpawefwtx5.us-east-1.rds.amazonaws.com) `-u admin -p`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686717574311/293d97bc-d406-41af-b9de-967a4f7883fc.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686717654782/9b358ab9-f9c2-4bcc-aba7-2435f89b8251.png align="center")

And above we can see that we have a database named **accounts** that we configured during RDS configuration.

### Initialization

Now to initialize the **account** database we have to apply the schema that is there with our project and to do so we have to clone the project to our DB-Client instance.

`git clone` [`https://github.com/rkn1999/vprofile-project.git`](https://github.com/rkn1999/vprofile-project.git) and move to the branch where our DB schema is available `db_backup.sql`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686718273934/32ff43eb-e2d4-4e44-8343-21bb1aa249fc.png align="center")

We have to initialize the DB with the above schema `db_backup.sql` with the same command `mysql -h` [`vprofile-mysql-db.cezpawefwtx5.us-east-1.rds.amazonaws.com`](http://vprofile-mysql-db.cezpawefwtx5.us-east-1.rds.amazonaws.com) `-u admin -p accounts > db_backup.sql` within the same directory where this schema is present.

And we got 3 tables created in our database **accounts:**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686719047129/ee271fa5-e36c-4e65-b4ed-d2af06d2dd66.png align="center")

And with this, our DB initialization has been completed.

# Step-7(Setting up Elastic Beanstalk)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686720528865/3b54fb76-f062-4c3c-9445-997568dbbb4d.png align="center")

### Creating IAM Roles for Beanstalk

Beanstalk will launch several instances on our behalf so, we have to create an IAM Role as we usually do for EC2 instances and other services. Beanstalk also creates some roles by itself. To create roles move to IAM and create a role with the below permissions:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686721027807/9d166a2e-d0dc-43f9-a1c4-9705cf67b251.png align="center")

And the role is created:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686721293984/a970a43a-4bf1-4ef3-a077-455a2c5bd316.png align="center")

## Creating the Elastic Beanstalk

So, after creating the Roles we'll configure the Beanstalk for our application and the configuration would be as below:

* Our application is a webpage so for environment we are selecting Web Server, name has been given to our project and the domain name is `vprofileapp-by-ritesh.`[`us-east-1.elasticbeanstalk.com`](http://us-east-1.elasticbeanstalk.com) that has to be unique. Platform is chosen as **Tomcat** as our application artifact will be deployed to Tomcat server so beanstalk will take care of that.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686881005696/bf5751f8-232a-4200-b5fe-30d8ca55a3e6.png align="center")

* Below we have given respective role configuration as well as the key-pair for the EC2 instances that will be created by Beanstalk.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686881352876/2efdc383-c767-4469-b6a5-9805824c1081.png align="center")
    
* At this step, VPC is set to default,we have chosen multiple subnets so that it can launch instances on different subnets and high availability. For **Databases** we are using RDS explicitly hence, here we have left this blank, this database will be created by Beanstalk and it will also be deleted along with the instance deletion so, it is not recommended for **production** usage, it's good for Dev and Test environments.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686881633235/cde09a5a-4f1f-4a43-91c5-696acc9d3171.png align="center")
    
    * At this step we have configured volume, load balancing and autoscaling based on the network traffir.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686882295171/0b5401ab-1459-4958-bc25-9d801edb923d.png align="center")
        

At this step we have configured update and monitoring. the rolling update section is very much important that works on basis of the deployment policies.

[Deployment Policies](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.rolling-version-deploy.html)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686882758675/ef708805-afef-4271-afbd-06339e60fa51.png align="center")

And finally, our stack is complete as we have submited the stack which is getting created as below, once it is done we can be able to access .

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686883104174/b90ec49a-4837-4a4b-92c8-ade1225e76fd.png align="center")

* With the configuration our stack is completed and it has deployed a dummy application as below that will be replaced by our application:
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686883299327/1287d2d0-f4fd-4c66-b865-0007b32b06cb.png align="center")
    

# Step-8(Updating Security Group & ELB)

Here we'll do 3 things:

1) Enable ACL in S3 Bucket

2) Update the health check in the Target Group

3) Update Security Group

### Enable ACL in S3 Bucket

**ACL** stands for **A**ccess **C**ontrol **L**ist, It is a mechanism that allows you to control access to your S3 buckets and objects within them. ACLs are used to specify who can perform specific actions (such as read, write, or delete) on your S3 resources. With ACLs, you can grant permissions to individual AWS accounts or predefined groups (such as Amazon S3 Log Delivery group) to control access at the bucket or object level.

Here we'll be enabling the ACL for **Elastic Bean Stalk** so that it can access the artifact that will be uploaded to S3 bucket without any error at S3-bucket level. To do so we have to **Enable** **the ACL** with t***he below configuration*** which means only the bucket owner can have the access so that, it can while beanstalk resources of my account will try to access the objects in this [elasticbeanstalk-us-east-1-017228101718](https://s3.console.aws.amazon.com/s3/buckets/elasticbeanstalk-us-east-1-017228101718) bucket, it will allow the request:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686892703612/e59582d4-26e1-4675-805b-43c21a66fb61.png align="center")

### Update the Health Check Path in the Target Group

Our application Vprofile listens at the URL **"/login"**and the Target Group will perform the Health Check on this path. So that, we have to change the in the **health check path** as below and to do so: move to the [`Elastic Beanstalk`](https://us-east-1.console.aws.amazon.com/elasticbeanstalk/home?region=us-east-1#) `>` [`Environments`](https://us-east-1.console.aws.amazon.com/elasticbeanstalk/home?region=us-east-1#/environments) `>`[`VProfile-application-prod`](https://us-east-1.console.aws.amazon.com/elasticbeanstalk/home?region=us-east-1#/environment/dashboard?environmentId=e-c4jpvudv37) `> Configuration > Instance traffic and scaling > Processes` :

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686893691791/52bd9666-7bd7-4702-9d11-c8df96daf70d.png align="center")

Adding **Listener:**

Here we are adding HTTPS listener at the port 443 to access the application on this on this port:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686894890237/70746023-07aa-4154-8bd8-a6f37c80f0c2.png align="center")

### Update Security Group

We have update the security group because all our backend services such as RDS, Amazon MQ, and Elastic Cache are in backend security group. Instances of the Beanstalk will access these backend services at their respective port numbers so that

Here we have to take the Security Group ID of the instances that are up by Elastic Beanstalk for our application deployment and have to tie it up with Backend Security group so that it will have access through this:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686896835203/c73b44e5-3262-4560-afe3-5de3bac66c87.png align="center")

Now the backend security group is allowing the traffic from the front-end security group(Instance Security Group).

# Step-9(Project Build & Upload Artifacts)

At this step we'll build our application source code with maven that will generate the deployable artifacts which will further be uploaded to S3 bucket.

Before building the source code we have to update the `application.properities` file in the project with all our backend server information which is like the below where we have added the endpoints of the services and saved it.

```ruby
#JDBC Configutation for Database Connection
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://vprofile-mysql-db.cezpawefwtx5.us-east-1.rds.amazonaws.com:3306/accounts?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull
jdbc.username=admin
jdbc.password=LinkDYymco62ZWFy7rRG

#Memcached Configuration For Active and StandBy Host
#For Active Host
memcached.active.host=vprofile-elasticache-svc.plafv0.0001.use1.cache.amazonaws.com
memcached.active.port=11211
#For StandBy Host
memcached.standBy.host=127.0.0.2
memcached.standBy.port=11211

#RabbitMq Configuration
rabbitmq.address=b-8e65c9d5-6d03-4828-bcb5-75c427966934.mq.us-east-1.amazonaws.com
rabbitmq.port=5671
rabbitmq.username=RabbitMQ
rabbitmq.password=Test@riteshthedrvops123

#Elasticesearch Configuration
elasticsearch.host =192.168.1.85
elasticsearch.port =9300
elasticsearch.cluster=vprofile
elasticsearch.node=vprofilenode
```

### Build( `mvn install` )

With mvn install we have built the project and generated the war file.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686898995836/ac6a8e56-3b99-4352-b610-d552378c356b.png align="center")

And we have got the .war file in target folder which we'll be uploading to S3:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686899081967/cb20dc4c-4706-406b-a02b-24d27fb54475.png align="center")

# Step-10(Upload & Deploy)

With this we are uploading the artifact from the beanstalk environment :

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686899542253/28a040a3-e747-4b08-8197-5ec8b6b691d4.png align="center")

And here you can see the artifact is listen in Application version and we need to upload it:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686900045120/070e7464-840f-4633-80e7-08357fe2b448.png align="center")

And then from the actions drop down we can deploy it to the **environment** we want. Here we have only on environment that is Prod so we are deploying to that environment:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686900174576/4e922cae-51e0-4cdd-9faf-e985941bf129.png align="center")

And finally the deployment has been completed to the beanstalk:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686901051994/77f9b3ac-c012-4aa8-ab8b-2996e07784d3.png align="center")

# Step-11(Updating CNAME record in GoDaddy DNS)

Now, the domain of the Beanstack Environment [vprofileapp-by-ritesh.us-east-1.elasticbeanstalk.com](http://vprofileapp-by-ritesh.us-east-1.elasticbeanstalk.com) will be added to the CNAME record in GoDaddy DNA as below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686903832968/0d7a2f91-681a-40ab-9fe6-003a5f1542a6.png align="center")

Finally the app is up :

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686903937323/ba1929b0-98e4-40a4-8c16-7e0fefd8818d.png align="center")

# CloudFront( Content Delivery Network by AWS)

AWS Cloud Front is a content delivery network by AWS that helps us to serve our application across multiple Regions.

Currently our application is hosted in N. Viriginia region and with the use of CloudFront service this application can be deployed across multiple Regions.