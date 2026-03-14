## Pipelines Big Picture. 
The entire ML pipeline has 4 major phases:  
```
Phase 1: CI (Continuous Integration)    → Code push → Test → Build
Phase 2: CT (Continuous Training)       → Train → Evaluate → Register
Phase 3: CD (Continuous Deployment)     → Deploy model to production
Phase 4: CM (Continuous Monitoring)     → Monitor → Detect drift → Retrain
```
## GitHub repository Folder structure
```
/home/runner/work/ml-project/ml-project/
├── .github/
│   └── workflows/
│       └── ml-pipeline.yml              ← GitHub Actions CI workflow
│
├── src/
│   ├── train.py                         ← model training
│   ├── evaluate.py                      ← model evaluation
│   ├── preprocess.py                    ← data preprocessing
│   └── predict.py                       ← inference logic
│
├── pipelines/                           ← ALL Kubeflow related files
│   ├── kfp_pipeline.py                  ← FILE 1: pipeline definition
│   │   ├── @component definitions       ← pipeline components
│   │   │   ├── preprocess component
│   │   │   ├── train component
│   │   │   └── evaluate component
│   │   │
│   │   └── @pipeline assembly           ← connects all components
│   │
│   └── submit_pipeline.py               ← FILE 2: compile + submit + poll
│       ├── compile logic                ← compile KFP pipeline YAML
│       ├── submit logic                 ← submit run to Kubeflow
│       └── polling loop                 ← monitor pipeline status
│
├── tests/
│   └── test_preprocess.py               ← unit tests
│
├── dvc.yaml                             ← DVC pipeline stages
├── dvc.lock                             ← data version tracking
├── params.yaml                          ← hyperparameters
├── requirements.txt                     ← Python dependencies
├── Dockerfile                           ← container image for pipeline
│
└── README.md                            ← project documentation
```
---  

## ML LIFECYCLE STEPS
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
   - 8a. Training (KFP component)  
   - 8b. Experiment Tracking (MLflow — logs params, metrics, artifacts to S3)  
   - 8c. Model Evaluation & Validation (KFP component — gates pipeline on metric threshold)  
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
## ML END-TO-END LIFECYCLE (CI, CT, CD, CM)
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
This is the heart of the entire lifecycle. The runner now has code ✅, dependencies ✅, and data ✅. It's time to fire actual model training. 

In a production MLOps setup, GitHub Actions runners do not perform the actual model training. Training usually requires high compute resources, persistent volumes, GPUs/CPUs, and long execution time, which are not suitable for GitHub runners. Instead, the runner’s role is only to orchestrate the training process on a dedicated Kubeflow Pipelines (KFP) cluster.

When the github Actions reaches the “Model Training” step, GitHub Actions simply triggers the training pipeline. The runner first **compiles the Kubeflow pipeline**, which means converting the pipeline Python file into a YAML specification that Kubeflow can understand and execute. After compilation, the runner **submits** this kfp_pipeline.yaml file to the Kubeflow Pipelines API server.

Once submitted, the Kubeflow Pipelines system schedules and runs the pipeline inside the Kubernetes cluster, where the actual compute resources are available. The GitHub Actions **runner then polls** the pipeline status periodically until the execution finishes (Succeeded or Failed).

> **NOTE:** we wrote kfp_pipeline.py in Python, But Kubeflow cluster does NOT understand Python files. Kubeflow only understands YAML (Kubernetes language).

#### The Full Journey — Step by Step
```
GitHub Actions Runner              Kubeflow Cluster (Kubernetes)
──────────────────────             ─────────────────────────────

PHASE 1: SUBMIT
───────────────

submit_pipeline.py runs
      │
      │ 1. Connect to Kubeflow API
      ▼
client = kfp.Client(host="https://kubeflow.internal")
      │
      │ 2. Send kfp_pipeline.yaml + params via HTTP POST
      ▼
POST https://kubeflow.internal/apis/v2beta1/runs
Body: {
  pipeline_spec: <contents of pipeline.yaml>,
  run_name: "run-a3f8c21d",
  experiment_name: "churn-prediction",
  arguments: {
    n_estimators: 200,
    max_depth: 10,
    ...
  }
}
      │
      │ 3. Kubeflow receives → creates Run object
      ▼                        returns run_id instantly
run_id = "abc123xyz"          ◄────────────────────────────────
      │
      │ 4. Runner gets run_id back
      │    Kubeflow is NOW working in background
      │    Runner does NOT wait — it starts polling
      ▼


PHASE 2: KUBEFLOW EXECUTES (background)
────────────────────────────────────────

                                Kubeflow Pipeline Controller reads kfp_pipeline.yaml
                                      │
                                      ▼
                                Creates Kubernetes Pods for each component:

                                Pod 1 (train_component):
                                  image: python:3.10-slim
                                  command: pip install... && python3 -c "..."
                                  resources: CPU=8, RAM=32Gi, GPU=1
                                  STATUS: Running
                                      │
                                      │ (training finishes — 30-40 mins)
                                      ▼
                                Pod 1 STATUS: Completed ✅
                                model.pkl saved to shared storage (MinIO)
                                      │
                                      ▼
                                Pod 2 (mlflow_tracking_component):
                                  Starts ONLY after Pod 1 completes
                                  Reads model.pkl from shared storage
                                  Logs to MLflow
                                  STATUS: Running → Completed ✅
                                      │
                                      ▼
                                Pod 3 (evaluate_and_gate_component):
                                  Starts ONLY after Pod 2 completes
                                  Checks recall > 0.85
                                  STATUS: Running → Completed ✅ or Failed ❌

                                Overall Run STATUS: Succeeded or Failed


PHASE 3: POLLING (runner checks every 30 seconds)
──────────────────────────────────────────────────

Runner polls Kubeflow API:
      │
      ├── [0s]   GET /apis/v2beta1/runs/abc123xyz → status: Running
      ├── [30s]  GET /apis/v2beta1/runs/abc123xyz → status: Running
      ├── [60s]  GET /apis/v2beta1/runs/abc123xyz → status: Running
      │           ...
      │           (20-40 minutes pass)
      │           ...
      ├── [1800s] GET /apis/v2beta1/runs/abc123xyz → status: Succeeded
      │
      ▼
status == "Succeeded" → sys.exit(0)  ← runner continues to Step 9
status == "Failed"    → sys.exit(1)  ← runner stops, pipeline fails
```

**What submit_pipeline.py Contains:** compile logic + submit logic + polling loop 
```
# pipelines/submit_pipeline.py
# Written by: MLOps Engineer (ONE file, THREE responsibilities)
# ─────────────────────────────────────────────────────────────
# RESPONSIBILITY 1: Compile   (Python → YAML)
# RESPONSIBILITY 2: Submit    (YAML → Kubeflow API)
# RESPONSIBILITY 3: Poll      (check status every 30s)
# ─────────────────────────────────────────────────────────────

import kfp
import kfp.compiler as compiler
import argparse
import yaml
import time
import sys
import os
from pipelines.kfp_pipeline import ml_pipeline   # imports FILE 1


def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument("--endpoint",    required=True)
    parser.add_argument("--experiment",  required=True)
    parser.add_argument("--run-name",    required=True)
    parser.add_argument("--params-file", required=True)
    return parser.parse_args()


def main():
    args   = parse_args()
    params = yaml.safe_load(open(args.params_file))

    # ══════════════════════════════════════════════════════════
    # RESPONSIBILITY 1: COMPILE
    # Converts kfp_pipeline.py Python → pipeline.yaml (YAML)
    # ══════════════════════════════════════════════════════════
    print("Compiling pipeline...")
    compiler.Compiler().compile(
        pipeline_func=ml_pipeline,        # from kfp_pipeline.py
        package_path="pipeline.yaml"      # output on runner disk
    )
    print("Compiled → pipeline.yaml ✅")

    # ══════════════════════════════════════════════════════════
    # RESPONSIBILITY 2: SUBMIT
    # Sends pipeline.yaml to Kubeflow API
    # Returns run_id immediately — does not block
    # ══════════════════════════════════════════════════════════
    print("Connecting to Kubeflow...")
    client = kfp.Client(host=args.endpoint)

    print("Submitting pipeline run...")
    run = client.create_run_from_pipeline_package(
        pipeline_file="pipeline.yaml",
        arguments={
            "n_estimators":         params["train"]["n_estimators"],
            "max_depth":            params["train"]["max_depth"],
            "random_state":         params["train"]["random_state"],
            "target_column":        params["train"]["target_column"],
            "min_recall_threshold": params["train"]["min_recall_threshold"],
            "mlflow_tracking_uri":  os.environ["MLFLOW_TRACKING_URI"],
            "git_commit":           os.environ["GITHUB_SHA"],
        },
        run_name=args.run_name,
        experiment_name=args.experiment,
    )

    run_id = run.run_id
    print(f"Submitted ✅  Run ID: {run_id}")
    print(f"Kubeflow UI:  {args.endpoint}/#/runs/details/{run_id}")

    # ══════════════════════════════════════════════════════════
    # RESPONSIBILITY 3: POLL
    # Checks run status every 30 seconds
    # Blocks runner until Kubeflow finishes
    # Passes result back via sys.exit code
    # ══════════════════════════════════════════════════════════
    poll_interval = 30
    timeout       = 7200    # 2 hours max
    elapsed       = 0

    while elapsed < timeout:
        status = client.get_run(run_id).run.status
        print(f"[{elapsed}s] Status: {status}")

        if status == "Succeeded":
            print("Pipeline SUCCEEDED ✅")

            # Write MLflow run_id to GitHub Actions env
            # Step 9 (register_model.py) reads this
            mlflow_run_id = extract_mlflow_run_id(client, run_id)
            with open(os.environ["GITHUB_ENV"], "a") as f:
                f.write(f"MLFLOW_RUN_ID={mlflow_run_id}\n")

            sys.exit(0)     # ← GitHub Actions sees ✅ → Step 9 starts

        elif status in ("Failed", "Error", "Cancelled"):
            print(f"Pipeline {status} ❌")
            sys.exit(1)     # ← GitHub Actions sees ❌ → pipeline stops

        time.sleep(poll_interval)
        elapsed += poll_interval

    print("Timeout exceeded.")
    sys.exit(1)


def extract_mlflow_run_id(client, run_id):
    """Parse MLflow run ID from KFP run artifacts."""
    run_detail = client.get_run(run_id)
    # KFP stores component outputs as artifacts
    # mlflow_tracking_component wrote run_id as an output artifact
    for artifact in run_detail.run.run_details.task_details:
        if "mlflow_run_id" in artifact.outputs:
            return artifact.outputs["mlflow_run_id"]


if __name__ == "__main__":
    main()
```  

#### Training vs Inference Images
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
## Step 12: Push Docker Image to ECR  
The image is built and smoke-tested on the runner ✅. This final step pushes it to Amazon ECR — the central image registry where your Kubernetes/ECS inference infrastructure pulls from to actually serve predictions.  

 
  <details>
    <summary><i> Complete ML CI Pipeline — ml-pipeline.yml </i></summary>  
   
```  
   
# .github/workflows/ml-pipeline.yml
# ═══════════════════════════════════════════════════════════════════════════════
# ML END-TO-END CI PIPELINE
# Steps: Code Quality → Data Pull → KFP Training → Registration →
#        Promotion → Docker Build → ECR Push
# ═══════════════════════════════════════════════════════════════════════════════

name: ML End-to-End Pipeline

# ───────────────────────────────────────────────────────────────────────────────
# STEP 1 & 2: TRIGGER RULES
# Fires on push to main (only if relevant files changed)
# or manually via workflow_dispatch
# ───────────────────────────────────────────────────────────────────────────────
on:
  push:
    branches:
      - main
    paths:
      - 'src/**'
      - 'pipelines/**'
      - 'params.yaml'
      - 'dvc.yaml'
      - 'requirements.txt'
      - 'Dockerfile'

  pull_request:
    branches:
      - main
    paths:
      - 'src/**'
      - 'params.yaml'

  workflow_dispatch:
    inputs:
      force_retrain:
        description: 'Force model retraining even if no changes'
        required: false
        default: 'false'

# ───────────────────────────────────────────────────────────────────────────────
# CONCURRENCY: Cancel in-progress runs on same branch
# Prevents duplicate model versions in MLflow Registry
# ───────────────────────────────────────────────────────────────────────────────
concurrency:
  group: ml-pipeline-${{ github.ref }}
  cancel-in-progress: true

# ───────────────────────────────────────────────────────────────────────────────
# GLOBAL ENVIRONMENT VARIABLES
# Secrets injected from GitHub → Settings → Secrets and Variables → Actions
# ───────────────────────────────────────────────────────────────────────────────
env:
  AWS_ACCESS_KEY_ID:      ${{ secrets.AWS_ACCESS_KEY_ID }}
  AWS_SECRET_ACCESS_KEY:  ${{ secrets.AWS_SECRET_ACCESS_KEY }}
  AWS_DEFAULT_REGION:     us-east-1
  MLFLOW_TRACKING_URI:    ${{ secrets.MLFLOW_TRACKING_URI }}
  KFP_ENDPOINT:           ${{ secrets.KFP_ENDPOINT }}
  ECR_REGISTRY:           ${{ secrets.ECR_REGISTRY }}
  MODEL_NAME:             churn-prediction-model
  IMAGE_NAME:             churn-inference
  PYTHON_VERSION:         '3.10'


# ═══════════════════════════════════════════════════════════════════════════════
# JOB 1: CODE QUALITY
# Steps 3 → 4 → 5 → 6
# Fast, cheap — runs before any expensive compute
# ═══════════════════════════════════════════════════════════════════════════════
jobs:
  code-quality:
    name: "Steps 3-6 | Code Quality (Checkout → Lint → Test)"
    runs-on: ubuntu-latest

    steps:

      # ─────────────────────────────────────────────────────────────────────────
      # STEP 3: CODE CHECKOUT
      # Checks out exact commit SHA that triggered this run
      # fetch-depth: 0 = full git history (needed for DVC diffs)
      # ─────────────────────────────────────────────────────────────────────────
      - name: "Step 3 | Checkout Repository"
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
          lfs: false
          token: ${{ secrets.GITHUB_TOKEN }}

      # ─────────────────────────────────────────────────────────────────────────
      # STEP 4: INSTALL DEPENDENCIES
      # Pins Python version, caches pip downloads
      # cache: 'pip' saves ~90 seconds on cache hit
      # ─────────────────────────────────────────────────────────────────────────
      - name: "Step 4 | Set Up Python ${{ env.PYTHON_VERSION }}"
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}
          cache: 'pip'

      - name: "Step 4 | Install Dependencies"
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements-dev.txt

      - name: "Step 4 | Verify Critical Packages"
        run: |
          python -c "import sklearn;  print('sklearn: ',  sklearn.__version__)"
          python -c "import mlflow;   print('mlflow:   ', mlflow.__version__)"
          python -c "import kfp;      print('kfp:      ', kfp.__version__)"
          python -c "import dvc;      print('dvc:      ', dvc.__version__)"
          aws --version
          pip list

      # ─────────────────────────────────────────────────────────────────────────
      # STEP 5: LINT CHECK (flake8)
      # Catches: undefined names, unused imports, style violations,
      #          overly complex functions (McCabe complexity > 10)
      # Exit code 1 = pipeline stops, no compute wasted
      # ─────────────────────────────────────────────────────────────────────────
      - name: "Step 5 | Lint Check (flake8)"
        run: |
          flake8 src/ pipelines/ \
            --max-line-length=88 \
            --max-complexity=10 \
            --ignore=E203,W503 \
            --statistics \
            --count

      # ─────────────────────────────────────────────────────────────────────────
      # STEP 6: UNIT TESTS (pytest)
      # Validates: preprocessing logic, training function, metric calculation
      # --cov-fail-under=80 → fails if coverage drops below 80%
      # -x → stops on first failure (no point running 50 tests after one breaks)
      # ─────────────────────────────────────────────────────────────────────────
      - name: "Step 6 | Unit Tests (pytest)"
        run: |
          pytest tests/ \
            --cov=src \
            --cov-report=xml \
            --cov-report=term-missing \
            --cov-fail-under=80 \
            -v \
            -x
        env:
          PYTHONPATH: ${{ github.workspace }}

      - name: "Step 6 | Upload Coverage Report"
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: coverage-report
          path: coverage.xml


# ═══════════════════════════════════════════════════════════════════════════════
# JOB 2: ML PIPELINE
# Steps 7 → 8 → 9
# Only runs if Job 1 (code-quality) fully passes
# Touches real data, real compute, real MLflow Registry
# ═══════════════════════════════════════════════════════════════════════════════
  ml-pipeline:
    name: "Steps 7-9 | ML Pipeline (Data → Train → Register)"
    runs-on: ubuntu-latest
    needs: code-quality                  # HARD dependency — Job 1 must pass

    steps:

      - name: "Checkout Repository"
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: "Set Up Python ${{ env.PYTHON_VERSION }}"
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}
          cache: 'pip'

      - name: "Install Dependencies"
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      # ─────────────────────────────────────────────────────────────────────────
      # STEP 7: DATA PULL & VERSIONING (DVC + S3)
      # Reads dvc.lock → pulls exact data version tied to this Git commit
      # --run-cache → skips pipeline stages whose inputs haven't changed
      # ─────────────────────────────────────────────────────────────────────────
      - name: "Step 7 | Configure AWS Credentials"
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id:     ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region:            us-east-1

      - name: "Step 7 | Configure DVC Remote (S3)"
        run: |
          dvc remote add -d s3remote s3://ml-project-data/files
          dvc remote modify s3remote region us-east-1

      - name: "Step 7 | Pull Data from S3 (DVC)"
        run: |
          dvc pull --run-cache
          echo "Data pull complete. Verifying files..."
          ls -lh data/train/
          ls -lh data/test/

      # ─────────────────────────────────────────────────────────────────────────
      # STEP 8: MODEL TRAINING PIPELINE (Kubeflow Pipelines)
      # Compiles kfp_pipeline.py → submits to Kubeflow cluster
      # Polls every 30 seconds until pipeline succeeds or fails
      # Inside Kubeflow:
      #   8a → Training pod (GPU, 32GB RAM)
      #   8b → MLflow tracking pod (logs params, metrics, artifact to S3)
      #   8c → Evaluation gate pod (recall > 0.85 or pipeline dies)
      # ─────────────────────────────────────────────────────────────────────────
      - name: "Step 8 | Compile & Submit KFP Pipeline"
        run: |
          python pipelines/submit_pipeline.py \
            --endpoint    ${{ secrets.KFP_ENDPOINT }} \
            --experiment  "churn-prediction" \
            --run-name    "run-${{ github.sha }}" \
            --params-file params.yaml
        env:
          MLFLOW_TRACKING_URI:   ${{ secrets.MLFLOW_TRACKING_URI }}
          AWS_ACCESS_KEY_ID:     ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          GITHUB_SHA:            ${{ github.sha }}
          GITHUB_ACTOR:          ${{ github.actor }}
          GITHUB_RUN_ID:         ${{ github.run_id }}

      # submit_pipeline.py writes MLFLOW_RUN_ID to GITHUB_ENV
      # All downstream steps read it via ${{ env.MLFLOW_RUN_ID }}

      # ─────────────────────────────────────────────────────────────────────────
      # STEP 9: MODEL REGISTRATION (MLflow Registry)
      # Finds model version created by Step 8b
      # Adds description + tags (git_commit, recall, dataset, ci_run_id)
      # Transitions: None → Staging
      # Exports MODEL_VERSION to GitHub Actions env for Steps 10-12
      # ─────────────────────────────────────────────────────────────────────────
      - name: "Step 9 | Register Model in MLflow Registry"
        run: |
          python pipelines/register_model.py \
            --tracking-uri ${{ secrets.MLFLOW_TRACKING_URI }} \
            --model-name   ${{ env.MODEL_NAME }} \
            --git-commit   ${{ github.sha }} \
            --run-id       ${{ env.MLFLOW_RUN_ID }}
        env:
          MLFLOW_TRACKING_URI:   ${{ secrets.MLFLOW_TRACKING_URI }}
          AWS_ACCESS_KEY_ID:     ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          GITHUB_RUN_ID:         ${{ github.run_id }}

      # register_model.py writes MODEL_VERSION to GITHUB_ENV


# ═══════════════════════════════════════════════════════════════════════════════
# JOB 3: HUMAN APPROVAL GATE
# Step 10 (Part 1) — pauses pipeline, notifies reviewers
# Senior Data Scientist / ML Lead reviews metrics in MLflow UI
# Pipeline waits up to 30 days for approval
# ═══════════════════════════════════════════════════════════════════════════════
  await-approval:
    name: "Step 10 | Await Human Approval for Production Promotion"
    runs-on: ubuntu-latest
    needs: ml-pipeline
    environment: production              # GitHub Environment with required reviewers

    steps:
      - name: "Step 10 | Display Model Info for Reviewer"
        run: |
          echo "══════════════════════════════════════════"
          echo "  Model Awaiting Production Promotion"
          echo "══════════════════════════════════════════"
          echo "  Model Name:    ${{ env.MODEL_NAME }}"
          echo "  Version:       ${{ env.MODEL_VERSION }}"
          echo "  Git Commit:    ${{ github.sha }}"
          echo "  Triggered By:  ${{ github.actor }}"
          echo "  CI Run:        ${{ github.run_id }}"
          echo "  MLflow UI:     ${{ secrets.MLFLOW_TRACKING_URI }}"
          echo "══════════════════════════════════════════"
          echo "  Review metrics before approving:"
          echo "  Recall, Precision, F1, ROC-AUC"
          echo "  Feature importance, fairness report"
          echo "══════════════════════════════════════════"


# ═══════════════════════════════════════════════════════════════════════════════
# JOB 4: PROMOTE MODEL
# Step 10 (Part 2) — runs only after human approves
# Transitions model: Staging → Production
# Archives old Production version (rollback always available)
# ═══════════════════════════════════════════════════════════════════════════════
  promote-model:
    name: "Step 10 | Promote Model (Staging → Production)"
    runs-on: ubuntu-latest
    needs: await-approval

    steps:
      - name: "Checkout Repository"
        uses: actions/checkout@v4

      - name: "Set Up Python ${{ env.PYTHON_VERSION }}"
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}
          cache: 'pip'

      - name: "Install MLflow"
        run: pip install mlflow==2.12.1

      - name: "Step 10 | Promote Model to Production"
        run: |
          python pipelines/promote_model.py \
            --tracking-uri ${{ secrets.MLFLOW_TRACKING_URI }} \
            --model-name   ${{ env.MODEL_NAME }} \
            --version      ${{ env.MODEL_VERSION }}
        env:
          MLFLOW_TRACKING_URI:   ${{ secrets.MLFLOW_TRACKING_URI }}
          AWS_ACCESS_KEY_ID:     ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          GITHUB_ACTOR:          ${{ github.actor }}
          GITHUB_RUN_ID:         ${{ github.run_id }}

      # promote_model.py writes PRODUCTION_MODEL_VERSION to GITHUB_ENV


# ═══════════════════════════════════════════════════════════════════════════════
# JOB 5: BUILD & PUSH DOCKER IMAGE
# Steps 11 → 12
# Only runs after model is officially in Production
# Packages model.pkl + FastAPI server into portable Docker image
# Pushes to ECR with 3 tags: git SHA, model version, environment
# ═══════════════════════════════════════════════════════════════════════════════
  build-and-push:
    name: "Steps 11-12 | Build Inference Image & Push to ECR"
    runs-on: ubuntu-latest
    needs: promote-model

    steps:
      - name: "Checkout Repository"
        uses: actions/checkout@v4

      - name: "Set Up Python ${{ env.PYTHON_VERSION }}"
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}
          cache: 'pip'

      - name: "Install Dependencies"
        run: pip install mlflow==2.12.1 boto3==1.34.69

      # ─────────────────────────────────────────────────────────────────────────
      # STEP 11 (Part 1): CONFIGURE AWS + ECR LOGIN
      # Gets temporary ECR auth token (valid 12 hours)
      # ─────────────────────────────────────────────────────────────────────────
      - name: "Step 11 | Configure AWS Credentials"
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id:     ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region:            us-east-1

      - name: "Step 11 | Login to Amazon ECR"
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      # ─────────────────────────────────────────────────────────────────────────
      # STEP 11 (Part 2): DOWNLOAD PRODUCTION MODEL FROM MLFLOW/S3
      # Pulls model artifact BEFORE Docker build
      # model/ folder is then COPY'd into the image
      # Model baked in = no S3 calls at inference time
      # ─────────────────────────────────────────────────────────────────────────
      - name: "Step 11 | Download Production Model from MLflow"
        run: |
          python scripts/download_model.py \
            --tracking-uri ${{ secrets.MLFLOW_TRACKING_URI }} \
            --model-name   ${{ env.MODEL_NAME }} \
            --stage        "Production" \
            --output-path  "./model"
          echo "Model artifact downloaded. Contents:"
          ls -lh ./model/
        env:
          MLFLOW_TRACKING_URI:   ${{ secrets.MLFLOW_TRACKING_URI }}
          AWS_ACCESS_KEY_ID:     ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

      # ─────────────────────────────────────────────────────────────────────────
      # STEP 11 (Part 3): BUILD DOCKER IMAGE
      # --build-arg injects MODEL_VERSION + GIT_COMMIT as ENV vars
      # Layer cache: pip install skipped if requirements.txt unchanged
      # ─────────────────────────────────────────────────────────────────────────
      - name: "Step 11 | Build Docker Image"
        run: |
          docker build \
            --build-arg MODEL_VERSION=${{ env.PRODUCTION_MODEL_VERSION }} \
            --build-arg GIT_COMMIT=${{ github.sha }} \
            --tag ${{ env.IMAGE_NAME }}:${{ github.sha }} \
            --tag ${{ env.IMAGE_NAME }}:latest \
            .
          echo "Image built. Size:"
          docker images ${{ env.IMAGE_NAME }}

      # ─────────────────────────────────────────────────────────────────────────
      # STEP 11 (Part 4): SMOKE TEST BUILT IMAGE
      # Starts container, hits /health and /predict endpoints
      # Catches broken inference server BEFORE pushing to ECR
      # ─────────────────────────────────────────────────────────────────────────
      - name: "Step 11 | Smoke Test Docker Image"
        run: |
          # Start container
          docker run -d \
            --name test-inference \
            -p 8080:8080 \
            ${{ env.IMAGE_NAME }}:${{ github.sha }}

          # Wait for FastAPI startup
          sleep 15

          # Health check
          echo "Testing /health endpoint..."
          curl -f http://localhost:8080/health
          echo ""

          # Single prediction
          echo "Testing /predict endpoint..."
          curl -f -X POST http://localhost:8080/predict \
            -H "Content-Type: application/json" \
            -d '{"age": 35, "tenure_months": 24, "monthly_spend": 80.0}'
          echo ""

          # Batch prediction
          echo "Testing /predict/batch endpoint..."
          curl -f -X POST http://localhost:8080/predict/batch \
            -H "Content-Type: application/json" \
            -d '{
              "instances": [
                {"age": 25, "tenure_months": 12, "monthly_spend": 50.0},
                {"age": 45, "tenure_months": 60, "monthly_spend": 150.0}
              ]
            }'
          echo ""

          # Tear down
          docker stop test-inference
          docker rm test-inference
          echo "Smoke tests passed ✅"

      # ─────────────────────────────────────────────────────────────────────────
      # STEP 12: PUSH DOCKER IMAGE TO ECR
      # Three tags pushed:
      #   1. Git SHA     → immutable, for rollback + audit
      #   2. v{N}-production → human readable, links to MLflow version
      #   3. production  → mutable, what Kubernetes pulls
      # Only changed layers uploaded (Docker layer deduplication)
      # ─────────────────────────────────────────────────────────────────────────
      - name: "Step 12 | Tag Images for ECR"
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        run: |
          ECR_BASE="$ECR_REGISTRY/${{ env.IMAGE_NAME }}"

          # Tag 1: Git SHA (immutable)
          docker tag ${{ env.IMAGE_NAME }}:${{ github.sha }} \
            $ECR_BASE:${{ github.sha }}

          # Tag 2: Model version (human readable)
          docker tag ${{ env.IMAGE_NAME }}:${{ github.sha }} \
            $ECR_BASE:v${{ env.PRODUCTION_MODEL_VERSION }}-production

          # Tag 3: Environment tag (Kubernetes pulls this)
          docker tag ${{ env.IMAGE_NAME }}:${{ github.sha }} \
            $ECR_BASE:production

          echo "ECR_BASE=$ECR_BASE" >> $GITHUB_ENV

      - name: "Step 12 | Push All Tags to ECR"
        run: |
          # Push all three tags
          docker push ${{ env.ECR_BASE }}:${{ github.sha }}
          docker push ${{ env.ECR_BASE }}:v${{ env.PRODUCTION_MODEL_VERSION }}-production
          docker push ${{ env.ECR_BASE }}:production

          # Export full image URI for deployment step
          echo "IMAGE_URI=${{ env.ECR_BASE }}:${{ github.sha }}" >> $GITHUB_ENV
          echo "Pushed successfully:"
          echo "  → ${{ env.ECR_BASE }}:${{ github.sha }}"
          echo "  → ${{ env.ECR_BASE }}:v${{ env.PRODUCTION_MODEL_VERSION }}-production"
          echo "  → ${{ env.ECR_BASE }}:production"

      # ─────────────────────────────────────────────────────────────────────────
      # STEP 12 (Part 2): ECR SECURITY SCAN
      # ECR automatically scans on push
      # Fail pipeline if any CRITICAL CVEs found
      # Protects production from known vulnerable dependencies
      # ─────────────────────────────────────────────────────────────────────────
      - name: "Step 12 | Wait for ECR Vulnerability Scan"
        run: sleep 60

      - name: "Step 12 | Check ECR Scan Results"
        run: |
          SCAN_RESULT=$(aws ecr describe-image-scan-findings \
            --repository-name ${{ env.IMAGE_NAME }} \
            --image-id imageTag=${{ github.sha }} \
            --region us-east-1)

          CRITICAL=$(echo $SCAN_RESULT | \
            python3 -c "
          import json, sys
          data = json.load(sys.stdin)
          findings = data.get('imageScanFindings', {})
          counts = findings.get('findingSeverityCounts', {})
          print(counts.get('CRITICAL', 0))
            ")

          echo "Critical vulnerabilities: $CRITICAL"

          if [ "$CRITICAL" -gt "0" ]; then
            echo "CRITICAL CVEs detected — blocking deployment."
            exit 1
          fi

          echo "Security scan passed ✅ — no critical vulnerabilities."

      # ─────────────────────────────────────────────────────────────────────────
      # STEP 12 (Part 3): VERIFY IMAGE IN ECR
      # Final confirmation image is queryable in registry
      # ─────────────────────────────────────────────────────────────────────────
      - name: "Step 12 | Verify Image in ECR"
        run: |
          aws ecr describe-images \
            --repository-name ${{ env.IMAGE_NAME }} \
            --image-ids imageTag=${{ github.sha }} \
            --region us-east-1
          echo "Image verified in ECR ✅"

      # ─────────────────────────────────────────────────────────────────────────
      # PIPELINE COMPLETE — Print full summary
      # ─────────────────────────────────────────────────────────────────────────
      - name: "Pipeline Complete — Summary"
        run: |
          echo "═══════════════════════════════════════════════════════"
          echo "  ML PIPELINE COMPLETE ✅"
          echo "═══════════════════════════════════════════════════════"
          echo "  Git Commit:      ${{ github.sha }}"
          echo "  Triggered By:    ${{ github.actor }}"
          echo "  Model Name:      ${{ env.MODEL_NAME }}"
          echo "  Model Version:   ${{ env.PRODUCTION_MODEL_VERSION }}"
          echo "  MLflow Run ID:   ${{ env.MLFLOW_RUN_ID }}"
          echo "  Image URI:       ${{ env.IMAGE_URI }}"
          echo "  ECR Tags:        :${{ github.sha }}"
          echo "                   :v${{ env.PRODUCTION_MODEL_VERSION }}-production"
          echo "                   :production"
          echo "═══════════════════════════════════════════════════════"
          echo "  Steps Completed:"
          echo "    ✅ Step  3  — Code Checkout"
          echo "    ✅ Step  4  — Dependencies Installed"
          echo "    ✅ Step  5  — Lint Passed (flake8)"
          echo "    ✅ Step  6  — Unit Tests Passed (pytest)"
          echo "    ✅ Step  7  — Data Pulled (DVC + S3)"
          echo "    ✅ Step  8a — Model Trained (KFP GPU Pod)"
          echo "    ✅ Step  8b — Experiment Tracked (MLflow)"
          echo "    ✅ Step  8c — Recall Gate Passed"
          echo "    ✅ Step  9  — Model Registered (Staging)"
          echo "    ✅ Step 10  — Human Approved + Promoted (Production)"
          echo "    ✅ Step 11  — Docker Image Built + Smoke Tested"
          echo "    ✅ Step 12  — Image Pushed to ECR + Scan Passed"
          echo "═══════════════════════════════════════════════════════"
          echo "  Kubernetes rolling update now in progress..."
          echo "  churn-inference:production → 3 pods running v${{ env.PRODUCTION_MODEL_VERSION }}"
          echo "═══════════════════════════════════════════════════════"
```

---

## Job Dependency Graph
```
PUSH TO MAIN
      │
      ▼
┌─────────────────────┐
│   JOB 1             │  Steps 3 → 4 → 5 → 6
│   code-quality      │  ~5 minutes
│   (ubuntu runner)   │
└──────────┬──────────┘
           │ needs: code-quality
           ▼
┌─────────────────────┐
│   JOB 2             │  Steps 7 → 8 → 9
│   ml-pipeline       │  ~40-50 minutes
│   (ubuntu runner    │  (polls Kubeflow cluster)
│    + KFP cluster)   │
└──────────┬──────────┘
           │ needs: ml-pipeline
           ▼
┌─────────────────────┐
│   JOB 3             │  Step 10 (Part 1)
│   await-approval    │  pauses indefinitely
│   environment:      │  until human approves
│   production        │  in GitHub UI
└──────────┬──────────┘
           │ needs: await-approval
           ▼
┌─────────────────────┐
│   JOB 4             │  Step 10 (Part 2)
│   promote-model     │  Staging → Production
│   (ubuntu runner)   │  ~2 minutes
└──────────┬──────────┘
           │ needs: promote-model
           ▼
┌─────────────────────┐
│   JOB 5             │  Steps 11 → 12
│   build-and-push    │  Build + Smoke Test
│   (ubuntu runner)   │  + Push to ECR
└─────────────────────┘  ~10 minutes
```

---

## GitHub Secrets Required
```
GitHub Repo → Settings → Secrets and Variables → Actions

Secret Name                Value
─────────────────────────  ──────────────────────────────────────────
AWS_ACCESS_KEY_ID          IAM user key with S3 + ECR permissions
AWS_SECRET_ACCESS_KEY      IAM user secret
MLFLOW_TRACKING_URI        https://mlflow.your-company.internal
KFP_ENDPOINT               https://kubeflow.your-company.internal
ECR_REGISTRY               123456789012.dkr.ecr.us-east-1.amazonaws.com
```

---

## Total Pipeline Timeline
```
Step 3-4   Install + Checkout       ~2  minutes
Step 5-6   Lint + Tests             ~3  minutes
Step 7     DVC Data Pull            ~5  minutes
Step 8     KFP Training             ~40 minutes
Step 9     Registration             ~1  minute
Step 10    Human Approval           variable (minutes to hours)
Step 10    Promotion                ~1  minute
Step 11    Docker Build + Test      ~5  minutes
Step 12    ECR Push + Scan          ~5  minutes
─────────────────────────────────────────────────
Total (excl. human approval):       ~62 minutes  
```
  </details> 

  </details>
  
  <details>
    <summary><i> --- CD Phase (ArgoCD + KServe) --- </i></summary>   
   
```  
  In real production these live in a SEPARATE Git repo:

ml-infra-repo/                      ← separate from ml-project repo
├── deployments/
│   └── churn-inference/
│       ├── inference.yaml          ← KServe InferenceService manifest
│       └── kustomization.yaml      ← optional: Kustomize config
└── argocd/
    └── app.yaml                    ← ArgoCD Application definition
```
> Why separate repo? So infra changes don't trigger ML retraining CI.   
And ML code changes don't accidentally touch infra configs.

## Step 1: Update KServe Manifest (```inference.yaml```)
This file describes how KServe should run your model.
Written once by MLOps Engineer, updated every deployment.  

```  
# deployments/churn-inference/inference.yaml

apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: churn-inference
  namespace: ml-production
  annotations:
    argocd.argoproj.io/sync-wave: "1"

spec:

  # ── PREDICTOR: the main model serving pod ────────────────────
  predictor:
    containers:
      - name: churn-inference
        # ↓ UPDATED every deployment (Step 11/12 output)
        image: 123456789012.dkr.ecr.us-east-1.amazonaws.com/churn-inference:a3f8c21d
        
        ports:
          - containerPort: 8080
            protocol: TCP

        env:
          - name: MODEL_VERSION
            value: "3"                # ← updated to new MLflow version
          - name: GIT_COMMIT
            value: "a3f8c21d"

        # ── Health checks ───────────────────────────────────────
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10

        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 5

        # ── Resources per pod ───────────────────────────────────
        resources:
          requests:
            cpu:    "2"
            memory: "4Gi"
          limits:
            cpu:    "4"
            memory: "8Gi"

  # ── AUTO-SCALING ──────────────────────────────────────────────
  # Scales from 1 to 5 pods based on traffic
  # Scales DOWN to 0 when no traffic (saves cost)
  scaleTarget: 100              # requests per second per pod
  scaleMetric: rps
  minReplicas: 1
  maxReplicas: 5
```

**What gets updated in this file every deployment:**
```
Old:  image: ...churn-inference:b2e1d90f    ← previous git SHA
New:  image: ...churn-inference:a3f8c21d    ← new git SHA from Step 12

Old:  MODEL_VERSION: "1"
New:  MODEL_VERSION: "3"                    ← new MLflow version
```

---

**Who Updates ```inference.yaml``` and How**
```
Two approaches in real production:

APPROACH 1: GitHub Actions updates it automatically
──────────────────────────────────────────────────
Add this step at the END of ml-pipeline.yml (after Step 12):

      - name: Update KServe Manifest
        run: |
          # Clone infra repo
          git clone https://github.com/your-org/ml-infra-repo.git
          cd ml-infra-repo

          # Update image tag with new git SHA
          sed -i "s|churn-inference:.*|churn-inference:${{ github.sha }}|g" \
            deployments/churn-inference/inference.yaml

          # Update model version
          sed -i "s|MODEL_VERSION.*|MODEL_VERSION\n            value: \"${{ env.PRODUCTION_MODEL_VERSION }}\"|g" \
            deployments/churn-inference/inference.yaml

          # Commit and push
          git config user.email "ci@your-company.com"
          git config user.name  "ML CI Bot"
          git add deployments/churn-inference/inference.yaml
          git commit -m "deploy: churn-inference v${{ env.PRODUCTION_MODEL_VERSION }} (${{ github.sha }})"
          git push

APPROACH 2: MLOps Engineer updates manually
──────────────────────────────────────────────────
For teams that want explicit human control
Change image tag → git push → ArgoCD picks it up  
```
## Step 2: Git Commit & Push Deployment Config  
```
# What happens in infra repo after update:

git log --oneline deployments/churn-inference/inference.yaml

a3f8c21  deploy: churn-inference v3 (a3f8c21d)   ← just pushed
b2e1d90  deploy: churn-inference v1 (b2e1d90f)   ← previous
c9f3a44  deploy: churn-inference v0 (c9f3a44e)   ← initial deploy
```
**ArgoCD Application Definition — Written Once by MLOps**   
#### argocd/app.yaml  
#### Tells ArgoCD: "watch THIS repo, apply to THIS cluster"  
```  
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: churn-inference
  namespace: argocd

spec:
  project: ml-production

  # ── WHERE to get config from (Git) ───────────────────────────
  source:
    repoURL:        https://github.com/your-org/ml-infra-repo
    targetRevision: main
    path:           deployments/churn-inference   # watch this folder

  # ── WHERE to deploy to (Kubernetes) ──────────────────────────
  destination:
    server:    https://kubernetes.default.svc
    namespace: ml-production

  # ── HOW to sync ───────────────────────────────────────────────
  syncPolicy:
    automated:
      prune:    true      # delete resources removed from Git
      selfHeal: true      # re-apply if someone manually changes cluster

    syncOptions:
      - CreateNamespace=true
```  

## Step 3: ArgoCD Detects Drift & Syncs
```
BEFORE git push:
────────────────────────────────────────────────────────
Git repo:          inference.yaml → image: ...a3f8c21d
Kubernetes cluster: running pod  → image: ...b2e1d90f

ArgoCD compares these every 3 minutes (default polling).
Detects: GIT ≠ CLUSTER → STATUS: OutOfSync

AFTER git push:
────────────────────────────────────────────────────────
ArgoCD detects drift immediately (webhook or next poll)
Pulls latest inference.yaml from Git
Applies it to Kubernetes cluster
STATUS: Syncing → Synced
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

## Step 4: KServe Deploys Model Pod

```
KServe Rolling Update Process:
──────────────────────────────────────────────────────

BEFORE (old pods running):
  Pod 1: churn-inference:b2e1d90f  ← serving traffic
  Pod 2: churn-inference:b2e1d90f  ← serving traffic
  Pod 3: churn-inference:b2e1d90f  ← serving traffic

ROLLING UPDATE BEGINS:

  Step A: Start 1 new pod
    Pod 4: churn-inference:a3f8c21d  ← starting up
    Pulls image from ECR
    Runs /health check
    Waits until readinessProbe passes

  Step B: New pod is healthy → shift traffic
    Pod 1: churn-inference:b2e1d90f  ← TERMINATED
    Pod 2: churn-inference:b2e1d90f  ← serving traffic
    Pod 3: churn-inference:b2e1d90f  ← serving traffic
    Pod 4: churn-inference:a3f8c21d  ← serving traffic

  Step C: Repeat for remaining old pods
    ...

AFTER (all new pods running):
  Pod 4: churn-inference:a3f8c21d  ← serving traffic
  Pod 5: churn-inference:a3f8c21d  ← serving traffic
  Pod 6: churn-inference:a3f8c21d  ← serving traffic

ZERO DOWNTIME — traffic never drops to zero
```  

### The Prediction Endpoint After Deployment  
```
KServe exposes standard endpoint:

POST https://churn-inference.ml-production.svc.cluster.local/v1/models/churn-inference:predict

Request:
{
  "instances": [
    {"age": 35, "tenure_months": 24, "monthly_spend": 80.0}
  ]
}

Response:
{
  "predictions": [
    {
      "churn_probability": 0.2341,
      "churn_prediction": 0
    }
  ],
  "model_version": "3"
}  
```  
### Canary Deployment — Traffic Splitting
```
# Send 10% traffic to new model, 90% to old model
# Validate new model in production before full rollout

spec:
  predictor:
    canaryTrafficPercent: 10      # 10% → new model (v3)
                                  # 90% → current model (v1)
    canary:
      containers:
        - image: ...churn-inference:a3f8c21d   # new version

# Monitor metrics for 30 mins
# If recall holds → bump to 50% → 100%
# If recall drops → set canaryTrafficPercent: 0 → instant rollback
```

---

### Rollback — How It Works in CD
```
Rollback is just a Git revert:

# Something wrong with v3 in production
git revert a3f8c21d                    # reverts inference.yaml to old image tag
git push

ArgoCD detects Git changed
Syncs cluster back to old image
KServe rolling update runs in reverse
Old pods (b2e1d90f) come back up
New pods (a3f8c21d) terminated

Total rollback time: ~2-3 minutes
```

---

### Full CD Flow — End to End
```
CI Phase Complete (image in ECR) ✅
        │
        ▼
Step 1: inference.yaml updated
        (new image tag + model version)
        │
        ▼
Step 2: Git push to ml-infra-repo
        (deployment history recorded)
        │
        ▼
Step 3: ArgoCD detects drift
        (Git ≠ Cluster → OutOfSync)
        ArgoCD syncs → applies inference.yaml
        │
        ▼
Step 4: KServe rolling update
        ├── Pulls image from ECR
        ├── Starts new pods one by one
        ├── Health checks pass
        ├── Traffic shifts to new pods
        └── Old pods terminated
        │
        ▼
Production: /predict endpoint serving
            churn-inference v3
            model.pkl from MLflow v3
            zero downtime ✅
```
  </details> 
 
  <details>
    <summary><i> --- Monitoring & Retraining Loop --- </i></summary>  
     
   ### What This Phase Is in Simple Words  
```  
     CI built the model.
CD deployed it.
Now the model is serving real predictions 24/7.

But models don't stay good forever.
Real world data keeps changing.
Customer behaviour changes.
Business patterns shift.

This phase answers three questions:
  1. Is the model still performing well?     (Step 17 — Monitoring)
  2. Has incoming data changed?              (Step 18 — Drift Detection)
  3. What do we do when something goes wrong?(Steps 19-22 — Alert + Retrain)
```
### The Full Loop Visual  
```
Model Serving in Production (KServe)
              │
              │ every prediction request
              ▼
Step 17: Prometheus scrapes metrics
              │
              ▼
Step 18: Drift detection runs
              │
         ┌────┴────┐
         │         │
    All good    Problem detected
         │         │
         │         ▼
         │   Step 19: Alertmanager fires alert
         │         │
         │         ▼
         │   Step 20: Webhook → GitHub Actions triggered
         │         │
         │         ▼
         │   Step 21: New KFP run → new model version
         │         │
         └────────►▼
              Step 22: Redeploy (loops to CD Step 13)
```
## Step 17: Monitoring (Prometheus + Grafana)  
- What Prometheus Does:  
```
Prometheus is a time-series database.
It scrapes (pulls) metrics from your running pods
every 15 seconds and stores them.

Your KServe pod exposes a /metrics endpoint.
Prometheus calls it → stores the numbers → forever.  
```  
## Step 18: Data / Concept Drift Detection

### What Is Drift — Simple Explanation
```
DATA DRIFT — inputs have changed
─────────────────────────────────────────────────
During training:   average customer age = 34
In production now: average customer age = 52

The model was never trained on 52-year-old customer data.
Its predictions become unreliable silently.
No errors thrown. No crashes. Just wrong answers.

CONCEPT DRIFT — relationship between input and output changed
─────────────────────────────────────────────────
During training:   high spend = low churn (loyal customers)
After economy hit: high spend = high churn (customers leaving)

Same input features, completely different meaning now.
Model is confidently predicting the wrong thing.
```
## Step 19: Alert Trigger (Prometheus Alertmanager)
- Alerting Rules — Written by MLOps Engineer:  
- Alertmanager Config — Where Alerts Go
- What the Slack message looks like:

## Step 20: Retraining Pipeline Trigger
- Webhook → GitHub Actions
## Step 21: New Model Version Generated  
## Step 22: Redeployment (Loops Back to CD)  

## The Complete Loop — All Together
```
Production Serving
      │
      │ (every 15 seconds)
      ▼
Prometheus scrapes /metrics
      │
      │ (every hour)
      ▼
Drift Detector CronJob runs
      │
      ├── No drift → continue monitoring
      │
      └── Drift detected
                │
                ▼
           Prometheus Alert fires
                │
                ├── Slack notification → team knows
                │
                └── Alertmanager webhook → GitHub API
                          │
                          ▼
                   GitHub Actions triggered
                   (repository_dispatch event)
                          │
                          ▼
                   Full ML pipeline runs again
                   Steps 3 → 4 → 5 → 6 → 7
                          │
                          ▼
                   Step 8: KFP trains on fresh data
                   Step 9: New version registered
                   Step 10: Human approves
                   Steps 11-12: New image built + pushed
                          │
                          ▼
                   CD Phase:
                   inference.yaml updated
                   ArgoCD syncs
                   KServe deploys new version
                          │
                          ▼
                   Back to: Production Serving
                   (with better model)
```


  </details> 
  
</details>  




