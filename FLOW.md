# FLOW 

## ML Lifecycle:  
Business problem  
Data collection  
Data validation  
Feature engineering  
Model training  
Experiment tracking  
Model evaluation  
Model registry  
Deployment  
Monitoring  
Retraining  

## Project Approach:  
“I generally explain the methodology in three layers:  
First, from a project execution perspective, teams may use Agile or Scrum.  
Second, from an ML development perspective, we work iteratively through experimentation, feature engineering, training, and validation.  
Third, from an MLOps engineering perspective, our stack is best described as a pipeline-based CI/CD/CT architecture with GitOps deployment and continuous monitoring.  

### Agile Methodology
Agile is not a single method — it's a mindset and set of principles for building software iteratively. Scrum, Kanban, and others are frameworks that implement Agile.   
  > Agile thinking:      "Plan a little → Build a little → Get feedback → Repeat"

### Waterfall Methodology 
It's the oldest and simplest approach — everything flows in one direction, top to bottom, like a waterfall. You complete one phase fully before moving to the next. No going back. In Waterfall, during testing, the client says: Actually, we also need UPI payments. you're stuck — going back means restarting from requirements. That's expensive and slow.  when Requirements are 100% clear and won't change then we go waterfall.  
  > Waterfall thinking:  "Plan everything → Build everything → Deliver once"  

### Scrum Methodology
Scrum is an Agile framework where you build software in small, repeatable cycles called Sprints (usually 2 weeks). Instead of delivering everything at the end like Waterfall, you deliver a working piece every sprint.  
```
Scrum:      Sprint 1 → Deliver login
            Sprint 2 → Deliver transfers
            Sprint 3 → Deliver bill pay
            Sprint 4 → Deliver statements
            ...every 2 weeks, something working is delivered
```



### CRISP-DM (Data scientists approach)
Data scientists follow CRISP-DM (Cross-Industry Standard Process for Data Mining). It's the most widely used methodology in data science — like how Scrum is to software development.   
`Data Scientist -> CRISP-DM -> Problem → Data → Validation -> Feature Eng -> Model Training → Evaluation -> Deploy(Locally)`

1. Microsoft MLOps Maturity Model (Levels 0–4) : Your company uses Azure stack (Azure ML, Azure DevOps, AKS)
2. Uber's Michelangelo Approach : Very large company with 50+ data scientists
3. Netflix's Metaflow Approach  : Small-to-medium team that can't afford heavy MLOps infra
4. Feature Store Centric Approach (Feast/Tecton) : Multiple models share the same features
5. GitOps-Native MLOps (What You Actually Do) : Regulated industries (automotive, healthcare, finance)
6. Google's MLOps Level 2
   > `ML/MLOps Engineer -> **MLOps Level 2: CI/CD pipeline automation with GitOps-native deployment** -> Pipeline → CI/CD → Deploy → Monitor → Retrain`
   
### MLOps Level 2: CI/CD pipeline automation with GitOps-native deployment / Google's MLOps Level 2 with GitOps deployment Methodology
**Level 0 — Manual Process:**  
Everything is done by hand.  
A data scientist writes code in a Jupyter notebook, trains a model locally, tests it manually, and hands it over to an engineer who deploys it somehow. When the model degrades, someone notices eventually and the whole process repeats manually.  
Problem: Slow, error-prone, no reproducibility, breaks in production silently.  

**Level 1 — ML Pipeline Automation :** 
Instead of manual steps, you build an automated pipeline that handles the full flow: data ingestion → validation → preprocessing → training → evaluation → deployment.  The key addition here is Continuous Training (CT) — the pipeline can retrain automatically when triggered by new data or a schedule.  
Problem: The pipeline itself is still deployed manually. If a data scientist changes the training code, there's no CI/CD to test and deploy that pipeline change.  

**Level 2 — CI/CD + ML Pipeline Automation:**  
This is the full picture. Level 2 adds CI/CD for the pipeline code itself, not just for the model. Every component of the ML system — data processing, training logic, serving config — is treated as software and goes through automated testing, building, and deployment.
  - Level 2 — Complete Flow Using Your Stack

> **Team Flow:** Platform Engineering → Data Engineering → Data Science → MLOps Engineering

## 1. Project Initiation & Tech Stack:
I am working on Daimler Project which comes under automotive domain/Industry, their i am dealing with Battery Fault Classification. The project aims to classify battery cell faults in real time using sensor data from the Battery Management System (BMS). This helps detect issues like overheating, overcharging, internal short circuits, and cell degradation early — before they become safety-critical. The main purpose is to improve EV battery safety, and extend battery lifespan.  
> **Tech stack:**  Kubeflow Pipelines(KFP), Mlflow, ArgoCD, Kserve, Prometheus & Grafana, GitHub Actions, AWS, Git, GitHub, DVC, Docker, Kubernetes, Python, Linux.  
> **Classes(Output):** normal, thermal_runaway, cell_imbalance, sensor_drift, over_discharge  
## 2. What the Data Scientists hand over to MLOps

**Data ingestion from Vehical to s3**
The raw telemetry data lands in S3 via the ingestion pipeline from Data engineering teams. Now the data science team pull this raw data from s3, did their EDA, cleaned it, and future engineering steps and pushes the processed training dataset back into S3 as **Parquet files. i.e train.parquet**.
> so, s3 contains raw data, processed data, model artifacts, etc...

**Data scientists:** own everything related to model development — choosing the architecture, deciding whether to fine-tune a pre-trained model or train from scratch, selecting hyperparameters, running experiments in Jupyter notebooks, and evaluating which approach gives the best metrics. Fine-tuning is a modeling decision, so it falls squarely in their domain.  

**as a Senior MLOps Engineer.** My responsibility starts from the point where we need to productionize that model. That means — how do we take this model from a Jupyter notebook, build a proper training pipeline around it, package it, deploy it to production, serve it in real time, monitor it, and retrain it.   

### Fine-Tuning:
Fine-tuning means taking a pre-trained model (Transformer-based time-series pre-trained model like PatchTST) that already learned general patterns. You remove/freeze the last layer and replace it with a new layer for ev battery classes. Then we fine-tune on our dataset.  

**Types:**
Fixed Feature Extraction → Freeze the pre-trained backbone, train only the  head i.e last layer(Classification Layer).  
                 Small dataset + similar task → Fixed Feature Extractor.  
You stop updating their weights during training.  
Full Fine-Tuning → Unfreeze more layers, retrain them on your dataset.  
                 Bigger dataset + different task → Full Fine-Tuning.  
Many teams start with fixed extractor, and if results are not good, they move to progressive/full fine-tuning.  

**What is Classification Layer/Head:**  
At the end of every model, there is a final layer that gives predictions. Its job is to take all the features the earlier layers have extracted and map them to output classes. 

**Hyperparameter tuning:** — you're adjusting knobs like learning rate, max depth, number of estimators, etc. to get the best performance. The architecture and data stay the same

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





