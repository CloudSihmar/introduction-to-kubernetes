# PART I - What is Kubernetes?

## Definition of Kubernetes

Kubernetes is a portable, extensible, open-source platform for managing containerized workloads and services, that facilitates both declarative configuration and automation. It has a large, rapidly growing ecosystem. Kubernetes services, support, and tools are widely available.

The name Kubernetes originates from Greek, meaning helmsman or pilot. K8s as an abbreviation results from counting the eight letters between the "K" and the "s". 

## History of Kubernetes

Kubernetes was founded by Joe Beda, Brendan Burns, and Craig McLuckie, who were quickly joined by other Google engineers including Brian Grant and Tim Hockin, and was first announced by Google in mid-2014. 

Its development and design are heavily influenced by Google's Borg system and many of the top contributors to the project previously worked on Borg. 

The original codename for Kubernetes within Google was Project 7, a reference to the Star Trek ex-Borg character Seven of Nine.The seven spokes on the wheel of the Kubernetes logo are a reference to that codename. 

The original Borg project was written entirely in C++, but the rewritten Kubernetes system is implemented in Go.

|||
|-|-|
|2003-2004: Birth of the Borg System|Borg was a large-scale internal cluster management system, which ran hundreds of thousands of jobs, from many thousands of different applications, across many clusters, each with up to tens of thousands of machines.|
|2013: From Borg to Omega|Following Borg, Google introduced the Omega cluster management system, a flexible, scalable scheduler for large compute clusters.|
|2014: Google Introduces Kubernetes|Google introduced Kubernetes as an open source version of Borg|
|2015: The year of Kube v1.0 & CNCF|Kubernetes v1.0 gets released. Along with the release, Google partnered with the Linux Foundation to form the Cloud Native Computing Foundation (CNCF). |

<b>Revision History of Kubernetes </b>

<img src=".\images\p1_kubernetes_versions.jpg"/>

### Future of Kubernetes

Nowadays, there is a growing excitement about ‘serverless’ technologies,and Lambda-based architectures, and some would argue that Kubernetes is moving in the opposite direction.

However, Kubernetes has it’s place in ‘increasingly serverless’ world. Kubernetes has the opportunity to be the servers of the serverless world. Tools like Kubeless and Fission providing equivalents to functions-as-a-service but running within Kubernetes. 

Kubernetes will be everywhere in coming years and more exciting technologies like mesh networks, multi-region Clusters, multi-cloud Clusters, serverless reimagined will be built on top of it.

## Why do we need Kubernetes and what it can do? 

Development and deployment of applications has changed in recent years. This change is both a consequence of splitting big monolithic apps into smaller microservices and of the changes in the infrastructure that runs those apps.

<b>Moving from monolithic apps to microservices</b>

The problems in monolithic applications have forced the community to start splitting complex monolithic applications into smaller independently deployable components called <b>microservices</b>. Each microservice runs as an independent process and communicates with other microservices through simple, well-defined interfaces (APIs).

<img src=".\images\p1_monolith_to_distributed.jpg"/>

Microservices allows to horizontally scale the parts that allow scaling out, and scale the parts that don’t, vertically instead of horizontally.

<img src=".\images\p1_traditional_scaling_microservice.jpg"/>

Microservices also have drawbacks. When your system consists of only a small number of deployable components, managing those components is easy. It’s trivial to decide where to deploy each component, because there aren’t that many choices. When the number of those components increases, deployment-related decisions become increasingly difficult because not only does the number of deployment combinations increase, but the number of inter-dependencies between the components increases by an even greater factor.

Microservices perform their work together as a team, so they need to find and talk to each other. When deploying them, someone or something needs to configure all of them properly to enable them to work together as a single system. With increasing numbers of microservices, this becomes tedious and error-prone, especially when you consider what the ops/sysadmin teams need to do when a server fails.

<b>There is no silver bullet.</b>

Solutions follows: 
* Providing a consistent and isolated environment to each application
* Moving to continuous delivery (DevOps)

<img src=".\images\p1_kubernetes_container_evolution.jpg"/>

Kubernetes provides you with a framework to run distributed systems resiliently. It takes care of scaling and failover for your application, provides deployment patterns, and more.

Kubernetes provides you with:

* <b>Service discovery and load balancing:</b> Kubernetes can expose a container using the DNS name or using their own IP address. If traffic to a container is high, Kubernetes is able to load balance and distribute the network traffic so that the deployment is stable.

* <b>Storage orchestration:</b> Kubernetes allows you to automatically mount a storage system of your choice, such as local storages, public cloud providers, and more.

* <b>Automated rollouts and rollbacks:</b> You can describe the desired state for your deployed containers using Kubernetes, and it can change the actual state to the desired state at a controlled rate. For example, you can automate Kubernetes to create new containers for your deployment, remove existing containers and adopt all their resources to the new container.

* <b>Automatic bin packing:</b> You provide Kubernetes with a cluster of nodes that it can use to run containerized tasks. You tell Kubernetes how much CPU and memory (RAM) each container needs. Kubernetes can fit containers onto your nodes to make the best use of your resources.

* <b>Self-healing:</b> Kubernetes restarts containers that fail, replaces containers, kills containers that don't respond to your user-defined health check, and doesn't advertise them to clients until they are ready to serve.

* <b>Secret and configuration:</b> management Kubernetes lets you store and manage sensitive information, such as passwords, OAuth tokens, and SSH keys. You can deploy and update secrets and application configuration without rebuilding your container images, and without exposing secrets in your stack configuration.

Kubernetes enables you to run your software applications on thousands of computer nodes as if all those nodes were a single, enormous computer. It abstracts away the underlying infrastructure and, by doing so, simplifies development, deployment, and management for both development and the operations teams.

<img src=".\images\p1_kubernetes_userview.jpg"/>

    Helps dev teams focus on the core application features
    Helps ops teams achieve better resource utilization

Deploying applications through Kubernetes is always the same, whether your cluster contains only a couple of nodes or thousands of them. The size of the cluster makes no difference at all. Additional cluster nodes simply represent an additional amount of resources available to deployed apps.

Kubernetes will run your containerized app somewhere in the cluster, provide information to its components on how to find each other, and keep all of them running. Because your application doesn’t care which node it’s running on, Kubernetes can relocate the app at any time, and by mixing and matching apps, achieve far better resource utilization than is possible with manual scheduling.

## What Kubernetes is not

Kubernetes is not a traditional, all-inclusive PaaS (Platform as a Service) system. 

* Does not limit the types of applications supported. Kubernetes aims to support an extremely diverse variety of workloads, including stateless, stateful, and data-processing workloads. If an application can run in a container, it should run great on Kubernetes.

* Does not deploy source code and does not build your application. 

* Does not provide application-level services, such as middleware (for example, message buses), data-processing frameworks (for example, Spark), databases (for example, MySQL), caches, nor cluster storage systems (for example, Ceph) as built-in services. 

* Does not dictate logging, monitoring, or alerting solutions. 

* Does not provide nor mandate a configuration language/system (for example, Jsonnet). It provides a declarative API that may be targeted by arbitrary forms of declarative specifications.

* Does not provide nor adopt any comprehensive machine configuration, maintenance, management, or self-healing systems.

* Kubernetes is not a mere orchestration system. 

## Running an Application in Kubernetes

To run an application in Kubernetes:
* Package it up into one or more container images
* Push the images to an image registry
* Post a description of your app to the Kubernetes API server

The description includes information about: 
* Container image or images that contain your application components, 
* The relationship between the components, 
* Number of copies (replicas) to be run 
* Whether the components provide a service to either internal or external clients 
* Should be exposed through a single IP address and made discoverable to the other components.

### How it works

<img src=".\images\p1_kubernetes_working_logic.jpg"/>

When the API server processes your app’s description, the Scheduler schedules the specified groups of containers onto the available worker nodes based on computational resources required by each group and the unallocated resources on each node at that moment. The Kubelet on those nodes then instructs the Container Runtime (Docker, for example) to pull the required container images and run the containers.

Once the application is running, Kubernetes continuously makes sure that the deployed state of the application always matches the description you provided. 
    
    For example, if you specify that you always want five instances of a web server running, Kubernetes will always keep exactly five instances running. If one of those instances stops working properly, like when its process crashes or when it stops responding, Kubernetes will restart it automatically.

Similarly, if a whole worker node dies or becomes inaccessible, Kubernetes will select new nodes for all the containers that were running on the node and run them on the newly selected nodes.  

### Scaling the number of copies

While the application is running, you can decide you want to increase or decrease the number of copies, and Kubernetes will spin up additional ones or stop the excess ones, respectively. 

You can even leave the job of deciding the optimal number of copies to Kubernetes. It can automatically keep adjusting the number, based on real-time metrics, such as CPU load, memory consumption, queries per second, or any other metric your app exposes.

## Pros & Cons

Pros:
* Simplify application development
* Achieve better utilization of hardware
* Health checking and self-healing
* Automatic scaling

Cons:
* Can be an overkill for simple applications
* Complex and can reduce productivity
* Transition to Kubernetes can be cumbersome

If done right, investing the time to learn and adopt Kubernetes will often pay off in the future due to better service quality, a higher productivity level and a more motivated workforce.

# PART II - Kubernetes Components

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

# PART III - Kubernetes Objects

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


## Pod

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
Pods are usually created by posting a JSON or YAML manifest to the Kubernetes REST API endpoint. 

The pod definition consists following main parts:
* apiVersion: Kubernetes API version used in the YAML
* kind: the type of resource the YAML is describing
* metadata: Includes the name, namespace, labels, and other information about the pod.
* spec: Contains the actual description of the pod’s contents, such as the pod’s containers, volumes, and other data.
* <i>status: contains the current information about the running pod, such as what condition the pod is in, the description and status of each container, and the pod’s internal IP and other basic info.</i>

    The status part contains read-only runtime data that shows the state of the resource at a given moment. When creating a new pod, you never need to provide the status part.

<img src=".\images\p3_pod_example_yaml.jpg"/>

### Organizing Pods with Labels
With microservices architectures, the number of deployed microservices can easily reach high values. Those components will probably be replicated (multiple copies of the same component will be deployed) and multiple versions or releases (stable, beta, canary, and so on) will run concurrently. This can lead to hundreds of pods in the system. Without a mechanism for organizing them, you end up with a big, incomprehensible mess.

<img src=".\images\p3_uncategorized_pods_example.jpg"/>

Organizing pods and all other Kubernetes objects is done through labels.

<img src=".\images\p3_categorized_pods_example.jpg"/>

Labels go hand in hand with label selectors. Label selectors allow you to select a subset of pods tagged with certain labels and perform an operation on those pods. A label selector is a criterion, which filters resources based on whether they include a certain label with a certain value.

### Using labels and selectors to constrain pod scheduling

Labels and label selectors can be used to constrain pod scheduling. As an example, let one of the nodes in your cluster contains a GPU to be used for general-purpose GPU computing. You add the label gpu=true to the nodes showing this feature.

Now imagine you want to deploy a new pod that needs a GPU to perform its work. To ask the scheduler to only choose among the nodes that provide a GPU, you’ll add a node selector to the pod’s YAML.

<img src=".\images\p3_nodeselector_example_yaml.jpg"/>

### Removing Pods

Delete pod by name: <i>kubectl delete po pod-name</i>

Deleting pods using label selectors: <i>kubectl delete po -l label=value</i>

Deleting all pods in a namespace :<i>kubectl delete po --all</i>

Deleting pods by deleting the whole namespace: <i>kubectl delete ns custom_namespace</i>


## ReplicationController

In real-world use cases, you want your deployments to stay up and running automatically and remain healthy without any manual intervention. To do this, you almost never create pods directly. Instead, you create other types of resources to create and manage the actual pods.

A ReplicationController is a Kubernetes resource that ensures its pods are always kept running. If the pod disappears for any reason, such as in the event of a node disappearing from the cluster or because the pod was evicted from the node, the ReplicationController notices the missing pod and creates a replacement pod.

<img src=".\images\p3_replicationcontroller_createpod.jpg"/>

A ReplicationController constantly monitors the list of running pods and makes sure the actual number of pods of a “type” always matches the desired number. If too few such pods are running, it creates new replicas from a pod template. If too many such pods are running, it removes the excess replicas.

Replication-Controllers operate on sets of pods that match a certain label selector.

<img src=".\images\p3_replicationcontroller_reconciliation_loop.jpg"/>

ReplicationControllerprovides the following features:

* It makes sure a pod (or multiple pod replicas) is always running by starting a new pod when an existing one goes missing.
* When a cluster node fails, it creates replacement replicas for the pods that were under the Replication-Controller’s control and running on the failed node.
* It enables horizontal scaling of pods—both manual and automatic.

    > A pod instance is never relocated to another node. Instead, the Replication-Controller creates a completely new pod instance that has no relation to the instance it’s replacing.

### Creating a ReplicationController

ReplicationController is created by posting a JSON or YAML descriptor to the Kubernetes API.server.

<img src=".\images\p3_replicationcontroller_yaml_example.jpg"/>

> Don’t specify a pod selector when defining a ReplicationController. Let Kubernetes extract it from the pod template. This will keep your YAML shorter and simpler.

### Responding Pod/Node Failures

The controller responds to the deletion of a pod by creating a new replacement pod. Technically, it isn’t responding to the deletion itself, but the resulting state—the inadequate number of pods.

In the non-Kubernetes world, If a node fails, the ops team would need to migrate the applications running on that node to other machines manually. Kubernetes, on the other hand, does that automatically. Soon after the ReplicationController detects that its pods are down, it will spin up new pods to replace them.

### Moving pods in and out of the scope of a ReplicationController

ReplicationController manages pods that match its label selector. By changing a pod’s labels, it can be removed from or added to the scope of a ReplicationController. It can even be moved from one ReplicationController to another.

You need to either remove matching label(s) or change its value to move the pod out of the ReplicationController’s scope. Adding another label will have no effect, because the ReplicationController doesn’t care if the pod has any additional labels. It only cares whether the pod has all the labels referenced in the label selector.

<img src=".\images\p3_replicationcontroller_change_pod_label.jpg"/>

> Removing a pod from the scope of the ReplicationController comes in handy when you want to perform actions on a specific pod. For example, you might have a bug that causes your pod to start behaving badly after a specific amount of time or a specific event. If you know a pod is malfunctioning, you can take it out of the ReplicationController’s scope, let the controller replace it with a new one, and then debug or play with the pod in any way you want. Once you’re done, you delete the pod.

> What happens if we change label-selector of ReplicationController? 
> 
> It would make all the pods fall out of the scope of the ReplicationController and create three new pods.

> What happends if we change pod template?
>
> A ReplicationController’s pod template can be modified at any time. Changing the pod template only affect the pods created afterwards. To modify the old pods, you’d need to delete them and let the Replication-Controller replace them with new ones based on the new template.

<img src=".\images\p3_replicationcontroller_change_pod_template.jpg"/>

### Horizontally scaling pods
Scaling the number of pods up or down is executed by changing the value of the replicas field in the ReplicationController resource. After the change, the Replication-Controller will either delete some existing pods (when scaling down) or create additional pods (when scaling up).

* Edit yaml file and apply new configuration: kubectl apply -f replication-set.yaml

* kubectl scale rc kubia --replicas=6

### Deleting a ReplicationController

When you delete a ReplicationController through kubectl delete, the pods are also deleted. 

 pods created by a ReplicationController aren’t an integral part of the ReplicationController, and are only managed by it, you can delete only the ReplicationController and leave the pods running by using --cascade=false flag.

 ## ReplicaSets
 Initially, ReplicationControllers were the only Kubernetes component for replicating pods and rescheduling them when nodes failed. Later, a similar resource called a ReplicaSet was introduced. It’s a new generation of ReplicationController and replaces it completely (ReplicationControllers will eventually be deprecated).

 A ReplicaSet behaves exactly like a ReplicationController, but it has more expressive pod selectors. Whereas a ReplicationController’s label selector only allows matching pods that include a certain label, a ReplicaSet’s selector also allows matching pods that lack a certain label or pods that include a certain label key, regardless of its value.
> For example, a single ReplicationController can’t match pods with the label env=production and those with the label env=devel at the same time. It can only match either pods with the env=production label or pods with the env=devel label. But a single ReplicaSet can match both sets of pods and treat them as a single group.

Similarly, a ReplicationController can’t match pods based merely on the presence of a label key, regardless of its value, whereas a ReplicaSet can. 

> For example, a ReplicaSet can match all pods that include a label with the key env, whatever its actual value is (you can think of it as env=*).

## DeamonSets

Certain cases exist when you want a pod to run on each and every node in the cluster and each node needs to run exactly one instance of the pod.

<img src=".\images\p3_deamonsets.jpg"/>

Those cases include infrastructure-related pods that perform system-level operations. For example, you’ll want to run a log collector and a resource monitor on every node. Another good example is Kubernetes’ own kube-proxy process, which needs to run on all nodes to make services work.

Pods created by a DaemonSet have a target node specified and skip the Kubernetes Scheduler. They aren’t scattered around the cluster randomly. A DaemonSet makes sure it creates as many pods as there are nodes and deploys each one on its own node.

If a node goes down, the DaemonSet doesn’t cause the pod to be created elsewhere. But when a new node is added to the cluster, the DaemonSet immediately deploys a new pod instance to it. It also does the same if someone inadvertently deletes one of the pods, leaving the node without the DaemonSet’s pod.

A DaemonSet deploys pods to all nodes in the cluster, unless you specify that the pods should only run on a subset of all the nodes. This is done by specifying the node-Selector property in the pod template.

## Jobs
Kubernetes includes support for creation of pods that terminates after process running inside finishes successfully and not restarted again.

In the event of a node failure, the pods on that node that are managed by a Job will be rescheduled to other nodes.

In the event of a failure of the process itself (when the process returns an error exit code), the Job can be configured to either restart the container or not.

* Run multiple pod instances in a Job
    * Run job pods sequentially
    * Run job pods in parallel
    * Scale a Job
* Limit the time allowed for a Job pod to complete    

## CronJobs
Job resources run their pods immediately when you create the Job resource. But many batch jobs need to be run at a specific time in the future or repeatedly in the specified interval. In Linux- and UNIX-like operating systems, these jobs are better known as cron jobs. Kubernetes supports them, too.

A cron job in Kubernetes is configured by creating a CronJob resource. At the configured time, Kubernetes will create a Job resource according to the Job template configured in the CronJob object. When the Job resource is created, one or more pod replicas will be created and started according to the Job’s pod template. 

## Deployment

## Services

## ConfigMaps

## Secrets

## StatefulSets