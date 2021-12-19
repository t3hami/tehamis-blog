---
title: "Understand Autoscaling Applications In Kubernetes Before Next Peak Hours"
date: 2021-12-07T23:20:15+05:00
draft: false
author: "Muhammad Tehami"
tags: ["Kubernnetes", "Cloud Computing", "Autoscaling", "Metrics Server", "Nginx"]
categories: ["DevOps"]
---

## What is Autoscaling?

Automatic scaling, is a cloud computing strategy that dynamically modifies the amount of computational resources required for an application. Normally measured by the number of active servers based on the load on the farm. For example, the number of servers hosting a web application can automatically increase or decrease based on the number of active users on your site. Because such metrics can fluctuate substantially during the day, and servers are a limited resource that cost money to run even when inactive, there is often an incentive to run “just enough” servers to support the current demand while still being able to handle sudden and large surges in activity. Autoscaling is useful for such situations because it may reduce the number of active servers when activity is low, and it can also increase the number of active servers when activity is high.

![How the Autoscaling System Works](/images/posts/post_3/how-the-autoscaling-system-works.gif)

Before moving ahead to deploy our application on Kubernetes and Autoscale it, there are a couple of terms we need to be familiar with.

* **Instance:** A single server or machine that is in itself is an independent unit or we can say microservice.
Autoscaling Group: The set of instances that are subject to autoscaling, as well as all associated policies and state data.
* **Size:** The number of instances in the autoscaling group at the moment.
* **Desired Size:** At any given time, the number of instances that the autoscaling group should have. The autoscaling group will try to launch (provision and attach) new instances if the size is less than the intended size. The autoscaling group will try to eliminate (detach and terminate) instances if the size exceeds the specified size.
* **Minimum Size:** The number of instance from which the targeted capacity is not allowed to go below.
* **Maximum Size:** The number of instance from which the targeted capacity is not allowed to go beyond.
* **Metric:** A measurement connected with the autoscaling group for which a time series of data points is created on a regular basis (such as CPU use, memory consumption, and network usage). Metric thresholds can be used to create autoscaling policies.
* **Autoscaling Policy:** In response to metrics reaching particular thresholds, a policy that specifies a modification to the autoscaling group’s targeted capacity (or, in some cases, its minimum and maximum size). Cooldown periods can be connected with scaling policies, preventing future scaling actions from occurring shortly after one. Changes to intended capacity could be incremental (increasing or decreasing by a specified number) or give a new desired capacity value. Policies that enhance desired capacity are referred to as “scaling out” or “scaling up,” while policies that reduce desired capacity are referred to as “scaling in” or “scaling down”.
* **Vertical Scaling:** Vertical scaling preserves your current infrastructure while increasing processing power. All you have to do is run it on an existing number of machines but with improved specs. By scaling up, you can improve the capacity and throughput of a single system. .
* **Horizontal Scaling:** Horizontal scaling increases the number of machine instances without improving the existing specs. Scale out to distribute processing power and load balancing across multiple servers.

We will mainly focus on Horizontal Scaling in this article.

## Setting Up Environment

### Minikube

![Minikube Logo](/images/posts/post_3/minikube.png) 

Minikube is a local Kubernetes that focuses on making Kubernetes easy to learn and develop locally. Kubernetes is only a single command away if you have Docker (or similarly compatible) container tooling or a Virtual Machine environment. Head to the following docs: https://minikube.sigs.k8s.io/docs/start/

### Kubectl

Kubernetes clusters can be managed through the kubectl command line tool.
https://minikube.sigs.k8s.io/docs/handbook/kubectl/

Now we have minikube and kubectl to control our cluster from cli let’s start our very first Kubernetes Cluster:

```
$ minikube start
```

## Deploying Nginx Server on Kubernetes

Let’s deploy an application on Kubernetes. For the demo purpose we are using the most simplest of the deployment; an Nginx server.

Create autoscale namespace.

```
$ kubectl create ns autoscale
```

Create nginx-deployment.yaml file with following content:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: autoscale
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 250m
            memory: 100Mi
          limits:
            cpu: 300m
            memory: 150Mi
```

Create deployment

```
$ kubectl apply -f nginx-deployment.yaml
```

Now we have to expose our application using NodePort service. Create nginx-service.yaml file with following content:

```
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: autoscale
  labels:
    app: nginx
spec:
  type: NodePort
  ports:
  - port: 8080
    nodePort: 31934
    targetPort: 80
    protocol: TCP
    name: http
  selector:
    app: nginx
```

Create service

```
$ kubectl apply -f nginx-service.yaml
```

Verify the Nginx server started and can be accessed using configured Node Port

```
$ curl http://`minikube ip`:31934
```

We have successfully deployed a Nginx server on our Kubernetes Cluster. Now the problem is if our application starts getting a lot of traffic it’s performance will start to decline as there is only one replica(instance) of our deployment(application) running in the cluster. We can manually update the number of replicas in our deployment before peak hours but then it will be unnecessary use of resources during the time there is very less traffic.

## Autoscaling with Kubernetes Metric Server

For Kubernetes built-in autoscaling pipelines, Metrics Server offers a scalable and efficient source of container resource metrics.

The Metrics API in the Kubernetes apiserver collects resource metrics from Kubelet and makes them available to Horizontal Pod Autoscaler and Vertical Pod Autoscaler. kubectl top may also access the metrics API.

### Installing Metric Server

By default there is no metric server installed in Cluster created by Minikube. Install the metric server using the following command:

```
$ minikube addons disable heapster
$ minikube addons enable metrics-server
```

_Note_: Use following Helm Chart to install Metric Server if you are not using Minikube: https://github.com/kubernetes-sigs/metrics-server/tree/master/charts/metrics-server

Validate Metric Server is running

```
$ kubectl get po -n kube-system
NAME                              READY   STATUS       RESTARTS  AGE
coredns-74ff55c5b-x42f6           1/1     Running       0         1d
etcd-minikube                     1/1     Running       0         1d
kube-apiserver-minikube           1/1     Running       0         1d
kube-controller-manager-minikube  1/1     Running       0         1d
kube-proxy-k87m2                  1/1     Running       0         1d
kube-scheduler-minikube           1/1     Running       0         1d
metrics-server-7894db45f8-wxzqd   1/1     Running       0         3m
storage-provisioner               1/1     Running       0         1d
```

Create nginx-autoscale.yaml with following content

```
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-autoscale
  namespace: autoscale
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

Now create Horizontal Pod Autoscaler

```
$ kubectl apply -f nginx-autoscale.yaml
```

Check current metrics

```
$ kubectl top pod -n autoscale
NAME                                CPU(cores)   MEMORY(bytes)
nginx-deployment-67459f4f86-6hq6g   0m           2Mi
```

There is only one pod running which is very obvious as we can see there’s no traffic currently coming to the Nginx server.

### Creating Load With Apache Bench

![Apache Bench Logo](/images/posts/post_3/apache-bench.png) 

The Apache Bench(ab) is a load testing and benchmarking tool for HTTP servers. It’s easy to use and may be started from the terminal. Install for your platform: https://httpd.apache.org/docs/2.4/programs/ab.html

Verify that you have it working by checking Apache Bench version.

```
$ ab -V
```

Now open two terminals

1st terminal to monitor pods and their resource usage

```
$ watch kubectl top pod -n autoscale
```

2nd terminal to send load to Nginx using Apache Bench. Here we are sending a total of 200000 requests and 200 concurrent requests. This will generate enough load to increase CPU utilization above 50%.

```
ab -n 200000 -c 200 http://`minikube ip`:31934/
```

Soon you will see the surge in resource usage and pods getting autoscale. In my machine the number of pods increased to 4.

```
$ NAME                                CPU(cores)   MEMORY(bytes)
nginx-deployment-67459f4f86-6hq6g   139m         2Mi
nginx-deployment-67459f4f86-h6f5m   116m         2Mi
nginx-deployment-67459f4f86-jt66v   52m          2Mi
nginx-deployment-67459f4f86-x55gv   56m          2Mi
```

Once all requests are completed the pods will downscale to 1 again.

## Conclusion

Kubernetes Horizontal Autoscaling takes care of up scaling and down scaling pods based on the resource usage metrics specified. It eliminates the need of manually changing the configuration to meet the current resource usage demand.
