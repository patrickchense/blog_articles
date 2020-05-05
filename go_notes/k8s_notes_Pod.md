## Pod && Container Design Pattern

### Why need Pod?

Pod is the idea of process group. Every Docker container is one process, sometimes we need multiple process to work together share the same resources, so that's why we need Pod.

The idea of process group (Pod) is coming from Google Brog Paper. 

#### Why Pod has to be the smallest unit?

Because the processes relates have to manage together, otherwise it's impossible to manage them. That's how K8s Scheduler schedules the container, related processes(container) have to be scheduled in the same Pod. 

* the processes may need exchange files
* the processes may communicate with local address or sockets
* the processes may share the same namespaces

### How to implement the Pod?

Pod itself is a logic idea. The key factor is sharing resources between multiple processes in an efficient way.

1. **Sharing Network** 

K8s create a Infra container to make containers in the Pod share the same Network Namespace.

Infra container is very small, like 100-200kb and always in "PAUSE" state. After Pod has this, all other containers can join Namespace to this infra container.

That's why the network device, IP, and MAC etc all looks the same in the Pod, they all come from the same infra container. 

The infra container will be the first container started in the Pod and share the same lifecycle with the Pod.

2. **Share Storage**

Very simple. Change the volume to Pod level, and all the containers can share the storage in the Pod. 



### Design Pattern

### Sidecar

**Sidecar** is container inside the Pod to help with the management. For example, `InitContainer` is the Sidecar help copy data to volume. 

Other ops?

* SSH stuff. 
* Log collection. 
* Debug
* Monitor

The most important thing about Sidecar is it's de-coupled from business containers, can be deployed independently. And the Sidecar can be shared among all the Pods.

#### Sidecar -- log collection

The business log writes to data volume in the Pod and shared by all the containers in the Pod. The Sidecar can read the log and store remotely or forward them, just like Fluented.

#### Sidecar -- proxy container

If the business container needs to  access the outside system or service, but maybe these are distributed, we need to find a way to easily access.

Using proxy Sidecar like a gateway encapsulate all the logic, and using proxy to connect all the outside service. The business container can easily communicates with proxy by local address, because they are in the same Namespace.

#### Sidecar -- adapter 

Create a Sidecar to adapte the API or interface, make it useable by outside service with another API version.

### Summary

* Pod is the core in K8s.
* Container Design Pattern is the best practice of Google Brog,  also the fundamental of K8s.
* The essence of all design pattern is **de-couple** and **reuse** 



