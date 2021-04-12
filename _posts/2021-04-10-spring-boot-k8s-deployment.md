---
layout: post
title: "Deploying Spring Boot Apps to Kubernetes 
| Part 3: Creating a Kubernetes Deployment"
date: 2021-04-09 12:00
tags: [spring-boot, kubernetes, docker, how-to]
summary: In this third post, we'll explore 
featured_image_thumbnail:
featured_image: /assets/images/posts/2021/deployments.jpg
featured: true
---

In [Part 2](2021-04-09-simple-spring-boot-on-k8s.md) we took the docker container for a basic Spring Boot application and ran it in a kubernetes Pod. In this post we'll be taking at the next level in kubernetes concept: Deployments. At a high level, Deployments allow declarative updates to Pods by expressing desired state. 

### Environment Setup
In this guide I won't focus too much on getting your local environment setup because there are already so many helpful resources out there. During this series, I'll be using docker desktop with kubernetes support enabled. I've found it to be the easiest way to get started with kubernetes quickly on a local machine. Here are a few helpful links to get your local environment configured:
* [Install Docker Desktop for Windows](https://docs.docker.com/docker-for-windows/install/)
* [Enable kubernetes Support in Docker Desktop for Windows](https://docs.docker.com/docker-for-windows/#kubernetes)
* [Docker with WSL2](https://docs.docker.com/docker-for-windows/wsl-tech-preview/)
* [Install Docker Desktop for Mac](https://docs.docker.com/docker-for-mac/install/)
* [Enable kubernetes Support in Docker Desktop for Mac](https://docs.docker.com/docker-for-mac/#kubernetes)
* [minikube](https://minikube.sigs.k8s.io/docs/start/) 

### Deployments
Previously, when working with Pods we were creating and managing a kubernetes resource directly. With Deployments, you create a kubernetes controller that manages other resources - in this case Pods. This is accomplished by expressing the desired state of the application. By state, think of concepts like:
* There should be a certain number of instances of an Application
* The Application should be restarted if it fails a certain condition
* Scale the number of instances of the Application based on certain conditions
Not only can you express the desired state, it can be updated through rolling out updates to Deployments. Read more about Deployments [here](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).

### Create a Deployment

### Configure Health Checks
https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-kubernetes-probes

### Scale a Deployment

### Update a Deployment