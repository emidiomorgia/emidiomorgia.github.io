---
layout: post
title: How to setup a pipeline with GitHub Action in order to build and publish a Docker image from your repo into Docker Hub.
slug: github-actions-pre
date_published: 2021-08-20T12:13:00.000Z
date_updated: 2021-10-18T18:34:04.000Z
tags: Github Actions, docker compose, NGINX, pipeline, Docker, CI/CD, docker hub
---

In this series of posts I will show you **how to set up a *GitHub Action*** that will be triggered when you commit to the repository. The *Action* in particular will test the code, **create a *Docker* image and publish it to the *docker hub***. Next we will use NGINX server in Ubuntu it as a reverse proxy to **run our *Docker* image downloaded from *Docker Hub*** and run through D*ocker Compose*  on a custom port exposed to the outside through *NGINX*.

I often find myself creating repositories on GitHub to study new topics or simply to keep a track of what I have learned and I want to be able to reuse it or maybe share it. For the need I bought a virtual server where I installed *Ubuntu *server in order to be able to publish the result of the projects as they evolve.
This configuration is not always necessary, in fact it is possible to use alternative and free solutions, within certain limits, such as the free tiers of *Azure *or *AWS *or *Heroku *(which offers a lot for free). Maybe in a dedicated post I will talk about these alternatives in detail...

In the past I have used pipelines with TravisCI or Jenkins but now that GitHub offers the possibility to create *pipelines *for free, I think it is very convenient to use a single environment to automate the phase of testing, creating images and publishing the content online. In this way, every time our project is enriched with new features, we will be able to publish on our server, or where we prefer, in complete safety and in an automated and repeatable way.

In this series of posts we will refer to a scenario where we will run the stack that we are going to create, even if consisting of a single container, on an *Ubuntu *server, even locally if you want, using *NGINX* and *Docker*.

The application we are going to transform into a container will be an initial *Angular* project obtained using the *CLI*.

**The articles will be structured in this order:**

- **[Part 1](__GHOST_URL__/github-actions-1/)** - What is a *GitHub* action and how to setup it
- **[Part 2](__GHOST_URL__/how-to-setup-a-pipeline-with-github-action-in-order-to-build-and-publish-a-docker-image-from-your-repo-into-docker-hub-part-2/)** - How to build and publish a *Docker *image from your repository with *GitHub Actions*
- **Part 3** - How to pull and run the published *Docker *image on *NGINX* using *Docker Compose*

Stay tuned for the [first article](__GHOST_URL__/github-actions-1/) that will talk about what *GitHub Actions* are and how to set them up for free on your open source project.
