---
layout: post
title: "Deploying Spring Boot Apps to Kubernetes 
| Part 2: Running a Container in a Pod"
date: 2021-04-09 12:00
tags: [spring-boot, kubernetes, docker, how-to]
summary: In this second post, we'll take the docker container we built in the previous post and deploy it to a local kubernetes cluster.
featured_image_thumbnail:
featured_image: /assets/images/posts/2021/pods.jpg
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
Now that we know a little about pods, let's get the docker image we built last time running in a pod (if you need to rebuild the image you can do that with `./gradlew bootBuildImage`). In order to run our application, we'll run a command using the kubernetes cli - `kubectl`:

 ```bash
 kubectl run demo --image=docker.io/library/springbooktk8s:0.0.1-SNAPSHOT --port=8080
 ``` 
 
 With this command kubernetes will run a pod named `demo` which can be used to reference the pod later. The `image` flag instructs kubernetes to run the image just built for the Spring Boot application as a container in the Pod. The `port` option allows for other nodes within the cluster to be able to access the container on the specified port.

To check that status of the Pod, run the command `kubectl get pods` (this will return information about any running Pods).

```
NAME              READY   STATUS    RESTARTS   AGE
demo              1/1     Running   0          4s
```

When running the cli command, kubernetes is creating a definition of a Pod for us. To take a look at that definition you can run the previous command but as a dry run:

```bash
kubectl run demo --image=docker.io/library/springbooktk8s:0.0.1-SNAPSHOT --dry-run=client -o yaml
```

Which should generate output that looks similar to this:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: demo
  name: demo
spec:
  containers:
  - image: docker.io/library/springbooktk8s:0.0.1-SNAPSHOT
    name: demo
    ports:
    - containerPort: 8080
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

There's a few things to note in this yaml:
* The `apiVersion` refers to the version of the kubernetes api
* `kind` determines what type of kubernetes resource being defined, in this case a Pod
* `metadata.name` is a mandatory field so that kubernetes has a way of identifying the Pod
* The `spec.containers` section defines the containers running in the Pod, what we see here should look familiar to `kubectl run` command used earlier.
For now we can ignore the rest of the fields, we'll go into details where necessary on those. 


### Accessing the Application
At this point, you might be tempted to point the browser to `http://localhost:8080/actuator/health` to see the running app's health. By default, Pods and their containers will only be accessible within the cluster. To access the application, the request will need to be made within the cluster. To do this, first find the Pod's IP Address by running:

```bash
kubect describe pod demo
```

In this output find a section that looks like:

```
Name:         demo
Namespace:    default
Priority:     0
Node:         docker-desktop/192.168.65.4
Start Time:   Sat, 10 Apr 2021 12:25:09 -0400
Labels:       run=demo
Annotations:  <none>
Status:       Running
IP:           10.1.0.18
```

For this example, the IP Address of the Pod's Cluster IP Address is `10.1.0.18`. Next, run a busybox container that the `wget` command can be run from:

```bash
kubectl run -i --tty --rm debug --image=busybox --restart=Never --
```

Once the shell prompt is up, make a request to the container using:

```bash
wget -qO- http://10.1.0.18:8080/actuator/health
```

And there should be an output similar to:

```json
{"status":"UP","groups":["liveness","readiness"]}
```

### Recap
In this post, you should have run a Docker image for a Spring Boot Application in a Kubernetes Pod. We then saw what the Pod definition looked like. Finally, we ran a busybox container in a different pod that could be used to access the application through the actuator. Thanks for reading!

### Other Useful commands
* `kubectl logs demo` to see the most recent logs
* `kubectl logs -f demo` to follow the logs
* `kubectl delete pod demo` to delete the pod 
* `kubectl get pod demo` to see the status of the Pod
* `kubectl describe pod demo` to see a detailed description of the Pod
* `kubectl get pod demo -o yaml|json` to see the full yaml or json definition of the Pod