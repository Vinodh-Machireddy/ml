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

#### What flake8 Actually Checks
flake8 bundles **three tools** internally:
```
flake8
  ├── PyFlakes     → logical errors (undefined names, unused imports)
  ├── pycodestyle  → PEP8 style violations (spacing, line length)
  └── McCabe       → cyclomatic complexity (tangled logic detection)
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

## Training vs Inference Images
- The CI workflow steps are identical for both. The only differences are what triggers them, what code gets copied in, and what dependencies get installed.  
- Training Image: ```Triggered when: src/pipeline/ or requirements.txt changes```  
- Inference Image: ```Triggered when: any code commit```  

Your main ML lifecycle CI consumes the Training Image at step 8, it does not build it. The Training Image was already built and sitting in ECR from its own separate CI workflow. Your main pipeline just references it by tag. If the training image doesn't exist in ECR yet, step 8 will fail because KFP won't be able to pull the image to create the pods.  

**NOTE:** pipelines/kfp_pipeline.py — The Pipeline Definition

### Step 9: Model Registration (MLflow Registry)

The pipeline gate passed ✅. The model proved it meets the recall threshold. Now it needs to be formally registered — given a name, a version number, and stored in a central catalog that the entire organization can reference.  

> **pipelines/register_model.py** — The Registration Script  
   
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

> **NOTE** pipelines/promote_model.py — The Promotion Script  

## Step 11: Build Inference Docker Image
The model is now officially Production ✅. Step 11 packages everything needed to serve predictions — the model, its dependencies, and a REST API — into a single portable Docker image.  
The Workflow Step:  

> **NOTE** scripts/download_model.py — Pull Model from MLflow Before Build

  </details>
  
  <details style="margin-left: 20px;">
    <summary><i> --- CD Phase (ArgoCD + KServe) --- </i></summary>
  </details> 
  
  <details style="margin-left: 20px;">
    <summary><i> --- Monitoring & Retraining Loop --- </i></summary>  
  </details> 
  
</details>  




