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

## Node Components

<img src=".\images\p2_kubernetes_components.jpg"/>

### kubelet

An agent that runs on each node in the cluster. It talks to the API server and manages containers on its node. 

### k-proxy

kube-proxy is a network proxy that runs on each node in the cluster. It maintains network rules on nodes. These network rules allow network communication to your Pods from network sessions inside or outside of your cluster.

### container run time

The container runtime is the software that is responsible for running containers.

Kubernetes supports several container runtimes: Docker, containerd, CRI-O, and any implementation of the Kubernetes CRI (Container Runtime Interface).