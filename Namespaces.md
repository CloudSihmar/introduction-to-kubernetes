## Namespace

A single cluster should be able to satisfy the needs of multiple users or groups of users. Kubernetes namespaces help different projects, teams, or customers to share a Kubernetes cluster. Kubernetes namespaces also provide a scope for objects names. Using multiple namespaces allows you to split complex systems with numerous components into smaller distinct groups. Resource names only need to be unique within a namespace. Two different namespaces can contain resources of the same name. 

Kubernetes starts with four initial namespaces:

* <i>default</i>:  The default namespace for objects with no other namespace

* <i>kube-system</i>: The namespace for objects created by the Kubernetes system

* <i>kube-public</i>: This namespace is created automatically and is readable by all users. This namespace is mostly reserved for cluster usage, in case that some resources should be visible and readable publicly throughout the whole cluster. 

* <i>kube-node-lease</i>: This namespace holds Lease objects associated with each node. Node leases allow the kubelet to send heartbeats so that the control plane can detect node failure

### Creating namespaces
A namespace is a Kubernetes resource like any other, so namespaces can be created by posting a YAML file to the Kubernetes API server.


<img src=".\images\p3_create_ns_yaml_example.jpg"/>

### Discovering namespaces

    >- kubectl get ns
    >- kubectl get po --ns kube-system
