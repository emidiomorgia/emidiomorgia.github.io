---
layout: post
title: How to setup a pipeline with GitHub Action in order to build and publish a Docker image from your repo into Docker Hub - Part 2
slug: how-to-setup-a-pipeline-with-github-action-in-order-to-build-and-publish-a-docker-image-from-your-repo-into-docker-hub-part-2
date_published: 2021-10-18T18:12:45.000Z
date_updated: 2021-10-18T18:12:45.000Z
tags: CI/CD, Docker, docker hub, Github Actions, Continuous Deploy, Continuos Integration, Continuos Delivery, pipeline
---

## How to build and publish a Docker image from your repository with GitHub Actions

## 1. Overview

Let's continue with the [series of posts](__GHOST_URL__/github-actions-pre) dedicated to **GitHub Actions** with the second article where we will talk about how to *build *and *publish *a **Docker image** from your *GitHub *repository.

If you alredy know about *Docker*, *containers*, *images *and *docker hub* you can jump into [paragraph 2](#paragraph-2).

Why is it important to automate the process of creating container images from our code repository? Because in this way, at each *commit* of our stable *branch*, we can execute a *workflow *that, for example: tests our code, creates an image and pushes it on a container registry. Next, we may have on the production environment, in cloud or on premises, a definition of our application stack (from *docker-compose* or *yaml*) that is run by an orchestrator such as *Kubernetes *or *Docker Swarm*. In this way, publishing a new version of our application, which could also be composed of more services, would simply mean downloading from the container registry the new versions of the container images defined in the application stack and updating the images of the running containers. 

## 1.1 What about Docker, containers and images

I find the [definition on wikipedia](https://en.wikipedia.org/wiki/Docker_(software)) complete: **Docker is a set of platform as a service (PaaS) products that use OS-level virtualization to deliver software in packages called containers**. Containers are isolated from one another and bundle their own software, libraries and configuration files;
they can communicate with each other through well-defined channels. Because all of the containers share the services of a single operating system kernel, **they use fewer resources than virtual machines**.

As we can read from [Docker official web site](https://www.docker.com/resources/what-container): A container is a standard unit of software that packages up code and all its dependencies so the application runs **quickly and reliably** from one computing environment to another.

Container images **become containers at runtime** and in the case of *Docker* containers, images become containers when they run on [Docker Engine](https://www.docker.com/products/container-runtime).

So why use containers and not old fashoned VMs? Again from [docker web site](https://www.docker.com/resources/what-container): Containers are an abstraction at the app layer that packages code and dependencies together. **Multiple containers can run on the same machine and share the OS kernel** with other containers, each running as isolated processes in user space. **Containers take up less space than VMs** (container images are typically tens of MBs in size), **can handle more applications and require fewer** VMs and Operating systems.

## 1.2 How to build a Docker image

When building an image, we usually start with a basic one that will be used to create the environment in which the content of our image will run.

For example, if we want to create a *container *that would run our *Angular *or *React *application, we could start from a base image that uses a Linux distribution on which nginx is installed and then go to insert the static files content into the execution folder of the web server.

The definition of the operations to be performed in order to create the new image must be placed inside a file, usually called **Dockerfile** of which you can find the definition of the syntax on the official Docker [documentation page](https://docs.docker.com/engine/reference/builder/).

The way to build a new image from an existing one is very particular and depends on the type of application you are creating. When it comes to using software that must be compiled before it can be run, there are usually two approaches: the first one is to **compile the sources files** before creating the image and then copy the results into the image.

The second approach consists in copying the sources inside the image, and **compiling them into executables using the same environment that will subsequently execute them**. I personally prefer this approach because it guarantees that the execution of the compiled files takes place within the same environment that compiled them from the sources.

Since the creation of the image is not a topic to be explored in this post, I will provide an **example of creating an image starting from the sources** of a web application in *dotnet core*. There are no limits to what can be built from the base images of the running operating system. A detailed list of examples of how to build images using different languages and frameworks is available at [this address](https://docs.docker.com/samples/).

Suppose we have written an application in *dotnet core* and are in the folder containing the *.csproj* file and use "*aspnetapp*" as the project name, with this Dockerfile we can create an image that, starting from the sources, **builds and publishes the binary files** by placing them in the "out" folder and pointing the container execution entrypoint to the *dll *containing the result:

    # syntax=docker/dockerfile:1
    FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build-env
    WORKDIR /app
    
    # Copy csproj and restore as distinct layers
    COPY *.csproj ./
    RUN dotnet restore
    
    # Copy everything else and build
    COPY ./ ./
    RUN dotnet publish -c Release -o out
    
    # Build runtime image
    FROM mcr.microsoft.com/dotnet/aspnet:5.0
    WORKDIR /app
    COPY --from=build-env /app/out .
    ENTRYPOINT ["dotnet", "aspnetapp.dll"]

Dockerfile for build a dotnetcore app from source
The command to use to create the *image*, assuming you are in the folder containing the file called ***Dockerfile***, is the following:

    $ docker build -t aspnetapp .

command for build the image
where "*aspnetapp*" is the tag that will uniquely identify the image and "*.*" is the parameter that indicates where to find the *Dockerfile*. For a complete guide see the official docker guide at [this address](https://docs.docker.com/engine/reference/commandline/build/).

Use the following command to check if the image was correctly created:

    $ docker images
    
    REPOSITORY                              TAG                 IMAGE ID            CREATED             SIZE
    aspnetapp                               latest              e6780479db63        4 days ago          190MB
    mcr.microsoft.com/dotnet/aspnet         5.0                 e6780479db63        4 days ago          190MB

list all images
## 1.3 How to run a docker image

Let's try to run the following command in orter to start the *container* based on the *image *we built in the previous point:

     $ docker run -d -p 8080:80 --name myapp aspnetapp

command to run the container
The parameter *-d* indicates that we want to execute the container in **detached mode**. The parameter *-p* means: **publish **the container port 80 to host port 8080. The parameter *--name myapp* indicates the **name of the created container** and the parameter *aspnetapp* is **the tag of the source image** used to run the container.

You can check the current running or created *container *with the following  command:

    $ docker ps -a
    
    CONTAINER ID    IMAGE            COMMAND                   CREATED           STATUS     PORTS    NAMES
    0f281cb3af99    aspnetapp        "dotnet NetCore.Dock…"    40 seconds ago    Created             myapp

command to show running containers
Find useful info at the [official documentation](https://docs.docker.com/engine/reference/commandline/run/).

## 1.4 What is Docker Hub

***Docker Hub*** is the world’s largest repository of container images where users get access to ***free public repositories*** for storing and sharing Docker image.

*Docker Hub* provides **repositories**, where you can *push *and *pull container images*, **automatic builds** from *GitHub *and Bitbucket, **trigger actions** after a successful *push *to a repository to integrate *Docker Hub* with other services. 

*Docker Hub* also provides a *CLI tool* (currently experimental) and an *API* that allows you to interact with *Docker Hub*.

From the [docker hub web](https://hub.docker.com/search?type=image) site you can **browse all public available images****that you can pull when executing *docker containers***.

![](__GHOST_URL__/content/images/2021/10/hub-images.png)

Public images from docker hub
Once you created an account you can have one private repository and as many public repositories as you want.

![](__GHOST_URL__/content/images/2021/10/hub-repositories.png)

Find in-depth info at the following [link](https://docs.docker.com/docker-hub/).

## 1.5 How to push an image into docker hub

According to the [official documentation](https://docs.docker.com/engine/reference/commandline/push/), once you have an image with *tag *you can **push it to a registry** using the following syntax: 

     $ docker push [OPTIONS] NAME[:TAG]

docker push syntax
Where *NAME *is in the form of *registry-address:port/account/repository*. For example to *push *on *docker hub* the *image* "*myimage" *with *tag "latest" *for the account "*myaccount" *you can use the following command:

     $ docker push myaccount/myimage:latest

In order to *push *an local *image *to remote registry, you have to create a tag from the local image.

A first option is to create the "*myaccount/myimage:latest"***while you build** the image using the *-t* parameter: 

    $ docker build -t myaccount/myimage .

Another option is to create a *tag *from an **existing **image:

    $ docker image tag myimage:latest registry-host:port-number/myaccount/myimage:latest

## 2. Github Actions and Docker: how to make GitHub Action build and publish our docker image

In the next paragraph we will create the *Dockerfile *that will produce our *image*. Following we will create the *GitHub Action* which will create the *Docker image *starting from the sources and send the *image *to the docker hub.

## 2.1 What we wanto to deploy

The *image *we are going to create will run an *Angular* app, published in the form of static *html, js and css* content, inside an *nginx *web server.
Referring to our *GitHub *repository the *Angular *sources will be located in the "*src/web"* directory. 

The creation of the *Angular *build is done by calling the *ng build* command which will create the compiled files in the *dist/web* folder. This setting is described in the *angular.json* file under the *outputPath *entry.

## 2.2 Dockerfile describing the image

The *Dockerfile *is placed under the root of the project, *src/web*.

    # Stage 1: Compile and Build angular codebase
    
    # Use official node image as the base image
    FROM node:latest as build
    
    # Set the working directory
    WORKDIR /usr/local/app
    
    # Add the source code to app
    COPY ./ /usr/local/app/
    
    # Install all the dependencies
    RUN npm install
    
    # Generate the build of the application
    RUN npm run build
    
    
    # Stage 2: Serve app with nginx server
    
    # Use official nginx image as the base image
    FROM nginx:latest
    
    # Copy the build output to replace the default nginx contents.
    COPY --from=build /usr/local/app/dist/web /usr/share/nginx/html
    
    # Expose port 80
    EXPOSE 80

Dockerfile
During the execution of the *build *of the *image*, first we create a *build environment* based on a *nodejs *image that we use to compile *typescript* sources to js:

    # Use official node image as the base image
    FROM node:latest as build

Then we set the working directory on the *build image* we just declared and copy all sources from the local folder:

    # Set the working directory
    WORKDIR /usr/local/app
    
    # Add the source code to app
    COPY ./ /usr/local/app/

Then we install all the *npm *client packages and run the command to start the *Angular *compilation:

    # Install all the dependencies
    RUN npm install
    
    # Generate the build of the application
    RUN npm run build

Now we can declare the **base image** for the execution environment which will be an *nginx *web server running on *linux*:

    # Stage 2: Serve app with nginx server
    
    # Use official nginx image as the base image
    FROM nginx:latest

The last thing to do is to copy the content of the output folder produced by *Angular *into the folder of *nginx *which will be served by the web server and *expose* the port 80 outside the container:

    # Copy the build output to replace the default nginx contents.
    COPY --from=build /usr/local/app/dist/web /usr/share/nginx/html
    
    # Expose port 80
    EXPOSE 80

If we stopped at this point we could use what we learned in the previous paragraphs to build a *docker image* starting from the definition of the *dockerfile*. This image would compile the *Angular *sources and insert them into the *nginx web *server. If we wanted, we could locally launch a *container *based on our new *image *and map a *port* of our choice to 80 of the container and visualize the result pointing at http://localhost:<port>. 

**Instead, what we want to achieve is, through the automation of a GitHub Action, the compilation and publish of the resulting image remotely, within GitHub, and then push the image obtained on the Docker Hub so that anyone can download and run our image publicly**.

## 2.3 GitHub Action file

As we said in the previous post, it is possible to create a *GitHub Action *by choosing from those available in the *marketplace *or, as in our case, to build a new one using, in some steps, the *actions *taken from the *marketplace*.

Let's start by creating a file called ***web-deploy.yml*** in the **.github/workflows** folder (we will create it if not existing).

The content of the file for the* GitHub Action* is as follows:

    # This is a basic workflow to help you get started with Actions
    
    name: web-deploy CI
    
    # Controls when the workflow will run
    on:
      # Triggers the workflow on push or pull request events but only for the main branch
      push:
        branches: [ main ]
      pull_request:
        branches: [ main ]
    
      # Allows you to run this workflow manually from the Actions tab
      workflow_dispatch:
    
    # A workflow run is made up of one or more jobs that can run sequentially or in parallel
    jobs:
      # This workflow contains a single job called "build"
      build:
        # The type of runner that the job will run on
        runs-on: ubuntu-latest
    
        # Steps represent a sequence of tasks that will be executed as part of the job
        steps:
          # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
          - uses: actions/checkout@v2
    
          - name: Login to Docker Hub
            uses: docker/login-action@v1
            with:
              username: ${{ secrets.DOCKER_HUB_USERNAME }}
              password: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}
              
          - name: Build and push
            id: docker_build
            uses: docker/build-push-action@v2
            with:
              context: ./src/web
              file: ./src/web/Dockerfile
              push: true
              tags: ${{ secrets.DOCKER_HUB_USERNAME }}/timetrails-web:latest
    

content of the GitHub Action: web-deploy.yml
After specified the action name, we provide information about** when the workflow will run**. In our case **after a push or pull request accepted on main branch**:

    # Controls when the workflow will run
    on:
      # Triggers the workflow on push or pull request events but only for the main branch
      push:
        branches: [ main ]
      pull_request:
        branches: [ main ]

Next thing to do is to **define a job** named "build" and specify that it will run on ubuntu:

    jobs:
      # This workflow contains a single job called "build"
      build:
        # The type of runner that the job will run on
        runs-on: ubuntu-latest

Then we need to declare **some steps in order to accomplish the tasks**. The first one is to *checkout *the source code from our *GitHub *repository into the *GitHub Action *environment. This is done using a marketplace action called "*actions/checkout*":

    - uses: actions/checkout@v2

The second step is to **login into the docker hub** with an existing account. Here we use some **secrets stored on *GitHub Secrets*** (secrets.DOCKER_HUB_USERNAME and secrets.DOCKER_HUB_ACCESS_TOKEN). 

This is done using a marketplace action called "*docker/login-action*" :

          - name: Login to Docker Hub
            uses: docker/login-action@v1
            with:
              username: ${{ secrets.DOCKER_HUB_USERNAME }}
              password: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}

The *parameters needed* to login into docker hub are provided by "*username*" and "*password*"

The last thing to do is **build and publish the image into docker hub** and it is done using a marketplace action called "*docker/build-push-action*":

          - name: Build and push
            id: docker_build
            uses: docker/build-push-action@v2
            with:
              context: ./src/web
              file: ./src/web/Dockerfile
              push: true
              tags: ${{ secrets.DOCKER_HUB_USERNAME }}/timetrails-web:latest

It will need some *parameters *in order to identify:

1. the starting folder which will be used as context (***context***)
2. the path of the dockerfile (***file***)
3. the confirm to push (***push***)
4. the tag name of the pushed image (***tags***). 

In the example we used the name of the project as the image name (*timetrails-web*) but **you can use the appropriate value as the image tag name**.

## 2.4 Testing push on commit

To **verify** that our *pipeline *works we can **apply a change to the source code within our repository and push the changes**.

Once the code has been *pushed *to the repository, the *GitHub Action* will start and we will be able to monitor the progress from the *Actions* panel.

![](__GHOST_URL__/content/images/2021/10/action-starting.png)

GitHub Action running after repository push
We can monitor the execution of our action by clicking on the relevant line in the action list by **choosing the one that shows the title of the commit we used** to activate the pipeline.

![](__GHOST_URL__/content/images/2021/10/action-finished.png)

The summary of GitHub Ation executed
We can see the detail of each step declared in the file *web-deploy.yml*.

Finally, in the *docker hub*, we can check if the image was correctly pushed (in our case the image is *timetrails-web*):

![](__GHOST_URL__/content/images/2021/10/docker-hub-after-action.png)

## 2.5 Run the image from docker hub

Now that we have built the image and pushed it into the docker hub we can try to pull the image in docker and run the container at a specified free port with the following command:

    $ docker run --name timetrails-web -p 8080:80 emidio78/timetrails-web

You can **check if the container is running** with the following command and you should receive the following result telling yout that the image was pulled and the container is running at port 8080:

    $ docker ps
    
    CONTAINER ID   IMAGE                     COMMAND                  CREATED              STATUS              PORTS                                   NAMES
    e0107a4d4c93   emidio78/timetrails-web   "/docker-entrypoint.…"   About a minute ago   Up About a minute   0.0.0.0:8080->80/tcp, :::8080->80/tcp   timetrails-web

**Opening the browser at http://localhost:8080** you should see the default Angular app generated by Angular cli:

![](__GHOST_URL__/content/images/2021/10/image-running.png)

Stay tuned for the third and final part of the series where we will talk about **how to run the docker image inside a docker-compose stack from an Ubuntu server** and **how to expose our service through a reverse proxy** such as NGINX.
