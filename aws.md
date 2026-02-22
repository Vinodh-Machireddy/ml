<div align="center">
  
  # AWS
</div>

```GitHub → GitHub Actions → Kubeflow Pipeline → MLflow Tracking → MLflow Registry → Docker → ArgoCD → KServe → Prometheus → Grafana```
## How deployment happens from local → production
Step 1: Develop locally  
Step 2: Push code to GitHub  
Step 3: GitHub Actions builds Docker image  
Step 4: Push image to registry  
Step 5: ArgoCD deploys to cloud Kubernetes  
Step 6: KServe exposes public endpoint  
Step 7: Customers access model  
<details>
   <summary><b>Click to expand: Production MLOps project structure</b></summary>
  <img width="387" height="436" alt="Screenshot 2026-02-21 at 4 14 39 PM" src="https://github.com/user-attachments/assets/81fd56f3-68ec-4d3d-be49-5a977a87bd07" />
  <img width="370" height="553" alt="Screenshot 2026-02-21 at 4 15 15 PM" src="https://github.com/user-attachments/assets/194c3596-7661-46ff-9a3b-d92b73ae2028" />  
</details>

## How customers access ML model in real production
```
Customer Application
      ↓
Internet
      ↓
Public URL (example: api.company.com)
      ↓
Load Balancer / Ingress
      ↓
KServe Model Endpoint
      ↓
Model Container
      ↓
Prediction Response
```
Example:  
Customer sends request:  ```POST https://api.company.com/v1/models/fraud-detection:predict```  
Request body:  
```
{
  "instances": [1000, 45, 0.67]
}
```
Model returns:  
```
{
  "predictions": [0.92]
}
```
### Where this model is hosted in production
Usually on cloud Kubernetes:  
AWS EKS  
Azure AKS  
Google GKE  


# EKS
1. Control Plane  
2. Data Plane
   
If we setup kubernetes using kops, kube Admins. we need to manage entire control plane(master node), it is lenghty process. 
- kube-API server  
- etcd(pronounced et-see-dee)  database  
- kube scheduler
- kube controller manager  
- cloud-controller-manager  

That's why we going with aws EKS which is aws managed service which manages complete control plane.  

Normal EKS (Without Fargate): 
In normal You must:  
- Create EC2 instances  
- Manage node groups  
- Scale nodes  
- Patch OS  
- Handle capacity  
- Pods run on EC2 worker nodes.  

  With Fargate in EKS:  
  - You run containers  
  - Without managing EC2 servers  
  - AWS manages infrastructure  
  -  No node groups. No instance selection.  

- EKS manages control plane  
- Fargate is a aws Fargate is serverless compute for containers which manages worker nodes.   
Note:- for worker nodes we can use EC2 Instances also.but, we need to take care of High Availability.



