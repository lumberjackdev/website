---
layout: post
title: "Deploying Spring Boot Apps to Kubernetes 
| Part 1: Containers"
date: 2020-04-19 08:50
tags: [spring-boot, kubernetes, docker, how-to]
summary: In this first post focused on deploying spring boot application to kubernetes, we'll take a look at containerizing spring boot applications.
featured_image_thumbnail:
featured_image: /assets/images/posts/2020/containers.jpg
featured: true
---

Let's face it, no matter where you look in the software development world you'll find mentions of kubernetes. And there are plenty of good reasons to that. Kubernetes is really powerful and becoming many organizations' first choice of deployment platform. So as software developers we often have to take it upon ourselves to figure kubernetes out and get our applications running there. In this series I'm going to focus on simplifying that process by breaking it down into steps (and hopefully include some helpful tips along the way). As a starting point, we'll be packaging a simple Spring Boot application in a docker container. 

### Environment Setup
In this guide I won't focus too much on getting your local environment setup because there are already so many helpful resources out there. During this series, I'll be using docker desktop with kubernetes support enabled. I've found it to be the easiest way to get started with kubernetes quickly on a local machine. Here are a few helpful links to get your local environment configured:
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
Whether your team uses a platform such as Cloud Foundry, deploys `.war` files to tomcat, has customized machine images for your applications, or some other form of deployment you're probably not using containers. For the purpose of this guide, I'll be using the word container to reference docker containers. For anyone new to the world of docker and containers, you can think of them as self-contained packages of your application and everything it needs to run. In a traditional deployment, you might provision a server or virtual machine and then have everything you need installed on it (ie. Java, standard linux libraries, etc.). With docker containers, all of that is packaged into a docker image that you can use to run your application anywhere docker is installed. 

### Containerizing a Spring Boot Application
With all of that said, let's get started with building a containerized Spring Boot application. As usual, the source code is available on [GitHub](https://github.com/lumberjackdev/springboot-on-k8s/tree/part-one).

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

And update the `application.yml` to expose all of the actuator endpoints:

```yaml
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

#### Building the Container
In order to build the container, we _could_ compile our application to a jar, write a custom Dockerfile by hand that uses that jar, and then manually build that image from the Dockerfile. Instead we'll be using the [Docker Spring Boot Application Gradle plugin](https://bmuschko.github.io/gradle-docker-plugin/#spring_boot_application_plugin) that automates all of this for us into a simple gradle task. So first add the plugin to the buildscript:

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
$ ./gradlew dockerBuildImage
```
**Note for Windows Users: You might need to run the dockerCreateDockerfile and build the image from that directly or use Docker within wsl2**

If you check the `build/docker` directory, you can see the generated Dockerfile. Notice that it's doing some clever things in order to copy libs, resources, and classes. Here's what the file looks like:

```Dockerfile
FROM openjdk:jre-alpine
LABEL maintainer=paul
WORKDIR /app
COPY libs libs/
COPY resources resources/
COPY classes classes/
ENTRYPOINT ["java", "-cp", "/app/resources:/app/classes:/app/libs/*", "com.lumberjackdev.springbooktk8s.Springbooktk8sApplication"]
EXPOSE 8080
```

Next we'll run the container that was created:

```bash
$ docker run -it -p 8080:8080 com.lumberjackdev/springbooktk8s:0.0.1-snapshot
```

If like me you're using a version of Java after 8, then you'll see the following error: `Exception in thread "main" java.lang.UnsupportedClassVersionError`. This exception is due to the fact that the default base image of our docker container (the first line in the Dockerfile) uses java 8. So let's configure the gradle plugin so that our base image uses a later version of Java. For our purposes, we'll  use the base image of `adoptopenjdk:11-jre-hotspot`:

```groovy
docker {
	springBootApplication {
		baseImage = 'adoptopenjdk:11-jre-hotspot'
	}
}
```

Now if we run our docker container, we'll see standard Spring Boot logs:

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.2.6.RELEASE)

2020-04-19 20:09:01.507  INFO 1 --- [           main] c.l.s.Springbooktk8sApplication          : Starting Springbooktk8sApplication on 7ef3d0063eac with PID 1 (/app/classes started by root in /app)
2020-04-19 20:09:01.513  INFO 1 --- [           main] c.l.s.Springbooktk8sApplication          : No active profile set, falling back to default profiles: default
2020-04-19 20:09:03.543  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2020-04-19 20:09:03.569  INFO 1 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2020-04-19 20:09:03.569  INFO 1 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.33]
2020-04-19 20:09:03.705  INFO 1 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2020-04-19 20:09:03.706  INFO 1 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 2048 ms
2020-04-19 20:09:04.317  INFO 1 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2020-04-19 20:09:04.634  INFO 1 --- [           main] o.s.b.a.e.web.EndpointLinksResolver      : Exposing 2 endpoint(s) beneath base path '/actuator'
2020-04-19 20:09:04.755  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2020-04-19 20:09:04.764  INFO 1 --- [           main] c.l.s.Springbooktk8sApplication          : Started Springbooktk8sApplication in 4.193 seconds (JVM running for 4.709)
2020-04-19 20:09:15.722  INFO 1 --- [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2020-04-19 20:09:15.723  INFO 1 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2020-04-19 20:09:15.757  INFO 1 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 34 ms
```

Since we mapped our local port `8080` to the same port on the container (`-p 8080:8080` from the run command), you can go to the actuator endpoint in your browser at `http://localhost:8080/actuator` like you would if you were running the application normally. If you go the `/env` endpoint, you'll see some interesting information about the underlying docker container that's worth checking out. But there you have it, a successfully containerized Spring Boot application!

#### Looking Ahead
In the milestone releases of Spring Boot 2.3, it was [announced](https://spring.io/blog/2020/01/27/creating-docker-images-with-spring-boot-2-3-0-m1) that there would be native support for building docker containers. The team supporting Spring Boot did some really cool stuff using buildpacks to make it possible. I tried this out on a different [branch](https://github.com/lumberjackdev/springboot-on-k8s/tree/part-one-using-spring-boot-2.3) and it was surprisingly easy to use. In the future release, you'll be able to create docker images without this setup by just using the gradle task `buildImage`. Once this version goes GA, I'd definitely recommend switching over to it because it's simpler and does some clever tricks to build the container. 

### Summing it Up
In this post, we covered some basics about kubernetes and docker containers. We then built a simple Spring Boot application and packaged it into a docker container through gradle. In the next post, we'll take the docker container we built and deploy it to a local kubernetes cluster. Thanks for reading!