# Introduction to Kubernetes

Editors: **Kaan Keskin, Sezen Erdem**

Date: November 2021

Available at: https://github.com/kaan-keskin/introduction-to-kubernetes

**Resources:**

> - Kubernetes Documentation - https://kubernetes.io/docs/home/
> - Kubernetes in Action - Marko Lukša 
> - Kubernetes Fundamentals (LFS258) - The Linux Foundation
> - Kubernetes for Developers (LFD259) - The Linux Foundation
> - Getting Started with Kubernetes - Sander van Vugt - Addison-Wesley Professional

**LEGAL NOTICE: This document is created for educational purposes, and it can not be used for any commercial purposes. If you find this document useful in any means please support the original authors for ethical reasons.** 

[Return to the README page.](README.md)

# Kubernetes API Objects

Kubernetes objects are persistent entities in the Kubernetes system. Kubernetes uses these entities to represent the state of your cluster. 

They can describe:

* What containerized applications are running (and on which nodes)

* The resources available to those applications

* The policies around how those applications behave, such as restart policies, upgrades, and fault-tolerance

A Kubernetes object is a "record of intent". By creating an object, you're effectively telling the Kubernetes system what you want your cluster's workload to look like; this is your cluster's desired state.

## Describing a Kubernetes object

To work with Kubernetes objects (create, modify, or delete) you'll need to use the Kubernetes API. <i>kubectl</i> is Kubernetes command-line tool. kubectl allows you to run commands (deploy applications, inspect and manage cluster resources, and view logs) against Kubernetes clusters via Kubernetes API.

When you create an object in Kubernetes, you must provide the object spec that describes its desired state, as well as some basic information about the object (such as a name).

Most often, the information about the objects are provided in .yaml (or .yml) file to kubectl. Kubernetes objects are usually created by posting a JSON or YAML manifest to the Kubernetes REST API endpoint. Defining Kubernetes objects from YAML files makes it possible to store them in a version control system.

### Required fields in YAML files

In the .yaml file for the Kubernetes object you want to create, you'll need to set values for the following fields:

* apiVersion - Which version of the Kubernetes API you're using to create this object

* kind - What kind of object you want to create
* metadata - Data that helps uniquely identify the object, including a name string, UID, and optional namespace
* spec - What state you desire for the object

The precise format of the object spec is different for every Kubernetes object, and contains nested fields specific to that object. 

<img src=".\images\p3_example_yaml.jpg"/>

## v1 API Group

The v1 API group is no longer a single group, but rather a collection of groups for each main object category. For example, there is a v1 group, a storage.k8s.io/v1 group, and an rbac.authorization.k8s.io/v1, etc. Currently, there are eight v1 groups.

We have touched on several objects in lab exercises. Click on each box to learn about some of these objects.

### Node

Represents a machine - physical or virtual - that is part of your Kubernetes cluster. You can get more information about nodes with the kubectl get nodes command. You can turn on and off the scheduling to a node with the kubectl cordon/uncordon commands.

### Service Account

Provides an identifier for processes running in a pod to access the API server and performs actions that it is authorized to do.

### Resource Quota

It is an extremely useful tool, allowing you to define quotas per namespace. For example, if you want to limit a specific namespace to only run a given number of pods, you can write a resourcequota manifest, create it with kubectl and the quota will be enforced.

### Endpoint

Generally, you do not manage endpoints. They represent the set of IPs for pods that match a particular service. They are handy when you want to check that a service actually matches some running pods. If an endpoint is empty, then it means that there are no matching pods and something is most likely wrong with your service definition.

## Discovering API Groups

We can take a closer look at the output of the request for current APIs. Each of the name values can be appended to the URL to see details of that group. 

For example, you could drill down to find included objects at this URL:

    https://localhost:6443/apis/apiregistration.k8s.io/v1beta1.

If you follow this URL, you will find only one resource, with a name of apiservices. If it seems to be listed twice, the lower output is for status. You'll notice that there are different verbs or actions for each. Another entry is if this object is namespaced, or restricted to only one namespace. In this case, it is not. 

```shell
​$ curl https://localhost:6443/apis --header "Authorization: Bearer $token" -k
```

```json
{ 
  "kind": "APIGroupList", 
  "apiVersion": "v1", 
  "groups": [ 
    { 
      "name": "apiregistration.k8s.io", 
      "versions": [ 
        { 
          "groupVersion": "apiregistration.k8s.io/v1", 
          "version": "v1" 
        } 
      ], 
      "preferredVersion": { 
        "groupVersion": "apiregistration.k8s.io/v1", 
        "version": "v1" 
      } 
```

You can then curl each of these URIs and discover additional API objects, their characteristics and associated verbs.

## Deploying an Application

Using the kubectl create command, we can quickly deploy an application. We have looked at the Pods created running the application, like nginx. Looking closer, you will find that a Deployment was created, which manages a ReplicaSet, which then deploys the Pod.

Let's take a closer look at each object. 

### Deployment

It is a controller which manages the state of ReplicaSets and the pods within. The higher level control allows for more flexibility with upgrades and administration. Unless you have a good reason, use a deployment.

### ReplicaSet

Orchestrates individual pod lifecycle and updates. These are newer versions of Replication Controllers, which differ only in selector support.

### Pod

As we've already mentioned, it is the lowest unit we can manage; it runs the application container, and possibly support containers.

## Kubernetes API Objects

> - [Kubernetes API Objects](KubernetesAPIObjects.md)
>   - [Node](Node.md)
>   - [Namespaces](Namespaces.md)
>   - [Pods](Pods.md)
>   - [ReplicationController](ReplicationController.md)
>   - [ReplicaSet](ReplicaSet.md)
>   - [DeamonSet](DeamonSets.md)
>   - [StatefulSet](StatefulSet.md)
>   - [Job and CronJob](JobAndCronJob.md)
>   - [Autoscaling](Autoscaling.md)
