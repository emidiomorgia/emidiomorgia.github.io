+++
title = "How to setup a pipeline with GitHub"
date = 2021-08-20T15:38:30+08:00
header_img = "t-k-9AxFJaNySB8-unsplash.jpg"
toc = true
tags = ["ci/cd","GitHub Actions"]
categories = []
series = ["github-actions"]
+++

In this series of posts I will show you **how to set up a _GitHub Action_** that will be triggered when you commit to the repository. The _Action_ in particular will test the code, **create a _Docker_ image and publish it to the _docker hub_**. Next we will use NGINX server in Ubuntu it as a reverse proxy to **run our _Docker_ image downloaded from _Docker Hub_** and run through D_ocker Compose_  on a custom port exposed to the outside through _NGINX_.

I often find myself creating repositories on GitHub to study new topics or simply to keep a track of what I have learned and I want to be able to reuse it or maybe share it. For the need I bought a virtual server where I installed _Ubuntu_ server in order to be able to publish the result of the projects as they evolve.  
This configuration is not always necessary, in fact it is possible to use alternative and free solutions, within certain limits, such as the free tiers of _Azure_ or _AWS_ or _Heroku_ (which offers a lot for free). Maybe in a dedicated post I will talk about these alternatives in detail...

In the past I have used pipelines with TravisCI or Jenkins but now that GitHub offers the possibility to create _pipelines_ for free, I think it is very convenient to use a single environment to automate the phase of testing, creating images and publishing the content online. In this way, every time our project is enriched with new features, we will be able to publish on our server, or where we prefer, in complete safety and in an automated and repeatable way.

In this series of posts we will refer to a scenario where we will run the ostack that we are going to create, even if consisting of a single container, on an _Ubuntu_ server, even locally if you want, using _NGINX_ and _Docker_.

The application we are going to transform into a container will be an initial _Angular_ project obtained using the _CLI_.

**The articles will be structured in this order:**

-   **[Part 1](posts/github-pipeline-1)** - What is a _GitHub_ action and how to setup it
-   **[Part 2](posts/github-pipeline-2)** - How to build and publish a _Docker_ image from your repository with _GitHub Actions_
-   **Part 3** - How to pull and run the published _Docker_ image on _NGINX_ using _Docker Compose_

Stay tuned for the [first article](posts/github-pipeline-1) that will talk about what _GitHub Actions_ are and how to set them up for free on your open source project.