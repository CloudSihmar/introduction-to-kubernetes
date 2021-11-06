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

## Node

A node is an API object created outside the cluster representing an instance. While a cp must be Linux, worker nodes can also be Microsoft Windows Server 2019. Once the node has the necessary software installed, it is ingested into the API server.

At the moment, you can create a cp node with kubeadm init and worker nodes by passing join. In the near future, secondary cp nodes and/or etcd nodes may be joined.

If the kube-apiserver cannot communicate with the kubelet on a node for 5 minutes, the default NodeLease will schedule the node for deletion and the NodeStatus will change from ready. The pods will be evicted once a connection is re-established. They are no longer forcibly removed and rescheduled by the cluster.

Each node object exists in the kube-node-lease namespace. To remove a node from the cluster, first use kubectl delete node <node-name> to remove it from the API server. This will cause pods to be evacuated. Then, use kubeadm reset to remove cluster-specific information. You may also need to remove iptables information, depending on if you plan on re-using the node.

To view CPU, memory, and other resource usage, requests and limits use the kubectl describe node command. The output will show capacity and pods allowed, as well as details on current pods and resource utilization.

