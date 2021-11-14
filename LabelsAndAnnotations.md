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

## Labels and Annotations

## Using Label Selectors

Labels allow for objects to be selected, which may not share other characteristics. For example, if a developer were to label their pods using their name, they could affect all of their pods, regardless of the application or deployment the pods were using.

Labels are how operators, also known as watch-loops, track and manage objects. As a result, if you were to hard-code the same label for two objects, they may be managed by different operators and cause issues. For example, one deployment may call for ten pods, while another with the same label calls for five. All the pods would be in a constant state of restarting, as each operator tries to start or stop pods until the status matches the spec.

Consider the possible ways you may want to group your pods and other objects in production. Perhaps you may use development and production labels to differentiate the state of the application. Perhaps you may want to add labels for the department, team, and primary application developer.

Selectors are namespace-scoped. Use the --all-namespaces argument to select matching objects in all namespaces.

The labels, annotations, name, and metadata of an object can be found near the top of

```shell
$ kubectl get object-name -o  yaml 
```

For example:

```shell
$ kubectl get pod examplepod-1234-vtlzd -o yaml
```

```yaml
apiVersion: v1 
kind: Pod 
metadata:   
  annotations:     
    cni.projectcalico.org/podIP: 192.168.113.220/32   
  creationTimestamp: "2020-01-31T15:13:08Z"   
  generateName: examplepod-1234-   
  labels:     
    app: examplepod     
    pod-template-hash: 1234   
  name: examplepod-1234-vtlzd 
....
```

Using this output, one way we could select the pod would be to use the app pod label. In the command line, the colon(:) would be replaced with an equals (=) sign and the space removed. The use of -l or --selector options can be used with kubectl.

```shell
$ kubectl -n test2 get --selector app=examplepod pod

NAME                   READY  STATUS   RESTARTS  AGE 
examplepod-1234-vtlzd  1/1    Running  0         25m
```

There are several built-in object labels. For example nodes have labels such as the arch, hostname, and os, which could be used for assigning pods to a particular node, or type of node.

```shell
$ kubectl get node worker

....
   creationTimestamp: "2020-05-12T14:23:04Z"
   labels:
     beta.kubernetes.io/arch: amd64
     beta.kubernetes.io/os: linux
     kubernetes.io/arch: amd64
     kubernetes.io/hostname: worker
     kubernetes.io/os: linux
   managedFields:
....
```

The nodeSelector: entry in the podspec could use this label to cause a pod to be deployed on a particular node with an entry such as:

```yaml
     spec:
       nodeSelector:
         kubernetes.io/hostname: worker
       containers:
```

## Using Annotations

Labels are used to work with objects or collections of objects; annotations are not.

Instead, annotations allow for metadata to be included with an object that may be helpful outside of the Kubernetes object interaction. Similar to labels, they are key to value maps. They are also able to hold more information, and more human-readable information than labels.
 
Having this kind of metadata can be used to track information such as a timestamp, pointers to related objects from other ecosystems, or even an email from the developer responsible for that object's creation. 

The annotation data could otherwise be held in an exterior database, but that would limit the flexibility of the data. The more this metadata is included, the easier it is to integrate management and deployment tools or shared client libraries. 

For example, to annotate only Pods within a namespace, you can overwrite the annotation, and finally delete it: 

```shell
$ kubectl annotate pods --all description='Production Pods' -n prod 

$ kubectl annotate --overwrite pod webpod description="Old Production Pods" -n prod 

$ kubectl -n prod annotate pod webpod description-
```
