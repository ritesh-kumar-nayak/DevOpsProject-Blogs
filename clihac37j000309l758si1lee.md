---
title: "E2E Lift & Shift Multi-tier Web Application to AWS"
datePublished: Sun Jun 04 2023 10:33:33 GMT+0000 (Coordinated Universal Time)
cuid: clihac37j000309l758si1lee
slug: e2e-lift-shift-multi-tier-web-application-to-aws
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1685364144541/d899f8c3-2f06-496c-95b2-c5f33f51b5b3.png
tags: aws, devops, devops-articles, 90daysofdevops

---

# Project Overview & Architecture

* In this project, we'll be hosting a **Multi-tier** Java Based social media web application called VProfile on **AWS** for **production** using **Lift & Shift Strategy.**
    
* The users will access the website using an URL and that URL will be pointing to an endpoint which will be an entry mentioned in GoDaddy DNS.
    
* This end-point from the user's browser will be connecting to the **ELB(Elastic Load Balancer)** by using HTTPS. This load balancer will be in a **Security Group** which only allows **HTTPS** requests.
    
* The **Certificate** for HTTPS encryption will be mentioned in **ACM(Amazon Certificate Manager)**.
    
* Then our application load balancer will route the application traffic to the **Tomcat server(EC2 instance where Tomcat is set up)** because our application will be hosted on the Tomcat server only. The tomcat servers will be managed by **Autoscaling Group** which can automatically scale in and scale out depending on the user load.
    
* **S3 Bucket** will be connected to the Tomcat server where our **project artifacts** will be stored.
    
* As we know our application needs certain backend services support such as **SQL DB, RabbitMQ, and Memcached.** These necessary services will be installed and set up in **3 different EC2 instances.** The Tomcat server needs the support of these servers altogether to make the application fully functional. The information about the backend services will be mentioned in a **Route53 private DNS zone.** The tomcat service will access the backend server with a name mentioned in **Route53 private DNS** where the **Private IP** addresses of our backend servers will be mentioned. These backend services will be attached to a separate security group.
    

**NB:** Earlier I had set up this project stack on my local system:

[Click here to see the Local Setup and Deployment of this project](https://hashnode.com/post/cli3e37kx000609l7gus78p5w)

# AWS Services to be Implemented

* **EC2 Instances:** VM for Tomcat, RabbitMQ, Memcached, and MySQL
    
* **ELB(Elastic Load Balancers):** Nginx Replacement
    
* **Autoscaling:** Automation for VM Scaling
    
* **S3/EFS Storage:** Shared Storage
    
* **Route53:** For private DNS service
    
* **ACM(Amazon Certificate Manager)**
    
* **IAM:** User and authentication management
    
* **EBS:** Elastic block storage
    

# The flow of Project Execution

1. Login to AWS Account
    
2. Create Key-Pairs
    
3. Create Security Groups
    
4. Lunch Instances with **User Data** Bash script
    
5. Update IP to Name Mapping in **Route53**
    
6. Build Application from source code in Local system
    
7. Upload artifacts to the **S3** bucket
    
8. Download Artifacts to the **Tomcat** EC2 instance
    
9. Setup **ELB** with HTTPS\[Certificate from Amazon Certificate Manager)
    
10. Mapping ELB Endpoint to the website name in **Godaddy DNS**(I have purchased DNS from GoDaddy).
    
11. Verify services are connected properly
    
12. Creating **Autoscaling Group** for **Tomcat** **Instances.**
    

# Step-1(Creating Security Groups)

A security group **acts as a firewall that controls the traffic allowed to and from the resources in your virtual private cloud (VPC)**. You can choose the ports and protocols to allow for inbound traffic and outbound traffic.

**NB: For now we are just creating the security group for all the EC2 instances(Servers). We'll attach these security groups while lunching the instances and configuring them.**

## Step-1.0(Creating a Security Group for ELB)

As you can see in the above project architecture, we need a load balancer for routing the request to our application. For that, we need to specify certain security rules that will define the kind of requests that are allowed to be routed or to access our application.

**Login to AWS console--&gt; click on EC2--&gt; Click on Security Group--&gt; Create a security group**

Initially, you will see only **default** security groups as below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685457727304/78d208a6-7cce-446c-ab73-c07f54f69842.png align="center")

`SG Name: VProfile-ELB-SG`

### **Editing in-bound rules:**

In the Inbound rule section, we define the IP address range from which our application will be accessible. So our inbound rule for this load balancer will look like the below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685458245916/5e56e4ab-9e96-4462-92f5-74aa4f15df12.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685458196528/28436332-f71c-4380-8a2e-7545de0b405d.png align="center")

As our load balancer will encounter access from normal users, we have given access to HTTP and HTTPS ports.

## Step-1.1(Creating Security Group for Tomcat Server)

`SG Name: VProfile-app-SG`

### **Editing in-bound rules:**

The Tomcat server runs on port **8080** hence, we are opening port 8080 below.

For the Tomcat server, the request will only come from our **ELB** hence, we just need to allow access to our ELB by attaching the **Security Group** of the ELB as the **Source.** So that, when ELB redirects the request to the Tomcat instance, it'll validate from its security group.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685459179689/b9b9f3aa-c9c2-4da4-86f6-7e851dde1503.png align="center")

## Step-1.2(Creating Security Group for Backend Services)

`SG Name: VProfile-BackendServics`

Our backend services include **MySQL, Memcached, and RabbitMQ** all together.

These services will interact with each other as well as with the Tomcat Server.

So, in this Security Group, we have added **4 Inbound Rules. 3** for different ports such as **5672, 11211, and 3306** for **RabbitMQ, Memcached, and MySQL** respectively. The **highlighted one** allows **all traffic** that enables internal routing/communication among all these **3** backend services hence its source is **this** **security group** itself.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685461656922/3a1777d4-0346-483f-8642-1134bdf3ef04.png align="center")

### Editing outbound rules:

Outbound rules define the requests that can go out **from** our **servers.** And our servers should be able to route traffic everywhere and should be able to send responses to all IP addresses hence, the outbound rules remain unchanged to **All Traffic**:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685458501996/7d80e585-b02c-4e33-acd9-e600e10b3afa.png align="center")

# Step-2(Creating Login Key Pair)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685462364450/8eda12b7-8754-4c06-8c72-fb1247c62283.png align="center")

**Click on Key Pairs--&gt; Create Key Pair--&gt; Fill in the configuration--&gt; Click on Create Key Pair at the bottom**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685462466095/accbae56-2042-40b5-9021-7c75d20bbac2.png align="center")

Here I have chosen **.pem** type as the Key format as I'll be connecting to the instance through **GitBash**. If you are connecting with Putty you can go with **.ppk.**

# Step-3(Lunching EC2 Instances)

After creating the Security Key, we'll lunch our required EC2 instances for our respective services such as Tomcat, RabbitMQ, MySQL, etc.

While launching the instances we'll use **User Data** and we'll provide the **Shell Script** we prepared previously for each service while doing manual provisioning.

**User Data:** it is an advanced way of launching EC2 instances that will automate the dependency installation, service installation, etc through the **shell script** provided and will give us a fully prepared server for our use. This user data eliminates the effort for manual provisioning or setup of services after launching the instance.

## Step-3.1(Lunching Instance for MySQL-DB)

### **Instance OS:** `CentOS 7`

For MySQL we require CentOS 7 hence, we are selecting the same from AWS Market Place.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685597229177/3d64642d-6b50-41fe-8c11-f0dbc7b01d27.png align="center")

### **Instance Type:** `t2.micro`

Our instance type should be t2.micro which provides the below configuration and it's a free tire.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685597286566/47a87dd1-5936-4048-95f6-cbe44c7ce2eb.png align="center")

### **Instance Name and tags:** `VProfile-DB01, Project: VProfile`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685598437066/679a87d7-3d51-4f5d-b448-abc45b276306.png align="center")

### **Key Pair:** `VProfile-Prod-Key`

Every time we lunch any instance we need a Key Pair and for our instances, we have already created a Key-Pair which is **VProfile-Prod-Key** hence, we are just using that instead of creating a new pair.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685597337347/bce5a45e-f0cc-4f8a-83b3-3fb76ace9546.png align="center")

### **Security Group:** `VProfile-BackendServices-SG`

Whenever any instance is launched, it requires a **Firewall(Security Group)** to be attached to it to control the traffic. We have already prepared the **Security Group** configuration in **Step-1.2** for Backend Services. As **MySQL** is one of the Backend Services it will be attached to the respective Security Group.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685597523954/224de013-9683-415a-b74c-d31fd8733bf0.png align="center")

### **Advanced Details-&gt; User Data:**

In this section, we have defined the shell script for Service installation and configuration. And the script is also written below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685597728245/39ff0fda-e501-4f9e-943c-11c0a19bd4d1.png align="center")

```bash
#!/bin/bash
DATABASE_PASS='admin123'
sudo yum update -y
sudo yum install epel-release -y
sudo yum install git zip unzip -y
sudo yum install mariadb-server -y


# starting & enabling mariadb-server
sudo systemctl start mariadb
sudo systemctl enable mariadb
cd /tmp/
git clone -b vp-rem https://github.com/devopshydclub/vprofile-repo.git
#restore the dump file for the application
sudo mysqladmin -u root password "$DATABASE_PASS"
sudo mysql -u root -p"$DATABASE_PASS" -e "UPDATE mysql.user SET Password=PASSWORD('$DATABASE_PASS') WHERE User='root'"
sudo mysql -u root -p"$DATABASE_PASS" -e "DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1', '::1')"
sudo mysql -u root -p"$DATABASE_PASS" -e "DELETE FROM mysql.user WHERE User=''"
sudo mysql -u root -p"$DATABASE_PASS" -e "DELETE FROM mysql.db WHERE Db='test' OR Db='test\_%'"
sudo mysql -u root -p"$DATABASE_PASS" -e "FLUSH PRIVILEGES"
sudo mysql -u root -p"$DATABASE_PASS" -e "create database accounts"
sudo mysql -u root -p"$DATABASE_PASS" -e "grant all privileges on accounts.* TO 'admin'@'localhost' identified by 'admin123'"
sudo mysql -u root -p"$DATABASE_PASS" -e "grant all privileges on accounts.* TO 'admin'@'%' identified by 'admin123'"
sudo mysql -u root -p"$DATABASE_PASS" accounts < /tmp/vprofile-repo/src/main/resources/db_backup.sql
sudo mysql -u root -p"$DATABASE_PASS" -e "FLUSH PRIVILEGES"

# Restart mariadb-server
sudo systemctl restart mariadb


#starting the firewall and allowing the mariadb to access from port no. 3306
sudo systemctl start firewalld
sudo systemctl enable firewalld
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --zone=public --add-port=3306/tcp --permanent
sudo firewall-cmd --reload
sudo systemctl restart mariadb
```

### **Lunch Instance:**

Finally, the instance for MySQL DB is up and running. It will take some time to configure and be stable as it will be executing the above shell script for setting up the entire Database.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685598691606/f39cf279-f44a-4527-916c-f5b68b645543.png align="center")

### **SSH Connection to DB Instance:**

So, to connect to the DB instance through SSH we have to **Open** **SSH port 22** in the security Group and the **Source** should only be **My IP** as it should only allow me to access the server.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685599091831/2eb2d894-c510-4d2b-a8f0-7d8b907dcbb1.png align="center")

Then move into the directory where the **Security Key** is downloaded and make an SSH request with the command: `$ ssh -i directoryName/VProfile-Prod-Key.pem centos@publicIPOfInstance`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685599606397/53b8c197-1d49-4ee1-bf5a-e519d8b74060.png align="center")

### Checking the process completion: `ef -ps`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685600054122/fdd9cf51-9ca3-4314-be05-e838c3d870b5.png align="center")

DB Setup has been completed.

### Checking DB Status: `systemctl status mariadb`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685599922929/dcfc1229-f2ff-43ca-b5b0-393475b749c1.png align="center")

### Logging into the DB: `mysql -u root -p`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685600489279/ddec06ff-cbea-4d38-b4ff-15f0fb4660d3.png align="center")

Finally, we have completed our MySQL provision.

## Step-3.2(Lunching Instance for Memcache)

### **Instance OS:** `CentOS 7`

For Memcached we require CentOS 7 hence, we are selecting the same from AWS Market Place.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685597229177/3d64642d-6b50-41fe-8c11-f0dbc7b01d27.png align="center")

### **Instance Type: t2.micro**

Our instance type should be t2.micro which provides the below configuration and it's a free tire.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685597286566/47a87dd1-5936-4048-95f6-cbe44c7ce2eb.png?auto=compress,format&format=webp align="left")

### **Instance Name and tags:** `VProfile-Memcache01, Project: VProfile`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685600849100/47bf76ad-e969-4646-b965-8fe9cc00d356.png align="center")

### **Key Pair:** `VProfile-Prod-Key`

Every time we lunch any instance we need a Key Pair and for our instances, we have already created a Key-Pair which is **VProfile-Prod-Key** hence, we are just using that instead of creating a new pair.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685597337347/bce5a45e-f0cc-4f8a-83b3-3fb76ace9546.png?auto=compress,format&format=webp align="left")

### **Security Group:** `VProfile-BackendServices-SG`

Whenever any instance is launched, it requires a **Firewall(Security Group)** to be attached to it to control the traffic. We have already prepared the **Security Group** configuration in **Step-1.2** for Backend Services. As **Memcache** is one of the Backend Services it will be attached to the respective Security Group.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685597523954/224de013-9683-415a-b74c-d31fd8733bf0.png?auto=compress,format&format=webp align="left")

### **Advanced Details-&gt; User Data:**

In this section, we have defined the shell script for Service installation and configuration. And the script is also written below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685601237228/3ca4b228-e06b-4f4f-90c1-10cc18d4ca4f.png align="center")

```bash
#!/bin/bash
sudo yum install epel-release -y
sudo yum install memcached -y
sudo systemctl start memcached
sudo systemctl enable memcached
sudo systemctl status memcached
sudo memcached -p 11211 -U 11111 -u memcached -d
```

### **Lunch Instance:**

Finally, the instance for MySQL DB is up and running. It will take some time to configure and be stable as it will be executing the above shell script for setting up the entire Memcached service.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685601579614/aa0955b8-babf-4e76-9f80-8b792ce44069.png align="center")

### SSH to Memcache

The same steps will be followed to connect through SSH.

`$ ssh -i directoryName/VProfile-Prod-Key.pem centos@publicIPOfInstance`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685610757673/75b17105-03af-49a0-87dd-0d7c592013cf.png align="center")

Memcached has been successfully set up.

## Step-3.3(Lunching Instance for RabbitMQ)

### **Instance OS:** `CentOS 7`

For RabbitMQ we require CentOS 7 hence, we are selecting the same from AWS Market Place.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685597229177/3d64642d-6b50-41fe-8c11-f0dbc7b01d27.png align="center")

### **Instance Type:** `t2.micro`

Our instance type should be t2.micro which provides the below configuration and it's a free tire.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685597286566/47a87dd1-5936-4048-95f6-cbe44c7ce2eb.png?auto=compress,format&format=webp align="left")

### **Instance Name and tags:** `VProfile-RabbitMQ01, Project: VProfile`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685601971812/a290c3aa-1bc3-4003-b0d6-c0961606e1ce.png align="center")

### **Key Pair:** `VProfile-Prod-Key`

Every time we lunch any instance we need a Key Pair and for our instances, we have already created a Key-Pair which is **VProfile-Prod-Key** hence, we are just using that instead of creating a new pair.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685597337347/bce5a45e-f0cc-4f8a-83b3-3fb76ace9546.png?auto=compress,format&format=webp align="left")

### **Security Group:** `VProfile-BackendServices-SG`

Whenever any instance is launched, it requires a **Firewall(Security Group)** to be attached to it to control the traffic. We have already prepared the **Security Group** configuration in **Step-1.2** for Backend Services. As **RabbitMQ** is one of the Backend Services it will be attached to the respective Security Group.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685597523954/224de013-9683-415a-b74c-d31fd8733bf0.png?auto=compress,format&format=webp align="left")

### **Advanced Details-&gt; User Data:**

In this section, we have defined the shell script for Service installation and configuration. And the script is also written below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685602258828/38f2d804-f7ba-464f-a4b0-99645ce06705.png align="center")

```bash
#!/bin/bash
sudo yum install epel-release -y
sudo yum update -y
sudo yum install wget -y
cd /tmp/
wget http://packages.erlang-solutions.com/erlang-solutions-2.0-1.noarch.rpm
sudo rpm -Uvh erlang-solutions-2.0-1.noarch.rpm
sudo yum -y install erlang socat
curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.rpm.sh | sudo bash
sudo yum install rabbitmq-server -y
sudo systemctl start rabbitmq-server
sudo systemctl enable rabbitmq-server
sudo systemctl status rabbitmq-server
sudo sh -c 'echo "[{rabbit, [{loopback_users, []}]}]." > /etc/rabbitmq/rabbitmq.config'
sudo rabbitmqctl add_user test test
sudo rabbitmqctl set_user_tags test administrator
sudo systemctl restart rabbitmq-server
```

### Lunch RabbitMQ

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685611124350/b46c8ade-bed9-48c2-b227-2fabe6cd716f.png align="center")

### SSH to RabbitMQ

The same steps will be followed to connect through SSH.

`$ ssh -i directoryName/VProfile-Prod-Key.pem centos@publicIPOfInstance`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685612949586/8e387732-b3c7-4180-995b-f3906daec58e.png align="center")

## Step-3.4(Lunching Instance for Tomcat)

### **Instance OS:** `Ubuntu 20`

For RabbitMQ we'll use Ubuntu hence, we are selecting the same from AWS Market Place.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685613424580/ed61e1bc-2aaf-4945-868d-d3bcbe431762.png align="center")

### **Instance Type:** `t2.micro`

Our instance type should be t2.micro which provides the below configuration and it's a free tire.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685597286566/47a87dd1-5936-4048-95f6-cbe44c7ce2eb.png?auto=compress,format&format=webp align="left")

### **Instance Name and tags:** `VProfile-RabbitMQ01, Project: VProfile`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685613576221/7792a5dc-1143-40c8-a66d-6841c70c08a7.png align="center")

### **Key Pair:** `VProfile-Prod-Key`

Every time we lunch any instance we need a Key Pair and for our instances, we have already created a Key-Pair which is **VProfile-Prod-Key** hence, we are just using that instead of creating a new pair.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685597337347/bce5a45e-f0cc-4f8a-83b3-3fb76ace9546.png?auto=compress,format&format=webp align="left")

### **Security Group:** `VProfile-app-SG`

Whenever any instance is launched, it requires a **Firewall(Security Group)** to be attached to it to control the traffic. We have already prepared the **Security Group** configuration in **Step-1.1** for Tomcat Service. As **Tomcat** is a service where our app will be hosted hence it has a separate configuration of Security Group.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685613995589/41dcbf0c-e147-446b-91b5-01d628066c3d.png align="center")

### **Advanced Details-&gt; User Data:**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685614110788/1999517b-20e5-4fc2-8058-214f23b24d8b.png align="center")

```bash
#!/bin/bash
sudo apt update
sudo apt upgrade -y
sudo apt install openjdk-8-jdk -y
sudo apt install tomcat9 tomcat9-admin tomcat9-docs tomcat9-common git -y
```

We have taken Ubuntu here just because the **Tomcat** installation is much simpler in Ubuntu than in CentOS.

### Lunch Tomcat

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685614477012/b3d0717b-8255-4d20-a7d1-b1a76b538bb1.png align="center")

### Accessing tomcat

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685619934747/882ea9b0-8814-43a9-bba1-5fb06bd3146f.png align="center")

# Step-4(Updating Route53 Private DNS Zone)

This will be the hosted zone for **VProfile backend services.** A private hosted zone **only responds to queries coming from within the associated VPC and it is not used for hosting a website that needs to be publicly accessed**. So, this will enable our backend services such as MySQL, RabbitMQ, and Memcached to interact with each other and the Tomcat server through a given domain name instead of its IP address directly.

**Move to Route53 Dashboard:**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685616634376/ff631dcd-a412-4007-a43f-b755bde23d9b.png align="center")

## Step-4.1(Creating a hosted zone)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685617263051/a0d0e658-cc9c-48a2-beb2-ca9c7904d127.png align="left")

Here we are creating a **Private** hosted zone for our backend services. On the above image you can see, the **domain name** for the hosted zone is **Vprofile.in.**

As the type is chosen as **Private Hosted Zone,** this will route traffic within a defined VPC. It is not accessible on Open Internet

The region is N. Virginia and the **VPC** is a default VPC.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685618032411/9dcd7118-7a5c-45af-83d0-6a7981cb7753.png align="center")

## Step-4.2(Creating Records)

In the hosted zone **vprofile.in** we'll create records of our **backend services**.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685618185215/ead8c6c4-0c9a-41e8-93bb-4639a052d142.png align="center")

We'll be creating **3 records** for **MySQL(DB01), Memcached(mc01), and RabbitMQ(rmq01)** respectively:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685619126841/80a08d72-e11e-4668-a9a9-35794a172e9d.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685619157311/26b69c52-4b2f-4012-b5c4-c506552bec5c.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685619177256/72d3e8a5-f71d-48a4-9303-8212358b2c5a.png align="center")

In all the above 3 records are **simple A** records. we have defined the name, record type, and value as the **Private IP** address of the respective server.

Finally, you can see all highlighted private IP address values are assigned against a record name **e.g** MySQL server is assigned to a name [db01.vprofile.in](http://db01.vprofile.in)

Now, we can use [db01.vprofile.in](http://db01.vprofile.in) instead of directly using the IP address as 172.31.22.44

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685619511039/272ce023-a885-4d57-b748-b5c84b198477.png align="center")

# Step-5(Building Artifact from Source Code)

Here we'll build the project that will create the artifacts(a **deployable** `.war` file) and then we'll deploy the application as a `.war` file to the Tomcat server.

**To do so:**

* We'll first build the project in our **Local System** and create the artifacts(.war file)
    
* Then we'll upload the artifact to **S3 Bucket**
    
* From the **S3 Bucket**, Tomcat will fetch the artifact and it will be deployed in the Tomcat server.
    

## Step-5.1(Git clone the project)

Clone the project to the local system :

`git clone` [`https://github.com/rkn1999/vprofile-project.git`](https://github.com/rkn1999/vprofile-project.git)

Checkout to the branch: `git checkout aws-LiftAndShift`

move to the project directory `vprofile-project/src/main/resources` where the [`application.`](http://application.properties)`properties` file is present

## Step-5.2(Configuring [`application.`](http://application.properties)`properties` )

```bash
#JDBC Configutation for Database Connection
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://db01.vprofile.in:3306/accounts?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull
jdbc.username=admin
jdbc.password=admin123

#Memcached Configuration For Active and StandBy Host
#For Active Host
memcached.active.host=mc01.vprofile.in
memcached.active.port=11211
#For StandBy Host
memcached.standBy.host=127.0.0.2
memcached.standBy.port=11211

#RabbitMq Configuration
rabbitmq.address=rmq01.vprofile.in
rabbitmq.port=5672
rabbitmq.username=test
rabbitmq.password=test

#Elasticesearch Configuration
elasticsearch.host =192.168.1.85
elasticsearch.port =9300
elasticsearch.cluster=vprofile
elasticsearch.node=vprofilenode
```

In the above configuration, you can see that we have used the **Private DNS from route53** for MySQL, RabbitMQ, and Memcached services instead of using direct IP addresses.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685679397332/157534c3-9713-4acb-8415-b9d9c179b8ed.png align="center")

## Step-5.3(Building Artifact in Local System)

To build the project and generate the artifact we'll use the **Maven** build tool.

So, now move back to the project root directory where the **pom.xml** file is present, This pom.xml file will help us to build and generate the artifact.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685679769441/04f9cfb0-1697-4402-a464-3576fdcaea35.png align="center")

Now, start the build simply use: `mvn install` and it will download the dependency, compile the Java code and generate the artifact in our local system as shown below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685679970507/e0a8a8dc-e0c8-4be9-bd31-ef0704cd1c41.png align="center")

Now, the build has been successful:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685680016979/bb237495-dcbe-48d3-9529-32f28b74c2e4.png align="center")

Now, a **target** directory has been created additionally after the build is successful;

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685680078374/2a238548-89ef-4344-b26d-8bd482d2b703.png align="center")

Inside the target directory, we have our `vprofile-v2.war` file which we'll upload to the **S3 Bucket** currently it's present in our local system:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685680132716/649c3c37-93ce-491c-bb10-a4ecc0a24b79.png align="center")

# Step-6(Uploading Artifact to S3)

As we have generated the `.war` file in the Build section, now we'll upload this artifact `vprofile-v2.war` to a **S3 Bucket.** To do so:

* We have to connect to the AWS console via an **IAM** user through AWS CLI
    
* Configure AWS CLI and create an S3 bucket through AWS CLI
    
* The IAM user should have the permission of [AmazonS3FullAccess](https://us-east-1.console.aws.amazon.com/iamv2/home?region=us-east-1#/policies/details/arn%3Aaws%3Aiam%3A%3Aaws%3Apolicy%2FAmazonS3FullAccess) to access the S3 bucket
    

Create an access key for the particular user to access AWS CLI

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685681516693/7404e2d2-f2db-4cf3-9211-11cb6b584fd9.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685681656884/5ef28947-b341-4582-ab9f-674202527e41.png align="center")

## Step-6.1(Configuring AWS CLI)

[AWS S3 Commands](https://docs.aws.amazon.com/cli/latest/userguide/cli-services-s3-commands.html)

Use the command: `aws configure`

It will further prompt for the access key, secret access key, region and output format as below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685682053238/75a1ce41-1e1a-4305-932c-a460d34ff2c4.png align="center")

**AWS CLI** configuration is done.

## Step-6.2(Creating S3 Bucket through AWS CLI)

Once the configuration is done with the AWS CLI through the IAM access key and secret key, we need to create an **S3 Bucket** where our **Artifact** will be uploaded. To do so below is the syntax:

`aws s3 mb s3://bucket-name`

**NB: the bucket name should be unique around the world.**

Command: `aws s3 mb s3://vprofile-artifact-repo-ritesh100`

Finally, the bucket has been created with the name: `vprofile-artifact-repo-ritesh100`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685693715832/5c738ae0-eac3-42a9-80b3-621581700fcd.png align="center")

## Step-6.3(Uploading Artifacts to S3 Bucket)

Now, we'll simply upload the `vprofile-v2.war` to our bucket `vprofile-artifact-repo-ritesh100` .

The Syntax: `aws s3 cp filename.txt s3://bucket-name`

**Copying:**

`aws s3 cp vprofile-v2.war s3://vprofile-artifact-repo-ritesh100/vprofile-v2.war`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685694959052/9c61deec-b215-4f36-aa3f-2af080252d5e.png align="center")

`aws s3 ls s3://vprofile-artifact-repo-ritesh100` : verifying from the CLI

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685695137163/e2b81219-bae2-4453-8412-5d77d5e7c83b.png align="center")

Now the upload from CLI has been completed. Let's verify from S3 console:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685695016219/38c918f3-4a64-429d-931c-90eb24fc0b8a.png align="center")

Hence, we conclude the artifact upload. !!

# Step-7(Downloading Artifact to Tomcat Instance from S3)

Our next goal is to download this artifact to the Tomcat EC2 instance. As we have uploaded the artifact to the S3 bucket now, we'll download the artifact to the **EC2 Instance** where the **Tomcat** server is up and running. To do so, we have to follow below steps:

* Create a new role that has S3 access and attach this role to the **Tomcat EC2 instance**
    
* Open the SSH connection port for the Tomcat instance
    

## Step-7.1(Creating Role S3 access role for Tomcat Instance)

Here firstly, we'll create a **Role** with the permission of **S3 AllAccess** that will enable our **Tomcat Server** to access the S3 bucket and download the artifact.

To do so: **IAM --&gt; Roles --&gt; Create Role**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685717423169/07dc8ab9-02e4-45d7-9de0-855c1c845da2.png align="center")

Keep the configuration as below as we are creating this role for the **EC2 instance** --&gt; click Next

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685717480098/12069696-940f-4891-b7dd-0c77cbfc8212.png align="center")

In the permission section choose **AmazonS3FullAccess** as shown below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685717605347/116150d9-6b5e-4d04-8b5c-a37743783d37.png align="center")

And create the role accordingly.

## Step-7.2(Attaching S3 access Role to Tomcat Instance)

Now, we'll be attaching the role created for S3 access to our Tomcat Server.

To do so:

Go to the particular EC2 instance and in **Actions** change go to **Security** and **Modify IAM Role**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685782798538/4f227b97-ce98-442d-b358-532dd4149cb2.png align="center")

Inside Modify IAM role attach the role named **vprofile-artifact-s3-role** and update the IAM role.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685782901575/86888525-c217-49bd-a8e5-0cb95bb6a9dd.png align="center")

Finally, the Role for accessing **S3 Bucket** has been attached and now our Tomcat server will be able to connect to S3 and access the Artifact.

## Step-7.3(Connecting to S3 from Tomcat Instance)

Now we'll connect to the S3 bucket from our Tomcat instance using AWS CLI.

**NB: EC2 instances do not have AWS CLI pre-installed. We have to install and configure it.**

`sudo apt update && sudo apt install awscli -y`

let's verify if we can see the .war file in the Tomcat server using AWS CLI

`aws s3 ls s3://vprofile-artifact-repo-ritesh100`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685784208780/b61b3fda-6567-4727-ba95-cd8370a8373b.png align="center")

## Step-7.4(Copying Artifact from S3 to Tomcat Server)

Now as we are able to see the war file we can copy it to the Tomcat instance by:

`aws s3 cp s3://vprofile-artifact-repo-ritesh100/vprofile-v2.war /tmp/vprofile-v2.war`

Using the above command we have copied the `vprofile-v2.war` artifact from `s3://vprofile-artifact-repo-ritesh100` bucket to `/tmp/` directory of the Tomcat Instance with the same name.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685784633181/01e5aec4-afb7-4450-8548-b4ddc11e1851.png align="center")

**Finally, we have got the artifact in our Tomcat Instance that we created from our source code in our local system via using S3 Bucket.**

# Step-8(Application Deployment to Tomcat Service)

As we know the **Artifact** `vprofile-v2.war` is currently present in the /**tmp** directory. We have to deploy this in Tomcat. Before deploying we have to shut down the Tomcat service by the command :`systemctl stop tomcat9`

Now, we have to move to its **web apps** directory where Tomcat's default dummy application is present inside `the` **ROOT** directory:

`cd /var/lib/tomcat9/webapps/`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685786163661/3e0d0047-9abb-4733-b31d-2f5b1724b7c5.png align="center")

And now we have to replace this **ROOT** directory with our `vprofile-v2.war` simply by copying it to the **/webapps** folder. To do so:

we have to remove the ROOT first: `rm -rf ROOT`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685786426145/be39d2f1-f58c-4ff5-99dc-b5b0a794607d.png align="center")

Now finally, we'll copy our artifact over here using the below command:

`cp /tmp/vprofile-v2.war ./ROOT.war`

Here we have copied our `vprofile-v2.war` from /tmp folder to /webapps folder as `ROOT.war` and `.` refers to the current directory(/webapps)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685786723612/ce93e651-8a90-40e4-af8e-c282b778f6bc.png align="center")

Now, let's restart the **Tomcat Service**:

`systemctl start tomcat9` : with this now another ROOT folder has been created where `vprofile-v2.war` is extracted and stored as a deployable Application.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685787033339/68eaa348-b4ff-4b82-930b-e3a641e6cfe0.png align="center")

Now, let's **verify** if our Application Server(Tomcat) is able to connect to the **Backend services(DB, Memcached, RabbitMQ).** We can do that by doing **telnet** to the backend server as below:

`telnet` [`db01.vprofile.in`](http://db01.vprofile.in) `3306` : telneting to DB

`telnet` [`mc01.vprofile.in`](http://mc01.vprofile.in) `11211` : telneting to Memcached

`telnet` [`rmq01.vprofile.in`](http://rmq01.vprofile.in) `5672` : telnetting to RabbitMQ

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685788270697/b8843953-bddc-4476-b129-a2a2f0af92b9.png align="left")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685788385785/6a134505-e1ad-4171-8b67-5adf1b22db4e.png align="left")

# Step-9(Setting up Load Balancer)

As we have completed the application deployment to the Tomcat server, our next goal is to make it accessible to the Users on the internet. To do so, we need a mediator between our tomcat Server and the public internet. Previously we have used **Nginx** for this purpose however, now we'll be using **AWS ELB(Elastic Load Balancer).**

**ELB (Elastic Load Balancer)** and **Nginx** are both popular solutions for load balancing in web application architectures.

## Step-9.1(Creating a Target Group)

Before creating an Application Load Balancer, we have to create a target group.

In **Elastic Load Balancer (ELB)**, a ***target group*** is a logical grouping of instances or services that the load balancer directs traffic to. **It acts as a routing mechanism** within the load balancer, determining how incoming requests are distributed among the targets.

Now, under EC2 move to Load Balancing, and under Load Balancing there is Target Group. Click on that and create a Target Group.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685863177083/88c0a025-e68f-4db3-bfbc-044a4fb8aa14.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685863469170/1e687545-dbd4-49be-bb52-ecb75b79678a.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685863923493/47bda833-0807-4bc1-91c5-6626300128db.png align="center")

So, our Target Group name will be `VProfile-App-TG` , the target group is created for **Ec2** **Instance(Tomcat Server)** so the target type is `Instance` , the Tomcat service runs on port 8080 so the port is defined as `8080` , VPC remains the default, and the path refers to the URL path that is used for health checks against the registered targets so, here our path for the app is `/login` .

Now our Target Group is created:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685864021155/1283d54c-0ab5-445b-ade1-4cfdcbce13bd.png align="center")

## Step-9.2(Registering the Target Group with Tomcat Instance)

After creating the Target Group, we'll have to register it with our **targeted EC2** instance which is our **Tomcat Instance**. To do so, select the target group and in actions click on Register and deregister instance/ IP targets:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685864250870/1018ddb9-81f7-419d-a4c0-4fd1c9a59bc2.png align="center")

And then choose the respective Instance **VProfile-Tomcat01** click on **Add to registered on port 8080** and then save as below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685864346482/7a0ad669-d68c-4963-9870-8cd1cbbeeeca.png align="center")

Finally, our **Target Group** has been created and attached to Tomcat Server. Currently, it demands a Load Balancer to be fully functional:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685865038301/97df1f8f-2f19-49e2-aa0f-cf03d19fe9f3.png align="center")

## Step-9.3(Creating Load balancer)

here we'll be creating the Load Balancer and this load balancer will further be attached to the Target Group named `VProfile-App-TG` .

To create the Load Balancer: go to Load Balancer --&gt; click on Create Load Balancer and then choose the first one that says **Application Load Balancer**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685866043144/cce58207-cadf-4ad9-b4eb-10bcc0aa06ac.png align="center")

**And make the configuration as below:**

The name of the ELB is `VProfile-prod-ELB` .

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685866251745/7d3e5629-0b9d-4320-b622-d57ac1ca1dbe.png align="center")

VPC is set to Default and Mappings can be **2 or more than 2 Availability Zones**. Here we have selected all the AZs that will make our application highly available:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685866397736/40e59b85-205d-4967-a223-6a733be98fe2.png align="center")

**Security Group** is set to the security group `VProfile-ELB-SG` that we created especially for Load Balancers at the very beginning as a basic configuration:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685866468637/05602644-772d-4595-b4f0-08bfb866b95e.png align="center")

**Listeners** contain the ports to which our ELB will respond. Here we have opened the ports HTTP and HTTPS. It's recommended to only open HTTPS:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685866720675/32f680fc-b91f-443a-ba29-b475ad510a76.png align="center")

And in the Secure Listeners setting we have to attach the SSL/TSL certificate for our website. I have already purchased a domain from GoDaddy and validated it with **ACM**(Amazon Certificate Manager). Am attaching the Load Balancer to that SSL domain `devopswithritesh.in`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685866912124/7b8990ce-916d-4c41-97c8-886309ce69be.png align="center")

Now simply click on Create Load Balancer.

And our Load Balancer is now active:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685867215852/266ae59a-6201-4ab9-9bab-322d50590b37.png align="center")

## Step-9.4(Registring ELB DNS with the Domain provider GoDaddy)

As we have created the Load Balancer now, we have to create a CNAME record with our domain name DNS in our Domain provider portal(GoDaddy).

To do so:

We have to take the DNS of our Load Balancer `VProfile-prod-ELB` from the description and create a CNAME record in `devopswithritesh.in` .

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685867852517/5eaa57c7-82cc-42b5-b257-2a879ae1de06.png align="center")

Finally, we have created a CNAME and the website will be referred as:

`https://vprofileapp.devopswithritesh.in`

When the users hit the above URL, it will point to our Load Balancer `VProfile-prod-ELB` and the load balancer will further redirect the traffic to the Tomcat Server where our application is situated.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685868270260/ffda62b0-c4cd-4475-b058-01065c4828d2.png align="center")

Finally, our app is accessible on the open internet with the above-said URL.

# Auto-Scaling

Autoscaling is a technique used in cloud computing to automatically adjust the capacity of a system or application based on the current demand. So, to make our application highly scalable we have to create Autoscaling Group and attach it to our App server.

To do so we have to go to the auto-scaling group section:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685870776655/54be509c-40fd-4fb4-adb0-f457926a8ecc.png align="center")

## Pre-requisite for Autoscaling Group

### AMI(Amazon Machine Image) Creation

To create an autoscaling group we need an **Image** of the **Tomcat Instance**. This image will further be used while creating the autoscaling group. And the auto scaling group will fire up instances with the same configuration as the existing Tomcat Instance when traffic increases.

To do so: select the tomcat instance and follow the below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685871164633/58c873ab-c3dc-4192-b050-335f915034bb.png align="center")

And with the below configuration create the image.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685871282842/cf770965-886a-4493-95cb-732eec241195.png align="center")

Now the AMI(Amazon Machine Image) is being created:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685871433647/533cf31d-f215-4c1e-99cc-718896353d76.png align="center")

### Creating lunch configuration(`Vprofile-app-LunchConfiguration`)

A launch configuration specifies various parameters for instances, including the Amazon Machine Image (AMI) to use, instance type, storage options, etc. Launch configurations are often used in conjunction with autoscaling groups, which manage the scaling behavior and policies. Autoscaling groups rely on launch configurations to create and configure instances automatically based on the defined rules and triggers.

While creating the Lunch Configuration make sure all the configuration should be the same as the server whose AMI is being used. Our configuration is as below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685871913953/4fc915dc-1e7c-4861-b277-e51bda45c69e.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685871972115/c61de233-6c89-433f-8eb2-3b123b540f75.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685871996069/fa60708a-c7cf-402a-9045-f1fb9d3b6249.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685872012676/4e09786e-cc83-4f34-89de-4d58aee35ffa.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685872046678/c79d2f6f-2086-4525-907b-a4e82ae1ef96.png align="center")

Lunch configuration has been created:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685872239060/cc9859b7-4dd4-4058-9cca-b3d877c92d32.png align="center")

## Creating Autoscaling Group(`VProfile-app-ASG` )

The autoscaling group configuration should be as below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685872648771/7ff8a9be-d041-4243-99fb-be8c95e5c50f.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685872662420/cbfaa079-99e3-4a0e-9e66-a75d537df264.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685872680732/e6626d88-624e-43e4-8d6a-e5b70695eb82.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685872697875/a67bbc0e-8e60-47b1-a969-4865defc60dc.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685872721231/c12b2ed0-de9f-47b9-9cc0-4fedd2ce8ef7.png align="center")

Now finally, our autoscaling group has been created:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685872792027/b38e0885-3f2d-49c9-9346-2aba65cb7c29.png align="center")

# Summary

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685874470267/fd01ee97-824a-4c2d-9d9f-6f1894f2b9b5.png align="center")

Finally, we are done with a scalable deployment of a Java-based social media application. Here we have used Lift & Shift strategy.

* The user will use the URL [`https://vprofileapp.devopswithritesh.in/login`](https://vprofileapp.devopswithritesh.in/login)
    
* This URL is hosted on my personal domain from GoDaddy.
    
* The user request from the URL will be redirected to the ELB(Elastic Load Balancer)
    
* The ELB further redirects it to the **Tomcat Server** where our application is hosted as a `.war` **file.** The Tomcat service further interacts with several backend services such as **MySQL, RabbitMQ, and Memcached.**
    
* The Tomcat Service fetched this `.war` file from the **S3 bucket.**
    
* The `.war` file was built from the Source code using the **Maven** build tool in our local system and was uploaded to an **S3 Bucket**
    
* The source code was cloned to our local system from the repository: [https://github.com/rkn1999/vprofile-project.git](https://github.com/rkn1999/vprofile-project.git)