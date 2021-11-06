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

## Operators

An important concept for orchestration is the use of operators, otherwise known as controllers or watch-loops. Various operators ship with Kubernetes, and you can create your own, as well. A simplified view of an operator is an agent, or Informer, and a downstream store. Using a DeltaFIFO queue, the source and downstream are compared. A loop process receives an obj or object, which is an array of deltas from the FIFO queue. As long as the delta is not of the type Deleted, the logic of the operator is used to create or modify some object until it matches the specification.

The Informer which uses the API server as a source requests the state of an object via an API call. The data is cached to minimize API server transactions. A similar agent is the SharedInformer; objects are often used by multiple other objects. It creates a shared cache of the state for multiple requests.

A Workqueue uses a key to hand out tasks to various workers. The standard Go work queues of rate limiting, delayed, and time queue are typically used. 

The endpoints, namespace, and serviceaccounts operators each manage the eponymous resources for Pods. Deployments manage replicaSets, which manage Pods running the same podSpec, or replicas.

## Service Operator

With every object and agent decoupled we need a flexible and scalable agent which connects resources together and will reconnect, should something die and a replacement is spawned. A service is an operator which listens to the endpoint operator to provide a persistent IP for Pods. Pods have ephemeral IP addresses chosen from a pool. 

Then the service operator sends messages via the kube-apiserver which forwards settings to kube-proxy on every node, as well as the network plugin such as calico-kube-controllers.

A service also handles access policies for inbound requests, useful for resource control, as well as for security. 

- Connect Pods together 
- Expose Pods to Internet 
- Decouple settings 
- Define Pod access policy.
