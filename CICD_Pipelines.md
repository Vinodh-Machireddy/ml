# INTERVIEW TARGET
1. Ml Pipelines : Kubeflow Pipelines
2. Experiment Tracking + Model Registry: Mlflow 
3. Deployment and serving : ArgoCD + Kserve
4. Monitoring and alerting : Prometheus & Grafana
5. Ci/cd Pipelines : GitHub Actions
6. Cloud : AWS S3, ECR, EKS, IAM, SageMaker, Cost optimization, Security (AWS Secrets Manager)

# Production Ready Tech Stack: 
```Kubeflow Pipelines(KFP), Mlflow, ArgoCD, Kserve, Prometheus & Grafana, GitHub Actions, AWS, Git, GitHub, DVC, Docker, Kubernetes, Python, Linux.```

<div align="center"> 
 
# CI/CD Pipeline Using GitHub Actions

</div>

### To start ci/cd we need few source files like. 
- train.py which has main training script.
- api.py
- Dataset
- requirements.txt: file where dependencies (libraries/packages) that your project needs to run
- GitHub actions workflow file i.e main.yml:  This file defines the entire ci/cd pipeline. 
- .gitignore File: To avoid pushing unnecessary, temporary, or sensitive files into the repo. 
NOTE:- This files we get from DS and ML teams.

### What we do?(Manual work)
1. Initial Setup
2. DVC Setup (Data Version Control)
3. Push Model to S3
4. Kubernetes Cluster
5. KServe Setup
6. Test KServe Inference

To Automate Test, Build, And Deploy we use ci/cd pipelines using GitHub actions.

[GitHub Actions Repo Structure](<img width="281" height="442" alt="Screenshot 2026-02-11 at 3 11 23 PM" src="https://github.com/user-attachments/assets/97f4d798-7b12-43a0-9f4e-655594f48a66"/>)

### Ci/cd Pipeline Flow:
1. Checkout code
2. Setup Python
3. Install Dependencies
4. Unit tests (pytest)  # Code Validation
5. Lint check (flake8)
6. Security Scan (Trivy)
7. Generate Dataset 
8. Train Model 
9. Configure AWS Credentials
10. Install DVC
11. Push Model to S3 via DVC
12. Login to Amazon ECR
13. Build Inference Docker Image
14. Push Docker Image to ECR.  #CI Ends Here (Artifact + Image ready)
15. Update KServe Manifest inference.yaml with new image tag #CD starts here
16. Git Commit Deployment Config
17. ArgoCD Syncs Deployment
18. KServe Deploys Model pod
19. Monitoring + Alerts  
`Note:- Trigger Kubeflow Training pipeline (optional). Big companies go instead of running training in GitHub runner it runes in cluster, it is heavy lift.`

### Continuous Integration (CI)
This job is triggered on every push, PR, workflow_dispatch to the main branch.

### Continuous Deployment (CD)
 -  ArgoCD
 -  KServe

<details>
  <summary><b>Click to expand: MLOps ci/cd Pipeline Stages</b></summary>

  ### MLOps ci/cd Pipeline
```  
on:  
  push:  
    branches: [cicd]  
  workflow_dispatch:  
  repository_dispatch:  

env:  
  AWS_REGION: us-east-1  
  S3_BUCKET: churn-model-bucket-cicd-abhi  
  ECR_REPOSITORY: churn-model-repo  
  IMAGE_TAG: ${{ github.sha }}  

jobs:  
  train-and-deploy:  
    runs-on: ubuntu-latest  

    permissions:  
      contents: write  

    steps:  

    # ✅ 1. Checkout Code
    - name: Checkout code
      uses: actions/checkout@v3
      with:
        token: ${{ secrets.GITHUB_TOKEN }}

    # ✅ 2. Setup Python
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.11'

    # ✅ 3. Install Dependencies
    - name: Install dependencies
      run: |
        pip install -r requirements.txt
        pip install pytest flake8

    # ==================================================
    # ✅ NEW STAGE — CODE VALIDATION
    # ==================================================

    # ✅ Unit Tests
    - name: Run Unit Tests
      run: pytest

    # ✅ Lint Check
    - name: Run Lint Check
      run: flake8 .

    # ✅ Security Scan (Trivy)
    - name: Security Scan using Trivy
      uses: aquasecurity/trivy-action@master
      with:
        scan-type: 'fs'
        scan-ref: '.'

    # ==================================================
    # ✅ ML TRAINING
    # ==================================================

    # ✅ 4. Generate Dataset
    - name: Generate dataset
      run: python generate_data.py

    # ✅ 5. Train Model
    - name: Train model
      run: python train.py

    # ==================================================
    # ✅ MODEL VERSIONING
    # ==================================================

    # ✅ 6. Configure AWS Credentials
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ env.AWS_REGION }}

    # ✅ 7. Install DVC
    - name: Install DVC
      run: pip install dvc[s3]

    # ✅ 8. Push Model to S3 via DVC
    - name: Push model using DVC
      run: |
        dvc push
        echo "Model pushed to S3"

    # ==================================================
    # ✅ CONTAINER BUILD + REGISTRY
    # ==================================================

    # ✅ 9. Login to Amazon ECR
    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1

    # ✅ 10. Build Docker Image
    - name: Build Docker image
      run: |
        docker build -t ${{ steps.login-ecr.outputs.registry }}/${{ env.ECR_REPOSITORY }}:${{ env.IMAGE_TAG }} .

    # ✅ 11. Push Docker Image to ECR
    - name: Push Docker image
      run: |
        docker push ${{ steps.login-ecr.outputs.registry }}/${{ env.ECR_REPOSITORY }}:${{ env.IMAGE_TAG }}

    # ==================================================
    # ✅ GITOPS DEPLOYMENT PREP
    # ==================================================

    # ✅ 12. Update KServe inference.yaml
    - name: Update inference.yaml with new image
      run: |
        NEW_IMAGE="${{ steps.login-ecr.outputs.registry }}/${{ env.ECR_REPOSITORY }}:${{ env.IMAGE_TAG }}"
        sed -i "s|image: .*|image: $NEW_IMAGE|g" k8s/inference.yaml

    # ✅ 13. Commit Changes → Triggers ArgoCD Sync
    - name: Commit updated inference.yaml
      run: |
        git config --local user.email "action@github.com"
        git config --local user.name "GitHub Action"
        git add k8s/inference.yaml
        git commit -m "Update inference image to $IMAGE_TAG [skip ci]" || echo "No changes"
        git push || echo "No changes to push"

    # ==================================================
    # ✅ OPTIONAL — DEPLOYMENT NOTIFICATION / MONITORING HOOK
    # ==================================================

    - name: Deployment Notification
      run: echo "Deployment committed. ArgoCD + KServe will take over.”
    Note:- ArgoCD / KServe / Monitoring → cannot be inside GitHub YAML (they run after commit)

    # ------------------------------------------------
    # ✅ Wait for ArgoCD Sync
    # ------------------------------------------------
    - name: Wait for ArgoCD Sync
      run: |
        echo "Waiting for ArgoCD deployment..."
        sleep 60

    # ------------------------------------------------
    # ✅ Health Check After Deploy
    # ------------------------------------------------
    - name: KServe Health Check
      run: |
        curl -f https://your-kserve-endpoint/health || exit 1

    # ------------------------------------------------
    # ✅ Slack Notification
    # ------------------------------------------------
    - name: Slack Notification
      if: success()
      uses: slackapi/slack-github-action@v1
      with:
        payload: |
          {
            "text": "✅ Churn Model Deployment Success - Image Tag: ${{ env.IMAGE_TAG }}"
          }
      env:
        SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
   14.  ArgoCD
   15.  KServe
   16. Monitoring + alerting
```
   </details>

### Continuous Training(CT)
CT means automatically retraining the model when new data or drift is detected.

#### CT Triggered When
 1. Scheduled retraining time 
 2. and New Data Uploaded 
#### CT Steps Flow
1. Night 2 AM → New production data pulled
2. Data validation
3. Train Model
4. Evaluate Model (If accuracy > threshold → register model)
5. Push Model to DVC/S3
6. Trigger CD → Deploy new model

#### Ex:-
```on:
  schedule:
    - cron: "0 2 * * *"    # Runs daily at 2 AM
  workflow_dispatch:       # Manual trigger also allowed
```
#### Ex:-
```on:
  repository_dispatch:
    types: [new-data-arrived]
```

 


--- CI Phase (GitHub Actions) ---
```
1.  Code Commit (Git Push)
2.  CI Trigger (GitHub Actions)
3.  Code Checkout
4.  Install Dependencies
5.  Lint Check (flake8)
6.  Unit Tests (pytest)
7.  Data Pull & Versioning (DVC + S3)
8.  Model Training Pipeline — KFP Run triggered by GitHub Actions (polls for completion)
        8a. Training            (KFP component)
        8b. Experiment Tracking (MLflow — logs params, metrics, artifacts to S3)
        8c. Model Evaluation & Validation (KFP component — gates pipeline on metric threshold)
9.  Model Registration (MLflow Registry — model artifact stored in S3 via MLflow)
10. Model Promotion (Staging → Production in MLflow Registry)
11. Build Inference Docker Image
12. Push Docker Image to ECR

--- CD Phase (ArgoCD + KServe) ---

13. Update KServe Manifest (inference.yaml — new image tag + S3 model URI)
14. Git Commit & Push Deployment Config
15. ArgoCD Detects Drift & Syncs
16. KServe Deploys Model Pod (pulls image from ECR, model from S3)

--- Monitoring & Retraining Loop ---

17. Monitoring (Prometheus + Grafana — latency, throughput, error rate)
18. Data/Concept Drift Detection (custom Prometheus metrics)
19. Alert Trigger (Prometheus Alertmanager)
20. Retraining Pipeline Trigger (webhook → GitHub Actions → new KFP run)
21. New Model Version Generated (loops back to step 8)
22. Redeployment (loops back to step 13)  
```  
NOTE 8b: 
- When MLflow logs the model inside the KFP pipeline (step 8b), it writes directly to S3 automatically. There is no separate explicit upload action. 
- model artifact already stored in S3 via MLflow artifact store"


## How the KFP Training Pipeline Runs on Kubernetes  
When GitHub Actions reaches the "Model Training" step, it does not run the training code directly inside the GitHub Actions runner. Instead, it just sends a trigger (an API call) to the Kubeflow Pipelines API server, which then schedules and runs the actual training pipeline on your Kubernetes cluster.  
So the GitHub Actions step looks something like this in practice:  
```
# inside your GitHub Actions workflow step
kfp_client = kfp.Client(host="http://<your-kfp-endpoint>")
kfp_client.create_run_from_pipeline_func(
    training_pipeline,
    arguments={"data_path": "s3://...", "epochs": 10}
)
```
GitHub Actions owns CI orchestration (lint, test, build, push). KFP owns ML orchestration (data prep, train, evaluate, register). Mixing them would make your CI pipeline fragile and hard to debug.  

The only thing to be careful about
GitHub Actions needs to wait for the KFP run to finish before proceeding to the next steps (evaluation, S3 push, image build). If you fire-and-forget the KFP trigger, your CI pipeline will move on before training is done. So your GitHub Actions step should poll the KFP run status and only proceed on a successful completion status.  

### 1. Who Creates the Pods?
When GitHub Actions sends the API call to the KFP API server, the following chain happens:
```
GitHub Actions
    → API call to KFP API Server
        → KFP API Server creates an Argo Workflow object in Kubernetes
            → Argo Workflow Controller (running in the cluster) reads that object
                → Argo Controller creates Pods for each pipeline step
```  
So the direct answer is Argo Workflow Controller creates the pods. KFP under the hood uses Argo Workflows as its execution engine. You never create pods manually. KFP compiles your pipeline into an Argo Workflow YAML, submits it, and Argo takes over from there.  Each pipeline step runs in its own individual pod.  

## Training vs Inference Images
The CI workflow steps are identical for both. The only differences are what triggers them, what code gets copied in, and what dependencies get installed.  
Training Image: ```Triggered when: src/pipeline/ or requirements.txt changes```  
Inference Image: ```Triggered when: any code commit```  

```
Two Separate CI Flows Running in Parallel (independently)
│
├── Training Image CI (separate workflow)
│       Triggered when: src/pipeline/ or requirements.txt changes
│       Steps:
│           1. Code Checkout
│           2. Install Dependencies
│           3. Build Training Docker Image
│           4. Push Training Image to ECR
│               → ecr/training-image:sha-abc123
│       This flow is independent of your main ML lifecycle
│
└── Main ML Lifecycle CI (what you finalized)
        Triggered when: any code commit
        ...
        Step 8: Model Training Pipeline (KFP Run)
                    KFP pulls training-image:sha-abc123 from ECR
                    Spins up pods using this image
                    Runs your pipeline components inside those pods
```


### What's Inside Each Image
#### Training Image
```
FROM python:3.10-slim

WORKDIR /app

# ML training dependencies
COPY requirements.txt .
RUN pip install \
    torch \
    scikit-learn \
    pandas \
    numpy \
    mlflow \        # for experiment tracking & model logging
    boto3 \         # for S3 access (data + artifacts)
    kfp \           # for KFP component decorators
    dvc \           # for data versioning
    great-expectations  # for data validation

# Your pipeline source code
COPY src/pipeline/ ./src/pipeline/
# contains:
#   data_ingestion.py
#   data_validation.py
#   feature_engineering.py
#   train.py
#   evaluate.py
#   register.py
```
So when KFP spins up a pod for the train.py step, the pod has Python, all ML libraries, and your training code already inside it. It just runs.  

#### Inference Image
```
FROM python:3.10-slim

WORKDIR /app

# Inference/serving dependencies only
COPY requirements.txt .
RUN pip install \
    fastapi \       # or flask — to expose prediction endpoint
    uvicorn \       # ASGI server to serve fastapi
    mlflow \        # to load model from MLflow/S3
    boto3 \         # to pull model artifact from S3
    numpy \
    scikit-learn    # or torch — only what's needed to run the model

# Your inference source code
COPY src/inference/ ./src/inference/
# contains:
#   server.py       (FastAPI app — /predict endpoint)
#   model_loader.py (pulls model from S3/MLflow at startup)
#   preprocessor.py (same feature engineering logic as training)

EXPOSE 8080
CMD ["uvicorn", "src.inference.server:app", "--host", "0.0.0.0", "--port", "8080"]
```

