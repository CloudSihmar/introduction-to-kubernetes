## Namespace

A single cluster should be able to satisfy the needs of multiple users or groups of users. Kubernetes namespaces help different projects, teams, or customers to share a Kubernetes cluster. Kubernetes namespaces also provide a scope for objects names. Using multiple namespaces allows you to split complex systems with numerous components into smaller distinct groups. Resource names only need to be unique within a namespace. Two different namespaces can contain resources of the same name. 

The term namespace is used to reference both the kernel feature and the segregation of API objects by Kubernetes. Both are means to keep resources distinct. 

Every API call includes a namespace, using default if not otherwise declared: 
    
    https://10.128.0.3:6443/api/v1/namespaces/default/pods. 

Namespaces, a Linux kernel feature that segregates system resources, are intended to isolate multiple groups and the resources they have access to work with via quotas. Eventually, access control policies will work on namespace boundaries, as well. One could use labels to group resources for administrative reasons. 

Kubernetes starts with four initial namespaces:

* <i>default</i>:  The default namespace for objects with no other namespace. This is where all the resources are assumed, unless set otherwise.

* <i>kube-system</i>: The namespace for objects created by the Kubernetes system. This namespace contains infrastructure pods.

* <i>kube-public</i>: This namespace is created automatically and is readable by all users. This namespace is mostly reserved for cluster usage, in case that some resources should be visible and readable publicly throughout the whole cluster. A namespace readable by all, even those not authenticated. General information is often included in this namespace.

* <i>kube-node-lease</i>: This namespace holds Lease objects associated with each node. Node leases allow the kubelet to send heartbeats so that the control plane can detect node failure. This is the namespace where worker node lease information is kept.

Should you want to see all the resources on a system, you must pass the --all-namespaces option to the kubectl command.

### Creating namespaces

A namespace is a Kubernetes resource like any other, so namespaces can be created by posting a YAML file to the Kubernetes API server.

<img src=".\images\p3_create_ns_yaml_example.jpg"/>

### Discovering namespaces

    >- kubectl get ns
    >- kubectl get po --ns kube-system

### Working with Namespaces

Take a look at the following commands:​

```shell
​$ kubectl get ns
$ kubectl create ns linuxcon
$ kubectl describe ns linuxcon
$ kubectl get ns/linuxcon -o yaml
$ kubectl delete ns/linuxcon​
```

The above commands show how to view, create and delete namespaces. Note that the describe subcommand shows several settings, such as Labels, Annotations, resource quotas, and resource limits, which we will discus later in the course.

Once a namespace has been created, you can reference it via YAML when creating a resource: 

```shell
$ cat redis.yaml
```

```yaml
apiVersion: V1
kind: Pod
metadata: 
    name: redis 
    namespace: linuxcon
...
```
