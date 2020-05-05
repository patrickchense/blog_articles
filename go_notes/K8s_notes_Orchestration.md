## App Orchestration & Management

### BackGround

Cluster manage all the Pods, app A, B, C belongs one of the Pods, which is  spread among the cluster.

#### Problems Rasied

* How to manage the available Pod size? If some of the four Pod of App A have errors or network issues, how to maintain the size?
* How to update mirror for all the Pods? 
* How to keep service availbility during the upgrade?
* If error occurs during the upgrade, how to rollback?

All this maybe related to the **Deployment** component.

First K8s schedules different App(A,B,C) into different Deployments, each Deployment manages the same App, we considers the same App has the same version. What Deployment can do here?

1. Deployment defines the Pod size (estimation),  then controller helps to maintain the Pod size. If the Pod occurs error, controller helps to recovery or start other Pod, keep the size stable.
2. Deployment configs Pod deploy sloution,  which means controller update Pod based on user policy, and can set the un-useable Pod size  range during the updating.
3. Deployment provides one command or one button to rollback the App.

### Deployment grammer

```yaml
apiVersion: apps/v1
kind: Development
metadata:
	name: nginx-deployment
	labels:
		app: nginx
spec:
	replicas: 3
	selector: 
		matchLabels: 
			app: nginx
	template:
		metadata:
			labels:
				app: nginx
		spec:
			containers:
			- name: nginx
				image: nginx:1.7.9
				ports:
				-	containerPort: 80
```

very simple Deployment yaml configuration.

- apiVersion: apps/v1, Deployment group is apps, version v1. metadata is Deployment related.
- Pod template contains two parts:
  - Pod metadata, labels, matches `selector.matchLabels` 
  - Pod.spec, Pod mirror and version

#### check Deployment status

> kubectl get deployment

#### check Pod

> kubectl get pod

#### update mirror

> kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.9.1

* kubectl has fixed ways to set image
* **deployment.v1.apps**, is build up with resource name, resource version, resource group. 
* **nginx-deployment**, deployment name
* **nginx**=, means template, also the Pod [container.name](#container.name)
* **nginx:1.9.1**, updated version

#### rollback

> kubectl rollout undo

or + verison

> kubectl rollout undo deployment/nginx-deployment

check which version?

> kubectl rollout undo deployment.v1.apps/nginx-deployment --to-revision=2

#### Deployment status

```
`![deploymentstatus](https://static001.infoq.cn/resource/image/9c/00/9c8dfbbcc989d37c70f5fcbed3a4ba00.png "DeploymentStatus")`|![deploymentstatus](https://static001.infoq.cn/resource/image/9c/00/9c8dfbbcc989d37c70f5fcbed3a4ba00.png "DeploymentStatus")
```

... not finished 



[refer](#https://www.infoq.cn/article/bShSloggOsz3RyV6azfL)

