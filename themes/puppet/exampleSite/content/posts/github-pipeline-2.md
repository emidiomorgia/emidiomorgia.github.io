+++
title = "How to setup a pipeline with GitHub (part 2)"
date = 2021-10-18T15:38:30+08:00
header_img = "ian-taylor-jOqJbvo1P9g-unsplash.jpg"
toc = true
tags = ["ci/cd","GitHub Actions"]
categories = []
series = ["github-actions"]
+++

## 1\. Overview

Let's continue with the [series of posts](series/github-actions) dedicated to **GitHub Actions** with the second article where we will talk about how to _build_ and _publish_ a **Docker image** from your _GitHub_ repository.

If you alredy know about _Docker_, _containers_, _images_ and _docker hub_ you can jump into [paragraph 2](posts/github-pipeline-2/#2-github-actions-and-docker-how-to-make-github-action-build-and-publish-our-docker-image).

Why is it important to automate the process of creating container images from our code repository? Because in this way, at each _commit_ of our stable _branch_, we can execute a _workflow_ that, for example: tests our code, creates an image and pushes it on a container registry. Next, we may have on the production environment, in cloud or on premises, a definition of our application stack (from _docker-compose_ or _yaml_) that is run by an orchestrator such as _Kubernetes_ or _Docker Swarm_. In this way, publishing a new version of our application, which could also be composed of more services, would simply mean downloading from the container registry the new versions of the container images defined in the application stack and updating the images of the running containers.

## 1.1 What about Docker, containers and images

I find the [definition on wikipedia](https://en.wikipedia.org/wiki/Docker_(software)) complete: **Docker is a set of platform as a service (PaaS) products that use OS-level virtualization to deliver software in packages called containers**. Containers are isolated from one another and bundle their own software, libraries and configuration files;  
they can communicate with each other through well-defined channels. Because all of the containers share the services of a single operating system kernel, **they use fewer resources than virtual machines**.

As we can read from [Docker official web site](https://www.docker.com/resources/what-container): A container is a standard unit of software that packages up code and all its dependencies so the application runs **quickly and reliably** from one computing environment to another.

Container images **become containers at runtime** and in the case of _Docker_ containers, images become containers when they run on [Docker Engine](https://www.docker.com/products/container-runtime).

So why use containers and not old fashoned VMs? Again from [docker web site](https://www.docker.com/resources/what-container): Containers are an abstraction at the app layer that packages code and dependencies together. **Multiple containers can run on the same machine and share the OS kernel** with other containers, each running as isolated processes in user space. **Containers take up less space than VMs** (container images are typically tens of MBs in size), **can handle more applications and require fewer** VMs and Operating systems.

## 1.2 How to build a Docker image

When building an image, we usually start with a basic one that will be used to create the environment in which the content of our image will run.

For example, if we want to create a _container_ that would run our _Angular_ or _React_ application, we could start from a base image that uses a Linux distribution on which nginx is installed and then go to insert the static files content into the execution folder of the web server.

The definition of the operations to be performed in order to create the new image must be placed inside a file, usually called **Dockerfile** of which you can find the definition of the syntax on the official Docker [documentation page](https://docs.docker.com/engine/reference/builder/).

The way to build a new image from an existing one is very particular and depends on the type of application you are creating. When it comes to using software that must be compiled before it can be run, there are usually two approaches: the first one is to **compile the sources files** before creating the image and then copy the results into the image.

The second approach consists in copying the sources inside the image, and **compiling them into executables using the same environment that will subsequently execute them**. I personally prefer this approach because it guarantees that the execution of the compiled files takes place within the same environment that compiled them from the sources.

Since the creation of the image is not a topic to be explored in this post, I will provide an **example of creating an image starting from the sources** of a web application in _dotnet core_. There are no limits to what can be built from the base images of the running operating system. A detailed list of examples of how to build images using different languages and frameworks is available at [this address](https://docs.docker.com/samples/).

Suppose we have written an application in _dotnet core_ and are in the folder containing the _.csproj_ file and use "_aspnetapp_" as the project name, with this Dockerfile we can create an image that, starting from the sources, **builds and publishes the binary files** by placing them in the "out" folder and pointing the container execution entrypoint to the _dll_ containing the result:

```
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
```

Dockerfile for build a dotnetcore app from source

The command to use to create the _image_, assuming you are in the folder containing the file called **_Dockerfile_**, is the following:

```
$ docker build -t aspnetapp .
```

command for build the image

where "_aspnetapp_" is the tag that will uniquely identify the image and "_._" is the parameter that indicates where to find the _Dockerfile_. For a complete guide see the official docker guide at [this address](https://docs.docker.com/engine/reference/commandline/build/).

Use the following command to check if the image was correctly created:

```
$ docker images

REPOSITORY                              TAG                 IMAGE ID            CREATED             SIZE
aspnetapp                               latest              e6780479db63        4 days ago          190MB
mcr.microsoft.com/dotnet/aspnet         5.0                 e6780479db63        4 days ago          190MB
```

list all images

## 1.3 How to run a docker image

Let's try to run the following command in orter to start the _container_ based on the _image_ we built in the previous point:

```
 $ docker run -d -p 8080:80 --name myapp aspnetapp
```

command to run the container

The parameter _\-d_ indicates that we want to execute the container in **detached mode**. The parameter _\-p_ means: **publish** the container port 80 to host port 8080. The parameter _\--name myapp_ indicates the **name of the created container** and the parameter _aspnetapp_ is **the tag of the source image** used to run the container.

You can check the current running or created _container_ with the following  command:

```
$ docker ps -a

CONTAINER ID    IMAGE            COMMAND                   CREATED           STATUS     PORTS    NAMES
0f281cb3af99    aspnetapp        "dotnet NetCore.Dock…"    40 seconds ago    Created             myapp
```

command to show running containers

Find useful info at the [official documentation](https://docs.docker.com/engine/reference/commandline/run/).

## 1.4 What is Docker Hub

**_Docker Hub_** is the world’s largest repository of container images where users get access to **_free public repositories_** for storing and sharing Docker image.

_Docker Hub_ provides **repositories**, where you can _push_ and _pull container images_, **automatic builds** from _GitHub_ and Bitbucket, **trigger actions** after a successful _push_ to a repository to integrate _Docker Hub_ with other services.

_Docker Hub_ also provides a _CLI tool_ (currently experimental) and an _API_ that allows you to interact with _Docker Hub_.

From the [docker hub web](https://hub.docker.com/search?type=image) site you can **browse all public available images** **that you can pull when executing _docker containers_**.

![](/hub-images.png)

Public images from docker hub

Once you created an account you can have one private repository and as many public repositories as you want.

![](/hub-repositories.png)

Find in-depth info at the following [link](https://docs.docker.com/docker-hub/).

## 1.5 How to push an image into docker hub

According to the [official documentation](https://docs.docker.com/engine/reference/commandline/push/), once you have an image with _tag_ you can **push it to a registry** using the following syntax:

```
 $ docker push [OPTIONS] NAME[:TAG]
```

docker push syntax

Where _NAME_ is in the form of _registry-address:port/account/repository_. For example to _push_ on _docker hub_ the _image_ "_myimage"_ with _tag "latest"_ for the account "_myaccount"_ you can use the following command:

```
 $ docker push myaccount/myimage:latest
```

In order to _push_ an local _image_ to remote registry, you have to create a tag from the local image.

A first option is to create the "_myaccount/myimage:latest"_ **while you build** the image using the _\-t_ parameter:

```
$ docker build -t myaccount/myimage .
```

Another option is to create a _tag_ from an **existing** image:

```
$ docker image tag myimage:latest registry-host:port-number/myaccount/myimage:latest
```

## 2\. Github Actions and Docker: how to make GitHub Action build and publish our docker image

In the next paragraph we will create the _Dockerfile_ that will produce our _image_. Following we will create the _GitHub Action_ which will create the _Docker image_ starting from the sources and send the _image_ to the docker hub.

## 2.1 What we wanto to deploy

The _image_ we are going to create will run an _Angular_ app, published in the form of static _html, js and css_ content, inside an _nginx_ web server.  
Referring to our _GitHub_ repository the _Angular_ sources will be located in the "_src/web"_ directory.

The creation of the _Angular_ build is done by calling the _ng build_ command which will create the compiled files in the _dist/web_ folder. This setting is described in the _angular.json_ file under the _outputPath_ entry.

## 2.2 Dockerfile describing the image

The _Dockerfile_ is placed under the root of the project, _src/web_.

```
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
```

Dockerfile

During the execution of the _build_ of the _image_, first we create a _build environment_ based on a _nodejs_ image that we use to compile _typescript_ sources to js:

```
# Use official node image as the base image
FROM node:latest as build
```

Then we set the working directory on the _build image_ we just declared and copy all sources from the local folder:

```
# Set the working directory
WORKDIR /usr/local/app

# Add the source code to app
COPY ./ /usr/local/app/
```

Then we install all the _npm_ client packages and run the command to start the _Angular_ compilation:

```
# Install all the dependencies
RUN npm install

# Generate the build of the application
RUN npm run build
```

Now we can declare the **base image** for the execution environment which will be an _nginx_ web server running on _linux_:

```
# Stage 2: Serve app with nginx server

# Use official nginx image as the base image
FROM nginx:latest
```

The last thing to do is to copy the content of the output folder produced by _Angular_ into the folder of _nginx_ which will be served by the web server and _expose_ the port 80 outside the container:

```
# Copy the build output to replace the default nginx contents.
COPY --from=build /usr/local/app/dist/web /usr/share/nginx/html

# Expose port 80
EXPOSE 80
```

If we stopped at this point we could use what we learned in the previous paragraphs to build a _docker image_ starting from the definition of the _dockerfile_. This image would compile the _Angular_ sources and insert them into the _nginx web_ server. If we wanted, we could locally launch a _container_ based on our new _image_ and map a _port_ of our choice to 80 of the container and visualize the result pointing at [http://localhost](http://localhost/):<port>.

**Instead, what we want to achieve is, through the automation of a GitHub Action, the compilation and publish of the resulting image remotely, within GitHub, and then push the image obtained on the Docker Hub so that anyone can download and run our image publicly**.

## 2.3 GitHub Action file

As we said in the previous post, it is possible to create a _GitHub Action_ by choosing from those available in the _marketplace_ or, as in our case, to build a new one using, in some steps, the _actions_ taken from the _marketplace_.

Let's start by creating a file called _**web-deploy.yml**_ in the **.github/workflows** folder (we will create it if not existing).

The content of the file for the _GitHub Action_ is as follows:

```
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
```

content of the GitHub Action: web-deploy.yml

After specified the action name, we provide information about **when the workflow will run**. In our case **after a push or pull request accepted on main branch**:

```
# Controls when the workflow will run
on:
  # Triggers the workflow on push or pull request events but only for the main branch
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
```

Next thing to do is to **define a job** named "build" and specify that it will run on ubuntu:

```
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest
```

Then we need to declare **some steps in order to accomplish the tasks**. The first one is to _checkout_ the source code from our _GitHub_ repository into the _GitHub Action_ environment. This is done using a marketplace action called "_actions/checkout_":

```
- uses: actions/checkout@v2
```

The second step is to **login into the docker hub** with an existing account. Here we use some **secrets stored on _GitHub Secrets_** (secrets.DOCKER\_HUB\_USERNAME and secrets.DOCKER\_HUB\_ACCESS\_TOKEN).

This is done using a marketplace action called "_docker/login-action_" :

```
      - name: Login to Docker Hub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKER_HUB_USERNAME }}
          password: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}
```

The _parameters needed_ to login into docker hub are provided by "_username_" and "_password_"

The last thing to do is **build and publish the image into docker hub** and it is done using a marketplace action called "_docker/build-push-action_":

```
      - name: Build and push
        id: docker_build
        uses: docker/build-push-action@v2
        with:
          context: ./src/web
          file: ./src/web/Dockerfile
          push: true
          tags: ${{ secrets.DOCKER_HUB_USERNAME }}/timetrails-web:latest
```

It will need some _parameters_ in order to identify:

1.  the starting folder which will be used as context (_**context**_)
2.  the path of the dockerfile (_**file**_)
3.  the confirm to push (_**push**_)
4.  the tag name of the pushed image (_**tags**_).

In the example we used the name of the project as the image name (_timetrails-web_) but **you can use the appropriate value as the image tag name**.

## 2.4 Testing push on commit

To **verify** that our _pipeline_ works we can **apply a change to the source code within our repository and push the changes**.

Once the code has been _pushed_ to the repository, the _GitHub Action_ will start and we will be able to monitor the progress from the _Actions_ panel.

![](/action-starting.png)

GitHub Action running after repository push

We can monitor the execution of our action by clicking on the relevant line in the action list by **choosing the one that shows the title of the commit we used** to activate the pipeline.

![](/action-finished.png)

The summary of GitHub Ation executed

We can see the detail of each step declared in the file _web-deploy.yml_.

Finally, in the _docker hub_, we can check if the image was correctly pushed (in our case the image is _timetrails-web_):

![](/docker-hub-after-action.png)

## 2.5 Run the image from docker hub

Now that we have built the image and pushed it into the docker hub we can try to pull the image in docker and run the container at a specified free port with the following command:

```
$ docker run --name timetrails-web -p 8080:80 emidio78/timetrails-web
```

You can **check if the container is running** with the following command and you should receive the following result telling yout that the image was pulled and the container is running at port 8080:

```
$ docker ps

CONTAINER ID   IMAGE                     COMMAND                  CREATED              STATUS              PORTS                                   NAMES
e0107a4d4c93   emidio78/timetrails-web   "/docker-entrypoint.…"   About a minute ago   Up About a minute   0.0.0.0:8080->80/tcp, :::8080->80/tcp   timetrails-web
```

**Opening the browser at [http://localhost:8080](http://localhost:8080/)** you should see the default Angular app generated by Angular cli:

![](/image-running.png)

Stay tuned for the third and final part of the series where we will talk about **how to run the docker image inside a docker-compose stack from an Ubuntu server** and **how to expose our service through a reverse proxy** such as NGINX.he project) and then **publish the content** on a Docker Hub repository.