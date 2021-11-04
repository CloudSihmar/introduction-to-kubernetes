## Kubernetes Components

Kubernetes cluster is split into two parts:

* The Kubernetes Control Plane
    * etcd
    * API server
    * Controller manager
    * Scheduler
* The worker nodes
    * kubelet
    * Container run time
    * kube-proxy
* Add-on components
    * The Kubernetes DNS server
    * The Dashboard
    * An Ingress controller
    * Heapster
    * The Container Network Interface network plugin

    <img src=".\images\Kubernetes_Architecture_Simplified.jpg"/>

    Kubernetes system components communicate only with the API server. They don’t talk to each other directly. The API server is the only component that communicates with etcd. None of the other components communicate with etcd directly, but instead modify the cluster state by talking to the API server.

    The components on the worker nodes all need to run on the same node, the components of the Control Plane can easily be split across multiple servers. There can be more than one instance of each Control Plane component running to ensure high availability. While multiple instances of etcd and API server can be active at the same time and do perform their jobs in parallel, only a single instance of the Scheduler and the Controller Manager may be active at a given time—with the others in standby mode.

    ### What API server does

    The Kubernetes API server is the central component used by all other components and by clients, such as kubectl. It provides a CRUD (Create, Read, Update, Delete) interface for querying and modifying the cluster state over a RESTful API. It stores that state in etcd.

    In addition to providing a consistent way of storing objects in etcd, it also performs validation of those objects, so clients can’t store improperly configured objects 

    ### Understanding the Scheduler
    Scheduler wait for newly created pods through the API server’s watch mechanism and assign a node to each new pod that doesn’t already have the node set.

    All the Scheduler does is update the pod definition through the API server. The API server then notifies the Kubelet (through the watch mechanism) that the pod has been scheduled. As soon as the Kubelet on the target node sees the pod has been scheduled to its node, it creates and runs the pod’s containers.

    ### Controllers running inside Controller Manager

    The single Controller Manager process currently combines a multitude of controllers performing various reconciliation tasks.

    The list of these controllers includes the

    * Replication Manager (a controller for ReplicationController resources)
    * ReplicaSet, DaemonSet, and Job controllers
    * Deployment controller
    * StatefulSet controller
    * Node controller
    * Service controller
    * Endpoints controller
    * Namespace controller
    * PersistentVolume controller
    * Others

    ### What the Kubelet does
    The Kubelet is the component responsible for everything running on a worker node. Its initial job is to register the node it’s running on by creating a Node resource in the API server. Then it needs to continuously monitor the API server for Pods that have been scheduled to the node, and start the pod’s containers. It does this by telling the configured container runtime (which is Docker, CoreOS’ rkt, or something else) to run a container from a specific container image. The Kubelet then constantly monitors running containers and reports their status, events, and resource consumption to the API server.
