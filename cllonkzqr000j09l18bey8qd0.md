---
title: "Containerization of 3-tier Application"
datePublished: Thu Aug 24 2023 04:17:53 GMT+0000 (Coordinated Universal Time)
cuid: cllonkzqr000j09l18bey8qd0
slug: containerization-of-3-tier-application
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1692588313705/a897e554-ba7e-489b-9711-2e31afc7ab77.png
tags: docker, dockerfile, docker-compose, docker-images

---

# Project Overview

In this project, we'll containerize a Java-based multi-tier web application. Here we'll containerize each of the services that are being used such as Nginx, MySQL, Tomcat, etc which will help us reduce resource usage. Further, we'll launch those containers using Docker Compose and test them.

# Architecture

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692587527292/387b6385-bc9e-43a7-b6aa-39359500578d.png align="center")

* We'll fetch the project source code from Git Hub.
    
* Write the docker file for customizing the base images as per our project requirement. Base images will be pulled from the Docker Hub
    
* Build the image using the `docker build` command for Tomcat, MySQL, and Nginx.
    
* Once images are built we'll test them on Docker composed by the docker-`compose.yaml file`
    
* And then after verification we'll push the docker images to our image repository on Docker Hub.
    

# Docker File for Services

Here to minimize the image size we'll write multi-stage docker files. In a multi-stage docker file, we separate the base image from which the main container image inherits only the minimum required dependencies reducing the size of the main image.

## Docker File for Tomcat

```yaml
FROM openjdk:11 AS Build_Image

RUN apt update && apt install maven -y

RUN git clone https://github.com/ritesh-kumar-nayak/vprofile-project-forked.git

RUN cd vprofile-project-forked && git checkout docker && mvn install


FROM tomcat:9-jre11

LABEL project="Vpro"
LABEL Author="Ritesh"

RUN rm -rf /usr/local/tomcat/webapps/*
COPY --from=Build_Image vprofile-project-forked/target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080
CMD [ "catalina.sh", "run" ]
```

1. **Build Stage (**`Build_Image`**):**
    
    * Use the OpenJDK 11 base image.
        
    * Update package information and install Maven.
        
    * Clone the Git repository.
        
    * Change to the cloned repository directory.
        
    * Switch to the `docker` branch using Git.
        
    * Build the project using Maven.
        
2. **Tomcat Stage:**
    
    * Use the Tomcat 9 image with Java 11.
        
    * Remove any existing content from Tomcat's webapps directory.
        
    * Copy the built WAR file from the `Build_Image` stage to the Tomcat webapps directory.
        
3. **Expose Port:**
    
    * Expose port 8080 for the Tomcat server.
        
4. **CMD:**
    
    * Set the command to start the Tomcat server using [`catalina.sh`](http://catalina.sh) `run`.
        

## Docker File for MySQL

```yaml
FROM mysql:8.0.33
LABEL project="Vpro"
LABEL Author="Ritesh"

ENV MYSQL_ROOT_PASSWORD="vprodbpass"
ENV MYSQL_DATABASE="accounts"

ADD db_backup.sql /docker-entrypoint-initdb.d/db_backup.sql
```

1. **Base Image:**
    
    * Use the official MySQL 8.0.33 image as the base image.
        
2. **Labels:**
    
    * Define some labels to provide information about the project and author.
        
3. **Environment Variables:**
    
    * Set the root password for the MySQL instance to "vprodbpass".
        
    * Set the name of the default database to "accounts".
        
4. **Add SQL File:**
    
    * Add a SQL file named `db_backup.sql` to the `/docker-entrypoint-initdb.d/` directory inside the container.
        
    * This SQL file will be executed when the container is started for the first time, populating the database with initial data.
        

## Docker File for Nginx

```yaml
FROM nginx
LABEL project="Vpro"
LABEL Author="Ritesh"

RUN rm -rf /etc/nginx/conf.d/default.conf

COPY nginvproapp.conf /etc/nginx/conf.d/nginvproapp.conf
```

### Application Configuration

```java
upstream vproapp {
 server vproapp:8080; #we have to run container with this name in Docker Environment and Service in K8S environment
}
server {
  listen 80;
location / {
  proxy_pass http://vproapp;
}
}
```

# Docker Compose

Now we'll test our multi-container setup locally using docker compose. Just like the way we can create multiple virtual machines using a single Vagrant file, **Docker Compose** does the same job for containers. In a single file named `docker-compose.yaml` We can place all our container configurations, build the image, and run them simultaneously.

[Docker-Compose documentation](https://docs.docker.com/compose/gettingstarted)

```yaml
version: '3.3'
services:
  vprodb:
    build: 
      context: ./Docker-files/db
    image: ritesh1999/vprofile_db
    container_name: vprodb  
    ports:
      - "3306:3306"
    volumes:
      - vprodbdata:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD:=vprodbpass  

  
  vprocache01:
    image: memcached
    ports:
      - "11211:11211"
  vpromq01:
    image: rabbitmq
    ports:
      - "15672:15672"
    environment:
      - RABBITMQ_DEFAULT_USER=guest
      - RABBITMQ_DEFAULT_PASS=guest
  
  vproapp:
    build: 
      context: ./Docker-files/app
    image: ritesh1999/vprofile_app
    container_name: vproapp
    ports:
      - "8080:8080"
    volumes:
      - vproappdata:/usr/local/tomcar/webapps

  vproweb:
    build: 
      context: ./Docker-files/web
    image: ritesh1999/vprofile-web
    container_name: vproweb  
    ports:
      - "80:80"

volumes:
  vprodbdata: {}
  vproappdata: {}
```

## Build & Run

Here we have listed all the docker files in one place and built it using `docker-compose build`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692848590452/1197dd06-1c99-4f0e-b0b1-2555212a8e31.png align="center")

As we have built the images now, we can start the container by `docker-compose up -d`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692849082352/92f39e4b-0c1c-4588-b4d0-eaedf588e6b7.png align="center")

Now with the command `ip addr show` We found out the host IP and were able to access it:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692849246904/ee43e2b8-61d4-4586-9acb-43e1371b8d09.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692849308529/98504a01-1fe6-477e-90c9-5e8f33cf6926.png align="center")

# Pushing the Images to DockerHub

Login successful

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692849546109/2cc0a360-54dd-46c2-92eb-c609eb874289.png align="center")

Now we can push the images by `docker push <image_name>`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692849860724/aac13de6-cdc8-4425-9c35-1e7f665751c0.png align="center")

# Stopping the Containers

Now, we can stop the container by `docker-compose stop` and `docker-compose rm container_name`

OR

We can use `docker-compose down` to stop and remove it in a single go.

AND we can also remove the images by `docker system prune -a`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1692850369363/f5182564-105b-45a0-8a31-488576510843.png align="center")

Now we have completely cleaned up our system.