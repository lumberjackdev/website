---
layout: post
title: "Deploying Spring Boot Apps to Kubernetes 
| Part 2: Simple Kubernetes Deployments"
date: 2020-05-16 12:00
tags: [spring-boot, kubernetes, docker, how-to]
summary: In this second post, we'll take the docker container we built in the previous post and deploy it to a local kubernetes cluster.
featured_image_thumbnail:
featured_image: /assets/images/posts/2020/pods.jpg
featured: true
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

### Pods
Pods are the most basic unit in a Kubernetes cluster. They are capable of running multiple containers, but typically you'll see a one container per pod philosophy. This makes it much easier to manage and conceptualize because you can think of them as a layer around your containers that allows kubernetes to manage your container more easily. Kubernetes provides networking and volume storage capabilities to pods. The [kubernetes docs](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) have an in-depth overview of what pods are. 

### Running a Spring Boot Application in a Pod
Now that we know a little about pods, let's get the docker image we built last time running in a pod (if you need to rebuild the image you can do that with `./gradlew bootBuildImage`). In order to run our application, we'll define a yaml file that describes what we want our pod to look like:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: spring-boot-app
spec:
  containers:
    - name: spring-boot-app
      image: docker.io/library/springbooktk8s:0.0.1-SNAPSHOT
      ports:
        - containerPort: 8080
```

There's a few things to note in this yaml file.
* The `apiVersion` refers to the version of the kubernetes api we'll use
* `kind` determines what type of kubernetes resource we want to define, in this case it's Pod because that's what we want to create
* `metadata.name` is a mandatory field so that kubernetes has a way of identifying your Pod
* The `spec.containers` section defines what containers we want to run in our Pod. In this case we want the container to be our image we built. And we'll give the container access to port `8080` in the `ports` section. 

Now let's run our Pod using the kubernetes cli `kubectl apply -f pod.yml`. And you should see the output `pod/spring-boot-app created`. To check that status of the Pod, run the command `kubectl get pods` (this will return information about any running Pods).

```
NAME              READY   STATUS    RESTARTS   AGE
spring-boot-app   1/1     Running   0          4s
```

At this point, you might be tempted to point your browser to `http://localhost:8080/actuator/health` to see your running app's health. Unfortunately, if you do this you won't get anything back. With our current Pod definition, the underlying container is only accessible within the cluster and not from outside. So let's fix that 