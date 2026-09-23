```
CLUSTER
  │
  └── NODE
       │
       └── POD
            │
            └── CONTAINER


DEPLOYMENT
     │
     ▼
REPLICASET
     │
     ▼
   PODS


SERVICE
     │
     └── stable endpoint
             │
             ▼
        matching Pods


INGRESS
     │
     ▼
INGRESS CONTROLLER
     │
     ▼
  SERVICE
     │
     ▼
   PODS


CONFIGMAP ──┐
            ├──→ POD
SECRET ─────┘


POD
 │
 ▼
PVC
 │
 ▼
PV
 │
 ▼
STORAGE


METRICS SERVER
      │
      ▼
     HPA
      │
      ▼
 DEPLOYMENT
      │
      ▼
    PODS
```