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

 
# ML END-TO-END LIFECYCLE
```
--- CI Phase (GitHub Actions) ---

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
- The CI workflow steps are identical for both. The only differences are what triggers them, what code gets copied in, and what dependencies get installed.  
- Training Image: ```Triggered when: src/pipeline/ or requirements.txt changes```  
- Inference Image: ```Triggered when: any code commit```  

Your main ML lifecycle CI consumes the Training Image at step 8, it does not build it. The Training Image was already built and sitting in ECR from its own separate CI workflow. Your main pipeline just references it by tag. If the training image doesn't exist in ECR yet, step 8 will fail because KFP won't be able to pull the image to create the pods.  

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

<details>
<summary><b>Click to expand: ML END-TO-END LIFECYCLE</b></summary>
<details style="margin-left: 20px;">
<summary><i> --- CI Phase (GitHub Actions) --- </i></summary>  
This job is triggered on every push, PR, workflow_dispatch to the main branch.  
 
### 1. Code Commit (Git Push)
  This is the trigger point for the entire pipeline. Here's exactly what happens in real-world production:  
   
```   
git push origin main
        │
        ▼
GitHub detects a push event
        │
        ▼
Reads .github/workflows/ml-pipeline.yml
        │
        ▼
Spins up a GitHub Actions Runner  ← Step 2 begins here  
```  
### 2. CI Trigger (GitHub Actions)
   This is where GitHub detects your push and spins up the automation engine.  
   Think of it as a server that watches your repo and reacts to events. The moment your push lands on GitHub, it reads your workflow file and starts executing   jobs.  
   
```  
Your Machine          GitHub               GitHub Actions Runner
     │                   │                        │
     │── git push ──────►│                        │
     │                   │── detects push event   │
     │                   │── reads workflow yml ──►│
     │                   │                        │── spins up VM
     │                   │                        │── begins Step 3+  
```  
```
# .github/workflows/ml-pipeline.yml

name: ML End-to-End Pipeline

# ─────────────────────────────────────────
# TRIGGER RULES — what events fire this?
# ─────────────────────────────────────────
on:
  push:
    branches:
      - main                  # fires on every push to main
    paths:                    # ONLY if these files changed
      - 'src/**'              # any code change
      - 'params.yaml'         # hyperparameter change
      - 'dvc.yaml'            # pipeline structure change
      - 'requirements.txt'    # dependency change

  pull_request:
    branches:
      - main                  # fires when PR targets main
    paths:
      - 'src/**'
      - 'params.yaml'

  workflow_dispatch:          # allows MANUAL trigger from GitHub UI
    inputs:
      force_retrain:
        description: 'Force model retraining even if no changes'
        required: false
        default: 'false'
```
### Step 3: Code Checkout  
This is the first actual step that runs inside the GitHub Actions Runner VM. Remember — the VM is completely blank. It has no idea what your project looks like. Code Checkout fixes that.  

What the Runner Looks Like BEFORE Checkout  
```
GitHub Actions Runner VM (just provisioned)
├── /home/runner/
├── /home/runner/work/          ← empty
└── No code, no files, nothing
```
The Checkout Action  
```
# Inside .github/workflows/ml-pipeline.yml
jobs:
  code-quality:
    runs-on: ubuntu-latest

    steps:
      # ── STEP 3: Code Checkout ──────────────────────────────
      - name: Checkout Repository
        uses: actions/checkout@v4       # official GitHub action
        with:
          fetch-depth: 0                # fetch FULL git history (explained below)
          lfs: false                    # we don't use Git LFS (DVC handles large files)
          token: ${{ secrets.GITHUB_TOKEN }}  # auto-injected by GitHub
```
What actions/checkout@v4 Does Internally  
Under the hood, this action runs a sequence of git commands on the runner:  
```
# 1. Initialize a git repo in the runner's workspace
git init /home/runner/work/ml-project/ml-project

# 2. Add your GitHub repo as the remote origin
git remote add origin https://github.com/your-org/ml-project.git

# 3. Authenticate using the injected token
git config http.extraheader \
  "AUTHORIZATION: Bearer <GITHUB_TOKEN>"

# 4. Fetch the specific commit that triggered this run
git fetch --no-tags origin \
  refs/heads/main:refs/remotes/origin/main

# 5. Checkout exactly the commit that was pushed
git checkout --progress --force \
  a3f8c21d...   # ← the exact commit SHA from your git push
```

> **Critical:** It checks out the **exact commit SHA** that triggered the push — not "latest main." This ensures the pipeline runs on precisely what was pushed, even if someone else pushed 2 seconds later.

---

#### What the Runner Looks Like AFTER Checkout
```
/home/runner/work/ml-project/ml-project/
├── .github/
│   └── workflows/
│       └── ml-pipeline.yml
├── src/
│   ├── train.py
│   ├── evaluate.py
│   ├── preprocess.py
│   └── predict.py
├── pipelines/
│   └── kfp_pipeline.py
├── tests/
│   └── test_preprocess.py
├── dvc.yaml                    ← pipeline definition (no data yet)
├── dvc.lock                    ← data version pointers (no data yet)
├── params.yaml                 ← hyperparameters
├── requirements.txt
└── Dockerfile
```  
> **Notice:** Only code files exist. No actual data, no models. Those come in Step 7 (DVC Pull).

### Step 4: Install Dependencies
The runner has your code but can't run any of it yet. Python packages, CLI tools, nothing is installed. This step makes the environment execution-ready.  

The Workflow Step:
```
# Inside .github/workflows/ml-pipeline.yml

      # ── STEP 4: Install Dependencies ──────────────────────
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.10'        # pin exact Python version
          cache: 'pip'                  # cache pip downloads (explained below)

      - name: Install Dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
```
### Step 5: Lint Check (flake8) 
The runner now has code + dependencies. Before touching any expensive compute (training, data pulls), we validate code quality first. Catching a syntax error here saves 20–40 minutes of wasted pipeline time.  
The Workflow Step:  
```
# Inside .github/workflows/ml-pipeline.yml

      # ── STEP 5: Lint Check ─────────────────────────────────
      - name: Lint Check (flake8)
        run: |
          flake8 src/ pipelines/ \
            --max-line-length=88 \
            --max-complexity=10 \
            --ignore=E203,W503 \
            --statistics \
            --count
```
The .flake8 Config File (Production Standard)  
Rather than passing all flags inline, production teams use a config file:  
```
# .flake8  (lives at repo root, committed to git)

[flake8]
# ── Scope ──────────────────────────────────────────────
per-file-ignores =
    tests/*: E501           # allow long lines in test files
    pipelines/*: W503       # allow line breaks before binary operators

# ── Style Rules ────────────────────────────────────────
max-line-length = 88        # matches Black formatter standard
max-complexity  = 10        # McCabe complexity — catches overly tangled logic
indent-size     = 4

# ── Ignored Rules ──────────────────────────────────────
ignore =
    E203,   # whitespace before ':' — conflicts with Black
    W503    # line break before binary operator — conflicts with Black

# ── What to Scan ───────────────────────────────────────
filename =
    *.py

# ── What to Skip ───────────────────────────────────────
exclude =
    .git,
    __pycache__,
    .venv,
    build/,
    dist/,
    *.egg-info

# ── Output ─────────────────────────────────────────────
statistics = true           # summary count per error code
count = true                # total error count at end
show-source = true          # show the actual offending line
show-pep8 = true            # explain what rule was violated
```

---

#### What flake8 Actually Checks

flake8 bundles **three tools** internally:
```
flake8
  ├── PyFlakes     → logical errors (undefined names, unused imports)
  ├── pycodestyle  → PEP8 style violations (spacing, line length)
  └── McCabe       → cyclomatic complexity (tangled logic detection)
```
#### 1. PyFlakes — Catches Real Bugs:  
```
# src/train.py

import pandas as pd
import numpy as np
import requests          # ← F401: imported but unused

def train_model(df):
    X = df[features]     # ← F821: 'features' undefined — REAL BUG
    model.fit(X, y)      # ← F821: 'model' undefined — REAL BUG
    return modl          # ← F821: 'modl' undefined (typo) — REAL BUG
```
```
flake8 output:
src/train.py:3:1: F401 'requests' imported but unused
src/train.py:6:9: F821 undefined name 'features'
src/train.py:7:5: F821 undefined name 'model'
src/train.py:8:12: F821 undefined name 'modl'
4     F821 undefined name
1     F401 imported but unused
5
```
#### 2. pycodestyle — Catches Style Issues:  
```  
# src/preprocess.py

def preprocess(df,threshold,verbose=False):  # ← E231: missing spaces after ','
    if verbose==True:                         # ← E712: use 'if verbose:'
        x=df['col'].fillna(0)                # ← E225: missing whitespace around =
        very_long_variable_name = some_function_with_long_name(argument_one,argument_two,argument_three)  # ← E501: line too long
```
```
flake8 output:
src/preprocess.py:1:18: E231 missing whitespace after ','
src/preprocess.py:2:13: E712 comparison to True should be 'if cond is True:' or 'if cond:'
src/preprocess.py:3:10: E225 missing whitespace around operator
src/preprocess.py:4:89: E501 line too long (112 > 88 characters)
```
#### 3. McCabe — Catches Complex Logic
```
# src/evaluate.py

def evaluate_model(model, X_test, y_test, threshold, verbose, 
                   save_report, notify_slack, fallback):
    if threshold > 0.5:
        if verbose:
            if save_report:
                if notify_slack:
                    if fallback:        # ← deeply nested = high complexity
                        ...
```
```
flake8 output:
src/evaluate.py:1:1: C901 'evaluate_model' is too complex (complexity: 11 > 10)
```
### Step 6: Unit Tests (pytest)
Lint confirmed the code is syntactically clean. Unit tests now confirm the code logically works correctly. This is the last gate before any data or compute is touched.  
#### What Unit Tests Validate:  
Unit Tests verify:  
├── Data preprocessing logic produces correct outputs  
├── Feature engineering transformations are mathematically correct  
├── Model training function returns expected object types  
├── Evaluation metrics are calculated correctly  
├── Edge cases are handled (nulls, empty DataFrames, wrong dtypes)  
└── params.yaml is parsed correctly  
#### The Workflow Step:  
```
# Inside .github/workflows/ml-pipeline.yml

      # ── STEP 6: Unit Tests ─────────────────────────────────
      - name: Unit Tests (pytest)
        run: |
          pytest tests/ \
            --cov=src \
            --cov-report=xml \
            --cov-report=term-missing \
            --cov-fail-under=80 \
            -v \
            -x
        env:
          PYTHONPATH: ${{ github.workspace }}  # ensures src/ is importable
```

#### Flag Breakdown
```
pytest tests/               → scan everything inside tests/ folder
--cov=src                   → measure coverage of src/ code
--cov-report=xml            → output coverage.xml (for SonarCloud etc.)
--cov-report=term-missing   → print which exact lines are NOT covered
--cov-fail-under=80         → FAIL if coverage drops below 80%
-v                          → verbose: show each test name + pass/fail
-x                          → stop immediately on first failure
                              (don't run 50 tests after one breaks)
```

---

#### Project Test Structure
```
tests/
├── conftest.py                  ← shared fixtures (reusable test data)
├── test_preprocess.py           ← tests for src/preprocess.py
├── test_train.py                ← tests for src/train.py
├── test_evaluate.py             ← tests for src/evaluate.py
└── test_params.py               ← tests for params.yaml loading
```



  </details>
  <details style="margin-left: 20px;">
  <summary><i> --- CD Phase (ArgoCD + KServe) --- </i></summary>
  </details> 
  <details style="margin-left: 20px;">
  <summary><i> --- Monitoring & Retraining Loop --- </i></summary>
  </details> 
</details>  
