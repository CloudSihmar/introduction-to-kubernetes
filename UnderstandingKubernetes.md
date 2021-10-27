# Introduction to Kubernetes

Editors: **Kaan Keskin, Sezen Erdem**

Date: November 2021

Available at: https://github.com/kaan-keskin/introduction-to-kubernetes

**Resources:**

> - Kubernetes in Action - Marko Lukša 
> - Kubernetes Documentation - https://kubernetes.io/docs/home/

**LEGAL NOTICE: This document is created for educational purposes, and it can not be used for any commercial purposes. If you find this document useful in any means please support the original authors for ethical reasons.** 

[Return README page.](README.md)

# Understanding Kubernetes

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
