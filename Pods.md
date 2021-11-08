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

## Pods

The whole point of Kubernetes is to orchestrate the lifecycle of a container. We do not interact with particular containers. Instead, the smallest unit we can work with is a Pod. Some would say a pod of whales or peas-in-a-pod. Due to shared resources, the design of a Pod typically follows a one-process-per-container architecture.

Containers in a Pod are started in parallel. As a result, there is no way to determine which container becomes available first inside a pod. The use of InitContainers can order startup, to some extent. To support a single process running in a container, you may need logging, a proxy, or special adapter. These tasks are often handled by other containers in the same pod.

There is only one IP address per Pod, for almost every network plugin. If there is more than one container in a pod, they must share the IP. To communicate with each other, they can either use IPC, the loopback interface, or a shared filesystem.

While Pods are often deployed with one application container in each, a common reason to have multiple containers in a Pod is for logging. You may find the term sidecar for a container dedicated to performing a helper task, like handling logs and responding to requests, as the primary application container may not have this ability. The term sidecar, like ambassador and adapter, does not have a special setting, but refers to the concept of what secondary pods are included to do.

Pods are the smallest deployable units of computing that you can create and manage in Kubernetes.

A Pod is a group of one or more containers, with shared storage and network resources, and a specification for how to run the containers.

In Kubernetes, instead of deploying containers individually, you always deploy and operate on a pod of containers. It’s common for pods to contain only a single container. 

    When a pod contains multiple containers, all of them are always run on a single worker node—it never spans multiple worker nodes. 

<img src=".\images\p3_pod_node_relation.jpg"/>

    In Kubernetes, pods live and die, not individual containers.

    All containers of a pod: 
     - share the same set of Linux namespaces instead of each container having its own set.
     - share the same hostname and network interfaces.
     - share the same IP address and port space.   
     - run under the same IPC namespace and can communicate through IPC.

Pods are logical hosts and behave much like physical hosts or VMs in the non-container world. Processes running in the same pod are like processes running on the same physical or virtual machine, except that each process is encapsulated in a container.

A pod is also the basic unit of scaling. Kubernetes can’t horizontally scale individual containers; instead, it scales whole pods.

Pods in a Kubernetes cluster are used in two main ways:

* <b>Pods that run a single container:</b> The "one-container-per-Pod" model is the most common Kubernetes use case

* <b>Pods that run multiple containers that need to work together:</b> A Pod can encapsulate an application composed of multiple co-located containers that are tightly coupled and need to share resources. These co-located containers form a single cohesive unit of service. For example, one container serving data stored in a shared volume to the public, while a separate sidecar container refreshes or updates those files. The Pod wraps these containers, storage resources, and an ephemeral network identity together as a single unit. The main reason to put multiple containers into a single pod is when the application consists of one main process and one or more complementary processes.

Deciding when to use multiple containers in a pod:
* Do they need to be run together or can they run on different hosts?
* Do they represent a single whole or are they independent components?
* Must they be scaled together or individually?

A container shouldn’t run multiple processes. A pod shouldn’t contain multiple containers if they don’t need to run on the same machine.

<img src=".\images\p3_multicontainer_in_a_pod_misusage.jpg"/>

Usually you don't need to create Pods directly, even singleton Pods. Instead, create them using workload resources such as Deployment or Job.

### Creating Pods

Pods and other objects can be created in several ways. They can be created by using a generator, which. historically, has changed with each release:

```shell
$ kubectl run newpod --image=nginx --generator=run-pod/v1
```

Or, they can be created and deleted using properly formatted JSON or YAML files:

```shell
$ kubectl create -f newpod.yaml

$ kubectl delete -f newpod.yaml
```

Pods are usually created by posting a JSON or YAML manifest to the Kubernetes REST API endpoint. 

The pod definition consists following main parts:
* apiVersion: Kubernetes API version used in the YAML
* kind: the type of resource the YAML is describing
* metadata: Includes the name, namespace, labels, and other information about the pod.
* spec: Contains the actual description of the pod’s contents, such as the pod’s containers, volumes, and other data.
* <i>status: contains the current information about the running pod, such as what condition the pod is in, the description and status of each container, and the pod’s internal IP and other basic info.</i>

    The status part contains read-only runtime data that shows the state of the resource at a given moment. When creating a new pod, you never need to provide the status part.

The following is an example of a Pod which consists of a container running the image nginx:1.14.2.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

Other objects will be created by operators/watch-loops to ensure the specifications and current status are the same. 

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

The use of an initContainer allows one or more containers to run only if one or more previous containers run and exit successfully. For example, you could have a checksum verification scan container and a security scan container check the intended containers. Only if both containers pass the checks would the following group of containers be attempted. 

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

### Running Commands in a Container

Part of the testing may be to execute commands within the Pod. What commands are available depend on what was included in the base environment when the image was created. In keeping to a decoupled and lean design, it's possible that there is no shell, or that the Bourne shell is available instead of bash. After testing, you may want to revisit the build and add resources necessary for testing and production.

Use the -it options for an interactive shell instead of the command running without interaction or access.
If you have more than one container, declare which container:

```shell
$ kubectl exec -i​t <Pod-Name> 
```

### Organizing Pods with Labels

With microservices architectures, the number of deployed microservices can easily reach high values. Those components will probably be replicated (multiple copies of the same component will be deployed) and multiple versions or releases (stable, beta, canary, and so on) will run concurrently. This can lead to hundreds of pods in the system. Without a mechanism for organizing them, you end up with a big, incomprehensible mess.

<img src=".\images\p3_uncategorized_pods_example.jpg"/>

Organizing pods and all other Kubernetes objects is done through labels.

<img src=".\images\p3_categorized_pods_example.jpg"/>

Labels go hand in hand with label selectors. Label selectors allow you to select a subset of pods tagged with certain labels and perform an operation on those pods. A label selector is a criterion, which filters resources based on whether they include a certain label with a certain value.

### Using labels and selectors to constrain pod scheduling

Labels and label selectors can be used to constrain pod scheduling. As an example, let one of the nodes in your cluster contains a GPU to be used for general-purpose GPU computing. You add the label gpu=true to the nodes showing this feature.

Now imagine you want to deploy a new pod that needs a GPU to perform its work. To ask the scheduler to only choose among the nodes that provide a GPU, you’ll add a node selector to the pod’s YAML.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubia-gpu
spec:
  nodeSelector:
    gpu: "true"
  containers:
  - name: kubia
    image: luksa/kubia
```
nodeSelector tells Kubernetes to deploy this pod only to nodes containing the gpu=true label. 

### Multi-Container Pod

The idea of multiple containers in a Pod goes against the architectural idea of decoupling as much as possible. One could run an entire operating system inside a container, but would lose much of the granular scalability Kubernetes is capable of. But there are certain needs in which a second or third co-located container makes sense. By adding a second container, each container can still be optimized and developed independently, and both can scale and be repurposed to best meet the needs of the workload.

It may not make sense to recreate an entire image to add functionality like a shell or logging agent. Instead, you could add another container to the Pod, which would provide the necessary tools.

Each container in the Pod should be transient and decoupled. If adding another container limits the scalability or transient nature of the original application, then a new build may be warranted.

Every container in a Pod shares a single IP address and namespace. Each container has equal potential access to storage given to the Pod. Kubernetes does not provide any locking, so your configuration or application should be such that containers do not have conflicting writes.

One container could be read only while the other writes. You could configure the containers to write to different directories in the volume, or the application could have built in locking. Without these protections, there would be no way to order containers writing to the storage.

There are three terms often used for multi-container pods: ambassador, adapter, and sidecar. Based off of the documentation, you may think there is some setting for each. There is not. Each term is an expression of what a secondary pod is intended to do. All are just multi-container pods.

### Multi-Container Pod Terminology

#### Ambassador

Ambassador is, an open source, Kubernetes-native API gateway for microservices built on Envoy.

It allows for access to the outside world without having to implement a service or another entry in an ingress controller: proxy local connection, reverse proxy, limits HTTP requests, re-route from the main container to the outside world.

This type of secondary container would be used to communicate with outside resources, often outside the cluster. Using a proxy, like Envoy or other, you can embed a proxy instead of using one provided by the cluster. It is helpful if you are unsure of the cluster configuration.

#### Adapter

The basic purpose of an adapter container is to modify data, either on ingress or egress, to match some other need. Perhaps, an existing enterprise-wide monitoring tools has particular data format needs. An adapter would be an efficient way to standardize the output of the main container to be ingested by the monitoring tool, without having to modify the monitor or the containerized application. An adapter container transforms multiple applications to singular view.

This type of secondary container is useful to modify the data generated by the primary container. For example, the Microsoft version of ASCII is distinct from everyone else. You may need to modify a datastream for proper use.

#### Sidecar

The idea for a sidecar container is to add some functionality not present in the main container. Rather than bloating code, which may not be necessary in other deployments, adding a container to handle a function such as logging solves the issue, while remaining decoupled and scalable. Prometheus monitoring and Fluentd logging leverage sidecar containers to collect data.

Similar to a sidecar on a motorcycle, it does not provide the main power, but it does help carry stuff. A sidecar is a secondary container which helps or provides a service not found in the primary application. Logging containers are a common sidecar.

### readinessProbe, livenessProbe, and startupProbe

#### readinessProbe

Oftentimes, our application may have to initialize or be configured prior to being ready to accept traffic. As we scale up our application, we may have containers in various states of creation. Rather than communicate with a client prior to being fully ready, we can use a readinessProbe. The container will not accept traffic until the probe returns a healthy state.

With the exec statement, the container is not considered ready until a command returns a zero exit code. As long as the return is non-zero, the container is considered not ready and the probe will keep trying.

Another type of probe uses an HTTP GET request (httpGet). Using a defined header to a particular port and path, the container is not considered healthy until the web server returns a code 200-399. Any other code indicates failure, and the probe will try again.

The TCP Socket probe (tcpSocket) will attempt to open a socket on a predetermined port, and keep trying based on periodSeconds. Once the port can be opened, the container is considered healthy.

#### livenessProbe

Just as we want to wait for a container to be ready for traffic, we also want to make sure it stays in a healthy state. Some applications may not have built-in checking, so we can use livenessProbes to continually check the health of a container. If the container is found to fail a probe, it is terminated. If under a controller, a replacement would be spawned.

#### startupProbe

Becoming beta recently, we can also use the startupProbe. This probe is useful for testing an application which takes a long time to start.

If kubelet uses a startupProbe, it will disable liveness and readiness checks until the application passes the test. The duration until the container is considered failed is failureThreshold times periodSeconds. For example, if your periodSeconds was set to five seconds, and your failureThreshold was set to ten, kubelet would check the application every five seconds until it succeeds, or is considered failed after a total of 50 seconds. If you set the periodSeconds to 60 seconds, kubelet would keep testing for 300 seconds, or five minutes, before considering the container failed.

### Testing

With the decoupled and transient nature and great flexibility, there are many possible combinations of deployments. Each deployment would have its own method for testing. No matter which technology is implemented, the goal is the end user getting what is expected. Building a test suite for your newly deployed application will help speed up the development process and limit issues with the Kubernetes integration.

In addition to overall access, building tools which ensure the distributed application functions properly, especially in a transient environment, is a good idea.

While custom-built tools may be best at testing a deployment, there are some built-in kubectl arguments to begin the process. The first one is describe and the next would be logs.

You can see the details, conditions, volumes and events for an object with describe. The Events at the end of the output can be helpful during testing, as it presents a chronological and node view of cluster actions and associated messages. A simple nginx pod may show the following output:

```shell
$ kubectl describe pod test1
```

A next step in testing may be to look at the output of containers within a pod. Not all applications will generate logs, so it could be difficult  to know if the lack of output is due to error or configuration.

```shell
$ kubectl logs test1
```

A different pod may be configured to show lots of log output, such as the etcd pod:

```shell
$ kubectl -n kube-system logs etcd-master    ##<-- Use Tab to complete for your etcd pod
```

### Removing Pods

```shell
kubectl delete po pod-name

kubectl delete po -l label=value

kubectl delete po --all

kubectl delete ns custom_namespace
```
