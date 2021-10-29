## StatefulSet

According to Kubernetes documentation, a StatefulSet is the workload API object used to manage stateful applications. Pods deployed using a StatefulSet use the same Pod specification. How this is different than a Deployment is that a StatefulSet considers each Pod as unique and provides ordering to Pod deployment. 

In order to track each Pod as a unique object, the controllers uses an identity composed of stable storage, stable network identity, and an ordinal. This identity remains with the node, regardless of which node the Pod is running on at any one time.

The default deployment scheme is sequential, starting with 0, such as app-0, app-1, app-2, etc. A following Pod will not launch until the current Pod reaches a running and ready state. They are not deployed in parallel.

StatefulSets are stable as of Kubernetes v1.9.
