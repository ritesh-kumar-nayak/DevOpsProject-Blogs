---
title: "Reddit Clone Deployment with Kubernetes"
datePublished: Sat May 13 2023 11:42:43 GMT+0000 (Coordinated Universal Time)
cuid: clhlx4axt000v09l376f16t3g
slug: reddit-clone-deployment-with-kubernetes
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1683979609842/0a4b05a6-3612-4d8c-bc8c-ab2d20ea00de.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1683979899363/fb9c96ca-93e7-40f7-8526-cbf0a97fda99.png
tags: aws, kubernetes, reddit, minikube, trainwithshubham

---

# Project Overview

In this project, we'll be fetching the code of a Reddit Clone app from git repository using the CI server. And will be deploying it to a deployment server using Docker and Kubernetes.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683977952304/fcbdcdf3-35ee-4f92-b1d1-32ad5b94108b.png align="center")

# Prerequisite

* 2 EC2 instances out of which one will be a **t2 medium** and another can be **t2 micro** of **Ubuntu AMI**
    
* Kubernetes should be installed in the **t2 medium** one for scaling
    
* An account on the Docker hub for pushing the image
    
* Git repository where the code exits
    
* Docker should be installed in the CI server
    
* Minikube
    
* Kubectl
    

# Step-1(Setting up the CI server)

After launching the EC2 instance of type **t2.micro** which will be primarily used for coder management and packaging we have to install **Docker** and clone the code from the git repository to the CI server.

### Step-1.1(Docker Installation)

Before moving forward we'll be installing docker by the command:

```bash
sudo apt install docker.io
```

### Step-1.2(Giving permission to Docker)

After the docker installation, to access docker commands we have to give user permission to the Docker using the below command.

```bash
sudo usermod -aG docker $USER && newgrp docker
```

### Step-1.3(Cloning the code from Git)

After Docker installation, we need the project code in our local machine to proceed further with image creation. To do so we have to clone the code from the git repository below:

```bash
git clone https://github.com/LondheShubham153/reddit-clone-k8s-ingress.git
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683965031670/32382496-41da-4b77-b862-9c98ee60a4fa.png align="center")

Now above you can see we have got our project directory named **reddit-clone-k8s-ingress**.

# Step-2(Containerizing the application with Docker)

After getting the code in our Local machine we have to create the docker image of the code which will further be deployed.

### Step-2.1(Writing the Docker file for containerization)

```yaml
FROM node:19-alpine3.15

WORKDIR /reddit-clone

COPY . /reddit-clone
RUN npm install

EXPOSE 3000
CMD ["npm","run","dev"]
```

This Dockerfile creates a Docker image for a Reddit clone application using Node.js version 19 on Alpine Linux version 3.15.

The `FROM` instruction specifies the base image for the Docker image, which in this case is the `node:19-alpine3.15` image.

The `WORKDIR` instruction sets the working directory for any subsequent instructions in the Dockerfile to `/reddit-clone`.

The `COPY` instruction copies the contents of the current directory (.) to the `/reddit-clone` directory in the Docker image.

The `RUN` instruction runs the `npm install` command to install the required dependencies for the Reddit clone application.

The `EXPOSE` instruction exposes port 3000 in the Docker image, which is the port used by the Reddit clone application.

The `CMD` instruction specifies the command to run when the Docker container is started, which in this case is the `npm run dev` command to start the application in development mode.

Overall, this Dockerfile sets up an environment to run a Node.js-based Reddit clone application, installs its dependencies, and starts the application in development mode on port 3000.

### Step-2.2(Building the Docker image from the above Dockerfile)

After writing the Dockerfile we can build a **Docker image** out of it by using the command docker `build . -t ritesh1999/reddit-clone` . this command builds a Docker image for the Reddit clone application based on the Dockerfile in the current directory and tags it with the name `ritesh1999/reddit-clone`:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683966458912/85b8316f-ed27-4bd0-b6cc-80fa31012ae3.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683966853210/8a89c204-8acb-43b8-be2c-fe324ebcf31f.png align="center")

Now above you can see an image `48ba72a7cd20` has been created and tagged. Using `docker images` you can see the image that has been created below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683966957643/955195c3-ee75-405d-b185-296a4f41d602.png align="center")

### Step-2.3(Logging in to Docker Hub)

Docker Hub is nothing but a hub of repositories containing docker Images. Once the image is built from the docker file, we have to push the image to the Docker hub so that during the deployment the image can be fetched from DockerHub itself. It works like GitHub.

So, to push the image we have to log in to our docker hub account from the CLI using the command: `docker login` followed by the username and password of your docker account.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683967375522/26708848-05d4-4276-ad5a-0af12c775a96.png align="center")

### Step-2.4(Pushing the image to Docker Hub)

Once the login is successful we can push the image by using the command below where `docker push username/tagname:latest` is the syntax:

```bash
 docker push ritesh1999/reddit-clone:latest
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683968392419/5d9c630d-bf35-40f2-a505-2cde06f5b303.png align="center")

Now from the above image from Docker Hub, you can see that the image has been pushed.

# Step-3(Setting up Deployment Server)

To set up the deployment server we have to install all the prerequisites as listed above in the prerequisite section.

As this is the Deployment server we need **Kubernetes** to be installed. We will be using **Minikube** to create a Kubernetes cluster.

And to support Minikube we also need Docker to be installed.

**NB: Minikube** **is** **a lightweight Kubernetes implementation that creates a VM on your local machine and deploys a simple cluster containing only one node**.

### Step-3.1(Docker and Minikube Installation)

Using `sudo apt install docker.io` we can install docker exactly as we did for CI server.

And give the permission using `sudo usermod -aG docker $USER && newgrp docker`

And below is the commands for Minikube and Kubernetes cluster setup.

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube 

sudo snap install kubectl --classic
minikube start --driver=docker
```

These commands are used to set up a Kubernetes development environment using Minikube with the Docker driver.

* The first command `curl -LO` [`https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64`](https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64) downloads the latest version of Minikube for Linux in the current directory using the `curl` command.
    
* The second command `sudo install minikube-linux-amd64 /usr/local/bin/minikube` installs the downloaded Minikube binary to `/usr/local/bin/minikube` which makes it accessible from anywhere in the terminal.
    
* The third command `sudo snap install kubectl --classic` installs the `kubectl` command-line tool using the Snap package manager.
    
* Finally, the fourth command `minikube start --driver=docker` starts the Minikube cluster with the Docker driver. The `--driver=docker` flag specifies that the Docker driver should be used as the runtime for the Kubernetes nodes in the Minikube cluster.
    

Overall, these commands set up a local Kubernetes cluster using Minikube with the Docker driver and install the necessary command-line tools such as `kubectl` to interact with the cluster.

With this, the setup has been completed.

# Step-4(App Deployment Preparation)

### Step-4.1(Writing the deployment.yaml File)

By creating and applying a `deployment.yaml` file, Kubernetes will create a Deployment object based on the specified configuration and ensure that the desired state of the Deployment is maintained, handling tasks such as scaling and rolling updates automatically.

So, for our application the deployment file will be as below:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reddit-clone-deployment
  labels:
    app: reddit-clone
spec:
  replicas: 3
  selector:
    matchLabels:
      app: reddit-clone
  template:
    metadata:
      labels:
        app: reddit-clone
    spec:
      containers:
      - name: reddit-clone
        image: ritesh1999/reddit-clone
        ports:
        - containerPort: 3000
```

### Step-4.2(Deployment.yaml explanation)

This is an example Kubernetes Deployment configuration file (`deployment.yaml`) for deploying the Reddit clone application using the image `ritesh1999/reddit-clone`.

The file starts with the `apiVersion` and `kind` fields, which define the Kubernetes API version and object type respectively. In this case, the object type is `Deployment` and the API version is `apps/v1`.

The `metadata` field includes the name and labels for the Deployment object. The `name` field specifies the name of the Deployment object as `reddit-clone-deployment`, and the `labels` field includes a label `app: reddit-clone`.

The `spec` field includes the specifications for the Deployment. The `replicas` field specifies that two replicas of the pod should be created and managed by the Deployment. The `selector` field specifies the label selector that identifies the set of pods managed by the Deployment. In this case, the selector matches the `app: reddit-clone` label.

The `template` field includes the specifications for the pod template that defines the characteristics of the pod. The `metadata` field includes the labels for the pod. The `spec` field includes the specifications for the container that runs in the pod. In this case, there is only one container named `reddit-clone`, which uses the `ritesh1999/reddit-clone` image and exposes port 3000.

Overall, this configuration file defines a Kubernetes Deployment for the Reddit clone application with two replicas, using the `ritesh1999/reddit-clone` image and exposing port 3000.

### Step-4.3(Applying the deployment.yaml)

After writing the **deployment.yaml** configuration we have to apply the configuration to complete the deployment.

```bash
kubectl apply -f deployment.yaml
```

The `kubectl apply -f deployment.yaml` command applies the Kubernetes Deployment configuration specified in the `deployment.yaml` file.

When you run this command, Kubernetes reads the `deployment.yaml` file and creates a new Deployment object or updates an existing one, based on the information provided in the file.

After running `kubectl apply -f deployment.yaml`, you can use `kubectl get deployments` to verify that the Deployment has been created or updated successfully as shown below in which we can see there are **3 Pods** available because we have defined 3 as **Replicas**:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683971845576/1f2666b8-5956-4fff-96c8-fe45a0ae31f8.png align="center")

### Step-4.4(Writing Service.yaml)

Currently, our Reddit Clone application is running inside the Pods or the container of the Kubernetes Cluster that's why it is not yet accessible to the outside world. This service.yaml file will help to expose and IP of the Cluster to the outside and will make is accessible for us.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: reddit-clone-service
  labels:
    app: reddit-clone
spec:
  type: NodePort
  ports:
  - port: 3000
    targetPort: 3000
    nodePort: 31000
  selector:
    app: reddit-clone
```

### Step-4.5(service.yaml explanation)

The file starts with the `apiVersion` and `kind` fields, which define the Kubernetes API version and object type respectively. In this case, the object type is `Service` and the API version is `v1`.

The `metadata` field includes the name and labels for the Service object. The `name` field specifies the name of the Service object as `reddit-clone-service`, and the `labels` field includes a label `app: reddit-clone`.

The `spec` field includes the specifications for the Service. The `type` field specifies the type of the Service as `NodePort`, which exposes the Service on a static port on each node in the cluster.

The `ports` field includes the port mappings for the Service. In this case, the Service is exposed on port 3000, which is the port on which the Reddit clone application is running in the container. The `targetPort` field specifies the port on which the application is listening inside the container, which is also set to 3000. The `nodePort` field specifies the port on which the Service is exposed on each node in the cluster. In this case, it is set to 31000.

The `selector` field specifies the label selector that identifies the set of pods that the Service should route traffic to. In this case, the selector matches the `app: reddit-clone` label.

Overall, this configuration file defines a Kubernetes Service for the Reddit clone application using a NodePort service type, exposing the Service on port 31000 on each node in the cluster and routing traffic to the pods with the `app: reddit-clone` label.

### Step-4.6(Applying service.yaml)

```bash
kubectl apply -f Service.yaml
```

When you run this command, Kubernetes reads the `Service.yaml` file and creates a new Service object or updates an existing one, based on the information provided in the file.

After applying the Service.yaml configuration we can see the app is deployed using the command:

```bash
minikube service reddit-clone-service
```

Here `reddit-clone-service` is the service name. This command also gives us an URL to access the application locally which is [http://192.168.49.2:31000](http://192.168.49.2:31000)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683973160014/b231a8a8-561c-488a-9afa-9eaad4c1baf9.png align="center")

**NB:** However, this URL is not yet accessible outside.

# Step-5(Exposing the Application)

### Step-5.1(Exposing the Deployment)

```bash
kubectl expose deployment reddit-clone-deployment --type=NodePort
```

The `kubectl expose deployment reddit-clone-deployment --type=NodePort` command creates a new Kubernetes Service object to expose the deployment named `reddit-clone-deployment` using the **NodePort** service type.

When you run this command, Kubernetes creates a new Service object and sets its selector to match the labels on the Pods managed by the `reddit-clone-deployment`. It also sets the Service's type to `NodePort`, which exposes the Service on a static port on each node in the cluster.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683973811451/9e67498f-f287-48dd-b5d2-daa392e65946.png align="center")

### Step-5.2(Exposing the Service)

```bash
kubectl port-forward svc/reddit-clone-service 3000:3000 --address 0.0.0.0 &
```

In the above step we have just exposed the deployment now we'll be exposing the Service using the **Port Forwarding** method.

The `kubectl port-forward svc/reddit-clone-service 3000:3000 --address 0.0.0.0 &` command creates a port forwarding tunnel between your local machine and the Kubernetes Service named `reddit-clone-service`.

When you run this command, Kubernetes listens on port 3000 of your local machine and forwards any traffic received on that port to port 3000 of the `reddit-clone-service` in the Kubernetes cluster. The `--address` flag specifies that the tunnel should be accessible from any IP address (0.0.0.0).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683974169563/9d2785e1-d338-4405-a8a2-fe42bc09ae09.png align="center")

**NB:** Note that this command runs in the **foreground**, so you will need to keep the terminal window open for as long as you want the port forwarding tunnel to remain active. If you want to run the command in the background, you can remove the `&` at the end of the command.

### Step-5.3(Exposing the IP of EC2 instance)

As you can see above the **port** has been forwarded to 3000. However, this is still not accessible from Browser externally. To access this we need to edit the inbound rule of our **Deployment Server.** Here we have to add port **3000** and give it IvP4-Everywhere permission as below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683974990896/3cba375b-c480-4752-a735-3b9d2dc55aeb.png align="center")

# Application Deployed

After completing the above steps the Reddit clone app is finally deployed and now accessible from outside:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683976946319/db6a7de6-b9df-40c5-be48-24fec0419ac5.png align="center")