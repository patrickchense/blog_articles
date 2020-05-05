[basic](https://www.infoq.cn/article/te70FlSyxhltL1Cr7gzM)

## Container & Mirror

### What's a container?   

It's a process group with view isolation, resource control and independent file system.

### What's a Mirror?

All the files needed when container running is called Mirror.

Normally, we use Dockerfile to build the mirror, because Docker provided very good understand of each step of the process. Every step changes the file system, these changes are called <changeset>.  The tier-changeset and reusability brings some benefits:

1. Improve the distribution efficiency. Big mirror can be split to small parts and download in parral.
2. Small parts can be share in the system.
3. Share parts can save a lot of storage.

### How to build Mirror?

Example based on Go:  

```dockerfile
#base on golang:1.12-alpine image
FROM goland:1.12-alpine

#setting current working dir (PWD -> /go/src/app)
WORKDIR /go/src/app

# copy local files into /go/src/app
COPY . .

# get all dependencies
RUN go get -d -v ./...

# build the application and install it
RUN go install -v ./...

# by default, run the app
CMD ["app"]
```

After creating the Dockerfile, then run `docker build` to build the app. The result normally store in local.

#### How to run the image in test or prod?

Need a transfer station or central storage, called `docker registry` to store all the docker images. We use `docker push` to push local image into the `docker registry`. After that, test env or prod env can download the images and run.

### How to run container?

Three steps:

1. Download the images from `docker registry`.
2. After successfully download, using `docker images` to check local images. There is a whole list of images and people can choose what they want.
3. After chosing the images, using `docker run` to run the container and this can be done multiple times with multiple containers. Every image is like a template, and each container is like the instance running the image. 

## Container Lifecycle

The container is a process group, so when `docker run` a image, the container chooses related program to run. The program is called `initial` process, when this `initial` process is starting, then container is starting and when `initial` process quit, then container quit too.

So the lifecycle of a container is go along with the `initial` process. And also the sub process created during the lifecycle is related to the `initial` process. 

But the data can't just be delete with the process, so there is a data volume in the mix, to persist the data from the container. The volume is not outside the container, looks like a independent part of the system. 

There is two ways to manage the data volume:

* `bind` data volume to the container. In this way, the system has to manage all the volumes
* leave all the control to the engine.

## Container Arch

### Engine Arch

moby is the most popular container engine arch so far. `mobydeamon` provides management about container, mirror, network and data volume. The most important the component is the `containerd`, it's a container runtime management engine, independent from `moby deamon`.

`containerd` manages container lifecycle, but container created based on different environment(images). So `containerd` uses `shim` to manage flexibly. `shim` developed based on all kinds of `containers`,  makes it separate from `containerd`, easily plugin into it.

The biggest feature about `shim` is dynamic takeover which means when any `containerd` or `container deamon` crushes, some other can take over immediately. 

## Container VS VM

### The difference between container & vm

VM using Hypervisor to mock CPU, Memory and other resources,  then install OS based on that. All VM bring better isolation, but cost a lot of resources compare to containers. 

Containers are small and depend on process, so it's lightweight. 





