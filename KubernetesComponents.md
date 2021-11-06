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

## Kubernetes Architecture

To quickly demistify Kubernetes, let's have a look at the Kubernetes Architecture graphic, which shows a high-level architecture diagram of the system components. Not all components are shown. Every node running a container would have kubelet and kube-proxy, for example.

<img src=".\images\kubernetes_architecture.png"/>

Kubernetes has the following main components:
- Control Plane and worker nodes 
- Operators 
- Services 
- Pods of containers 
- Namespaces and quotas 
- Network and policies 
- Storage

In its simplest form, Kubernetes is made of control plane nodes (aka cp nodes) and worker nodes, once called minions. The cp runs an API server, a scheduler, various controllers and a storage system to keep the state of the cluster, container settings, and the networking configuration.

Kubernetes exposes an API via the API server. You can communicate with the API using a local client called kubectl or you can write your own client and use curl commands. The kube-scheduler is forwarded the pod spec for running containers coming to the API and finds a suitable node to run those containers. Each node in the cluster runs two processes: a kubelet, which is often a systemd process, not a container, and kube-proxy. The kubelet receives requests to run the containers, manages any resources necessary and works with the container engine to manage them on the local node. The local container engine could be Docker, cri-o, containerd, or some other.

The kube-proxy creates and manages networking rules to expose the container on the network to other containers or the outside world.
Using an API-based communication scheme allows for non-Linux worker nodes and containers. Support for Windows Server 2019 was graduated to Stable with the 1.14 release. Only Linux nodes can be cp of the cluster at this time.

## Terminology

We have learned that Kubernetes is an orchestration system to deploy and manage containers. Containers are not managed individually; instead, they are part of a larger object called a Pod. A Pod consists of one or more containers which share an IP address, access to storage and namespace. Typically, one container in a Pod runs an application, while other containers support the primary application.

Kubernetes uses namespaces to keep objects distinct from each other, for resource control and multi-tenant considerations. Some objects are cluster-scoped, others are scoped to one namespace at a time. As the namespace is a segregation of resources, pods would need to leverage services to communicate.

Orchestration is managed through a series of watch-loops, also called controllers or operators. Each controller interrogates the kube-apiserver for a particular object state, then modifying the object until the declared state matches the current state. These controllers are compiled into the kube-controller-manager, but others can be added using custom resource definitions. The default and feature-filled operator for containers is a Deployment. A Deployment does not directly work with pods. Instead it manages ReplicaSets. The ReplicaSet is an operator which will create or terminate pods according to a podSpec. The podSpec is sent to the kubelet, which then interacts with the container engine to download and make available the required resources, then spawn or terminate containers until the status matches the spec.

The service operator requests existing IP addresses and information from the endpoint operator, and will manage the network connectivity based on labels. A service is used to communicate between pods, namespaces, and outside the cluster. There are also Jobs and CronJobs to handle single or recurring tasks, among other default operators.

To easily manage thousands of Pods across hundreds of nodes could be difficult. To make management easier, we can use labels, arbitrary strings which become part of the object metadata. These can then be used when checking or changing the state of objects without having to know individual names or UIDs. Nodes can have taints to discourage Pod assignments, unless the Pod has a toleration in its metadata.

There is also space in metadata for annotations which remain with the object but is not used as a selector. This information could be used by the containers, by third-party agents or other tools.

## Kubernetes Components

When you install Kubernetes on a system, you're actually installing the following components, 

> - [Control Plane Node Components](ControlPlaneNodeComponents.md): 
>   - kube-apiserver
>   - etcd
>   - kube-controller-manager
>   - cluster-controller-manager
>   - kube-scheduler
>   - kube-dns
>   - Container Runtime

> - [Worker Node Components](WorkerNodeComponents.md)    
>   - kubelet 
>   - kube-proxy
>   - Container Runtime

### Operators

An important concept for orchestration is the use of operators, otherwise known as controllers or watch-loops. Various operators ship with Kubernetes, and you can create your own, as well. A simplified view of an operator is an agent, or Informer, and a downstream store. Using a DeltaFIFO queue, the source and downstream are compared. A loop process receives an obj or object, which is an array of deltas from the FIFO queue. As long as the delta is not of the type Deleted, the logic of the operator is used to create or modify some object until it matches the specification.

The Informer which uses the API server as a source requests the state of an object via an API call. The data is cached to minimize API server transactions. A similar agent is the SharedInformer; objects are often used by multiple other objects. It creates a shared cache of the state for multiple requests.

A Workqueue uses a key to hand out tasks to various workers. The standard Go work queues of rate limiting, delayed, and time queue are typically used. 

The endpoints, namespace, and serviceaccounts operators each manage the eponymous resources for Pods. Deployments manage replicaSets, which manage Pods running the same podSpec, or replicas.

### Service Operator

With every object and agent decoupled we need a flexible and scalable agent which connects resources together and will reconnect, should something die and a replacement is spawned. A service is an operator which listens to the endpoint operator to provide a persistent IP for Pods. Pods have ephemeral IP addresses chosen from a pool. 

Then the service operator sends messages via the kube-apiserver which forwards settings to kube-proxy on every node, as well as the network plugin such as calico-kube-controllers.

A service also handles access policies for inbound requests, useful for resource control, as well as for security. 

- Connect Pods together 
- Expose Pods to Internet 
- Decouple settings 
- Define Pod access policy.

### Pods

The whole point of Kubernetes is to orchestrate the lifecycle of a container. We do not interact with particular containers. Instead, the smallest unit we can work with is a Pod. Some would say a pod of whales or peas-in-a-pod. Due to shared resources, the design of a Pod typically follows a one-process-per-container architecture.

Containers in a Pod are started in parallel. As a result, there is no way to determine which container becomes available first inside a pod. The use of InitContainers can order startup, to some extent. To support a single process running in a container, you may need logging, a proxy, or special adapter. These tasks are often handled by other containers in the same pod.

There is only one IP address per Pod, for almost every network plugin. If there is more than one container in a pod, they must share the IP. To communicate with each other, they can either use IPC, the loopback interface, or a shared filesystem.

While Pods are often deployed with one application container in each, a common reason to have multiple containers in a Pod is for logging. You may find the term sidecar for a container dedicated to performing a helper task, like handling logs and responding to requests, as the primary application container may not have this ability. The term sidecar, like ambassador and adapter, does not have a special setting, but refers to the concept of what secondary pods are included to do.

### Containers

While Kubernetes orchestration does not allow direct manipulation on a container level, we can manage the resources containers are allowed to consume. 

In the resources section of the PodSpec you can pass parameters which will be passed to the container runtime on the scheduled node: 

```yaml
resources: 
  limits: 
    cpu: "1" 
    memory: "4Gi" 
  requests: 
    cpu: "0.5" 
    memory: "500Mi"
```

Another way to manage resource usage of the containers is by creating a ResourceQuota object, which allows hard and soft limits to be set in a namespace. The quotas allow management of more resources than just CPU and memory and allows limiting several objects. 

A beta feature in v1.12 uses the scopeSelector field in the quota spec to run a pod at a specific priority if it has the appropriate priorityClassName in its pod spec.

### Init Containers

Not all containers are the same. Standard containers are sent to the container engine at the same time, and may start in any order. LivenessProbes, ReadinessProbes, and StatefulSets can be used to determine the order, but can add complexity. Another option can be an Init container, which must complete before app containers will be started. Should the init container fail, it will be restarted until completion, without the app container running. 

The init container can have a different view of the storage and security settings, which allows utilities and commands to be used, which the application would not be allowed to use.. Init containers can contain code or utilities that are not in an app. It also has an independent security from app containers.

The code below will run the init container until the ls command succeeds; then the database container will start.

```yaml
spec: 
  containers: 
  - name: main-app 
    image: databaseD 
  initContainers:
  - name: wait-database
    image: busybox
    command: ['sh', '-c', 'until ls /db/dir ; do sleep 5; done; '] 
```
