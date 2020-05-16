---
layout: post
title: "Deploying Spring Boot Apps to Kubernetes 
| Part 2: Simple Kubernetes Deployments"
date: 2020-04-26 18:05
tags: [spring-boot, kubernetes, docker, how-to]
summary: In this second post, we'll take the docker container we built in the previous post and deploy it to a local kubernetes cluster.
---

In [Part 1](2020-04-19-spring-boot-containers.md) we built a docker container for a basic Spring Boot application. In this post we'll be taking a look at some of the basic elements of kubernetes deployments and then see how we can deploy our application to a local kubernetes cluster. 

### Environment Setup
In this guide I won't focus too much on getting your local environment setup because there are already so many helpful resources out there. During this series, I'll be using docker desktop with kubernetes support enabled. I've found it to be the easiest way to get started with kubernetes quickly on a local machine. Here are a few helpful links to get your local environment configured:
* [Install Docker Desktop for Windows](https://docs.docker.com/docker-for-windows/install/)
* [Enable Kubernetes Support in Docker Desktop for Windows](https://docs.docker.com/docker-for-windows/#kubernetes)
* [Docker with WSL2](https://docs.docker.com/docker-for-windows/wsl-tech-preview/)
* [Install Docker Desktop for Mac](https://docs.docker.com/docker-for-mac/install/)
* [Enable Kubernetes Support in Docker Desktop for Mac](https://docs.docker.com/docker-for-mac/#kubernetes)
* [minikube](https://minikube.sigs.k8s.io/docs/start/) - I haven't used minikube for awhile so your results may vary with this option
