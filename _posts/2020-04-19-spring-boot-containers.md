---
layout: post
title: "Deploying Spring Boot Apps to Kubernetes Part 1: Containers"
date: 2020-04-19 08:50
tags: [spring-boot, kubernetes, docker, how-to]
summary: In this first post focused on deploying spring boot application to kubernetes, we'll take a look at containerizing spring boot applications.
---

Let's face it, no matter where you look in the software development world you'll find mentions of kubernetes. And there are plenty of good reasons to that. Kubernetes is really powerful and becoming many organizations' first choice of platform. So as software developers we often have to take it upon ourselves to figure kubernetes out and get our applications running in it. In this series I'm going to focus on simplifying that process by breaking it down into steps. We'll start with packaging a simple Spring Boot application in a container as our starting point. 

### Environment Setup
In this guide I won't focus too much on getting your local environment setup because there are already so many helpful guides out there. For this guide, I'll be using docker desktop with kubernetes support enabled because I've found it to be the easiest way to get started with kubernetes quickly. Here are a few helpful links to get your local environment setup:
* [Install Docker Desktop for Windows](https://docs.docker.com/docker-for-windows/install/)
* [Enable Kubernetes Support in Docker Desktop for Windows](https://docs.docker.com/docker-for-windows/#kubernetes)
* [Docker with WSL2](https://docs.docker.com/docker-for-windows/wsl-tech-preview/)
* [Install Docker Desktop for Mac](https://docs.docker.com/docker-for-mac/install/)
* [Enable Kubernetes Support in Docker Desktop for Mac](https://docs.docker.com/docker-for-mac/#kubernetes)
* [minikube](https://minikube.sigs.k8s.io/docs/start/) - I haven't used minikube for awhile so your results may vary with this option

### What is Kubernetes?
The [official](https://kubernetes.io/) definition of kubernetes is "Kubernetes (K8s) is an open-source system for automating deployment, scaling, and management of containerized applications." There's a lot going on there, but I like to think of kubernetes as a platform for deploying and running containers. Most of the features provided by kubernetes encourage [cloud native practices](https://tanzu.vmware.com/cloud-native) which draws the attention of many developers and organizations. If you'd like a more technical overview of what kubernetes is, then check out the [official docs](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/).

### Why Start with Containers?
You might be wondering, "Why start with containers? All of my applications are already containerized, so show me the k8s!". If you already have all of your applications build into containers, then that's awesome! This guide _might_ not be for you. You might want to wait for part two. However, I'd encourage you to read along because I'm going to discuss some new Spring Boot features that might make that process easier. 

For everyone else, containers are the natural starting point when beginning your journey into kubernetes deployments. Containers are the building blocks of kubernetes so we'll need applications packaged in containers before we can deploy to kubernetes. 

### So... What's a Container?
Whether your team uses a platform such as Cloud Foundry, deploys `.war` files to tomcat, has customized machine images for your applications, or some other form of deployment you're probably not using containers. For the purpose of this guide, I'll be using the word container to reference docker containers. For anyone new to the world of docker and containers, you can think of them as self-contained packages for everything your application needs to run. In a traditional deployment, you might provision a server or virtual machine and then have everything you need installed on it (ie. Java, standard linux libraries, etc.). With docker containers, all of that is packaged into a docker image that you can use to run your application anywhere docker is installed. 

### Containerizing a Spring Boot Application
With all of that said, let's get started with building a containerized Spring Boot Application. As usual, the source code is available on [GitHub](https://github.com/lumberjackdev/springboot-on-k8s/tree/part-one).

#### Getting Started
For now we'll be building a simple Spring Boot Web App with just the actuator dependency for health checks. The dependency section will look like this:

```groovy
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	implementation 'org.springframework.boot:spring-boot-starter-actuator'

	testImplementation('org.springframework.boot:spring-boot-starter-test') {
		exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
	}
}
```

And for now the `application.yml` will be empty. 

#### Building the Container
In order to build the container, we'll be using the [Docker Spring Boot Application Gradle plugin](https://bmuschko.github.io/gradle-docker-plugin/#spring_boot_application_plugin). So first add the plugin to the buildscript:

```groovy
plugins {
	id 'org.springframework.boot' version '2.2.6.RELEASE'
	id 'com.bmuschko.docker-spring-boot-application' version '6.4.0'
	id 'io.spring.dependency-management' version '1.0.9.RELEASE'
	id 'java'
}
```

This plugin adds a few configurations and tasks, but for now we're interested in the `dockerBuildImage` task. So let's go ahead and run that:

```bash
$ ./gradlew buildDockerImage
```

**Note for Windows Users: You might need to run the dockerCreateDockerfile and build the image from that directly or use Docker within wsl2**