## DeamonSets

Certain cases exist when you want a pod to run on each and every node in the cluster and each node needs to run exactly one instance of the pod.

<img src=".\images\p3_deamonsets.jpg"/>

Those cases include infrastructure-related pods that perform system-level operations. For example, you’ll want to run a log collector and a resource monitor on every node. Another good example is Kubernetes’ own kube-proxy process, which needs to run on all nodes to make services work.

Pods created by a DaemonSet have a target node specified and skip the Kubernetes Scheduler. They aren’t scattered around the cluster randomly. A DaemonSet makes sure it creates as many pods as there are nodes and deploys each one on its own node.

If a node goes down, the DaemonSet doesn’t cause the pod to be created elsewhere. But when a new node is added to the cluster, the DaemonSet immediately deploys a new pod instance to it. It also does the same if someone inadvertently deletes one of the pods, leaving the node without the DaemonSet’s pod.

A DaemonSet deploys pods to all nodes in the cluster, unless you specify that the pods should only run on a subset of all the nodes. This is done by specifying the node-Selector property in the pod template.
