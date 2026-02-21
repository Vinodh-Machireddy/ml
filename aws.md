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
```
  mlops-project/
│
├── README.md
├── .gitignore
│
├── requirements.txt
├── Dockerfile
│
├── src/
│   ├── train.py
│   ├── evaluate.py
│   ├── predict.py
│   ├── preprocess.py
│   ├── utils.py
│
├── pipelines/
│   ├── kubeflow_pipeline.py
│   ├── components/
│   │   ├── train_component.py
│   │   ├── evaluate_component.py
│   │   ├── register_component.py
│
├── deployment/
│   ├── kserve-inference.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│
├── argocd/
│   ├── application.yaml
│
├── mlflow/
│   ├── mlflow_config.yaml
│
├── monitoring/
│   ├── prometheus-config.yaml
│   ├── grafana-dashboard.json
│
├── .github/
│   └── workflows/
│       ├── ci.yaml
│       ├── cd.yaml
│
├── tests/
│   ├── test_train.py
│   ├── test_predict.py
│
└── configs/
    ├── config.yaml
```
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



