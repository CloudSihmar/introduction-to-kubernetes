# Introduction to Kubernetes

Editors: **Kaan Keskin, Sezen Erdem**

Date: November 2021

Available at: https://github.com/kaan-keskin/introduction-to-kubernetes

**Resources:**

> - Kubernetes in Action - Marko Lukša 
> - Kubernetes Documentation - https://kubernetes.io/docs/home/

**LEGAL NOTICE: This document is created for educational purposes, and it can not be used for any commercial purposes. If you find this document useful in any means please support the original authors for ethical reasons.** 

[Return README page.](README.md)

# Kubernetes Objects

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

ReplicationController is created by posting a JSON or YAML descriptor to the Kubernetes API server.

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

## Services
Pods need a way of finding other pods if they want to consume the services they provide. Configuring each client application by specifying the exact IP address or hostname of the server providing the service in the client’s configuration files will not work in Kubernetes because:

* Pods are ephemeral—They may come and go at any time
* Kubernetes assigns an IP address to a pod after the pod has been scheduled to a node and before it’s started—Clients thus can’t know the IP address of the server pod up front.
* Horizontal scaling means multiple pods may provide the same service—Each of those pods has its own IP address. 

A Kubernetes Service is a resource created to make a single, constant point of entry to a group of pods providing the same service. 

Each service has an IP address and port that never change while the service exists. Clients can open connections to that IP and port, and those connections are then routed to one of the pods backing that service. 

Clients of a service don’t know the location of individual pods providing the service, allowing those pods to be moved around the cluster at any time.

<img src=".\images\p3_KubernetesService.jpg"/>

A service can be backed by more than one pod. Connections to the service are load-balanced across all the backing pods. 

Label selectors determine which pods belong to the Service.

<img src=".\images\p3_service_labelselector.jpg"/>

### Creating Services
Service is created by posting a JSON or YAML descriptor to the Kubernetes API server.

<img src=".\images\p3_service_yamlexample.jpg"/>

### Exposing multiple ports in the same service

Services can support multiple ports. For example, if your pods listened on two ports, 8080 for HTTP and 8443 for HTTPS. You could use a single service to forward both port 80 and 443 to the pod’s ports 8080 and 8443. You don’t need to create two different services in such cases. Using a single, multi-port service exposes all the service’s ports through a single cluster IP.

    When creating a service with multiple ports, you must specify a name for each port.

<img src=".\images\p3_service_multipleport.jpg"/>

    The label selector applies to the service as a whole—it can’t be configured for each port individually. If you want different ports to map to different subsets of pods, you need to create two services.

You can give a name to each pod’s port and refer to it by name in the service spec. The biggest benefit of doing so is that it enables you to change port numbers later without having to change the service spec. 

<img src=".\images\p3_service_namedports_pod.jpg"/>

<img src=".\images\p3_service_namedports_service.jpg"/>

### Discovering Services

By creating a service, you now have a single and stable IP address and port that you can hit to access your pods. This address will remain unchanged throughout the whole lifetime of the service. Pods behind this service may come and go, their IPs may change, their number can go up or down, but they’ll always be accessible through the service’s single and constant IP address.

But how do the client pods know the IP and port of a service?

* Discover services through environment variables: When a pod is started, Kubernetes initializes a set of environment variables pointing to each service that exists at that moment. If you create the service before creating the client pods, processes in those pods can get the IP address and port of the service by inspecting their environment variables.

* Discovering services through DNS: The kube-system namespace includes a pod for DNS and a corresponding service with the name kube-dns. The pod runs a DNS server, which all other pods running in the cluster are automatically configured to use (Kubernetes does that by modifying each container’s /etc/resolv.conf file). Any DNS query performed by a process running in a pod will be handled by Kubernetes’ own DNS server, which knows all the services running in your system. Each service gets a DNS entry in the internal DNS server, and client pods that know the name of the service can access it through its fully qualified domain name (FQDN) instead of resorting to environment variables.

### Connecting to services living outside the cluster

Services don’t link to pods directly. Instead, a resource sits in between—the Endpoints resource. An Endpoints resource is a list of IP addresses and ports exposing a service. Pod selector defined in the service spec is not used directly when redirecting incoming connections. Instead, the selector is used to build a list of IPs and ports, which is then stored in the Endpoints resource. When a client connects to a service, the service proxy selects one of those IP and port pairs and redirects the incoming connection to the server listening at that location. 

### Exposing services to external clients

Services can be made accessible externally by:

* NodePort service type
* LoadBalancer service type
* Creating an Ingress resource

<b>Using a NodePort service</b>

By creating a NodePort service, you make Kubernetes reserve a port on all its nodes (the same port number is used across all of them) and forward incoming connections to the pods that are part of the service.

<img src=".\images\p3_service_nodeport_yaml.jpg"/>

NodePort service can be accessed not only through the service’s internal cluster IP, but also through any node’s IP and the reserved node port.

<img src=".\images\service_nodeport_access.jpg"/>

<b>Exposing a service through an external load balancer</b>

The service type is set to LoadBalancer. This type of service obtains a load balancer from the infrastructure hosting the Kubernetes cluster. The load balancer will have its own unique, publicly accessible IP address and will redirect all connections to your service. You can thus access your service through the load balancer’s IP address.

<img src=".\images\p3_service_loadbalancer_yaml.jpg"/>

<img src=".\images\p3_service_loadbalancer_access.jpg"/>

<b>Exposing services externally through an Ingress resource</b>

Each LoadBalancer service requires its own load balancer with its own public IP address, whereas an Ingress only requires one, even when providing access to dozens of services.

<img src=".\images\p3_service_ingress.jpg"/>

To make Ingress resources work, an Ingress controller needs to be running in the cluster.

The client first performed a DNS lookup of kubia.example.com, and the DNS server (or the local operating system) returned the IP of the Ingress controller. The client then sent an HTTP request to the Ingress controller and specified kubia.example.com in the Host header. From that header, the controller determined which service the client is trying to access, looked up the pod IPs through the Endpoints object associated with the service, and forwarded the client’s request to one of the pods.

<img src=".\images\p3_service_ingress_howitworks.jpg"/>

You can map multiple paths on the same host to different services.

You can use an Ingress to map to different services based on the host in the HTTP request instead of (only) the path

### Pod Readiness Probe
Pods are included as endpoints of a service if their labels match the service’s pod selector. As soon as a new pod with proper labels is created, it becomes part of the service and requests start to be redirected to the pod. But what if the pod isn’t ready to start serving requests immediately?

The pod may need time to load either configuration or data, or it may need to perform a warm-up procedure to prevent the first user request from taking too long and affecting the user experience. In such cases you don’t want the pod to start receiving requests immediately, especially when the already-running instances can process requests properly and quickly. It makes sense to not forward requests to a pod that’s in the process of starting up until it’s fully ready.

Kubernetes allows you to also define a readiness probe for your pod. The readiness probe is invoked periodically and determines whether the specific pod should receive client requests or not. When a container’s readiness probe returns success, it’s signaling that the container is ready to accept requests. A pod whose readiness probe fails is removed as an endpoint of a service.

Three types of readiness probes exist:

* An Exec probe, where a process is executed. The container’s status is determined by the process’ exit status code.
* An HTTP GET probe, which sends an HTTP GET request to the container and the HTTP status code of the response determines whether the container is ready or not.
* A TCP Socket probe, which opens a TCP connection to a specified port of the container. If the connection is established, the container is considered ready.

### Headless Services

Services can be used to provide a stable IP address allowing clients to connect to pods backing the services. Each connection to the service is forwarded to one randomly selected backing pod. But what if the client needs to connect to all of those pods? What if the backing pods themselves need to each connect to all the other backing pods? Connecting through the service clearly isn’t the way to do this.

For a client to connect to all pods, it needs to figure out the the IP of each individual pod. One option is to have the client call the Kubernetes API server and get the list of pods and their IP addresses through an API call, but because you should always strive to keep your apps Kubernetes-agnostic, using the API server isn’t ideal.

When you perform a DNS lookup for a service, the DNS server returns a single IP—the service’s cluster IP. But if you tell Kubernetes you don’t need a cluster IP for your service (you do this by setting the clusterIP field to None in the service specification), the DNS server will return the pod IPs instead of the single service IP.

### Troubleshooting services

When you’re unable to access your pods through the service, you should start by going through the following list:

* First, make sure you’re connecting to the service’s cluster IP from within the cluster, not from the outside.
* Don’t bother pinging the service IP to figure out if the service is accessible (remember, the service’s cluster IP is a virtual IP and pinging it will never work).
* If you’ve defined a readiness probe, make sure it’s succeeding; otherwise the pod won’t be part of the service.
* To confirm that a pod is part of the service, examine the corresponding Endpoints object with kubectl get endpoints.
* If you’re trying to access the service through its FQDN or a part of it (for example, myservice.mynamespace.svc.cluster.local or myservice.mynamespace) and it doesn’t work, see if you can access it using its cluster IP instead of the FQDN.
* Check whether you’re connecting to the port exposed by the service and not the target port.
* Try connecting to the pod IP directly to confirm your pod is accepting connections on the correct port.
* If you can’t even access your app through the pod’s IP, make sure your app isn’t only binding to localhost.

## ConfigMaps

ConfigMap is a map containing key/value pairs with the values ranging from short literals to full config files. The whole point of an app’s configuration is to keep the config options that vary between environments, or change frequently, separate from the application’s source code. 

<img src=".\images\p3_configmaps.jpg"/>

An application doesn’t need to read the ConfigMap directly or even know that it exists. The contents of the map are passed to containers as either environment variables or as files in a volume.

### Creating ConfigMaps

* By passing literals to the kubectl command

        kubectl create configmap myconfigmap --from-literal=foo=bar --from-literal=bar=baz --from-literal=one=two
* By defining in a yaml file
* From individual files or a directory

<img src=".\images\p3_configmaps_createfromdirectory.jpg"/>


## Deployment
The basic outline of an application running on Kubernetes is as follows.

<img src=".\images\p3_deployments_general_architecture.jpg"/>

The pods are backed by a ReplicationController or a ReplicaSet. A Service exists through which apps running in other pods or external clients access the pods. 

Initially, the pods run the first version of your application—let’s suppose its image is tagged as v1. You then develop a newer version of the app and push it to an image repository as a new image, tagged as v2. You’d next like to replace all the pods with this new version. 

You have two ways of updating all those pods:

* If you have a ReplicationController managing a set of v1 pods,  replace them by modifying the pod template so it refers to version v2 of the image. Then delete the old pod instances. The ReplicationController will notice that no pods match its label selector and it will spin up new instances.  

<img src=".\images\p3_deployments_update_pods_v1.jpg"/>

* Blue-Green Deployment: Start new pods and once they’re up, delete the old ones. 

    You can do this either by adding all the new pods and then deleting all the old ones at once, or sequentially, by adding new pods and removing old ones gradually.


<img src=".\images\p3_deployments_blue_green_update_v1.jpg"/>

<img src=".\images\p3_deployments_rolling_update.jpg"/>

A Deployment is a higher-level resource meant for deploying applications and updating them declaratively, instead of doing it through a ReplicationController or a ReplicaSet, which are both considered lower-level concepts.

When you create a Deployment, a ReplicaSet resource is created underneath. Replica-Sets replicate and manage pods. When using a Deployment, the actual pods are created and managed by the Deployment’s ReplicaSets, not by the Deployment directly.

### Creating a Deployment

A Deployment is composed of a label selector, a desired replica count, a pod template and a deployment strategy that defines how an update should be performed when the Deployment resource is modified.

### Updating a Deployment

The only thing you need to do is modify the pod template defined in the Deployment resource and Kubernetes will take all the steps necessary to get the actual system state to what’s defined in the resource. Similar to scaling a ReplicationController or ReplicaSet up or down, all you need to do is reference a new image tag in the Deployment’s pod template and leave it to Kubernetes to transform your system so it matches the new desired state.

<b>Deployment Strategies</b>

* RollingUpdate: Default strategy. The RollingUpdate strategy removes old pods one by one, while adding new ones at the same time, keeping the application available throughout the whole process, and ensuring there’s no drop in its capacity to handle requests
* Recreate: The Recreate strategy causes all old pods to be deleted before the new ones are created. Use this strategy when your application doesn’t support running multiple versions in parallel and requires the old version to be stopped completely before the new one is started. This strategy does involve a short period of time when your app becomes completely unavailable.

### Rolling back a deployment

Deployments make it easy to roll back to the previously deployed version by telling Kubernetes to undo the last rollout of a Deployment. You can roll back to a specific revision by specifying the revision in the undo command.

Deployment ensures that only a certain number of Pods are down while they are being updated. By default, it ensures that at least 75% of the desired number of Pods are up (25% max unavailable).

Deployment also ensures that only a certain number of Pods are created above the desired number of Pods. By default, it ensures that at most 125% of the desired number of Pods are up (25% max surge).

<img src=".\images\p3_deployments_maxsurge_maxunavailable.jpg"/>

### Scaling a Deployment

You can scale a Deployment by changing replica count. 

    kubectl scale deployment.v1.apps/nginx-deployment --replicas=10

You can setup an autoscaler for your Deployment and choose the minimum and maximum number of Pods you want to run based on the CPU utilization of your existing Pods.

    kubectl autoscale deployment.v1.apps/nginx-deployment --min=10 --max=15 --cpu-percent=80


### Deployment status

A Deployment enters various states during its lifecycle.
* Progressing:
Kubernetes marks a Deployment as progressing when one of the following tasks is performed:
    * The Deployment creates a new ReplicaSet.
    * The Deployment is scaling up its newest ReplicaSet.
    * The Deployment is scaling down its older ReplicaSet(s).
    * New Pods become ready or available (ready for at least MinReadySeconds).

* Complete: Kubernetes marks a Deployment as complete when it has the following characteristics:
    * All of the replicas associated with the Deployment have been updated to the latest version you've specified, meaning any updates you've requested have been completed.
    * All of the replicas associated with the Deployment are available.
    * No old replicas for the Deployment are running.

* Failed: Your Deployment may get stuck trying to deploy its newest ReplicaSet without ever completing. This can occur due to some of the following factors:
    * Insufficient quota
    * Readiness probe failures
    * Image pull errors
    * Insufficient permissions
    * Limit ranges
    * Application runtime misconfiguration

## Secrets

## StatefulSets