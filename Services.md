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

With every object and agent decoupled we need a flexible and scalable operator which connects resources together and will reconnect, should something die and a replacement is spawned. Each Service is a microservice handling a particular bit of traffic, such as a single NodePort or a LoadBalancer to distribute inbound requests among many Pods.

A Service also handles access policies for inbound requests, useful for resource control, as well as for security.

A service, as well as kubectl, uses a selector in order to know which objects to connect. There are two selectors currently supported:

- **equality-based**: Filters by label keys and their values. Three operators can be used, such as =, ==, and !=. If multiple values or keys are used, all must be included for a match.
- **set-based**: Filters according to a set of values. The operators are in, notin, and exists. For example, the use of status notin (dev, test, maint) would select resources with the key of status which did not have a value of dev, test, nor maint.

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

For a client to connect to all pods, it needs to figure out the IP of each individual pod. One option is to have the client call the Kubernetes API server and get the list of pods and their IP addresses through an API call, but because you should always strive to keep your apps Kubernetes-agnostic, using the API server isn’t ideal.

**When you perform a DNS lookup for a service, the DNS server returns a single IP—the service’s cluster IP. But if you tell Kubernetes you don’t need a cluster IP for your service (you do this by setting the clusterIP field to None in the service specification), the DNS server will return the pod IPs instead of the single service IP.**

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
