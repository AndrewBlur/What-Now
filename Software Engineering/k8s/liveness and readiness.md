### Liveness & Readiness Probes

Kubernetes uses probes to determine the health and availability of a Pod.
- **Readiness Probe** → _"Can this Pod receive traffic?"_
    - If it fails, Kubernetes removes the Pod from the Service's endpoints.
    - The container is **not restarted**.
    - Example: check whether PostgreSQL/Redis is available.
        
- **Liveness Probe** → _"Is this application still alive?"_
    - If it repeatedly fails, Kubernetes **restarts the container**.
    - Should generally be a lightweight check of the application itself.
        

```text
                 Pod
                  │
        ┌─────────┴─────────┐
        ▼                   ▼
   Readiness             Liveness
        │                   │
 "Send traffic?"      "Restart me?"
        │                   │
    YES → traffic       FAIL → restart
    NO  → no traffic
```

**Easy way to remember:**

> **Readiness controls traffic. Liveness controls restarts.**