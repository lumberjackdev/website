---
layout: post
title: "Deploying Spring Boot Apps to Kubernetes 
| Part 3: Creating a Kubernetes Deployment"
date: 2021-05-14 12:00
tags: [spring-boot, kubernetes, docker, how-to]
summary: In this third post, we explore kubernetes Deployments for Spring Boot Applications
featured_image_thumbnail:
featured_image: /assets/images/posts/2021/deployments.jpg
featured_image_href: https://unsplash.com/photos/eqwFWHfQipg
featured_image_credit: CHUTTERSNAP
featured: true
---

In [Part 2](2021-04-09-simple-spring-boot-on-k8s.md) we took the docker container for a basic Spring Boot application and ran it in a kubernetes Pod. In this post we explore the next level in kubernetes concept: Deployments. At a high level, Deployments allow declarative updates to Pods by expressing desired state. In this guide we create a kubernetes Deployment for a Spring Boot Application, inspect the definition for the Deployment, and update the deployment. 

### Environment Setup
In this guide I won't focus too much on getting your local environment setup because there are already so many helpful resources out there. During this series, I use docker desktop with kubernetes support enabled. I've found it to be the easiest way to get started with kubernetes quickly on a local machine. Here are a few helpful links to get your local environment configured:
* [Install Docker Desktop for Windows](https://docs.docker.com/docker-for-windows/install/)
* [Enable Kubernetes Support in Docker Desktop for Windows](https://docs.docker.com/docker-for-windows/#kubernetes)
* [Docker with WSL2](https://docs.docker.com/docker-for-windows/wsl-tech-preview/)
* [Install Docker Desktop for Mac](https://docs.docker.com/docker-for-mac/install/)
* [Enable Kubernetes Support in Docker Desktop for Mac](https://docs.docker.com/docker-for-mac/#kubernetes)
* [minikube](https://minikube.sigs.k8s.io/docs/start/) 

### Deployments
Previously, when working with Pods we were creating and managing a kubernetes resource directly. With Deployments, we create a kubernetes [controller](https://kubernetes.io/docs/concepts/architecture/controller/) that manages other resources - in this case Pods. This is accomplished by expressing the desired state of the application. By state, think of concepts like:
* There should be a certain number of instances of an Application
* The Application should be restarted if it fails a certain condition
* Scale the number of instances of the Application based on certain conditions

Not only can we express the desired state, it can be updated through rolling out updates to Deployments. Read more about Deployments [here](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).

### Creating a Deployment
The quickest way to create a deployment is using `kubectl`, for the demo Application we've been building we can do this with the following command:

```bash
kubectl create deployment springboot-demo --image-docker.io/library/springbooktk8s:0.0.1-SNAPSHOT --save-config
```

{% include note.html note="the save-config option stores additional information about how the Deployment was created and makes it easier to update later." %}

To check the status of the deployment run:

```bash
kubectl get deployments
```

And we should see output similar to:

```bash
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
springboot-demo   1/1     1            1           4s
```

For this deployment the only options specified were the name and the image to use for the underlying Pods. Because of this, there is only be a single Pod created to run the image. This can be checked with `kubectl get pods` to see an output similar to this:

```bash
NAME                               READY   STATUS    RESTARTS   AGE
springboot-demo-5797d86dfb-w8lw9   1/1     Running   0          7s
```

Like last time, we can take a look at the Deployment definition using the `--dry-run=client` option and output to yaml using:

```bash
kubectl create deployment springboot-demo --image-docker.io/library/springbooktk8s:0.0.1-SNAPSHOT  --save-config --dry-run=client -o yaml
```

And this outputs the following yaml:

```yaml
# kubernetes/original-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: springboot-demo
  name: springboot-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: springboot-demo
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: springboot-demo
    spec:
      containers:
        - image: docker.io/library/springbooktk8s:0.0.1-SNAPSHOT
          name: springbooktk8s
          resources: {}
status: {}
```

There's a few things to note in this yaml.
* This yaml could be used to create the Deployment to using `kubectl apply -f deployment.yaml`
* `spec.template.spec.containers` should appear similar to the Pod definition we saw in the last time because this is defining the Deployment's template for Pods.
* `spec.replicas` indicates how many Pod replicas of the Application matched by `spec.selector.matchLabels` to run. This matches the `spec.template.metadata.labels` variable for the Pod definition.


### Updating a Deployment
Previously when working directly with Pods, the configuration could not be updated directly. The Pods would need to be deleted and recreated in order to update their configuration in anyway (such as changing underlying container image). Deployments do not have this restriction. To demonstrate how to update the deployment created earlier, we add a container port to the Pod template of the Deployment:

```yaml
# kubernetes/deployment.yaml
spec:
  containers:
    - image: docker.io/library/springbooktk8s:0.0.1-SNAPSHOT
      name: springbooktk8s
      ports:
        - containerPort: 8080
      resources: {}
```

And then apply the change:

```bash
kubectl apply -f deployment.yaml --record
```


{% include note.html note="the record option stores information on the Deployment object about how an update was applied. We can see this when looking up the description of a deployment." %}


If we check the status of the Pods during the update with `kubectl get pods`, then the output shows the Pod without a container port terminating and a new Pod is running:

```bash
NAME                               READY   STATUS        RESTARTS   AGE
springboot-demo-5797d86dfb-sdjqh   1/1     Terminating   0          2m33s
springboot-demo-7b9646b797-pm4d5   1/1     Running       0          8s
```

In the case that we wanted to undo this change, we could undo the deployment with the following command:


```bash
kubectl rollout undo deployment springboot-demo
```

### Conclusion
In this post, we created and updated a kubernetes Deployment for a small Spring Boot Application. In later posts we look into how this Deployment could be configured further with useful features such as liveness and readiness probes or scaling. As usual, the source code over can be found on [github](https://github.com/lumberjackdev/springboot-on-k8s/tree/part-three). Thanks for reading!