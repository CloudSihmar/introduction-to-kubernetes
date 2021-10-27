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
