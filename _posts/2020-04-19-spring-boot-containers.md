---
layout: post
title: "Deploying Spring Boot Apps to Kubernetes 
| Part 1: Building Containers"
date: 2020-04-19 08:50
tags: [spring-boot, kubernetes, docker, how-to]
summary: In this first post focused on deploying spring boot application to kubernetes, we'll take a look at containerizing spring boot applications.
featured_image_thumbnail:
featured_image: /assets/images/posts/2020/containers.jpg
featured: true
---
Let's face it, no matter where you look in the software development world you'll find mentions of kubernetes. And there are plenty of good reasons to that. Kubernetes is really powerful and becoming many organizations' first choice of deployment platform. So as software developers we often have to take it upon ourselves to figure kubernetes out and get our applications running there. In this series I'm going to focus on simplifying that process by breaking it down into steps (and hopefully include some helpful tips along the way). As a starting point, we'll be packaging a simple Spring Boot application in a docker container. 

_5/16/2020: Updated for release of Spring Boot 2.3_

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

#### Building the Container Image (for Spring Boot 2.3.0.RELEASE and newer)
With the release of Spring Boot 2.3, it was [announced](https://spring.io/blog/2020/01/27/creating-docker-images-with-spring-boot-2-3-0-m1) that there would be native support for building docker containers. The team supporting Spring Boot did some really cool stuff using buildpacks to make it possible. So in order to make our lives easier, we'll be using this method.

We'll start by using to new gradle task to build the image: 
```bash
./gradlew bootBuildImage

> Task :bootBuildImage
Building image 'docker.io/library/springbooktk8s:0.0.1-SNAPSHOT'

 > Pulling builder image 'gcr.io/paketo-buildpacks/builder:base-platform-api-0.3' ..................................................
 > Pulled builder image 'gcr.io/paketo-buildpacks/builder@sha256:7996dd44d157f656bdea4f1063760ad8d4eae2dd41465f7fe6bf720f5b25ca4d'
 > Pulling run image 'gcr.io/paketo-buildpacks/run:base-cnb' ..................................................
 > Pulled run image 'gcr.io/paketo-buildpacks/run@sha256:15bccd9803f63b20a38a6834821a74d9c3949eb475cf759446047dc9586ca2a9'
 > Executing lifecycle version v0.7.5
 > Using build cache volume 'pack-cache-169a3d4a5785.build'

 > Running creator
    [creator]     ---> DETECTING
    [creator]     5 of 15 buildpacks participating
    [creator]     paketo-buildpacks/bellsoft-liberica 2.5.3
    [creator]     paketo-buildpacks/executable-jar    1.2.3
    [creator]     paketo-buildpacks/apache-tomcat     1.1.3
    [creator]     paketo-buildpacks/dist-zip          1.3.0
    [creator]     paketo-buildpacks/spring-boot       1.5.3
    [creator]     ---> ANALYZING
    [creator]     Previous image with name "docker.io/library/springbooktk8s:0.0.1-SNAPSHOT" not found
    [creator]     ---> RESTORING
    [creator]     ---> BUILDING
    [creator]
    [creator]     Paketo BellSoft Liberica Buildpack 2.5.3
    [creator]         Set $BPL_JVM_HEAD_ROOM to configure the headroom in memory calculation. Default 0.
    [creator]         Set $BPL_JVM_LOADED_CLASS_COUNT to configure the number of loaded classes in memory calculation. Default 35% of classes.
    [creator]         Set $BPL_JVM_THREAD_COUNT to configure the number of threads in memory calculation. Default 250.
    [creator]         Set $BP_JVM_VERSION to configure the Java version. Default 11.*.
    [creator]       BellSoft Liberica JRE 11.0.7: Contributing to layer
    [creator]         Downloading from https://github.com/bell-sw/Liberica/releases/download/11.0.7+10/bellsoft-jre11.0.7+10-linux-amd64.tar.gz
    [creator]         Verifying checksum
    [creator]         Expanding to /layers/paketo-buildpacks_bellsoft-liberica/jre
    [creator]         Writing env.launch/JAVA_HOME.override
    [creator]         Writing env.launch/MALLOC_ARENA_MAX.override
    [creator]         Writing profile.d/active-processor-count.sh
    [creator]       Memory Calculator 4.0.0: Contributing to layer
    [creator]         Downloading from https://github.com/cloudfoundry/java-buildpack-memory-calculator/releases/download/v4.0.0/memory-calculator-4.0.0.tgz
    [creator]         Verifying checksum
    [creator]         Expanding to /layers/paketo-buildpacks_bellsoft-liberica/memory-calculator
    [creator]         Writing profile.d/memory-calculator.sh
    [creator]       Class Counter: Contributing to layer
    [creator]         Copying to /layers/paketo-buildpacks_bellsoft-liberica/class-counter
    [creator]       JVMKill Agent 1.16.0: Contributing to layer
    [creator]         Downloading from https://github.com/cloudfoundry/jvmkill/releases/download/v1.16.0.RELEASE/jvmkill-1.16.0-RELEASE.so
    [creator]         Verifying checksum
    [creator]         Copying to /layers/paketo-buildpacks_bellsoft-liberica/jvmkill
    [creator]         Writing env.launch/JAVA_OPTS.append
    [creator]       Link-Local DNS: Contributing to layer
    [creator]         Copying to /layers/paketo-buildpacks_bellsoft-liberica/link-local-dns
    [creator]         Writing profile.d/link-local-dns.sh
    [creator]       Java Security Properties: Contributing to layer
    [creator]         Writing env.launch/JAVA_OPTS.append
    [creator]         Writing env.launch/JAVA_SECURITY_PROPERTIES.override
    [creator]       Security Providers Configurer: Contributing to layer
    [creator]         Copying to /layers/paketo-buildpacks_bellsoft-liberica/security-providers-configurer
    [creator]         Writing profile.d/security-providers-classpath.sh
    [creator]         Writing profile.d/security-providers-configurer.sh
    [creator]       OpenSSL Security Provider 1.0.2: Contributing to layer
    [creator]         Downloading from https://jitpack.io/com/github/paketo-buildpacks/openssl-security-provider/1.0.2/openssl-security-provider-1.0.2.jar
    [creator]         Verifying checksum
    [creator]         Copying to /layers/paketo-buildpacks_bellsoft-liberica/openssl-security-provider
    [creator]         Writing env.launch/SECURITY_PROVIDERS.append
    [creator]         Writing env.launch/SECURITY_PROVIDERS_CLASSPATH
    [creator]         Writing profile.d/openssl-security-provider.sh
    [creator]
    [creator]     Paketo Executable JAR Buildpack 1.2.3
    [creator]         Writing env.launch/CLASSPATH
    [creator]       Process types:
    [creator]         executable-jar: java -cp "${CLASSPATH}" ${JAVA_OPTS} org.springframework.boot.loader.JarLauncher
    [creator]         task:           java -cp "${CLASSPATH}" ${JAVA_OPTS} org.springframework.boot.loader.JarLauncher
    [creator]         web:            java -cp "${CLASSPATH}" ${JAVA_OPTS} org.springframework.boot.loader.JarLauncher
    [creator]
    [creator]     Paketo Spring Boot Buildpack 1.5.3
    [creator]       Image labels:
    [creator]         org.springframework.boot.spring-configuration-metadata.json
    [creator]         org.springframework.boot.version
    [creator]     ---> EXPORTING
    [creator]     Adding layer 'launcher'
    [creator]     Adding layer 'paketo-buildpacks/bellsoft-liberica:class-counter'
    [creator]     Adding layer 'paketo-buildpacks/bellsoft-liberica:java-security-properties'
    [creator]     Adding layer 'paketo-buildpacks/bellsoft-liberica:jre'
    [creator]     Adding layer 'paketo-buildpacks/bellsoft-liberica:jvmkill'
    [creator]     Adding layer 'paketo-buildpacks/bellsoft-liberica:link-local-dns'
    [creator]     Adding layer 'paketo-buildpacks/bellsoft-liberica:memory-calculator'
    [creator]     Adding layer 'paketo-buildpacks/bellsoft-liberica:openssl-security-provider'
    [creator]     Adding layer 'paketo-buildpacks/bellsoft-liberica:security-providers-configurer'
    [creator]     Adding layer 'paketo-buildpacks/executable-jar:class-path'
    [creator]     Adding 1/1 app layer(s)
    [creator]     Adding layer 'config'
    [creator]     *** Images (a07558e9af8b):
    [creator]           docker.io/library/springbooktk8s:0.0.1-SNAPSHOT

Successfully built image 'docker.io/library/springbooktk8s:0.0.1-SNAPSHOT'


BUILD SUCCESSFUL in 1m 10s
4 actionable tasks: 4 executed
```

From the build output, we can see some interesting bits where buildpacks are being used to construct our container image. We can also see that our image was tagged as `docker.io/library/springbooktk8s:0.0.1-SNAPSHOT`. 

We can use it to now run a container:

```bash
$ docker run -it -p 8080:8080 docker.io/library/springbooktk8s:0.0.1-SNAPSHOT
```

And you'll see the output:

```
Container memory limit unset. Configuring JVM for 1G container.
Calculated JVM Memory Configuration: -XX:MaxDirectMemorySize=10M -XX:MaxMetaspaceSize=83662K -XX:ReservedCodeCacheSize=240M -Xss1M -Xmx452913K (Head Room: 0%, Loaded Class Count: 12357, Thread Count: 250, Total M
emory: 1073741824)
Adding Security Providers to JVM

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.3.0.RELEASE)

2020-05-16 17:24:56.284  INFO 1 --- [           main] c.l.s.Springbooktk8sApplication          : Starting Springbooktk8sApplication on 0618def9b835 with PID 1 (/workspace/BOOT-INF/classes started by cnb in /works
pace)
2020-05-16 17:24:56.295  INFO 1 --- [           main] c.l.s.Springbooktk8sApplication          : No active profile set, falling back to default profiles: default
2020-05-16 17:24:58.317  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2020-05-16 17:24:58.338  INFO 1 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2020-05-16 17:24:58.338  INFO 1 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.35]
2020-05-16 17:24:58.447  INFO 1 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2020-05-16 17:24:58.448  INFO 1 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 2049 ms
2020-05-16 17:24:58.853  INFO 1 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2020-05-16 17:24:59.194  INFO 1 --- [           main] o.s.b.a.e.web.EndpointLinksResolver      : Exposing 2 endpoint(s) beneath base path '/actuator'
2020-05-16 17:24:59.367  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2020-05-16 17:24:59.399  INFO 1 --- [           main] c.l.s.Springbooktk8sApplication          : Started Springbooktk8sApplication in 3.898 seconds (JVM running for 4.431)

```

Since we mapped our local port `8080` to the same port on the container (`-p 8080:8080` from the run command), you can go to the actuator endpoint in your browser at `http://localhost:8080/actuator` like you would if you were running the application normally. If you go the `/env` endpoint, you'll see some interesting information about the underlying docker container that's worth checking out. But there you have it, a successfully containerized Spring Boot application!

#### Building the Container Image (For Older Spring Boot Versions)
For older versions of spring boot to build the container, we _could_ compile our application to a jar, write a custom Dockerfile by hand that uses that jar, and then manually build that image from the Dockerfile. Instead we'll be using the [Docker Spring Boot Application Gradle plugin](https://bmuschko.github.io/gradle-docker-plugin/#spring_boot_application_plugin) that automates all of this for us into a simple gradle task. You can find the source code for this method over on [github](https://github.com/lumberjackdev/springboot-on-k8s/tree/part-one-using-gradle-plugin). First add the plugin to the buildscript:

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

### Summing it Up
In this post, we covered some basics about kubernetes and docker containers. We then built a simple Spring Boot application and packaged it into a docker container through gradle. In the next post, we'll take the docker container we built and deploy it to a local kubernetes cluster. Thanks for reading!

Continue on in [Part 2](2020-05-09-simple-spring-boot-on-k8s.md)