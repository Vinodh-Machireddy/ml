# FLOW

## Project Approach:
“I generally explain the methodology in three layers:  
First, from a project execution perspective, teams may use Agile or Scrum.  
Second, from an ML development perspective, we work iteratively through experimentation, feature engineering, training, and validation.  
Third, from an MLOps engineering perspective, our stack is best described as a pipeline-based CI/CD/CT architecture with GitOps deployment and continuous monitoring.  


1. Project Initiation & Tech Stack
2. What the Data Scientists hand over — brief context, just enough to show you understand the ML side
3. Training Pipeline (KFP) — how you productionized their notebook into a repeatable pipeline (this is YOUR territory)
4. Experiment Tracking & Model Registry (MLflow) — how models are versioned and promoted (YOUR territory)
5. CI/CD Pipeline (GitHub Actions + ArgoCD + DVC) — how code and model changes flow to production (YOUR territory)
6. Model Serving (KServe) — real-time inference architecture (YOUR territory)
7. Monitoring & Observability (Prometheus + Grafana) — drift detection, alerting (YOUR territory)
8. Retraining Strategy — when and how retraining is triggered (YOUR territory)
> **Team Flow:** Platform Engineering → Data Engineering → Data Science → MLOps Engineering

## 1. Project Initiation & Tech Stack:
I am working on Daimler Project which comes under automotive domain/Industry, their i am dealing with Battery Fault Classification. The project aims to classify battery cell faults in real time using sensor data from the Battery Management System (BMS). This helps detect issues like overheating, overcharging, internal short circuits, and cell degradation early — before they become safety-critical. The main purpose is to improve EV battery safety, and extend battery lifespan.  
**Tech stack:**  Kubeflow Pipelines(KFP), Mlflow, ArgoCD, Kserve, Prometheus & Grafana, GitHub Actions, AWS, Git, GitHub, DVC, Docker, Kubernetes, Python, Linux.

## 2. What the Data Scientists hand over to MLOps
**as a Senior MLOps Engineer.** my job is not to build the model from scratch — we have data scientists for that. My responsibility starts from the point where we need to productionize that model. That means — how do we take this model from a Jupyter notebook, build a proper training pipeline around it, package it, deploy it to production, serve it in real time, monitor it, and retrain it.   

**Data ingestion from Vehical to s3**
The raw telemetry data lands in S3 via the ingestion pipeline from platform engineering teams. Now the data science team pull this raw data from s3, did their EDA, cleaned it, and future engineering steps and pushes the processed training dataset back into S3 as **Parquet files. i.e train.parquet**.
> so, s3 contains raw data, processed data, model artifacts, etc...    

### I received three essentially things from Data Scientists 
- A monolithic Jupyter notebook — with the full training code, preprocessing logic, feature engineering, model training, and evaluation.  (the original experiment)
- A requirements file — Python dependencies (xgboost, pandas, etc.)  
- The training dataset in S3 — versioned with DVC.    # DVC pointer file (NOT actual data)
  > When the data scientist saves train.parquet to S3, they also run: dvc add `data/train.parquet` which does two things: 
    > 1. It **uploads** the actual large file to S3
    > 2. It creates a **tiny pointer file** called `train.parquet.dvc` in git
    > 3. Now you will not lost the link between code and data, for reproduce old Model versions (v1, v2, v3,v4...).

My work starts from S3 onwards. My KFP pipeline picks up data from this processed S3 path as its input. we take this notebook and turn it into a production-grade, automated, repeatable, auditable training pipeline.  

## 3. Training Pipeline (KFP)  
**why KFP specifically?** Because our infrastructure is on Kubernetes (EKS on AWS), and KFP runs natively on Kubernetes. Each step of the pipeline runs as a separate container inside a Kubernetes pod. That gives us isolation, scalability, and reproducibility out of the box.
 
"So the first major thing I did was, I decomposed the monolithic Jupyter notebook into a Kubeflow Pipeline — KFP."  

**It has 6 stages, and each stage is a separate Python component packaged as a Docker container:**
Each pipeline stage is a separate Python file under a components folder. This gives us modularity — each component can be developed, tested, and maintained independently. The pipeline definition file imports all components and chains them together, passing outputs from one step as inputs to the next.  

**The pipeline.py imports and chains them:**   
```
from components.data_ingestion import data_ingestion
from components.data_validation import data_validation
from components.preprocessing import preprocessing
from components.model_training import model_training
from components.model_evaluation import model_evaluation
from components.model_registration import model_registration

@dsl.pipeline(name="bms-training-pipeline")
def training_pipeline(git_commit: str = "latest"):
    step1 = data_ingestion(git_commit=git_commit)
    step2 = data_validation(data=step1.output)
    step3 = preprocessing(data=step2.output)
    step4 = model_training(data=step3.output)
    step5 = model_evaluation(mlflow_run_id=step4.output)
    step6 = model_registration(mlflow_run_id=step5.output)
```  
### Stage 1 — Data Ingestion:  
This step simply pulls that data into the pipeline from s3. And here's the important part — the S3 path is not hardcoded. It's passed as a **pipeline parameter,** so I can point the same pipeline to different data versions without changing any code.  
```
# Pipeline definition
@dsl.pipeline(name="bms-training-pipeline")
def training_pipeline(
    s3_data_path: str,    # ← this is the parameter
    model_name: str
):
```
**What happens when KFP pipeline triggers:**  
it asks us to pass commit id during run time. once we pass, then it runs below 3 command internally. and get matching data from s3. 
```  
1. git clone https://github.com/daimler/bms-ml.git     ← get the repo
2. git checkout abc123  (if specific version)          ← go to that point
3. dvc pull                                            ← reads .dvc pointer file & downloads matching data from S3   
4. Data is now available inside the container          ← ready for next step
   > Step 2: Data Validation (receives data from Step 1) --> Step 3: Preprocessing --> ..... and so on                       
  
```
**KFP Triggering Ways:**  
**CLI:** kfp run submit --pipeline bms-training-pipeline --param git_commit=abc123  
**GitHub Actions (production):** params: { "git_commit": "${{ github.sha }}" }   # ← no human involved at all

### Stage 2 — Data Validation: 
### Stage 3 — Preprocessing & Feature Engineering:
### Stage 4 — Model Training:
**mlops add MLflow logging code inside the training step**
I took the data scientist's training code as-is and wrapped it inside a KFP component. The core training logic remains unchanged. What I added around it is the MLflow integration — `log_param`, `log_metric`, `log_model`, log_artifact. Every single training run gets a unique MLflow run ID. which gets passed to the next pipeline step for evaluation and registry. `with mlflow.start_run() as run:` Everything inside this block gets tracked.  

### Stage 5 — Model Evaluation:
- After training, the model is evaluated against the test set — which was held out and never seen during training. This step computes the final metrics and also checks against predefined thresholds. For example, we have a rule — if F1 score for any fault class drops below 0.85, the model is not promoted.
- his is a quality gate. If the model passes, the pipeline moves to the next step. If it fails, the pipeline stops, metrics are logged in MLflow, and the team is notified to investigate.

### Step 4: Experiment Tracking & Model Registry (MLflow):
####  Experiment Tracking:  
In our setup, MLflow is deployed on Kubernetes (EKS) as a centralized tracking server. It has two backends — PostgreSQL for storing metadata like parameters, metrics, tags, and run information — and S3 for storing heavy artifacts like trained model files and evaluation reports. MLflow itself doesn't store anything — it's an API layer that routes data to the right backend.

When you call **`log_model` or `log_artifact`,** in training code. MLflow uploads the model file to S3. MLflow handles it because you already told it "artifacts go to S3" during MLflow server setup.  
> Each run gets its own folder in S3. Everything is organized automatically.  

When I call **log_param or log_metric** in the training code, The server then stores that information in its PostgreSQL backend.  

```  
PostgreSQL:  log_param(), log_metric(), set_tag(), etc 
S3:          log_model, log_artifact, log_artifacts
```

**The MLFlow architecture:**
```
Once Your training Code runs → MLflow Server(API layer which calls) → PostgreSQL (to store metadata)
                             → S3 (artifacts)
```  
#### Model Registry:
Once a model clears the evaluation quality gates — for example, F1 score per class must be above 0.85 — the pipeline's registration step calls mlflow.register_model().  MLflow auto-increments the version number — v1, v2, v3 and so on. And the initial stage is always set to Staging.
```  
log_model()        → uploads model to S3              (happens during training)
register_model()   → creates a pointer to that S3 path (happens during registration)
                   → assigns version number (v1, v2, v3... auto-incremented)
                   → sets stage = "Staging"
```
#### The Promotion Flow:  
When a new model version lands in Staging, three things happen:  

**First** — the team gets a notification — a Slack alert or email saying 'Model v5 is in Staging with F1 weighted 0.96, per-class metrics are XYZ'. This is triggered by a GitHub Actions workflow that listens for registry events.

**Second** — the data science team and I review the model. We look at the MLflow UI — compare the new model's metrics against the current production model side by side. MLflow UI makes this very easy — you select two runs and it shows a comparison table. We check — did accuracy improve? Did any class-level F1 drop? Is there any sign of overfitting?"

**Third** — if everything looks good, we approve the promotion. This is done by updating the model stage from Staging to Production."

"Now the moment a model version is promoted to Production in MLflow registry — that's not the end. That's actually the trigger for deployment. A webhook or a CI job detects this stage change, updates the model version in the deployment manifest in GitHub, and ArgoCD picks up the change and deploys the new model to the KServe inference endpoint. So the registry promotion is the handshake between MLflow and ArgoCD — MLflow says 'this model is ready', ArgoCD says 'I'll deploy it'."
```  
Manual promotion    → separate script (scripts/promote_model.py)
                    → human reviews and decides
                    → triggered via GitHub Actions workflow_dispatch

Automated promotion → inside pipeline registration step (Stage 6) (model_registration.py component)
                    → compares new model vs current production model
                    → promotes only if new model is better
                    → no human involved, fully automatic
```

## Step 5: CI/CD Pipeline (GitHub Actions + ArgoCD + DVC)


**In our project, deployment can be triggered by two scenarios:**
```  
Scenario 1: CODE changes    →  i. DS pushes new ML code like (training & preprocessing logic, feature eng, changes hyperparameters.) 
                               ii. infra changes like (increase replicas, autoscaling config (min/max pods), change memory/CPU limits for serving pods.
Scenario 2: MODEL changes   → new model version promoted to Production in MLflow registry
```
```  
Repo 1: github.com/daimler/bms-ml/           ← application code (pipeline, components)
Repo 2: github.com/daimler/bms-deployment/    ← deployment manifests (Kubernetes YAML)  
```

**When ML Code changes:** The DS pushes ML Code to a feature branch and raises a Pull Request (PR) to main. The moment the PR is created, GitHub Actions kicks off the CI pipeline. and execute all stages and  CI finishes its job by updating Repo 2 deployment manifest with new image tag. CD starts its job by detecting that update. That's how they connect.   
```ML code changes/DVC Data pointers Changes        → triggers retraining pipeline    → produces new model → deploy```

**When infra code changes:** — Repo 2 is edited DIRECTLY: No Repo 1, ci pipeline involved. No Docker build. No new image tag. Because you're just changing config values like replica count, not the actual application code.  
```Infrastructure changes  → directly updates Repo 2 manifest → ArgoCD deploys (no training)```  

GitHub Actions handles CI — linting, testing, Docker build, ECR push, and manifest update. ArgoCD handles CD — it watches the deployment repo and syncs any change to the Kubernetes cluster. Code changes and model changes both flow through this same pipeline. Git is the single source of truth. No manual deployments. Full audit trail."  

### SCENARIO 2: Model Change — Model Promotion triggers deployment
When a new model is promoted to Production in MLflow, a webhook fires and triggers a separate GitHub Actions workflow — not the code CI, but a model deployment workflow. This workflow fetches the new model version from MLflow, builds a new Docker serving image with the model baked in, pushes it to ECR, and updates the deployment manifest in Repo 2 with the new image tag. ArgoCD picks up the change and deploys the new KServe endpoint. So whether it's a code change or a model change, the flow always converges at the same point — Repo 2 is updated, ArgoCD syncs. One consistent deployment path.


**triggering types:**  
ci triggers when a code chage on PR or push/merge to main branch
ct triggers when a New Data Trigger, schedule , drift trigger
CD Pipeline Trigger — “When deployment state changes”

                              
.github/workflows/
├── ci-pipeline.yml                → CI (Continuous Integration)
├── training-pipeline.yml          → CT (Continuous Training)
├── model-deploy-pipeline.yml      → CD (Continuous Deployment/Delivery)


**CI Pipeline (ci-pipeline.yml):**  
```
push to main
    ↓
1. Checkout code
2. Lint (flake8 / ruff)
3. Unit tests (pytest)
4. Build training Docker image
5. Push training image to ECR
```
**CT Pipeline (training-pipeline.yml):** 
```
CI pipeline success
    ↓
1. Checkout code
2. Trigger KFP pipeline (via KFP SDK / REST API)
    │
    └── Inside KFP (these are KFP component steps):
        3. Data validation (Great Expectations)
        4. Feature engineering (preprocessing)
        5. Model training (XGBoost)
        6. Model evaluation (accuracy, F1, confusion matrix)
        7. Register model to MLflow Model Registry
        8. Promote model to "Staging" / "Production" in MLflow
    │

```
**CD Pipeline (model-deploy-pipeline.yml):** 

### Create a webhook that fires when model moves to Production  
MLflow doesn't natively trigger GitHub Actions, so we set up an MLflow Model Registry Webhook. When the model stage transitions to Production, MLflow fires an HTTP POST to the **GitHub repository dispatch API**. This triggers our GitHub Actions workflow CD Pipeline(model-deploy-pipeline.yml) which builds the Docker image and pushes it to ECR.  

```
client.create_registry_webhook(
    events=["MODEL_VERSION_TRANSITIONED_TO_PRODUCTION"],
    http_url_spec={
        "url": "https://api.github.com/repos/<org>/<repo>/dispatches",
        "secret": "your-secret-token",
        "authorization": f"token {GITHUB_PAT}"
    }
)
```

GitHub Actions side — listens for this event:  

```
# .github/workflows/model-deploy-pipeline.yml  
on:
  repository_dispatch:
    types: [model-promoted-to-production]   ** <-- webhook fires this**

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Build and Push Docker Image
        run: |
          docker build -t my-model .
         - name: Push to ECR
           ...
         - name: Update K8s Manifest
           ... 
```
```  
Stage 1 → Checkout Code from GitHub
Stage 2 → Connect to AWS
Stage 3 → Login to ECR
Stage 4 → Build Docker Image (model + dependencies packaged)
Stage 5 → Push Docker Image to ECR
Stage 6 → Update K8s Manifest with new image tag → Push to Git
Stage 7 → ArgoCD detects Git change → Deploys to EKS automatically
```

**What ArgoCD does internally during sync:**
```  
ArgoCD Sync Process:
      │
      ▼
Pull latest inference.yaml from Git
      │
      ▼
Compare with current cluster state
      │
      ▼
Generate kubectl apply commands:
  kubectl apply -f inference.yaml -n ml-production
      │
      ▼
Kubernetes API receives new InferenceService spec
      │
      ▼
KServe controller detects InferenceService changed
      │
      ▼
Begins rolling update (Step 4)
```
KServe Deploys Model Pod 
## 6. Model Serving (KServe) 
The Prediction Endpoint After Deployment: KServe exposes standard endpoint: 

Canary Deployment 
Rollback
## 7. Monitoring & Observability
## 8. Retraining Strategy









                          
**Scripts** 
python file:  mlops engineer add MLflow logging code inside the training step  for `log_param`, `log_metric`, `log_model`, log_artifact.  
promote_model.py              ← manual promotion script   
rollback_model.py             ← rollback if something goes wrong   





