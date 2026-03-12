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
## ML END-TO-END LIFECYCLE 
<details>
<summary><b>Click-To-Expand: ML END-TO-END LIFECYCLE</b></summary>

<details>
<summary><b>CI Phase (GitHub Actions)</b></summary>

1. Code Commit (Git Push)  
2. CI Trigger (GitHub Actions)  
3. Code Checkout  
4. Install Dependencies  
5. Lint Check (flake8)  
6. Unit Tests (pytest)  
7. Data Pull & Versioning (DVC + S3)  

8. Model Training Pipeline — KFP Run triggered by GitHub Actions (polls for completion)

   <details>
   <summary>Training Pipeline Steps</summary>

   - 8a. Training (KFP component)  
   - 8b. Experiment Tracking (MLflow — logs params, metrics, artifacts to S3)  
   - 8c. Model Evaluation & Validation (KFP component — gates pipeline on metric threshold)  

   </details>
   NOTE 8b:  
   - When MLflow logs the model inside the KFP pipeline (step 8b), it writes directly to S3 automatically. There is no separate explicit upload action.  
   - model artifact already stored in S3 via MLflow artifact store  
9. Model Registration (MLflow Registry — model artifact stored in S3 via MLflow)  
10. Model Promotion (Staging → Production in MLflow Registry)  
11. Build Inference Docker Image  
12. Push Docker Image to ECR  

</details>

<details>
<summary><b>CD Phase (ArgoCD + KServe)</b></summary>

13. Update KServe Manifest (inference.yaml — new image tag + S3 model URI)  
14. Git Commit & Push Deployment Config  
15. ArgoCD Detects Drift & Syncs  
16. KServe Deploys Model Pod (pulls image from ECR, model from S3)  

</details>

<details>
<summary><b>Monitoring & Retraining Loop</b></summary>

17. Monitoring (Prometheus + Grafana — latency, throughput, error rate)  
18. Data / Concept Drift Detection (custom Prometheus metrics)  
19. Alert Trigger (Prometheus Alertmanager)  
20. Retraining Pipeline Trigger (webhook → GitHub Actions → new KFP run)  
21. New Model Version Generated (loops back to step 8)  
22. Redeployment (loops back to step 13)  

</details>

</details>  

--- 

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
### Step 7: Data Pull & Versioning (DVC + S3)
Code is verified clean. Now the pipeline needs real data. This step is where DVC bridges the gap between Git (code versioning) and S3 (data storage), pulling the exact data version that corresponds to this commit.  
DVC is an open-source tool designed to handle large files, datasets, and machine learning models — things that Git alone can't manage efficiently.  
Think of it this way:  
Git tracks your code changes (versions)  
DVC tracks your data changes (versions)  
Together, they give you complete version control over both code and data.  

S3 (Amazon Simple Storage Service)  
S3 is Amazon's cloud storage service. It acts as a remote storage location where your actual large data files are stored.  

### How DVC + S3 Work Together  
The problem:  
Git is not built to handle large files like datasets (e.g., 10GB CSV files). Pushing them to GitHub would be slow and impractical.  
The solution:  
DVC stores the actual data files in S3 (cloud) and keeps only a small pointer/reference file (.dvc file) in your Git repository.  

**Simple workflow:**
1. You add a large dataset using dvc add data.csv
2. DVC creates a small data.csv.dvc file (just a pointer)
3. You push the actual data to S3 using dvc push
4. The pointer file goes to Git, the real data goes to S3
5. When a teammate needs the data, they run dvc pull — it fetches the exact version from S3

**Data Pull** means downloading the correct version of data from S3 using DVC. Versioning means tracking every change to your dataset over time, so you can go back to any previous version.   

The Workflow Step:  
```
# Inside .github/workflows/ml-pipeline.yml
# This runs in Job 2: ml-pipeline (after code-quality job passed)

      # ── STEP 7: Data Pull & Versioning ────────────────────
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id:     ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region:            us-east-1

      - name: Configure DVC Remote
        run: |
          dvc remote add -d s3remote s3://ml-project-data/files
          dvc remote modify s3remote region us-east-1

      - name: Pull Data from S3 (DVC)
        run: |
          dvc pull --run-cache           # pulls data + cached pipeline outputs
```
### Step 8: Model Training Pipeline (KFP Run)
This is the heart of the entire lifecycle. The runner now has code ✅, dependencies ✅, and data ✅. It's time to fire actual model training — but NOT on the GitHub Actions runner itself. The runner orchestrates training on a dedicated Kubeflow Pipelines (KFP) cluster.  
> The runner's job in Step 8: compile the pipeline, submit it to Kubeflow, and poll until it finishes. The actual compute happens inside the Kubeflow cluster.  
The Big Picture of Step 8:

```  
GitHub Actions          →  "Hey KFP, run this pipeline"
KFP API Server          →  "OK, let me translate this into something Kubernetes understands"
Argo Workflow Object    →  "I'm just a YAML definition sitting in Kubernetes"
Argo Workflow Controller→  "I read that definition and CREATE the actual pods"
Kubernetes              →  "I schedule and RUN those pods on available nodes"
```
When GitHub Actions reaches the "Model Training" step, it does not run the training code directly inside the GitHub Actions runner. Instead, it just sends a trigger (an API call) to the Kubeflow Pipelines API server, which then schedules and runs the actual training pipeline on your Kubernetes cluster.  
So the GitHub Actions step looks something like this in practice:   
```
# Simply says "hey, run this pipeline"
kfp.Client(host="https://kfp-server-url").create_run_from_pipeline_package(
    pipeline_file="my_pipeline.yaml",
    arguments={"param1": "value1"}
)
```  
GitHub Actions owns CI orchestration (lint, test, build, push). KFP owns ML orchestration (data prep, train, evaluate, register). Mixing them would make your CI pipeline fragile and hard to debug.  

The only thing to be careful about GitHub Actions needs to wait for the KFP run to finish before proceeding to the next steps (evaluation, S3 push, image build). If you fire-and-forget the KFP trigger, your CI pipeline will move on before training is done. So your GitHub Actions step should poll the KFP run status and only proceed on a successful completion status.  

Step 1 — GitHub Actions (The Trigger)  
This is just the starting point. Your CI/CD pipeline makes an API call   

Step 2 — KFP API Server (The Translator) KFP receives the request and thinks:  
> "OK, this pipeline has 3 steps — preprocess, train, evaluate. Let me convert this into an Argo Workflow YAML and submit it to Kubernetes."
KFP does NOT create pods directly. It just creates an Argo Workflow object (a Kubernetes custom resource).

Step 3 — Argo Workflow Object (Just a Definition)
This is just a YAML file stored in Kubernetes. It describes WHAT should run, but by itself it does nothing. Think of it like a blueprint — it's just sitting there waiting.  
**Step 4 — Argo Workflow Controller (The REAL Pod Creator)**   
This is the **key player**. The Argo Controller is a process **constantly running inside your Kubernetes cluster**, watching for new Workflow objects. When it sees one:  

> "Oh, a new workflow appeared! Let me read it and create pods for each step."
```
Argo Controller sees the workflow object
    → Creates Pod for "preprocess" step
    → Waits for it to finish
    → Creates Pod for "train" step
    → Waits for it to finish
    → Creates Pod for "evaluate" step
```
**The Argo Controller is the one who actually creates the pods.**   

**Step 5 — Kubernetes (The Executor)**  
Once the pod is created, **Kubernetes scheduler** decides:  

> "Which node has enough CPU/memory to run this pod?"  

Then it schedules and runs the pod on an available node.    



#### pipelines/kfp_pipeline.py — The Pipeline Definition

### Step 9: Model Registration (MLflow Registry)

The pipeline gate passed ✅. The model proved it meets the recall threshold. Now it needs to be formally registered — given a name, a version number, and stored in a central catalog that the entire organization can reference.  
The Workflow Step:  
```
# Inside .github/workflows/ml-pipeline.yml

      # ── STEP 9: Model Registration ─────────────────────────
      - name: Register Model in MLflow
        run: |
          python pipelines/register_model.py \
            --tracking-uri  ${{ secrets.MLFLOW_TRACKING_URI }} \
            --model-name    "churn-prediction-model" \
            --git-commit    ${{ github.sha }} \
            --run-id        ${{ env.MLFLOW_RUN_ID }}
        env:
          MLFLOW_TRACKING_URI: ${{ secrets.MLFLOW_TRACKING_URI }}
          AWS_ACCESS_KEY_ID:   ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```
> **pipelines/register_model.py** — The Registration Script  
```
# pipelines/register_model.py
# Written by: MLOps Engineer (once at project setup)

import mlflow
from mlflow.tracking import MlflowClient
import argparse
import os
import sys


def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument("--tracking-uri", required=True)
    parser.add_argument("--model-name",   required=True)
    parser.add_argument("--git-commit",   required=True)
    parser.add_argument("--run-id",       required=True)
    return parser.parse_args()


def main():
    args = parse_args()

    mlflow.set_tracking_uri(args.tracking_uri)
    client = MlflowClient()

    # ── Fetch the run that just completed ─────────────────────
    run = client.get_run(args.run_id)
    metrics = run.data.metrics
    params  = run.data.params

    print(f"Run ID:    {args.run_id}")
    print(f"Recall:    {metrics['recall']:.4f}")
    print(f"F1:        {metrics['f1']:.4f}")
    print(f"ROC-AUC:   {metrics['roc_auc']:.4f}")

    # ── Get the latest version just created by Step 8b ────────
    # Step 8b already called mlflow.sklearn.log_model() which
    # auto-created a new version — we just need to find it
    latest_versions = client.get_latest_versions(
        name=args.model_name,
        stages=["None"]          # newly registered = "None" stage
    )

    if not latest_versions:
        print("ERROR: No model version found in 'None' stage")
        sys.exit(1)

    # Get the most recently created version
    model_version = sorted(
        latest_versions,
        key=lambda v: v.creation_timestamp,
        reverse=True
    )[0]

    version_number = model_version.version
    print(f"Model version found: v{version_number}")

    # ── Add description ────────────────────────────────────────
    client.update_model_version(
        name=args.model_name,
        version=version_number,
        description=(
            f"RandomForest churn model | "
            f"Recall: {metrics['recall']:.4f} | "
            f"F1: {metrics['f1']:.4f} | "
            f"ROC-AUC: {metrics['roc_auc']:.4f} | "
            f"Commit: {args.git_commit[:8]}"
        )
    )

    # ── Add tags to this version ───────────────────────────────
    client.set_model_version_tag(
        name=args.model_name,
        version=version_number,
        key="git_commit",
        value=args.git_commit
    )
    client.set_model_version_tag(
        name=args.model_name,
        version=version_number,
        key="recall",
        value=str(round(metrics["recall"], 4))
    )
    client.set_model_version_tag(
        name=args.model_name,
        version=version_number,
        key="dataset_version",
        value="customers_v3"       # could pull from dvc.lock dynamically
    )
    client.set_model_version_tag(
        name=args.model_name,
        version=version_number,
        key="ci_run_id",
        value=os.environ.get("GITHUB_RUN_ID", "unknown")
    )

    # ── Transition to Staging ──────────────────────────────────
    client.transition_model_version_stage(
        name=args.model_name,
        version=version_number,
        stage="Staging",
        archive_existing_versions=False   # keep old Staging versions visible
    )

    print(f"Model v{version_number} transitioned to Staging")

    # ── Export version number for downstream steps ─────────────
    # GitHub Actions reads this to pass to Step 10
    with open(os.environ["GITHUB_ENV"], "a") as f:
        f.write(f"MODEL_VERSION={version_number}\n")
        f.write(f"MLFLOW_RUN_ID={args.run_id}\n")

    print("Registration complete.")


if __name__ == "__main__":
    main()
```

---

#### What MLflow Registry Looks Like After Step 9
```
MLflow Registry
└── Registered Model: "churn-prediction-model"
      │
      ├── Version 1  [Production]    ← previous good model (still serving)
      │     ├── Stage:       Production
      │     ├── Recall:      0.861
      │     ├── Git commit:  b2e1d90f...
      │     ├── Registered:  2024-10-15 09:32:11
      │     └── Artifact:    s3://ml-project-mlflow/artifacts/
      │                        churn-prediction-model/1/model.pkl
      │
      ├── Version 2  [Archived]      ← old staging candidate (superseded)
      │     ├── Stage:       Archived
      │     ├── Recall:      0.823   ← failed threshold, archived
      │     └── Git commit:  c9f3a44e...
      │
      └── Version 3  [Staging]       ← just registered NOW in Step 9
            ├── Stage:       Staging
            ├── Recall:      0.887   ← passed gate ✅
            ├── F1:          0.910
            ├── ROC-AUC:     0.951
            ├── Git commit:  a3f8c21d...   ← exact commit that produced it
            ├── CI Run ID:   8473920156
            ├── Dataset:     customers_v3
            ├── Registered:  2024-11-20 14:17:43
            └── Artifact:    s3://ml-project-mlflow/artifacts/
                               churn-prediction-model/3/
                                 ├── model.pkl
                                 ├── MLmodel
                                 └── conda.yaml
```

---

#### What Lives in S3 After Registration
```
s3://ml-project-mlflow/
└── artifacts/
    └── churn-prediction-model/
        └── 3/                           ← version number
            └── model/
                ├── model.pkl            ← serialized RandomForest
                ├── MLmodel              ← MLflow model metadata
                │     flavor: sklearn
                │     python_version: 3.10.14
                │     sklearn_version: 1.4.1
                │     time_created: 2024-11-20T14:17:43
                ├── conda.yaml           ← conda environment spec
                └── requirements.txt     ← pip requirements for this model
```
> **NOTE:** The artifact path in S3 is permanent and immutable. MLflow never overwrites a registered version's artifacts — every version is preserved forever for audit and rollback.

### Step 10: Model Promotion (Staging → Production)
The model is in Staging ✅. This step is the final human + automated gate before a model is officially declared Production and inference infrastructure is built around it.  
The Two-Layer Promotion Design:  
```  
Gate 1 — Automated (Step 8c)
──────────────────────────────
  Did recall exceed 0.85?
  YES → proceed to Staging
  NO  → pipeline dies

Gate 2 — Human Approval (Step 10)
──────────────────────────────────
  Did a senior Data Scientist / ML Lead
  review the full metrics, fairness report,
  and business impact before approving?
  APPROVED → transition to Production
  REJECTED → stays in Staging, pipeline stops
```
#### How Human Approval Works in GitHub Actions:  
```  
# Inside .github/workflows/ml-pipeline.yml  

jobs:

  # ── Job 3: Human Approval Gate ────────────────────────────
  await-approval:
    name: Await Human Approval for Production Promotion
    runs-on: ubuntu-latest
    needs: ml-pipeline              # runs after Step 9 completes
    environment: production         # ← THIS is the key

    steps:
      - name: Approval Gate
        run: |
          echo "Model v${{ env.MODEL_VERSION }} awaiting promotion approval"
          echo "Recall:  ${{ env.MODEL_RECALL }}"
          echo "F1:      ${{ env.MODEL_F1 }}"
          echo "Commit:  ${{ github.sha }}"
```

---

#### The `environment: production` Key — How GitHub Enforces Human Approval
```
GitHub Repo → Settings → Environments → production
──────────────────────────────────────────────────────
  ├── Required reviewers:
  │     ├── @john-ml-lead       ← must approve
  │     └── @sarah-data-science ← must approve (either one)
  │
  ├── Wait timer: 0 minutes     ← no auto-approval after delay
  │
  └── Deployment branches:
        └── main only           ← can't promote from feature branches
```
```
What the approver sees in GitHub UI:
──────────────────────────────────────
┌─────────────────────────────────────────────────┐
│  Workflow: ML End-to-End Pipeline               │
│  Job: Promote Model to Production               │
│                                                 │
│  Model v3 is ready for Production promotion     │
│  Recall:  0.887                                 │
│  F1:      0.910                                 │
│  Commit:  a3f8c21d                              │
│                                                 │
│  MLflow UI: https://mlflow.internal/models/...  │
│                                                 │
│  [ Approve and deploy ]   [ Reject ]            │
└─────────────────────────────────────────────────┘
```
> Pipeline is completely paused at this point. The GitHub Actions runner waits up to 30 days for approval. If nobody approves, it times out and the model stays in Staging.
#### After Approval — The Promotion Step Runs
```
# ── Job 4: Execute Promotion ──────────────────────────────
  promote-model:
    name: Promote Model to Production
    runs-on: ubuntu-latest
    needs: await-approval           # only runs after human approves

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: '3.10'
          cache: 'pip'

      - run: pip install mlflow==2.12.1

      - name: Promote Model to Production
        run: |
          python pipelines/promote_model.py \
            --tracking-uri  ${{ secrets.MLFLOW_TRACKING_URI }} \
            --model-name    "churn-prediction-model" \
            --version       ${{ env.MODEL_VERSION }}
        env:
          MLFLOW_TRACKING_URI:   ${{ secrets.MLFLOW_TRACKING_URI }}
          AWS_ACCESS_KEY_ID:     ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```
#### pipelines/promote_model.py — The Promotion Script  
```
# pipelines/promote_model.py
# Written by: MLOps Engineer (once at project setup)

import mlflow
from mlflow.tracking import MlflowClient
import argparse
import os
import sys


def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument("--tracking-uri", required=True)
    parser.add_argument("--model-name",   required=True)
    parser.add_argument("--version",      required=True, type=int)
    return parser.parse_args()


def main():
    args = parse_args()

    mlflow.set_tracking_uri(args.tracking_uri)
    client = MlflowClient()

    # ── Fetch current Production version ──────────────────────
    current_production = client.get_latest_versions(
        name=args.model_name,
        stages=["Production"]
    )

    # ── Safety Check: confirm Staging version still exists ────
    staging_version = client.get_model_version(
        name=args.model_name,
        version=str(args.version)
    )

    if staging_version.current_stage != "Staging":
        print(
            f"ERROR: Version {args.version} is not in Staging. "
            f"Current stage: {staging_version.current_stage}"
        )
        sys.exit(1)

    print(f"Promoting v{args.version} from Staging → Production")

    # ── Promote new version to Production ─────────────────────
    client.transition_model_version_stage(
        name=args.model_name,
        version=str(args.version),
        stage="Production",
        archive_existing_versions=True   # ← auto-archives old Production
    )

    # ── Add promotion metadata ─────────────────────────────────
    client.set_model_version_tag(
        name=args.model_name,
        version=str(args.version),
        key="promoted_by",
        value=os.environ.get("GITHUB_ACTOR", "unknown")  # who approved
    )
    client.set_model_version_tag(
        name=args.model_name,
        version=str(args.version),
        key="promoted_at",
        value=str(os.environ.get("GITHUB_RUN_ID", "unknown"))
    )
    client.set_model_version_tag(
        name=args.model_name,
        version=str(args.version),
        key="replaced_version",
        value=str(current_production[0].version) if current_production else "none"
    )

    # ── Log what was archived ──────────────────────────────────
    if current_production:
        old_version = current_production[0].version
        print(f"Previous Production v{old_version} → Archived")
        print(f"New Production:      v{args.version}")
    else:
        print(f"First Production version: v{args.version}")

    # ── Export for downstream steps ───────────────────────────
    with open(os.environ["GITHUB_ENV"], "a") as f:
        f.write(f"PRODUCTION_MODEL_VERSION={args.version}\n")
        if current_production:
            f.write(f"ARCHIVED_MODEL_VERSION={current_production[0].version}\n")

    print("Promotion complete. Proceeding to Docker build.")


if __name__ == "__main__":
    main()
```

---

#### What MLflow Registry Looks Like After Promotion
```
MLflow Registry
└── Registered Model: "churn-prediction-model"
      │
      ├── Version 1  [Archived]      ← was Production, now archived
      │     ├── Stage:       Archived               (auto by archive_existing_versions=True)
      │     ├── Recall:      0.861
      │     └── replaced_by: v3
      │
      ├── Version 2  [Archived]      ← was Staging candidate, still archived
      │     ├── Stage:       Archived
      │     └── Recall:      0.823
      │
      └── Version 3  [Production]    ← just promoted NOW ✅
            ├── Stage:       Production
            ├── Recall:      0.887
            ├── F1:          0.910
            ├── ROC-AUC:     0.951
            ├── git_commit:  a3f8c21d...
            ├── promoted_by: john-ml-lead     ← GitHub actor who approved
            ├── promoted_at: 8473920156       ← CI run ID
            └── replaced_version: 1
```

---

#### The Rollback Story — Why Archiving Matters
```
Scenario: v3 goes to Production, but 2 hours later
          business reports a spike in false positives

Rollback in 2 commands:
────────────────────────────────────────────────────
# Re-promote the old archived version instantly
client.transition_model_version_stage(
    name="churn-prediction-model",
    version="1",                    ← old Production version
    stage="Production",
    archive_existing_versions=True  ← archives broken v3
)

# v1 model.pkl is still in S3 — never deleted
# inference server picks it up immediately
# no retraining needed
# full audit trail of what happened and when
```

---

#### How the Inference Server Knows What Changed
```
Inference server (Step 12+) loads model like this:

model = mlflow.pyfunc.load_model(
    model_uri="models:/churn-prediction-model/Production"
                                              ↑
                                   always points to current
                                   Production stage version
                                   
Before promotion: resolves to v1 → loads v1 model.pkl
After promotion:  resolves to v3 → loads v3 model.pkl

No hardcoded version numbers in serving code.
Stage alias handles the switching automatically.
```

---

#### Who Writes `promote_model.py`
```
Same pattern as register_model.py:

Written ONCE by:   MLOps Engineer during project setup
Never touched by:  Data Scientist, Data Engineer
Modified only if:  MLflow API changes
                   Approval workflow changes
                   New metadata tags required
                   Rollback logic added
```

---

#### The Complete Promotion Flow
```
Step 9: Model Registered as Staging v3 ✓
      │
      ▼
GitHub Actions pauses at environment: production
      │
      ├── Email sent to: john-ml-lead, sarah-data-science
      ├── They review MLflow UI (metrics, fairness, artifacts)
      ├── They review GitHub Actions run (full CI log)
      └── Decision:
            │
            ├── REJECTED → pipeline stops
            │               model stays in Staging
            │               team investigates
            │
            └── APPROVED → pipeline resumes
                  │
                  ▼
            promote_model.py runs:
              ├── v3: Staging    → Production ✅
              ├── v1: Production → Archived
              ├── promoted_by: john-ml-lead logged
              └── PRODUCTION_MODEL_VERSION=3 exported
                  │
                  ▼
            Step 11: Build Inference Docker Image ← next
```

---

#### Full Picture After Step 10
```
Step 9:  Model Registration ✓
      │  └── v3 in Staging
      ▼
Step 10: Model Promotion ✓
      │
      ├── Human approval gate passed
      ├── v3: Staging → Production
      ├── v1: Production → Archived (rollback available)
      ├── Full audit trail: who promoted, when, what replaced
      ├── Inference server alias "Production" now resolves to v3
      └── PRODUCTION_MODEL_VERSION=3 in GitHub Actions env
      │
      ▼
Step 11: Build Inference Docker Image ← next
```
## Step 11: Build Inference Docker Image
The model is now officially Production ✅. Step 11 packages everything needed to serve predictions — the model, its dependencies, and a REST API — into a single portable Docker image.  
The Workflow Step:  
```
# Inside .github/workflows/ml-pipeline.yml

  # ── Job 5: Build & Push Docker Image ──────────────────────
  build-and-push:
    name: Build Inference Docker Image
    runs-on: ubuntu-latest
    needs: promote-model              # only runs after promotion ✅

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      # ── Configure AWS credentials for ECR access ───────────
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id:     ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region:            us-east-1

      # ── Login to Amazon ECR ────────────────────────────────
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      # ── Download model artifact from MLflow/S3 ─────────────
      - name: Download Production Model from MLflow
        run: |
          python scripts/download_model.py \
            --tracking-uri ${{ secrets.MLFLOW_TRACKING_URI }} \
            --model-name   "churn-prediction-model" \
            --stage        "Production" \
            --output-path  "./model"
        env:
          MLFLOW_TRACKING_URI:   ${{ secrets.MLFLOW_TRACKING_URI }}
          AWS_ACCESS_KEY_ID:     ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

      # ── Build Docker Image ─────────────────────────────────
      - name: Build Docker Image
        run: |
          docker build \
            --build-arg MODEL_VERSION=${{ env.PRODUCTION_MODEL_VERSION }} \
            --build-arg GIT_COMMIT=${{ github.sha }} \
            --tag churn-inference:${{ github.sha }} \
            --tag churn-inference:latest \
            .
```
#### scripts/download_model.py — Pull Model from MLflow Before Build
#### The Dockerfile 
```
# Dockerfile
# Written by: MLOps Engineer (once at project setup)

# ── Base Image ─────────────────────────────────────────────────
FROM python:3.10-slim

# ── Build Arguments (injected by GitHub Actions) ───────────────
ARG MODEL_VERSION=unknown
ARG GIT_COMMIT=unknown

# ── Environment Variables (available at runtime) ───────────────
ENV MODEL_VERSION=${MODEL_VERSION}
ENV GIT_COMMIT=${GIT_COMMIT}
ENV PYTHONUNBUFFERED=1
ENV PYTHONDONTWRITEBYTECODE=1

# ── System Dependencies ────────────────────────────────────────
RUN apt-get update && apt-get install -y \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# ── Working Directory ──────────────────────────────────────────
WORKDIR /app

# ── Install Python Dependencies ────────────────────────────────
# Copy requirements FIRST — Docker layer cache optimization
# If requirements.txt unchanged → this layer is cached
# Even if code changes → pip install is skipped (fast rebuild)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# ── Copy Inference Server Code ─────────────────────────────────
COPY src/predict.py ./src/predict.py

# ── Copy Model Artifact ────────────────────────────────────────
# Downloaded from MLflow/S3 in the GitHub Actions step above
# Baked directly into the image — no S3 call at inference time
COPY model/ ./model/

# ── Security: non-root user ────────────────────────────────────
RUN useradd --create-home --shell /bin/bash appuser
USER appuser

# ── Expose Port ────────────────────────────────────────────────
EXPOSE 8080

# ── Health Check ───────────────────────────────────────────────
HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
    CMD curl -f http://localhost:8080/health || exit 1

# ── Start Server ───────────────────────────────────────────────
CMD ["uvicorn", "src.predict:app", \
     "--host", "0.0.0.0", \
     "--port", "8080", \
     "--workers", "4"]
```
## Step 12: Push Docker Image to ECR
The Workflow Step:  
```
# Inside .github/workflows/ml-pipeline.yml

      # ── STEP 12: Push Docker Image to ECR ──────────────────
      - name: Push Image to ECR
        id: push-image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_NAME:   churn-inference
          IMAGE_TAG:    ${{ github.sha }}
        run: |
          # Full ECR image URI
          ECR_IMAGE="$ECR_REGISTRY/$IMAGE_NAME:$IMAGE_TAG"
          ECR_LATEST="$ECR_REGISTRY/$IMAGE_NAME:latest"

          # Tag local image with full ECR URI
          docker tag churn-inference:$IMAGE_TAG $ECR_IMAGE
          docker tag churn-inference:$IMAGE_TAG $ECR_LATEST

          # Push both tags to ECR
          docker push $ECR_IMAGE
          docker push $ECR_LATEST

          # Export full image URI for downstream deployment steps
          echo "IMAGE_URI=$ECR_IMAGE" >> $GITHUB_ENV
          echo "Pushed: $ECR_IMAGE"

      # ── Verify image is in ECR ──────────────────────────────
      - name: Verify ECR Push
        run: |
          aws ecr describe-images \
            --repository-name churn-inference \
            --image-ids imageTag=${{ github.sha }} \
            --region us-east-1
```

  </details>
  
  <details style="margin-left: 20px;">
    <summary><i> --- CD Phase (ArgoCD + KServe) --- </i></summary>
  </details> 
  
  <details style="margin-left: 20px;">
    <summary><i> --- Monitoring & Retraining Loop --- </i></summary>  
  </details> 
  
</details>  




