

# Getting Started with EKS

## Overview

This lab introduces the basics of working with microservices and [EKS](https://aws.amazon.com/eks/). This includes: setting up the initial EKS cluster, deploying the sample application, service discovery and deployment of the containers. The sample application that will be running on EKS requires a couple Docker images built and placed in an ECR repository that your EKS cluster has access to.

The sample application consists of ColorGateway and ColorTeller applications. 

**ColorGateway**

Color-gateway is a simple http service written in go that is exposed to external clients and responds to http://service-name:port/color that responds with color retrieved from color-teller and histogram of colors observed at the server that responded so far. For e.g.

```
$ curl -s http://colorgateway.default.svc.cluster.local:9080/color
{"color":"blue", "stats": {"blue":"1"}}
```

color-gateway app runs as a service in EKS. 

**ColorTeller**

Color-teller is a simple http service written in go that is configured to return a color. This configuration is provided as environment variable and is run within a task. Multiple versions of this service are deployed each configured to return a specific color.

At the microservice level the application looks like this:

![img1]

[img1]:https://github.com/tohwsw/kubernetes-basics/blob/master/Lab1-Getting-Started-with-ECS/img/microservicesapp.png

They shall be deployed using AWS EKS, which makes it easy for you to run Kubernetes on AWS without needing to install and operate your own Kubernetes clusters. We shall deploy 1 instances of Gateway container and 2 instances of ColorTeller. 


**Note**: 
You'll need to have a working AWS account to use this lab.

