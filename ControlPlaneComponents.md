# Introduction to Kubernetes

Editors: **Kaan Keskin, Sezen Erdem**

Date: November 2021

Available at: https://github.com/kaan-keskin/introduction-to-kubernetes

**Resources:**

> - Kubernetes in Action - Marko Lukša 
> - Kubernetes Documentation - https://kubernetes.io/docs/home/

**LEGAL NOTICE: This document is created for educational purposes, and it can not be used for any commercial purposes. If you find this document useful in any means please support the original authors for ethical reasons.** 

[Return README page.](README.md)

[Return Kubernetes Components page.](KubernetesComponents.md)

## Control Plane Components

The Control Plane is what controls the cluster and makes it function. It consists of multiple components that can run on a single master node or be split across multiple nodes and replicated to ensure high availability. The container orchestration layer that exposes the API and interfaces to define, deploy, and manage the lifecycle of containers.

<img src=".\images\p2_kubernetes_components.jpg"/>

### kube-apiserver

The API server is a component of the Kubernetes control plane that exposes the Kubernetes API. The API server is the front end for the Kubernetes control plane.  You and the other Control Plane components communicate with Kuberntes Cluster via API server.

### kube-scheduler

Schedules the applications. Assigns a worker node to each deployable component of the application. Factors taken into account for scheduling decisions include: individual and collective resource requirements, hardware/software/policy constraints, affinity and anti-affinity specifications, data locality, inter-workload interference, and deadlines.

### etcd

Consistent and highly-available key value data store that persistently stores the cluster configuration.

### kube-controller-manager
Performs cluster-level functions, such as replicating components, keeping track of worker nodes, handling node failures, and so on

### cloud-controller-manager
A Kubernetes control plane component that embeds cloud-specific control logic. The cloud controller manager lets you link your cluster into your cloud provider's API, and separates out the components that interact with that cloud platform from components that only interact with your cluster.
