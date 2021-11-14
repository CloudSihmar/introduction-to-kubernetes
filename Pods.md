# Introduction to Kubernetes

Editors: **Kaan Keskin, Sezen Erdem**

Date: November 2021

Available at: https://github.com/kaan-keskin/introduction-to-kubernetes

**Resources:**

> - Kubernetes Documentation - https://kubernetes.io/docs/home/
> - Kubernetes in Action - Marko Lukša - Manning Publications
> - Kubernetes Fundamentals (LFS258) - Timothy Serewicz - The Linux Foundation
> - Kubernetes for Developers (LFD259) - Timothy Serewicz - The Linux Foundation
> - Certified Kubernetes Application Developer (CKAD) Study Guide - Benjamin Muschko - O'Reilly Media
> - Getting Started with Kubernetes - Sander van Vugt - Addison-Wesley Professional

**LEGAL NOTICE: This document is created for educational purposes, and it can not be used for any commercial intentions. If you find this document useful in any means please support the original authors for ethical reasons.** 

[Return to the README page.](README.md)

# Pods

The whole point of Kubernetes is to orchestrate the lifecycle of a container. We do not interact with particular containers. Instead, the smallest unit we can work with is a Pod. Some would say a pod of whales or peas-in-a-pod. Due to shared resources, the design of a Pod typically follows a one-process-per-container architecture.

Containers in a Pod are started in parallel. As a result, there is no way to determine which container becomes available first inside a pod. The use of InitContainers can order startup, to some extent. To support a single process running in a container, you may need logging, a proxy, or special adapter. These tasks are often handled by other containers in the same pod.

There is only one IP address per Pod, for almost every network plugin. If there is more than one container in a pod, they must share the IP. To communicate with each other, they can either use IPC, the loopback interface, or a shared filesystem.

While Pods are often deployed with one application container in each, a common reason to have multiple containers in a Pod is for logging. You may find the term sidecar for a container dedicated to performing a helper task, like handling logs and responding to requests, as the primary application container may not have this ability. The term sidecar, like ambassador and adapter, does not have a special setting, but refers to the concept of what secondary pods are included to do.

Pods are the smallest deployable units of computing that you can create and manage in Kubernetes.

A Pod is a group of one or more containers, with shared storage and network resources, and a specification for how to run the containers.

CRI interaction with Docker images:

<img src=".\images\CRI-interaction-with-Docker-images.png"/>

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

## Pod Life Cycle Phases

Because Kubernetes is a state engine with asynchronous control loops, it’s possible that the status of the Pod doesn’t show a Running status right away when listing the Pods. It usually takes a couple of seconds to retrieve the image and start the container. Upon Pod creation, the object goes through several life cycle phases.

<img src=".\images\Pod-Life-cycle-Phases.png"/>

Understanding the implications of each phase is important as it gives you an idea about the operational status of a Pod.

- **Pending**: The Pod has been accepted by the Kubernetes system, but one or more of the container images has not been created.
- **Running**: At least one container is still running, or is in the process of starting or restarting.
- **Succeeded**: All containers in the Pod terminated successfully.
- **Failed**: Containers in the Pod terminated, as least one failed with an error.
- **Unknown**: The state of Pod could not be obtained.

## Creating Pods

Pods and other objects can be created in several ways. They can be created by using a generator, which. historically, has changed with each release. 

The run command is the central entry point for creating Pods imperatively. 

```shell
$ kubectl run newpod --image=nginx --generator=run-pod/v1
```

They can be created and deleted using properly formatted JSON or YAML files:

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

## Containers

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

## Rendering Pod Details

The rendered table produced by the get command provides high-level information about a Pod. But what if you needed to have a deeper look at the details? The describe command can help:

```shell
$ kubectl describe pods hazelcast
```

The terminal output contains the metadata information of a Pod, the containers it runs and the event log, such as failures when the Pod was scheduled. You can expect the output to be very lengthy.

There’s a way to be more specific about the information you want to render. You can combine the **describe** command with a Unix **grep** command. Say you wanted to identify the image for running in the container:

```shell
$ kubectl describe pods hazelcast | grep Image:
```

## Accessing Logs of a Pod

As application developers, we know very well what to expect in the log files produced by the application we implemented. Runtime failures may occur when operating an application in a container. The logs command downloads the log output of a container. The following output indicates that the Hazelcast server started up successfully:

```shell
$ kubectl logs hazelcast
```

It’s very likely that more log entries will be produced as soon as the container receives traffic from end users. **You can stream the logs with the command line option -f**. This option is helpful if you want to see logs in real time.

Kubernetes tries to restart a container under certain conditions, such as if the image cannot be resolved on the first try. **Upon a container restart, you’ll not have access to the logs of the previous container anymore; the logs command only renders the logs for the current container. However, you can still get back to the logs of the previous container by adding the -p command line option.** You may want to use the option to identify the root cause that triggered a container restart.

## Running Commands in a Container

Part of the testing may be to execute commands within the Pod. What commands are available depend on what was included in the base environment when the image was created. In keeping to a decoupled and lean design, it's possible that there is no shell, or that the Bourne shell is available instead of bash. After testing, you may want to revisit the build and add resources necessary for testing and production.

Use the -it options for an interactive shell instead of the command running without interaction or access.
If you have more than one container, declare which container:

```shell
$ kubectl exec -i​t <Pod-Name> 
```

There are situations that require you to log into the container and explore the file system. Maybe you want to inspect the configuration of your application or debug the current state of your application. You can use the exec command to open a shell in the container to explore it interactively, as follows:

```shell
$ kubectl exec -it hazelcast -- /bin/sh
```

Notice that you do not have to provide the resource type. This command only works for a Pod. **The two dashes (--) separate the exec command and its options from the command you want to run inside of the container.**

It’s also possible to just execute a single command inside of a container. Say you wanted to render the environment variables available to containers without having to be logged in. Just remove the interactive flag -it and provide the relevant command after the two dashes:

```shell
$ kubectl exec hazelcast -- env
```

## Organizing Pods with Labels

With microservices architectures, the number of deployed microservices can easily reach high values. Those components will probably be replicated (multiple copies of the same component will be deployed) and multiple versions or releases (stable, beta, canary, and so on) will run concurrently. This can lead to hundreds of pods in the system. Without a mechanism for organizing them, you end up with a big, incomprehensible mess.

<img src=".\images\p3_uncategorized_pods_example.jpg"/>

Organizing pods and all other Kubernetes objects is done through labels.

<img src=".\images\p3_categorized_pods_example.jpg"/>

Labels go hand in hand with label selectors. Label selectors allow you to select a subset of pods tagged with certain labels and perform an operation on those pods. A label selector is a criterion, which filters resources based on whether they include a certain label with a certain value.

## Using labels and selectors to constrain pod scheduling

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

## Multi-Container Pod

The idea of multiple containers in a Pod goes against the architectural idea of decoupling as much as possible. One could run an entire operating system inside a container, but would lose much of the granular scalability Kubernetes is capable of. But there are certain needs in which a second or third co-located container makes sense. By adding a second container, each container can still be optimized and developed independently, and both can scale and be repurposed to best meet the needs of the workload.

It may not make sense to recreate an entire image to add functionality like a shell or logging agent. Instead, you could add another container to the Pod, which would provide the necessary tools.

Each container in the Pod should be transient and decoupled. If adding another container limits the scalability or transient nature of the original application, then a new build may be warranted.

Every container in a Pod shares a single IP address and namespace. Each container has equal potential access to storage given to the Pod. Kubernetes does not provide any locking, so your configuration or application should be such that containers do not have conflicting writes.

One container could be read only while the other writes. You could configure the containers to write to different directories in the volume, or the application could have built in locking. Without these protections, there would be no way to order containers writing to the storage.

There are three terms often used for multi-container pods: ambassador, adapter, and sidecar. Based off of the documentation, you may think there is some setting for each. There is not. Each term is an expression of what a secondary pod is intended to do. All are just multi-container pods.

Especially to beginners of Kubernetes, how to appropriately design a Pod isn’t necessarily apparent. Upon reading the Kubernetes user documentation and tutorials on the internet, you’ll quickly find out that you can create a Pod that runs multiple containers at the same time. The question often arises, “Should I deploy my microservices stack to a single Pod with multiple containers, or should I create multiple Pods, each running a single microservice?” The short answer is to operate a single microservice per Pod. This modus operandi promotes a decentralized, decoupled, and distributed architecture. Furthermore, it helps with rolling out new versions of a microservice without necessarily interrupting other parts of the system.

So what’s the point of running multiple containers in a Pod then? There are two common use cases. Sometimes, you’ll want to initialize your Pod by executing setup scripts, commands, or any other kind of preconfiguration procedure before the application container should start. This logic runs in a so-called init container. Other times, you’ll want to provide helper functionality that runs alongside the application container to avoid the need to bake the logic into application code. For example, you may want to massage the log output produced by the application. Containers running helper logic are called sidecars.

## Multi-Container Pod Terminology

Real-world scenarios call for running multiple containers inside of a Pod. An init container helps with setting the stage for the main application container by executing initializing logic. Once the initialized logic has been processed, the container will be terminated. The main application container only starts if the init container ran through its functionality successfully.

**Kubernetes enables implementing software engineering best practices like separation of concerns and the single-responsibility principle. Cross-cutting concerns or helper functionality can be run in a so-called sidecar container. A sidecar container lives alongside the main application container within the same Pod and fulfills this exact role.**

The adapter pattern helps with “translating” data produced by the application so that it becomes consumable by third-party services. The ambassador pattern acts as a proxy for the application container when communicating with external services by abstracting the “how.”

### Init Containers

Init containers provide initialization logic concerns to be run before the main application even starts. 

Init containers are always started before the main application containers, which means they have their own lifecycle. To split up the initialization logic, you can even distribute the work into multiple init containers that are run in the order of definition in the manifest. Of course, initialization logic can fail. If an init container produces an error, the whole Pod is restarted, causing all init containers to run again in sequential order. Thus, to prevent any side effects, making init container logic idempotent is a good practice. Figure below shows a Pod with two init containers and the main application.

Sequential and atomic lifecycle of init containers in a Pod

<img src=".\images\sequential-and-atomic-lifecycle-init-containers-pod.png"/>

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

A Pod defining an init container:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: business-app
spec:
  initContainers:
  - name: configurer
    image: busybox:1.32.0
    command: ['sh', '-c', 'echo Configuring application... && \
              mkdir -p /usr/shared/app && echo -e "{\"dbConfig\": \
              {\"host\":\"localhost\",\"port\":5432,\"dbName\":\"customers\"}}" \
              > /usr/shared/app/config.json']
    volumeMounts:
    - name: configdir
      mountPath: "/usr/shared/app"
  containers:
  - image: bmuschko/nodejs-read-config:1.0.0
    name: web
    ports:
    - containerPort: 8080
    volumeMounts:
    - name: configdir
      mountPath: "/usr/shared/app"
  volumes:
  - name: configdir
    emptyDir: {}
```

When starting the Pod, you’ll see that the status column of the get command provides information on init containers as well. The prefix Init: signifies that an init container is in the process of being executed. The status portion after the colon character shows the number of init containers completed versus the overall number of init containers configured:

```shell
$ kubectl create -f init.yaml

pod/business-app created

$ kubectl get pod business-app

NAME           READY   STATUS    RESTARTS   AGE
business-app   0/1     Init:0/1  0          2s

$ kubectl get pod business-app

NAME           READY   STATUS    RESTARTS   AGE
business-app   1/1     Running   0          8s
```

Errors can occur during the execution of init containers. You can always retrieve the logs of an init container by using the **--container** command-line option (or -c in its short form).

<img src=".\images\pod-targeting-specific-container.png"/>

The following command renders the logs of the configurer init container, which equates to the echo command we configured in the YAML manifest:

```shell
$ kubectl logs business-app -c configurer

Configuring application...
```

### The Sidecar Pattern

The lifecycle of an init container looks as follows: it starts up, runs its logic, then terminates once the work has been done. Init containers are not meant to keep running over a longer period of time. There are scenarios that call for a different usage pattern. For example, you may want to create a Pod that runs multiple containers continuously alongside one another.

Typically, there are two different categories of containers: the container that runs the application and another container that provides helper functionality to the primary application. In the Kubernetes space, the container providing helper functionality is called a sidecar. Among the most commonly used capabilities of a sidecar container are file synchronization, logging, and watcher capabilities. The sidecars are not part of the main traffic or API of the primary application. They usually operate asynchronously and are not involved in the public API.

The idea for a sidecar container is to add some functionality not present in the main container. Rather than bloating code, which may not be necessary in other deployments, adding a container to handle a function such as logging solves the issue, while remaining decoupled and scalable. Prometheus monitoring and Fluentd logging leverage sidecar containers to collect data.

Similar to a sidecar on a motorcycle, it does not provide the main power, but it does help carry stuff. A sidecar is a secondary container which helps or provides a service not found in the primary application. Logging containers are a common sidecar.

To illustrate the behavior of a sidecar, we’ll consider the following use case. The main application container runs a web server—in this case, NGINX. Once started, the web server produces two standard logfiles. The file '/var/log/nginx/access.log' captures requests to the web server’s endpoint. The other file, '/var/log/nginx/error.log', records failures while processing incoming requests.

As part of the Pod’s functionality, we’ll want to implement a monitoring service. The sidecar container polls the file’s error.log periodically and checks if any failures have been discovered. More specifically, the service tries to find failures assigned to the error log level, indicated by [error] in the log file. If an error is found, the monitoring service will react to it. For example, it could send a notification to the administrators of the system. We’ll keep the functionality as simple as possible. The monitoring service will simply render an error message to standard output. The file exchange between the main application container and the sidecar container happens through a Volume.

<img src=".\images\pod-sidecar-pattern.png"/>

The YAML manifest shown below sets up the described scenario. The most tricky portion of the code is the lengthy bash command. The command runs an infinite loop. As part of each iteration, we inspect the contents of the file error.log, grep for an error and potentially act on it. The loop executes every 10 seconds.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webserver
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: logs-vol
      mountPath: /var/log/nginx
  - name: sidecar
    image: busybox
    command: ["sh","-c","while true; do if [ \"$(cat /var/log/nginx/error.log | grep 'error')\" != \"\" ]; then echo 'Error discovered!'; fi; sleep 10; done"]
    volumeMounts:
    - name: logs-vol
      mountPath: /var/log/nginx
  volumes:
  - name: logs-vol
    emptyDir: {}
```

When starting up the Pod, you’ll notice that the overall number of containers will show 2. After all containers can be started, the Pod signals a Running status:

```shell
$ kubectl create -f sidecar.yaml

pod/webserver created

$ kubectl get pods webserver

NAME        READY   STATUS              RESTARTS   AGE
webserver   0/2     ContainerCreating   0          4s

$ kubectl get pods webserver

NAME        READY   STATUS    RESTARTS   AGE
webserver   2/2     Running   0          5s
```

You will find that error.log does not contain any failure to begin with. It starts out as an empty file. With the following commands, you’ll provoke an error on purpose. After waiting for at least 10 seconds, you’ll find the expected message on the terminal, which you can query for with the logs command:

```shell
$ kubectl logs webserver -c sidecar

$ kubectl exec webserver -it -c sidecar -- /bin/sh

# wget -O- localhost?unknown

Connecting to localhost (127.0.0.1:80)
wget: server returned error: HTTP/1.1 404 Not Found

# cat /var/log/nginx/error.log

2020/07/18 17:26:46 [error] 29#29: *2 open() "/usr/share/nginx/html/unknown" failed (2: No such file or directory), client: 127.0.0.1, server: localhost, request: "GET /unknown HTTP/1.1", host: "localhost"

# exit

$ kubectl logs webserver -c sidecar

Error discovered!
```

### The Adapter Pattern

The basic purpose of an adapter container is to modify data, either on ingress or egress, to match some other need. Perhaps, an existing enterprise-wide monitoring tools has particular data format needs. An adapter would be an efficient way to standardize the output of the main container to be ingested by the monitoring tool, without having to modify the monitor or the containerized application. An adapter container transforms multiple applications to singular view.

This type of secondary container is useful to modify the data generated by the primary container. For example, the Microsoft version of ASCII is distinct from everyone else. You may need to modify a datastream for proper use.

As application developers, we want to focus on implementing business logic. For example, as part of a two-week sprint, say we’re tasked with adding a shopping cart feature. **In addition to the functional requirements, we also have to think about operational aspects like exposing administrative endpoints or crafting meaningful and properly formatted log output. It’s easy to fall into the habit of simply rolling all aspects into the application code, making it more complex and harder to maintain. Cross-cutting concerns in particular need to be replicated across multiple applications and are often copied and pasted from one code base to another.**

In Kubernetes, we can avoid bundling **cross-cutting concerns** into the application code by running them in another container apart from the main application container. The adapter pattern transforms the output produced by the application to make it consumable in the format needed by another part of the system. Figure 4-4 illustrates a concrete example of the adapter pattern.

<img src=".\images\pod-adapter-pattern.png"/>

The business application running the main container produces timestamped information—in this case, the available disk space—and writes it to the file diskspace.txt. As part of the architecture, we want to consume the file from a third-party monitoring application. The problem is that the external application requires the information to exclude the timestamp. Now, we could change the logging format to avoid writing the timestamp, but what do we do if we actually want to know when the log entry has been written? This is where the adapter pattern can help. An adapter container executes transformation logic that turns the log entries into the format needed by the external system without having to change application logic.

The YAML manifest shown below illustrates what this implementation of the adapter pattern could look like. The app container produces a new log entry every five seconds. The transformer container consumes the contents of the file, removes the timestamp, and writes it to a new file. Both containers have access to the same mount path through a Volume.

An exemplary adapter pattern implementation:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: adapter
spec:
  containers:
  - args:
    - /bin/sh
    - -c
    - 'while true; do echo "$(date) | $(du -sh ~)" >> /var/logs/diskspace.txt; sleep 5; done;'
    image: busybox
    name: app
    volumeMounts:
      - name: config-volume
        mountPath: /var/logs
  - image: busybox
    name: transformer
    args:
    - /bin/sh
    - -c
    - 'sleep 20; while true; do while read LINE; do echo "$LINE" | cut -f2 -d"|" >> $(date +%Y-%m-%d-%H-%M-%S)-transformed.txt; done < /var/logs/diskspace.txt; sleep 20; done;'
    volumeMounts:
    - name: config-volume
      mountPath: /var/logs
  volumes:
  - name: config-volume
    emptyDir: {}
```

After creating the Pod, we’ll find two running containers. We should be able to locate the original file, '/var/logs/diskspace.txt', after shelling into the transformer container. The transformed data exists in a separate file in the user home directory:

```shell
$ kubectl create -f adapter.yaml

pod/adapter created

$ kubectl get pods adapter

NAME      READY   STATUS    RESTARTS   AGE
adapter   2/2     Running   0          10s

$ kubectl exec adapter --container=transformer -it -- /bin/sh

# cat /var/logs/diskspace.txt

Sun Jul 19 20:28:07 UTC 2020 | 4.0K	/root
Sun Jul 19 20:28:12 UTC 2020 | 4.0K	/root

# ls -l

total 40
-rw-r--r--  1  root  root  60 Jul 19 20:28 2020-07-19-20-28-28-transformed.txt

...

# cat 2020-07-19-20-28-28-transformed.txt

 4.0K	/root
 4.0K	/root
```

### The Ambassador Pattern

The ambassador pattern provides a proxy for communicating with external services.

There are many use cases that can justify the introduction of the ambassador pattern. **The overarching goal is to hide and/or abstract the complexity of interacting with other parts of the system.** Typical responsibilities include retry logic upon a request failure, security concerns like providing authentication or authorization, or monitoring latency or resource usage.

<img src=".\images\pod-ambassador-pattern.png"/>

In this example, we’ll want to implement rate-limiting functionality for HTTP(S) calls to an external service. For example, the requirements for the rate limiter could say that an application can only make a maximum of 5 calls every 15 minutes. Instead of strongly coupling the rate-limiting logic to the application code, it will be provided by an ambassador container. Any calls made from the business application need to be tunneled through the ambassador container. Example below shows a Node.js-based rate limiter implementation that makes calls to the external service Postman.

Node.js HTTP rate limiter implementation:

```javascript
const express = require('express');
const app = express();
const rateLimit = require('express-rate-limit');
const https = require('https');

const rateLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  message:
    'Too many requests have been made from this IP, please try again after an hour'
});

app.get('/test', rateLimiter, function (req, res) {
  console.log('Received request...');
  var id = req.query.id;
  var url = 'https://postman-echo.com/get?test=' + id;
  console.log("Calling URL %s", url);

  https.get(url, (resp) => {
    let data = '';

    resp.on('data', (chunk) => {
      data += chunk;
    });

    resp.on('end', () => {
      res.send(data);
    });

    }).on("error", (err) => {
      res.send(err.message);
    });
})

var server = app.listen(8081, function () {
  var port = server.address().port
  console.log("Ambassador listening on port %s...", port)
})
```

The corresponding Pod runs the main application container on a different port than the ambassador container. Every call to the HTTP endpoint of the container named business-app would delegate to the HTTP endpoint of the container named ambassador. It’s important to mention that containers running inside of the same Pod can communicate via localhost. No additional networking configuration is required.

An exemplary ambassador pattern implementation:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: rate-limiter
spec:
  containers:
  - name: business-app
    image: bmuschko/nodejs-business-app:1.0.0
    ports:
    - containerPort: 8080
  - name: ambassador
    image: bmuschko/nodejs-ambassador:1.0.0
    ports:
    - containerPort: 8081
```

Let’s test the functionality. First, we’ll create the Pod, shell into the container that runs the business application, and execute a series of curl commands. The first five calls will be allowed to the external service. On the sixth call, we’ll receive an error message, as the rate limit has been reached within the given time frame:

```shell
$ kubectl create -f ambassador.yaml

pod/rate-limiter created

$ kubectl get pods rate-limiter

NAME           READY   STATUS    RESTARTS   AGE
rate-limiter   2/2     Running   0          5s

$ kubectl exec rate-limiter -it -c business-app -- /bin/sh

# curl localhost:8080/test

{"args":{"test":"123"},"headers":{"x-forwarded-proto":"https", \
"x-forwarded-port":"443","host":"postman-echo.com", \
"x-amzn-trace-id":"Root=1-5f177dba-e736991e882d12fcffd23f34"}, \
"url":"https://postman-echo.com/get?test=123"}
...

# curl localhost:8080/test

Too many requests have been made from this IP, please try again after an hour.
```

Ambassador is, an open source, Kubernetes-native API gateway for microservices built on Envoy.

It allows for access to the outside world without having to implement a service or another entry in an ingress controller: proxy local connection, reverse proxy, limits HTTP requests, re-route from the main container to the outside world.

This type of secondary container would be used to communicate with outside resources, often outside the cluster. Using a proxy, like Envoy or other, you can embed a proxy instead of using one provided by the cluster. It is helpful if you are unsure of the cluster configuration.

## Observability

### readinessProbe

Oftentimes, our application may have to initialize or be configured prior to being ready to accept traffic. As we scale up our application, we may have containers in various states of creation. Rather than communicate with a client prior to being fully ready, we can use a readinessProbe. The container will not accept traffic until the probe returns a healthy state.

With the exec statement, the container is not considered ready until a command returns a zero exit code. As long as the return is non-zero, the container is considered not ready and the probe will keep trying.

Another type of probe uses an HTTP GET request (httpGet). Using a defined header to a particular port and path, the container is not considered healthy until the web server returns a code 200-399. Any other code indicates failure, and the probe will try again.

The TCP Socket probe (tcpSocket) will attempt to open a socket on a predetermined port, and keep trying based on periodSeconds. Once the port can be opened, the container is considered healthy.

### livenessProbe

Just as we want to wait for a container to be ready for traffic, we also want to make sure it stays in a healthy state. Some applications may not have built-in checking, so we can use livenessProbes to continually check the health of a container. If the container is found to fail a probe, it is terminated. If under a controller, a replacement would be spawned.

### startupProbe

Becoming beta recently, we can also use the startupProbe. This probe is useful for testing an application which takes a long time to start.

If kubelet uses a startupProbe, it will disable liveness and readiness checks until the application passes the test. The duration until the container is considered failed is failureThreshold times periodSeconds. For example, if your periodSeconds was set to five seconds, and your failureThreshold was set to ten, kubelet would check the application every five seconds until it succeeds, or is considered failed after a total of 50 seconds. If you set the periodSeconds to 60 seconds, kubelet would keep testing for 300 seconds, or five minutes, before considering the container failed.

## Testing

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

## Declaring Environment Variables

Applications need to expose a way to make their runtime behavior configurable. For example, you may want to inject the URL to an external web service or declare the username for a database connection. Environment variables are a common option to provide this runtime configuration.

Defining environment variables in a Pod YAML manifest is relatively easy. Add or enhance the section env of a container. **Every environment variable consists of a key-value pair, represented by the attributes name and value. Kubernetes does not enforce or sanitize typical naming conventions for environment variable keys. It’s recommended to follow the standard of using upper-case letters and the underscore character (_) to separate words.**

To illustrate a set of environment variables, have a look at example below. The code snippet describes a Pod that runs a Java-based application using the Spring Boot framework.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: spring-boot-app
spec:
  containers:
  - image: bmuschko/spring-boot-app:1.5.3
    name: spring-boot-app
    env:
    - name: SPRING_PROFILES_ACTIVE
      value: prod
    - name: VERSION
      value: '1.5.3'
```

## Defining a Command with Arguments

Many container images already define an **ENTRYPOINT** or **CMD** instruction. The command assigned to the instruction is automatically executed as part of the container startup process. For example, the Hazelcast image we used earlier defines the instruction CMD ["/opt/hazelcast/start-hazelcast.sh"].

**In a Pod definition, you can either redefine the image ENTRYPOINT and CMD instructions or assign a command to execute for the container if hasn’t been specified by the image.** You can provide this information with the help of the command and args attributes for a container. The command attribute overrides the image’s ENTRYPOINT instruction. The args attribute replaces the CMD instruction of an image.

Imagine you wanted to provide a command to an image that doesn’t provide one yet. As usual there are two different approaches, imperatively and declaratively. 

We’ll generate the YAML manifest with the help of the run command. The Pod should use the busybox image and execute a shell command that renders the current date every 10 seconds in an infinite loop:

```shell
$ kubectl run mypod --image=busybox -o yaml --dry-run=client --restart=Never > pod.yaml -- /bin/sh -c "while true; do date; sleep 10; done"
```

You could have achieved the same by a combination of the command and args attributes if you were to hand-craft the YAML manifest.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - command: ["/bin/sh"]
    args: ["-c", "while true; do date; sleep 10; done"]
    image: busybox
    name: mypod
  restartPolicy: Never
```

You can quickly verify if the declared command actually does its job. First, we create the Pod instance, then we tail the logs:

```shell
$ kubectl create -f pod.yaml
```

## Removing Pods

Sooner or later you’ll want to delete a Pod. You made a configuration mistake and want to start the question from scratch:

```shell
$ kubectl delete pod hazelcast
```

**Keep in mind that Kubernetes tries to delete a Pod gracefully.** This means that the Pod will try to finish active requests to the Pod to avoid unnecessary disruption to the end user. **A graceful deletion operation can take anywhere from 5-30 seconds.**

```shell
kubectl delete po pod-name

kubectl delete po -l label=value

kubectl delete po --all

kubectl delete ns custom_namespace
```

An alternative way to delete a Pod is to point the delete command to the YAML manifest you used to create it. The behavior is the same:

```shell
$ kubectl delete -f pod.yaml
```
