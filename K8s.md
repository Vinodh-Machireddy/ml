**StatefulSet vs Deployment:**
Deployment — for stateless applications. If a pod dies, Kubernetes replaces it with a brand new one — no identity, no persistent storage, no specific ordering. It doesn't matter which pod serves the request.  

StatefulSet — for stateful applications. Each pod has a stable identity (name, network address, storage) that persists across restarts. Pods are created and deleted in a specific order. If pod postgres-0 dies, Kubernetes recreates it with the same name and reattaches the same persistent volume — your data survives.  
```
Deployments (stateless):          StatefulSets (stateful):
├── KFP pipeline pods             ├── PostgreSQL (MLflow backend)
├── KServe inference pods         ├── Prometheus
├── ArgoCD                        └── Grafana (if using persistent dashboards)
└── MLflow server (API layer)
```
**"What's a PersistentVolumeClaim?":** — It's the storage request that a StatefulSet pod makes. Each pod gets its own PVC, backed by an EBS volume on AWS. Even if the pod is rescheduled to a different node, the PVC follows it.  
