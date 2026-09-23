so we have 

```
	 cluster ( collection of nodes) (A cluster is the collection of nodes and the Kubernetes control plane that manages the cluster.)
									| 
									\/
		these clusters have nodes (in windows its is docker-desktop)
									|
									\/
   Pods are scheduled onto Nodes (each pod can have one more container                                 running inside)
```

this is the basic flow i guess, 
we cant up multiple pods manually with command so we write a 
# Deployment.yaml, Pod.yaml, Service.yaml

Kubernetes resources are declared using YAML with fields such as `apiVersion`, `kind`, `metadata`, and resource-specific configuration; many resources use `spec`.

apiVersion, Kind, Metadata, Spec (it differs for different yaml , example kind HPA, service , ingress ) (mostly same )

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

> A Service doesn't simply "store" Pod IPs. It provides a stable virtual endpoint and Kubernetes maintains the set of backend endpoints corresponding to matching Pods.

so service manages this by selecting all labels of the app that we mention while creating a service , so once a pod is up with x label the service discovers this and keeps it 

> Pods can use the Service's DNS name to reach the Service, and the Service routes traffic to its selected backend Pods.

so we have solid setup with this , but service only knows the internal endpoints ips and port , we need a way to connect outside cluster traffic to inside the cluster , 
here is where ... 
Ingress comes it defines rules where and how the outside traffic should be routed to which service inside the cluster . 

but Ingress needs a controller based on ngnix or similar to enforce those rules 
(Ingress-Controller)

so now we have a flow , to do other optimizations to this we need 
configMap -> we can create this resource and tag it as referencing its name to a pod so we dont have to rewrite the envs across similar pods
similarly Secrets work but for Secrets !!

then we need a persisting storage for some pods like db -> postgres pod etc., 
we use PVC -> persistent Volume Claim -> docker has storage class hostpath which has automatic provisioning so it converts that claim to actual physical hold of some storage for our said pod , now we can give the name of this claim to the pod as volume and it will use it instead of creating new one every time 

- **PVC (PersistentVolumeClaim)** → a request for storage made by a workload.
- **PV (PersistentVolume)** → a Kubernetes resource representing available persistent storage.
- **StorageClass** → defines how storage can be dynamically provisioned.


Then comes HPA -> Horizontal Pod Autoscaling -> which watchs metrics of pods and scales them acorrdingly for given watch metric 

>HPA watches metrics and adjusts the number of Pod replicas belonging to a workload such as a Deployment.

with this basics of kubernetes is over !!! 





```
                         Internet
                            │
                            ▼
                     Ingress Controller
                            │
                            ▼
                         Ingress
                            │
                            ▼
                      app-service
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
          FastAPI        FastAPI        FastAPI
            Pod            Pod            Pod
             │              │              │
             └──────────────┼──────────────┘
                            │
                    managed by Deployment
                            │
                         ReplicaSet


                     ┌───────────────┐
                     │               │
                     ▼               ▼
             postgres-service   redis-service
                     │               │
                     ▼               ▼
              PostgreSQL Pod      Redis Pod
                     │
                     ▼
                    PVC
                     │
                     ▼
                    PV
                     │
                     ▼
             Actual storage
```