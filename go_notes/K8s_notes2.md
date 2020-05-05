## K8s Core Concept

### What is K8s?

K8s is a automatic container orchestration platform, it in charges of app deployment,  app flexibility and app management, all based on containers.

### K8s Key Function

* Service descovery and load balancing

* Container auot-boxing, also called `scheduling`. It puts the container into a machine in the cluster and help arrange storage, makes strorage lifecycle connects with container lifecycle.
* K8s supports container auto recovery. In the cluster, if the container fails because of the host or OS problem, K8s can auto recovery the container. 
* K8s support auto deployment and rollback. 
* K8s support batch job execution
* K8s support horizontal scalaing

### K8s Arch

### Master

K8s is classic 2-tier server-client arch. Master as centrol controller, connects with nodes.

All the UI, clients and users only connect the Master and Master sends the command to the nodes.

Master contains four major parts: API Server, Controller, Scheduler and ectd. 

* API Server: handles all the api. All the components in K8s connects with the API Server, they don't connect directly and communicates through APIs.
* Controller: manges cluster states. for example,  auto recovery and auto scalaing both depend on the states montoring. 
* Scheduler: for example, new container scheduling means finding host with enough CPU and memory resources for the container.
* etcd: A distributed storage system. All the data API Server needs is in the etcd. etcd is highly available, K8s availability based on that.

### Node

In K8s, Node is the component runs all the businesses,  every business runs as Pod. Each Pod has one or more containers,  the component `kubelet` in Pod is the key, it gets Pos status through API Server, and submits to the `container runtime`.

Four key components in `node`: `container runtime`, `storage plugin`, `network plugin`, `kube-proxy`. K8s doesn't operate network or storage directly(depends on `skim`?), but it uses plugins to operate and every vendor provides its own plugin to finish the job.

There is a network inside the k8s, it replies on `kube-proxy` iptables to build the network.

One example of the communication process:

> User submit Pod to K8s to deploy
>
> This Pod submits to API Server, and API Server stores it in etcd
>
> Scheduler get the notification by API Server watch.
>
> After Scheduler successfully handle the scheduling, communicates with API Server
>
> API Server stores the result in etcd, and notify the Node pod started ok. 
>
> Node gets the notification then notify the kubelet to run container runtime to actually starts the container and use plugin to handel network and storage.

### K8s Key Concept And API

#### Key Concept

##### Pod

The smallest unit in K8s. User use Pod API to generate Pod,  K8s schedules the Pod and puts it in the cluster to run. Pos is the abstract of a group of containers.

##### Volume

Manges K8s's storage, declares the dir can be accessed by the containers inside the Pod. One volume can bind to one or more containers inside the Pod.

Volume is an abstract definition, which can support multiple backend storage, such as local, distributed and cloud. 

##### Deployment

High level of Pods, it can define a group of Pod, replicas and version. Normally uers use `Deployment` to manage.

##### Service

Service provides one or more Pod instance stable address.

There may be more Pods in one Deployment, but the users doesn't care as long as they can access any of them by the same VIP and they dont' care about the real address of each Pod.

So it depends on the Deployment to do LB and provides the VIP. This funcation is called `Service`.

##### Namespace

`Namespace` used to logially isolate the resources inside the cluster. Each resources of the K8s mentioned above `Pod`, `Deployment`, `Server` all belongs to one `Namespace`,  resources in the same `Namespace` is unique. 

#### API

K8s API is all HTTP requests with JSON response.`kubectl`, UI and curl communicates with K8s all though HTTP+JSON.

e.g: /api/v1/namespaces/$NAMESPACE/pods/$NAME

