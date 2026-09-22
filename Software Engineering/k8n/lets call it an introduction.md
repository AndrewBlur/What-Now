so we have 
```
	 cluster ( collection of nodes) (A cluster is the collection of nodes and the Kubernetes control plane that manages the cluster.)
									| 
									\/
		these clusters have nodes (in windows its is docker-desktop)
									|
									\/
   inside these nodes we run pods (each pod can have one more container                                 running inside)
```

this is the basic flow i guess, 
we cant up multiple pods manually with command so we write a 
# Deployment.yaml, Pod.yaml, Service.yaml

all yaml files for k8n has certain structure
apiVersion, Kind, Metadata, Spec

for this file we write Spec 
	- replicas -> how many replicas of the pods we want
	- Selector -> Template Labels
	- Template -> the thing we want to replicate , we write its basically pod.yaml

for different yaml we only change the kind to Deployment, Service, Pod, etc. (i dont know if theres more)

> The Deployment doesn't necessarily "up multiple Pods" directly. It manages a ReplicaSet, which maintains the desired number of Pods.

use of creating a service (created using service.yaml)

inside the cluster 
- the pods can communicate with ip's so if we hard code the ips to our application here is where it will break
	-  if a pod gets deleted deployment takes care of upping another pod with different ip ,
	- so we need a way to route traffic to the application pods that we need even if the ip is changing 

so service manages this by selecting all labels of the app that we mention while creating a service , so once a pod is up with x label the service discovers this and keeps it and the traffic will route through the service name as a dns , 