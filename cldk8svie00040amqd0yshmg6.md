---
title: "Kubernetes(K8S) Complete Setup with Project-1"
datePublished: Tue Jan 31 2023 12:55:24 GMT+0000 (Coordinated Universal Time)
cuid: cldk8svie00040amqd0yshmg6
slug: kubernetesk8s-complete-setup-with-project-1
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1674973687325/da31a493-e096-4e3b-b45b-bc9698d4e4fc.png
tags: docker, kubernetes, k8s, bestk8sarticle, minikube-setup

---

## What is Kubernetes?

Kubernetes is nothing but an open-source platform for automating the deployment, scaling, and management of **containerized applications**. It is a multi-environment and multi-container deployment tool.

It is also known as **the Container Orchestration tool** developed by Google.

It is best suited for applications based on microservice architecture or loosely coupled applications.

It helps to reduce the failure and downtime of the application through its auto scaling and auto healing functionality.

# Docker-Swarm vs Kubernetes

Kubernetes and Docker Swarm are both open-source orchestration platforms for managing containers. However, they have some differences:

1. Architecture: Kubernetes has a more complex architecture with multiple components and is better suited for large-scale, production deployments. Docker Swarm has a simpler architecture, making it easier to set up and use, but with limited features compared to Kubernetes.
    
2. Scalability: Kubernetes can handle larger and more complex deployments, with automatic **scaling and self-healing** features. Docker Swarm is limited in its scalability and does not have as many features as Kubernetes.
    
3. Community: Kubernetes has a larger and more active community than Docker Swarm.
    
4. Flexibility: Kubernetes supports a wider range of deployment options and container configurations, while Docker Swarm focuses more on simplicity and ease of use.
    
5. Resource Management: Kubernetes has more advanced resource management and auto-scaling features.
    
6. Networking: Kubernetes has a more robust networking model, with built-in support for service discovery, load balancing, and network segmentation.
    

In summary, Kubernetes is more suited for complex, large-scale production deployments, while Docker Swarm is a good choice for smaller, simpler deployments.

## Why are Microservice architecture and Kubernetes connected?

In a microservice architecture, a single application consists of multiple components and every component of the application is independent of the other.

And when it comes to the containerization of this microservice architectured application, each component of the application gets containerized individually in different docker containers so that, the number of containers increases and it is extremely important and necessary to keep the containers up and running to make the entire application functional.

This is where the concept of **container orchestration** kicks in. Microservices are typically deployed as containers, and Kubernetes is designed for managing and orchestrating containers.

1. **Scalability**: Kubernetes provides features for scaling microservices, such as **horizontal pod scaling**, **automatic scaling based on resource utilization**, and **self-healing** capabilities.
    
2. **Load balancing**: Kubernetes can automatically distribute **incoming requests** to microservices, ensuring high availability and reliability.
    
3. **Deployment**: Kubernetes allows for easy deployment and management of microservices, with features such as rolling updates, rollbacks, and automatic rollouts.
    
4. **Observability**: Kubernetes provides tools for monitoring and logging microservices, making it easier to troubleshoot and diagnose issues.
    

Nowadays 90% of the applications work on microservice architecture. Whether it's social media apps like Facebook, Instagram or payments apps like phonePe, Paytm and GPay or video streaming platforms like youtube, Netflix and Hotstar etc work on microservice architecture. And they use Kubernetes extensively in their deployments and productions due to its production-friendly functionalities.

# Kubernetes Architecture

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1674977778694/851c2630-16f8-496c-93a0-5ada5836b85d.png align="center")

As you can see in the above picture, Kubernetes works on the Master and Worker node architecture. Nodes are nothing but **EC2 instances or servers.**

However, on a high level, there are several components consisting of which the entire architecture becomes functional. We will discuss all the components in detail below:

1. **Master Node:** This is the node whose sole responsibility is to manage, instruct and operate the worker node. Unlike docker swarm, it doesn't run any container by itself. This node contains all the components required to manage the workers.
    
2. **Worker Node**: worker node is the node where the application runs and it contains several Pods.
    
3. **API Server**: It allows the user(we) from the control panel(user interacts through the control panel/CLI using Kubectl) to connect or interact with Worker Nodes. It is the central component of Kubernetes, responsible for serving the Kubernetes API and processing REST operations. The API server doesn't work alone, it has other components like ETCD, Controller, and Scheduler having its back.
    
4. **Kubectl:** Kubectl is a command line tool that enables us to interact with the **API Server** using commands and the API will take the message forward to nodes. Kubectl presents in **Master Node** only.
    
5. **ETCD**: A distributed key-value store used to persist the configuration data of the cluster.
    
6. **Controller Manager:** A component responsible for tracking the desired state of the cluster. such as replicating pods, and handling failures. It keeps a track of what's happening in the cluster(A combination of master and worker node is called a cluster), gives updates to API servers and then the API server takes actions according.
    
7. **Scheduler:** As the name suggests, it helps to schedule a particular job at a particular time. It is responsible for scheduling pods onto nodes based on available resources and constraints. If at a business peak time, we need more containers than usual then the scheduler helps us here.
    
8. **Kubelet**: An agent that runs on each node and communicates with the API server to ensure that containers are running as expected. It acts as a receiver for Kubctl because "**Kubectl** gives the instructions to **API Server** and **API Server** acts as a **mediator** delivers the information and **Kubelet** interprets the information and implements in **Worker** Nodes".
    
9. **Container runtime**: A low-level component responsible for starting and managing containers, such as Docker or CRI-O.
    
10. **Pods**: The smallest and simplest unit in the Kubernetes object model, representing a single instance of a running process.
    
11. **Services**: A logical grouping of pods, providing a stable IP address and DNS name for access to the pods.
    
12. **Namespaces**: A mechanism for partitioning resources and access control within a cluster.
    

These components work together to provide a self-healing, highly available, and scalable platform for deploying and managing containerized applications.

**NB: All the above components themselves work inside individual containers which we will see in a hands-on illustration. For now, refer the below image:**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675154205641/2de9fcad-230e-45d2-bfe4-3bcd74cac5b7.png align="left")

### Node vs Pod

1-Pods are nothing but containers in the context of Kubernetes whereas nodes are EC2 instances or host machines. A single pod can have multiple containers when the containerized app has a closely coupled dependency. But it's recommended to run the single container in a single pod.

2-A Node in Kubernetes represents a worker machine in the cluster whereas, Pod represents a single instance of a running process in the cluster. Nodes are the worker machines in a cluster that host and run containers, while Pods are the smallest and simplest unit in the Kubernetes object model, representing a single instance of a running process.

3-A node can host multiple pods, but a pod is limited to running on a single node.

4-If a node fails, the pods running on that node are automatically rescheduled to other healthy nodes. Pods, on the other hand, can be designed to self-heal or be rescheduled in case of failures.

5-Pods have unique IP addresses and can communicate with each other within the cluster, while nodes have their IP address and host the network components of the cluster.

**Note: You can create 256 pods in a single node in EKS from AWS and GKE allows up to 110 pods in a single node. We can not exceed that limit for any node.**

# Setup and Hands-on

### Minimum Requirement for setup

\-&gt; **Minikube**: Minikube is a tool that makes it easy to run a single-node Kubernetes cluster on your local development machine. Minikube allows you to test and experiment with Kubernetes without the need for a full-fledged cluster, making it a popular choice for developers who want to **learn Kubernetes** or for testing purposes.

\-&gt; **Server**: An EC2 instance or your local system having 2 CPUs. You can go for t2 Medium which has 2 vCPU.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675141929048/a285f5ce-cc0b-4138-9a27-877bf6be6899.png align="left")

**NB:** In industry\*\*,\*\* they don't use this minikube inside EC2 or they don't spin multiple EC2 instances individually instead, they use EKS(Elastic Kubernetes Service) from AWS or GKE(Google Kubernetes Service) from GCP.

## Why Minikube?

As we know, to demonstrate Docker swarm we used multiple EC2 instances, installed all the dependencies individually in each server and established a connection between the master and worker nodes explicitly.

Here **Minikube** helps us eliminate this tedious task, it performs a similar operation and simulates the functionalities of the EKS cluster in our local system. Because of Minikube only we can avoid lunching multiple EC2 instances, installing Kubernetes and all its required components in each instance one by one.

### Step-1(Docker installation)

Install docker using the command `$ sudo apt install` [`docker.io`](http://docker.io) because for containerization we need docker to be installed.

### Step-2(minikube installation)

The below command is needed to install minikube.

`curl -LO` [`https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64`](https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64)

`sudo install minikube-linux-amd64 /usr/local/bin/minikube`

For more info please visit [https://minikube.sigs.k8s.io/docs/start/](https://minikube.sigs.k8s.io/docs/start/)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675150537828/9ca5d138-271b-45d5-ad30-c050f160d691.png align="center")

### Step-3(Starting the Minikube)

Once minikube is installed we need to make it up and running for that `$ minikube start` will help however, while installing for the first time we might encounter an error as below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675150931252/2ed4b349-9282-4631-afd2-87fca76d164c.png align="left")

What the above error defines is, while starting minikube it checked for the components that are needed and was unable to find them.

### Step-3.1(Resolving the error)

`$ minikube start --driver=docker` : the command will be able to explicitly instruct the server to use docker as its base driver and perform operations on it.

NB: make sure you have created the group **docker** and added the **USER** to it using `sudo usermod -aG docker $USER && newgrp docker` to avoid **the permission denied** error as before.

### Step-3.2(Starting the Kubernetes cluster through minikube)

Now, `$ minikube start --driver=docker` will start the minikube and install all the components of Kubernetes locally as shown below image:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675152250115/78ae8363-e51f-45ef-9377-c80c97ee7449.png align="left")

Now as you can see above, minikube has set up the Kubernetes cluster. There are **two CPUs** in which one is used for **the Master node** and all the required components such as the controller, scheduler, etcd etc are installed in the Master node.

### Step-3.3(Verifying if Cluster is up and running)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675153127253/32487df9-8831-4ae4-b149-ffd76445e44a.png align="center")

Above we can see that a minikube docker-container named gcr.io/k8s-minikube/kicbase:v0.0.37 is running.

### Step-4(Anatomy of Minikube)

As we know Kubernetes is running inside minikube hence, to verify all the Kubernetes components we can use `$ minikube ssh`

`$ minikube ssh` will enable us to interact with the Shell of Minikube'

### Step-4.1(Viewing the internal architecture of local Kubernetes)

Now as we are inside **the minikube shell,** by `$ docker ps` we can see all the containers running inside minikube. These containers are nothing but all the components as described earlier:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675154205641/2de9fcad-230e-45d2-bfe4-3bcd74cac5b7.png align="left")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675154262190/2ffa3ff8-e822-4e08-94ec-f0cd546f34ec.png align="center")

We can see above that each service such as the API server, kube-scheduler, etcd and controller manager are running in individual containers which satisfy the loosely coupled architecture.

**NB: We are inside Minikube SSH and all the containers shown above are running inside it. To exit this use** `$ exit` .

### Step-5(Installing `kubectl`)

As we know **kubectl** is a command line tool that helps us to interact with master node of Kubernetes however, it doesn't come preinstalled so when firing any command with kubectl we get the error command not found

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675157331200/9c251694-85bc-42ec-ba76-916b28489cdc.png align="left")

`sudo snap install kubectl --classic` to install kubectl we can use this command. Here we can notice instead of **sudo apt install** we are using **sudo snap install** because **snap** is used when we want to install containerized applications and kubectl itself is a containerized component of Kubernetes.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675157451894/cc79587c-df19-4973-b1e9-1e8f7cba723e.png align="left")

## Namespace

**The namespace** is nothing but A mechanism for partitioning resources and access control within a cluster.

Namespaces are used to organize and limit access to resources within a cluster. Some common use cases for namespaces include environments (e.g. dev, test, prod), teams, or applications.

In a single node, there will be multiple pods and to identify these pods we have to categorize them under some labels or names.

`$ kubectl get pods djangoPod` will display all the pods available with the namespace **djangoPod**.

### Types of Namespace

There are **4 types** of namespaces which are as below:

* **default**: It is created by Kubernetes itself so that you can use the cluster. If
    
* **kube-node-lease:** It is a Kubernetes object of the "Lease" resource type that is used to track the health of nodes in a cluster. It is stored in the "kube-system" namespace and is created by the controller manager. This helps ensure that the cluster remains healthy by detecting and removing failed nodes.
    
* **kube-public:** It is a namespace in Kubernetes that is created by default and is accessible cluster-wide. This namespace is intended for resources that should be readable by all users in the cluster, such as information about the cluster itself, cluster-level configuration, or shared resources.
    
* **kube-system:** is a special namespace in Kubernetes that is created by default and is used to store resources and objects that are related to the operation of the cluster itself. This namespace contains resources that are managed by the Kubernetes system components such as the control plane and the system daemonsets. Users should not create or modify resources in this namespace unless they understand the implications and potential impact on the operation of the cluster.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675162498615/ade2a146-e779-484b-9846-5426c7ee581e.png align="left")

`$ kubectl get pods` gives us the number of pods running in a working node. But when commanding this for the first time we get the error below because we have not created any pods yet under **the default** namespace.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675159160044/853d4f52-9e28-465f-b7f1-1a3be9d83844.png align="left")

However, this doesn't mean we do not have any Pods at all. Pods are nothing but containers. Even all the components of the master node also function as pods.

`$ kubectl get pods --namespace=kube-system` using this command we can see all the components present under **kube-system** namespace such as the API server, controller, scheduler etcd of the Master node below.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675161467963/d7a536c9-02e4-476e-b470-401b3ea7d7a5.png align="left")

Similarly, we can see pods under kube-public and kube-node-lease using `$ kubectl get pods --namespace=kube-public` and `$ kubectl get pods --namespace=kube-node-lease` respectively when they have some pods else it will through the error saying `No resources were found in namespace.`

### Creating a Namespace

`$ kubectl create namespace django-todo-app` this command will create a namespace with the name **django-todo-app.**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675162776465/4e5e875a-f77a-492c-a081-41a1d08974f7.png align="left")

Now, we can create a pod and run the django app under this newly created namespace called django-todo-app.

## Creating a POD & Running a Node app inside it

To create a pod we have to write a YAML configuration as shown below.

**Note: inside a pod, we run the docker container but we don't build images.**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675164615806/c647457a-924a-4200-9c1f-937ea4e525b2.png align="left")

`$ kubectl apply -f pod.yaml` once the yaml file is written as above, this command is fired to create a pod named **"node-todo-pod"**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675164811170/80cb0e19-3f17-4746-b4fe-cb035dab3e7b.png align="left")

### Verifying if the app is running inside the pod

We have already defined the namespace under which the app should run in the **pod.yaml** configuration file against the field **namespace: django-todo-app** which we have created by ourselves.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675165409065/c9dcdc2c-ad54-4999-b96d-fd2f9115f82f.png align="left")

Using `$ kubectl get namespaces` we can also cross-verify and see that the namespace created by us is listed.

Now using `$ kubectl get pods --namespace=django-todo-app` the command we can see that our pod is successfully up and running.

We can also check from **minikube ssh** by getting inside minikube shell as below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675165677395/54be13c1-7296-4ab5-89ee-eeb72c1e1ffe.png align="left")

## Deleting a Pod

Syntax:`$ kubectl delete pod --namespace=nameOfTheNamepace podname`

Command to delete the created pod: `$ kubectl delete pod --namespace=django-todo-app node-todo-pod`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675166173198/99b7235f-80ad-449a-ad74-7fd009b6270f.png align="left")

And finally our pod is deleted which you can see below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675166338015/c1e54635-19b4-4ed4-8e90-1f13ff02c480.png align="center")