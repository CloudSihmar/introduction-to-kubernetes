# Introduction to Kubernetes

Editors: **Kaan Keskin, Sezen Erdem**

Date: November 2021

Available at: https://github.com/kaan-keskin/introduction-to-kubernetes

**Resources:**

> - Kubernetes in Action - Marko Lukša 
> - Kubernetes Documentation - https://kubernetes.io/docs/home/

**LEGAL NOTICE: This document is created for educational purposes, and it can not be used for any commercial purposes. If you find this document useful in any means please support the original authors for ethical reasons.** 

[Return README page.](README.md)

# Kubernetes Components

## Cluster & Node

A <b>Kubernetes cluster</b> is a set of node machines (virtual or physical) for running containerized applications. 

    If you’re running Kubernetes, you’re running a cluster.

A <b>Node</b> is a machine (physical or virtual) on which Kubernetes is installed. A Node is a worker machine, and that is where containers will be launched by Kubernetes.

<img src=".\images\p2_cluster_single_node.jpg"/>

If the Node on which your application is running fails, our application goes down. so we need to have more than one Nodes. A Cluster is a set of Nodes grouped together, If one Node fails, our application is still accessible from the other Nodes. Moreover, having multiple Nodes helps in sharing load as well. 

<img src=".\images\p2_cluster_multi_node.jpg"/>

In a Cluster, we need a master who is responsible for managing the Cluster, store the information about members of the Cluster, monitor the status of Nodes and move the workload of the failed Node to another Worker Node. 

The Master is another Node with Kubernetes installed in it and is configured as a Master. The Master watches over the Nodes in the Cluster and is responsible for the actual orchestration of containers on the Worker Nodes. Master is not required to be a single node. So the management part is called as Control Plane. 

The control plane is responsible for maintaining the desired state of the cluster, such as which applications are running and which container images they use. Nodes actually run the applications and workloads.

<img src=".\images\p2_kubernetes_components.jpg"/>

When you install Kubernetes on a system, you're actually installing the following components, 

* Control Plane Components: 
    * API server 
    * etcd service
    * controllers
        * kube-controller-manager
        * cluster-controller-manager
    * scheduler 

* Worker Node Components    
    * kubelet service, 
    * kube-proxy
    * container runtime

## Control Plane Components

The Control Plane is what controls the cluster and makes it function. It consists of multiple components that can run on a single master node or be split across multiple nodes and replicated to ensure high availability. The container orchestration layer that exposes the API and interfaces to define, deploy, and manage the lifecycle of containers. 

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

## Node Components

### kubelet

An agent that runs on each node in the cluster. It talks to the API server and manages containers on its node. 

### k-proxy

kube-proxy is a network proxy that runs on each node in the cluster. It maintains network rules on nodes. These network rules allow network communication to your Pods from network sessions inside or outside of your cluster.

### container run time

The container runtime is the software that is responsible for running containers.

Kubernetes supports several container runtimes: Docker, containerd, CRI-O, and any implementation of the Kubernetes CRI (Container Runtime Interface).