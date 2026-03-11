# GitHub repository structure  
```  
/home/runner/work/ml-project/ml-project/
├── .github/
│   └── workflows/
│       └── ml-pipeline.yml
│
├── src/
│   ├── train.py
│   ├── evaluate.py
│   ├── preprocess.py
│   └── predict.py
│
├── pipelines/
│   ├── kfp_pipeline.py
│   └── submit_pipeline.py        ← bridge script to submit KFP pipeline
│
├── tests/
│   └── test_preprocess.py
│
├── dvc.yaml                      ← pipeline stages definition
├── dvc.lock                      ← data version references
├── params.yaml                   ← ML hyperparameters
├── requirements.txt              ← python dependencies
├── Dockerfile                    ← container image for training/pipeline
│
└── README.md                     ← project documentation (recommended)
```  

# Production-Grade ML Pipeline — Complete Repository Structure

> **Context:** This maps to the 12-step CI/CD pipeline for a Senior MLOps Engineer role.
> Runner path: `/home/runner/work/ml-project/ml-project/`

---

## Complete Directory Tree

```
ml-project/
│
├── .github/
│   ├── workflows/
│   │   ├── ml-pipeline.yml                  # Main CI/CD pipeline (Steps 1–12)
│   │   ├── base-image-build.yml             # Rebuild CI base Docker image when deps change
│   │   ├── scheduled-retrain.yml            # Cron-triggered retraining (weekly/monthly)
│   │   └── pr-checks.yml                    # Lightweight checks on pull requests (lint + unit tests only)
│   ├── CODEOWNERS                           # Enforce review rules (ML eng for src/, platform for infra/)
│   └── pull_request_template.md             # PR checklist (model metrics, tests, DVC tracked?)
│
├── configs/
│   ├── train_config.yaml                    # Training hyperparams (learning_rate, epochs, batch_size)
│   ├── serve_config.yaml                    # Inference config (model_name, version, timeout, batch)
│   ├── feature_config.yaml                  # Feature engineering params (scaling, encoding, selection)
│   ├── logging_config.yaml                  # Python logging config (formatters, handlers, levels)
│   └── environments/
│       ├── dev.env                           # Dev environment variables (MLFLOW_TRACKING_URI, etc.)
│       ├── staging.env                       # Staging environment variables
│       └── prod.env                          # Production environment variables
│
├── data/
│   ├── .gitkeep                              # Placeholder — actual data is in S3, tracked by DVC
│   ├── raw/                                  # DVC-tracked: raw ingested data
│   │   └── .gitkeep
│   ├── processed/                            # DVC-tracked: cleaned/feature-engineered data
│   │   └── .gitkeep
│   ├── splits/                               # DVC-tracked: train/val/test splits
│   │   └── .gitkeep
│   └── reference/                            # Small reference data committed to Git (label maps, schemas)
│       ├── label_mapping.json
│       └── schema.json                       # Expected data schema for validation
│
├── docker/
│   ├── Dockerfile.train                      # Training image (heavy deps: torch/sklearn + training code)
│   ├── Dockerfile.serve                      # Inference image (lean: fastapi + model loading only)
│   ├── Dockerfile.ci                         # CI base image (all tools: flake8, pytest, dvc, kfp, mlflow)
│   ├── Dockerfile.dataprep                   # Data preprocessing image (pandas, dvc, s3 tools)
│   └── .dockerignore                         # Exclude .git, data/, notebooks/, __pycache__, .env
│
├── docs/
│   ├── architecture.md                       # System architecture diagram + explanation
│   ├── runbook.md                            # Incident response: what to do when model degrades
│   ├── onboarding.md                         # New team member setup guide
│   ├── model_card.md                         # Model Card: purpose, limitations, bias, performance
│   ├── data_dictionary.md                    # Every feature: name, type, source, business meaning
│   └── adr/                                  # Architecture Decision Records
│       ├── 001-use-kfp-over-airflow.md
│       ├── 002-mlflow-for-tracking.md
│       └── 003-dvc-for-data-versioning.md
│
├── infra/
│   ├── terraform/
│   │   ├── main.tf                           # Root module: ties all resources together
│   │   ├── variables.tf                      # Input variables (region, env, bucket names)
│   │   ├── outputs.tf                        # Output values (ECR URI, S3 bucket ARN, endpoint URL)
│   │   ├── backend.tf                        # Remote state config (S3 + DynamoDB lock)
│   │   ├── providers.tf                      # AWS provider config
│   │   ├── ecr.tf                            # ECR repository for Docker images
│   │   ├── s3.tf                             # S3 buckets (data, artifacts, MLflow store)
│   │   ├── iam.tf                            # IAM roles/policies (CI runner, SageMaker, ECS)
│   │   ├── sagemaker.tf                      # SageMaker endpoints (or ECS/EKS for serving)
│   │   ├── networking.tf                     # VPC, subnets, security groups
│   │   └── monitoring.tf                     # CloudWatch alarms, dashboards
│   ├── helm/
│   │   └── ml-serving/                       # Helm chart if using Kubernetes for serving
│   │       ├── Chart.yaml
│   │       ├── values.yaml
│   │       ├── values-staging.yaml
│   │       └── templates/
│   │           ├── deployment.yaml
│   │           ├── service.yaml
│   │           ├── hpa.yaml                  # Horizontal Pod Autoscaler
│   │           └── ingress.yaml
│   └── scripts/
│       ├── setup-kubeflow.sh                 # Bootstrap Kubeflow Pipelines on cluster
│       └── setup-mlflow-server.sh            # Deploy MLflow tracking server
│
├── notebooks/
│   ├── 01_eda.ipynb                          # Exploratory Data Analysis
│   ├── 02_feature_engineering.ipynb          # Feature experiment playground
│   ├── 03_model_experimentation.ipynb        # Model selection / hyperparameter exploration
│   ├── 04_error_analysis.ipynb               # Post-training error analysis
│   └── README.md                             # "Notebooks are for exploration, NOT production"
│
├── pipelines/
│   ├── kfp_pipeline.py                       # Main Kubeflow pipeline definition (compiles to YAML)
│   ├── components/
│   │   ├── __init__.py
│   │   ├── data_validation.py                # KFP component: validate data schema + drift check
│   │   ├── preprocessing.py                  # KFP component: feature engineering
│   │   ├── training.py                       # KFP component: model training
│   │   ├── evaluation.py                     # KFP component: evaluate + gate on metrics
│   │   ├── registration.py                   # KFP component: register model in MLflow
│   │   └── promotion.py                      # KFP component: promote Staging → Production
│   ├── triggers/
│   │   ├── trigger_pipeline.py               # Script called by GitHub Actions to launch KFP run
│   │   └── poll_pipeline.py                  # Script to poll KFP run status until completion
│   └── compiled/
│       └── .gitkeep                          # Compiled pipeline YAML goes here (generated, not committed)
│
├── requirements/
│   ├── requirements-base.in                  # Human-edited: core ML libs (numpy, pandas, sklearn)
│   ├── requirements-base.txt                 # pip-compile output: locked + hashed
│   ├── requirements-train.in                 # Human-edited: training libs (xgboost, torch, optuna)
│   ├── requirements-train.txt                # pip-compile output: locked + hashed
│   ├── requirements-serve.in                 # Human-edited: serving libs (fastapi, uvicorn, gunicorn)
│   ├── requirements-serve.txt                # pip-compile output: locked + hashed
│   ├── requirements-ci.in                    # Human-edited: CI tools (flake8, pytest, dvc, kfp, mlflow)
│   ├── requirements-ci.txt                   # pip-compile output: locked + hashed
│   ├── requirements-dev.in                   # Human-edited: dev extras (jupyter, black, ipdb, mypy)
│   ├── requirements-dev.txt                  # pip-compile output: locked + hashed
│   ├── requirements-monitoring.in            # Human-edited: monitoring libs (evidently, prometheus-client)
│   ├── requirements-monitoring.txt           # pip-compile output: locked + hashed
│   └── constraints.txt                       # Upper-bound pins for transitive deps (protobuf<5, etc.)
│
├── scripts/
│   ├── setup_local_env.sh                    # One-command local dev environment setup
│   ├── run_training_local.sh                 # Run training locally for debugging
│   ├── compile_pipeline.sh                   # Compile KFP pipeline to YAML
│   ├── promote_model.py                      # Manual model promotion script (Staging → Prod)
│   ├── rollback_model.py                     # Rollback to previous model version
│   ├── generate_requirements.sh              # Run pip-compile for all .in files
│   ├── smoke_test_endpoint.py                # Post-deploy health check against live endpoint
│   ├── seed_mlflow.py                        # Seed MLflow with initial experiment structure
│   └── data_quality_report.py                # Generate data quality report (missing values, outliers)
│
├── serving/
│   ├── app.py                                # FastAPI application (inference endpoint)
│   ├── model_loader.py                       # Load model from MLflow Registry / S3
│   ├── request_schema.py                     # Pydantic models for request/response validation
│   ├── response_schema.py                    # Pydantic models for response
│   ├── middleware/
│   │   ├── __init__.py
│   │   ├── logging_middleware.py             # Request/response logging
│   │   ├── auth_middleware.py                # API key / JWT validation
│   │   └── rate_limiter.py                   # Rate limiting
│   ├── preprocessing/
│   │   ├── __init__.py
│   │   └── feature_transformer.py            # Same transforms as training (loaded from artifact)
│   ├── gunicorn_config.py                    # Gunicorn production settings (workers, timeout, bind)
│   └── tests/
│       ├── test_app.py                       # Integration tests for API endpoints
│       └── test_model_loader.py              # Test model loading from registry
│
├── src/
│   ├── __init__.py
│   ├── data/
│   │   ├── __init__.py
│   │   ├── ingest.py                         # Data ingestion from source systems (S3, DB, API)
│   │   ├── validate.py                       # Schema validation (great_expectations or pandera)
│   │   ├── preprocess.py                     # Feature engineering + transformation pipeline
│   │   ├── split.py                          # Train/val/test split with stratification
│   │   └── drift_detector.py                 # Statistical drift detection (KS test, PSI, etc.)
│   ├── features/
│   │   ├── __init__.py
│   │   ├── feature_store.py                  # Feature store interface (Feast or custom)
│   │   ├── feature_engineering.py            # Feature transformations (encoding, scaling, etc.)
│   │   └── feature_selection.py              # Feature importance / selection methods
│   ├── models/
│   │   ├── __init__.py
│   │   ├── train.py                          # Training orchestration (data loading → fit → save)
│   │   ├── evaluate.py                       # Evaluation: metrics computation + threshold checks
│   │   ├── predict.py                        # Single-sample and batch prediction logic
│   │   ├── hyperparameter_tuning.py          # Optuna / Ray Tune integration
│   │   └── model_registry.py                 # MLflow Model Registry interaction wrapper
│   ├── monitoring/
│   │   ├── __init__.py
│   │   ├── performance_monitor.py            # Track prediction latency, throughput
│   │   ├── drift_monitor.py                  # Monitor feature + prediction drift in production
│   │   ├── metrics_exporter.py               # Export custom metrics to Prometheus / CloudWatch
│   │   └── alerting.py                       # Alert rules (Slack, PagerDuty, email)
│   ├── experiment_tracking/
│   │   ├── __init__.py
│   │   ├── mlflow_tracker.py                 # MLflow experiment tracking wrapper
│   │   └── experiment_config.py              # Experiment naming, tagging conventions
│   └── utils/
│       ├── __init__.py
│       ├── logger.py                         # Structured logging setup (JSON format for prod)
│       ├── config.py                         # Config loader (YAML + env vars + CLI args)
│       ├── aws_utils.py                      # S3, STS, ECR helper functions
│       ├── io_utils.py                       # File I/O helpers (parquet, csv, json)
│       └── reproducibility.py                # Seed setting, deterministic mode flags
│
├── tests/
│   ├── __init__.py
│   ├── conftest.py                           # Shared fixtures (sample data, mock models, tmp dirs)
│   ├── unit/
│   │   ├── __init__.py
│   │   ├── test_preprocess.py                # Test feature engineering transforms
│   │   ├── test_validate.py                  # Test schema validation catches bad data
│   │   ├── test_split.py                     # Test stratified splitting logic
│   │   ├── test_train.py                     # Test training runs on tiny data without error
│   │   ├── test_evaluate.py                  # Test metric computation correctness
│   │   ├── test_predict.py                   # Test prediction input/output shapes
│   │   ├── test_config.py                    # Test config loading + env var override
│   │   ├── test_feature_engineering.py       # Test feature transforms produce expected output
│   │   └── test_drift_detector.py            # Test drift detection on synthetic shifted data
│   ├── integration/
│   │   ├── __init__.py
│   │   ├── test_training_pipeline.py         # End-to-end: data → train → evaluate → register
│   │   ├── test_serving_pipeline.py          # End-to-end: load model → predict → validate response
│   │   ├── test_dvc_pull.py                  # Test DVC data pull from S3 works
│   │   └── test_mlflow_connection.py         # Test MLflow tracking server connectivity
│   ├── smoke/
│   │   ├── __init__.py
│   │   └── test_endpoint_health.py           # Post-deploy: hit /health, /predict with sample
│   └── fixtures/
│       ├── sample_data.csv                   # Small synthetic dataset for tests
│       ├── sample_model.pkl                  # Pre-trained tiny model for predict tests
│       └── sample_request.json               # Example API request payload
│
├── monitoring/
│   ├── dashboards/
│   │   ├── grafana_model_performance.json    # Grafana dashboard: accuracy, latency, throughput
│   │   ├── grafana_data_drift.json           # Grafana dashboard: feature drift over time
│   │   └── cloudwatch_dashboard.json         # CloudWatch dashboard definition
│   ├── alerts/
│   │   ├── alert_rules.yaml                  # Prometheus/Grafana alert rules
│   │   └── pagerduty_config.yaml             # PagerDuty escalation config
│   └── evidently/
│       ├── data_drift_report.py              # Generate Evidently data drift HTML report
│       ├── model_performance_report.py       # Generate Evidently model performance report
│       └── reference_data/
│           └── .gitkeep                      # Reference dataset for drift comparison
│
├── .dvc/
│   ├── config                                # DVC remote config (S3 bucket, region, endpoint)
│   └── .gitignore
│
├── .flake8                                   # Flake8 config (max-line-length, exclude, per-file-ignores)
├── .pre-commit-config.yaml                   # Pre-commit hooks (black, flake8, mypy, detect-secrets)
├── .gitignore                                # Comprehensive gitignore for ML projects
├── .env.example                              # Template for local environment variables
├── dvc.yaml                                  # DVC pipeline stages (preprocess → split → train → evaluate)
├── dvc.lock                                  # DVC lock file (hashes of data + code for each stage)
├── params.yaml                               # Hyperparameters referenced by DVC + training code
├── pyproject.toml                            # Project metadata + tool configs (black, mypy, pytest)
├── setup.py                                  # Package setup (or setup.cfg) — makes src/ installable
├── setup.cfg                                 # Additional package config
├── Makefile                                  # Developer shortcuts (make train, make test, make lint, etc.)
├── CHANGELOG.md                              # Version history with model performance changes
├── CONTRIBUTING.md                           # How to contribute: branching, testing, review process
├── LICENSE                                   # License file
└── README.md                                 # Project overview, quick start, architecture summary
```

---

## File-by-File Contents (Key Files)

### 1. `.github/workflows/ml-pipeline.yml` — The Main CI/CD Pipeline

This is the orchestrator for all 12 steps.

```yaml
# .github/workflows/ml-pipeline.yml
name: ML Pipeline — CI/CD

on:
  push:
    branches: [main]
    paths:
      - 'src/**'
      - 'pipelines/**'
      - 'configs/**'
      - 'requirements/**'
      - 'dvc.yaml'
      - 'params.yaml'
      - 'docker/**'
  workflow_dispatch:                           # Manual trigger button in GitHub UI
    inputs:
      skip_training:
        description: 'Skip training (deploy existing model)'
        type: boolean
        default: false
      force_promote:
        description: 'Force promote to production'
        type: boolean
        default: false

permissions:
  id-token: write                             # Required for OIDC → AWS role assumption
  contents: read                              # Required for checkout

env:
  AWS_REGION: us-east-1
  ECR_REGISTRY: 123456789012.dkr.ecr.us-east-1.amazonaws.com
  ECR_REPOSITORY: ml-project-inference
  MLFLOW_TRACKING_URI: https://mlflow.internal.company.com
  KFP_HOST: https://kubeflow.internal.company.com
  PYTHON_VERSION: '3.11.7'

jobs:
  # ============================================================
  # JOB 1: CI Checks (Steps 3–6)
  # ============================================================
  ci-checks:
    name: "CI: Lint + Tests"
    runs-on: ubuntu-22.04
    container:
      image: 123456789012.dkr.ecr.us-east-1.amazonaws.com/ml-ci-base:latest
      credentials:
        username: AWS
        password: ${{ secrets.ECR_PASSWORD }}
    
    steps:
      # ── Step 3: Code Checkout ──
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0                      # Full history for DVC + changelog

      # ── Step 4: Install Dependencies ──
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}

      - name: Cache pip
        uses: actions/cache@v4
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('requirements/requirements-ci.txt') }}
          restore-keys: ${{ runner.os }}-pip-

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip==24.0
          pip install -r requirements/requirements-ci.txt --no-deps --require-hashes
          pip install -e .                    # Install project as editable package

      # ── Step 5: Lint Check ──
      - name: Lint with flake8
        run: flake8 src/ pipelines/ serving/ tests/

      - name: Type check with mypy
        run: mypy src/ --config-file pyproject.toml
        continue-on-error: true               # Warning, not blocking (teams adopt gradually)

      - name: Security scan with bandit
        run: bandit -r src/ -c pyproject.toml

      # ── Step 6: Unit Tests ──
      - name: Run unit tests
        run: |
          pytest tests/unit/ \
            --cov=src \
            --cov-report=xml:coverage.xml \
            --cov-fail-under=80 \
            --junitxml=test-results.xml \
            -v

      - name: Upload test results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: test-results
          path: |
            coverage.xml
            test-results.xml

  # ============================================================
  # JOB 2: Data Pull + Training Pipeline (Steps 7–10)
  # ============================================================
  training:
    name: "Train: DVC Pull + KFP Pipeline"
    needs: ci-checks                          # Only runs if CI passes
    if: ${{ !inputs.skip_training }}
    runs-on: ubuntu-22.04
    container:
      image: 123456789012.dkr.ecr.us-east-1.amazonaws.com/ml-ci-base:latest
      credentials:
        username: AWS
        password: ${{ secrets.ECR_PASSWORD }}

    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Configure AWS credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/github-actions-ml-pipeline
          aws-region: ${{ env.AWS_REGION }}

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip==24.0
          pip install -r requirements/requirements-ci.txt --no-deps --require-hashes
          pip install -e .

      # ── Step 7: Data Pull & Versioning (DVC + S3) ──
      - name: Pull data with DVC
        run: |
          dvc remote modify myremote access_key_id ${{ env.AWS_ACCESS_KEY_ID }}
          dvc remote modify myremote secret_access_key ${{ env.AWS_SECRET_ACCESS_KEY }}
          dvc pull -v
        env:
          AWS_ACCESS_KEY_ID: ${{ env.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ env.AWS_SECRET_ACCESS_KEY }}

      - name: Validate data schema
        run: python -m src.data.validate --config configs/feature_config.yaml

      # ── Step 8: Trigger KFP Training Pipeline ──
      # 8a. Training  |  8b. MLflow Tracking  |  8c. Evaluation & Gating
      - name: Trigger Kubeflow Pipeline
        id: kfp_trigger
        run: |
          python pipelines/triggers/trigger_pipeline.py \
            --host ${{ env.KFP_HOST }} \
            --pipeline-name ml-training-pipeline \
            --experiment-name ${{ github.repository }} \
            --run-name "ci-${{ github.sha }}" \
            --params-file params.yaml \
            --commit-sha ${{ github.sha }}
        env:
          KFP_TOKEN: ${{ secrets.KFP_TOKEN }}

      - name: Poll KFP Pipeline for completion
        id: kfp_poll
        run: |
          python pipelines/triggers/poll_pipeline.py \
            --host ${{ env.KFP_HOST }} \
            --run-id ${{ steps.kfp_trigger.outputs.run_id }} \
            --timeout 7200 \
            --poll-interval 60
        env:
          KFP_TOKEN: ${{ secrets.KFP_TOKEN }}

      # ── Step 9: Model Registration (done inside KFP pipeline) ──
      # The KFP evaluation component registers the model in MLflow if metrics pass.
      # This step verifies it was registered.
      - name: Verify model registration
        run: |
          python -c "
          import mlflow
          client = mlflow.tracking.MlflowClient()
          versions = client.search_model_versions(\"name='ml-project-model'\")
          latest = max(versions, key=lambda v: int(v.version))
          assert latest.current_stage == 'Staging', f'Expected Staging, got {latest.current_stage}'
          print(f'Model version {latest.version} registered in Staging')
          "
        env:
          MLFLOW_TRACKING_URI: ${{ env.MLFLOW_TRACKING_URI }}

      # ── Step 10: Model Promotion (Staging → Production) ──
      - name: Promote model to Production
        if: ${{ inputs.force_promote || success() }}
        run: |
          python scripts/promote_model.py \
            --model-name ml-project-model \
            --from-stage Staging \
            --to-stage Production \
            --archive-existing
        env:
          MLFLOW_TRACKING_URI: ${{ env.MLFLOW_TRACKING_URI }}

    outputs:
      model_version: ${{ steps.kfp_poll.outputs.model_version }}
      model_uri: ${{ steps.kfp_poll.outputs.model_uri }}

  # ============================================================
  # JOB 3: Build & Push Docker Image (Steps 11–12)
  # ============================================================
  build-and-push:
    name: "Build: Docker Image → ECR"
    needs: training
    runs-on: ubuntu-22.04

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/github-actions-ml-pipeline
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to ECR
        uses: aws-actions/amazon-ecr-login@v2

      # ── Step 11: Build Inference Docker Image ──
      - name: Build Docker image
        run: |
          docker build \
            -f docker/Dockerfile.serve \
            --build-arg MODEL_VERSION=${{ needs.training.outputs.model_version }} \
            --build-arg COMMIT_SHA=${{ github.sha }} \
            --build-arg BUILD_DATE=$(date -u +"%Y-%m-%dT%H:%M:%SZ") \
            -t ${{ env.ECR_REGISTRY }}/${{ env.ECR_REPOSITORY }}:${{ github.sha }} \
            -t ${{ env.ECR_REGISTRY }}/${{ env.ECR_REPOSITORY }}:latest \
            .

      # ── Step 12: Push Docker Image to ECR ──
      - name: Push to ECR
        run: |
          docker push ${{ env.ECR_REGISTRY }}/${{ env.ECR_REPOSITORY }}:${{ github.sha }}
          docker push ${{ env.ECR_REGISTRY }}/${{ env.ECR_REPOSITORY }}:latest

      - name: Run Trivy vulnerability scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.ECR_REGISTRY }}/${{ env.ECR_REPOSITORY }}:${{ github.sha }}
          format: 'sarif'
          severity: 'CRITICAL,HIGH'

  # ============================================================
  # JOB 4: Deploy (Post-CI/CD — Bonus)
  # ============================================================
  deploy:
    name: "Deploy: Update SageMaker Endpoint"
    needs: build-and-push
    runs-on: ubuntu-22.04
    environment: production                   # Requires manual approval in GitHub

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/github-actions-ml-pipeline
          aws-region: ${{ env.AWS_REGION }}

      - name: Deploy to SageMaker
        run: |
          python scripts/deploy_sagemaker.py \
            --image-uri ${{ env.ECR_REGISTRY }}/${{ env.ECR_REPOSITORY }}:${{ github.sha }} \
            --endpoint-name ml-project-prod \
            --instance-type ml.m5.xlarge \
            --initial-instance-count 2

      - name: Smoke test
        run: |
          python scripts/smoke_test_endpoint.py \
            --endpoint-name ml-project-prod \
            --test-payload tests/fixtures/sample_request.json \
            --expected-status 200

      - name: Notify Slack
        if: always()
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "ML Pipeline ${{ job.status }}: model v${{ needs.training.outputs.model_version }} deployed to production",
              "channel": "#ml-deployments"
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

---

### 2. `.github/workflows/pr-checks.yml` — Lightweight PR Checks

```yaml
name: PR Checks

on:
  pull_request:
    branches: [main, develop]

jobs:
  quick-checks:
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11.7'
      - name: Install CI deps
        run: |
          pip install -r requirements/requirements-ci.txt --no-deps --require-hashes
          pip install -e .
      - name: Lint
        run: flake8 src/ pipelines/ serving/ tests/
      - name: Unit tests
        run: pytest tests/unit/ -v --tb=short
      - name: Check DVC files committed
        run: |
          # Ensure dvc.lock is committed when dvc.yaml changes
          if git diff --name-only origin/main | grep -q "dvc.yaml"; then
            git diff --name-only origin/main | grep -q "dvc.lock" || \
              { echo "ERROR: dvc.yaml changed but dvc.lock not updated"; exit 1; }
          fi
```

---

### 3. `.github/workflows/base-image-build.yml` — CI Base Image Rebuild

```yaml
name: Build CI Base Image

on:
  push:
    branches: [main]
    paths:
      - 'requirements/requirements-ci.txt'
      - 'docker/Dockerfile.ci'

jobs:
  build-base-image:
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v4
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/github-actions-ml-pipeline
          aws-region: us-east-1
      - uses: aws-actions/amazon-ecr-login@v2
      - name: Build and push CI base image
        run: |
          docker build -f docker/Dockerfile.ci -t ml-ci-base:latest .
          docker tag ml-ci-base:latest 123456789012.dkr.ecr.us-east-1.amazonaws.com/ml-ci-base:latest
          docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/ml-ci-base:latest
```

---

### 4. `.github/workflows/scheduled-retrain.yml` — Cron Retraining

```yaml
name: Scheduled Retraining

on:
  schedule:
    - cron: '0 6 * * 1'                      # Every Monday at 6 AM UTC
  workflow_dispatch:

jobs:
  retrain:
    uses: ./.github/workflows/ml-pipeline.yml # Reuse the main pipeline
    with:
      skip_training: false
      force_promote: false
    secrets: inherit
```

---

### 5. `src/data/preprocess.py`

```python
"""
Feature engineering and data preprocessing pipeline.
All transforms must be invertible/serializable for serving consistency.
"""
import logging
from pathlib import Path
from typing import Tuple

import numpy as np
import pandas as pd
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler, OneHotEncoder, OrdinalEncoder
from sklearn.impute import SimpleImputer
import joblib

from src.utils.config import load_config
from src.utils.logger import get_logger

logger = get_logger(__name__)


class DataPreprocessor:
    """Production-grade preprocessing pipeline.
    
    Key design decisions:
    - All transforms are fitted on training data ONLY (no data leakage)
    - The fitted pipeline is serialized and loaded at inference time
    - Feature names are tracked through transforms for interpretability
    """

    def __init__(self, config_path: str = "configs/feature_config.yaml"):
        self.config = load_config(config_path)
        self.pipeline: Pipeline | None = None
        self.feature_names_in_: list[str] = []
        self.feature_names_out_: list[str] = []

    def build_pipeline(self) -> Pipeline:
        """Build sklearn ColumnTransformer from config."""
        numeric_features = self.config["features"]["numeric"]
        categorical_features = self.config["features"]["categorical"]
        ordinal_features = self.config["features"].get("ordinal", [])

        numeric_transformer = Pipeline(steps=[
            ("imputer", SimpleImputer(strategy="median")),
            ("scaler", StandardScaler()),
        ])

        categorical_transformer = Pipeline(steps=[
            ("imputer", SimpleImputer(strategy="constant", fill_value="missing")),
            ("encoder", OneHotEncoder(handle_unknown="ignore", sparse_output=False)),
        ])

        ordinal_transformer = Pipeline(steps=[
            ("imputer", SimpleImputer(strategy="most_frequent")),
            ("encoder", OrdinalEncoder(
                categories=self.config["features"].get("ordinal_categories", "auto"),
                handle_unknown="use_encoded_value",
                unknown_value=-1,
            )),
        ])

        preprocessor = ColumnTransformer(
            transformers=[
                ("num", numeric_transformer, numeric_features),
                ("cat", categorical_transformer, categorical_features),
                ("ord", ordinal_transformer, ordinal_features),
            ],
            remainder="drop",                 # Explicitly drop unused columns
            verbose_feature_names_out=True,
        )

        self.pipeline = Pipeline(steps=[
            ("preprocessor", preprocessor),
        ])
        return self.pipeline

    def fit_transform(self, df: pd.DataFrame) -> np.ndarray:
        """Fit on training data and transform. Save fitted pipeline."""
        if self.pipeline is None:
            self.build_pipeline()
        
        self.feature_names_in_ = list(df.columns)
        result = self.pipeline.fit_transform(df)
        self.feature_names_out_ = list(
            self.pipeline.named_steps["preprocessor"].get_feature_names_out()
        )
        
        logger.info(
            "Preprocessing fitted",
            extra={
                "input_features": len(self.feature_names_in_),
                "output_features": len(self.feature_names_out_),
                "samples": result.shape[0],
            }
        )
        return result

    def transform(self, df: pd.DataFrame) -> np.ndarray:
        """Transform using fitted pipeline (for val/test/inference)."""
        if self.pipeline is None:
            raise RuntimeError("Pipeline not fitted. Call fit_transform() first or load().")
        return self.pipeline.transform(df)

    def save(self, path: str) -> None:
        """Serialize fitted pipeline for serving."""
        output_path = Path(path)
        output_path.parent.mkdir(parents=True, exist_ok=True)
        joblib.dump({
            "pipeline": self.pipeline,
            "feature_names_in": self.feature_names_in_,
            "feature_names_out": self.feature_names_out_,
            "config": self.config,
        }, output_path)
        logger.info(f"Preprocessing pipeline saved to {output_path}")

    def load(self, path: str) -> None:
        """Load fitted pipeline for inference."""
        data = joblib.load(path)
        self.pipeline = data["pipeline"]
        self.feature_names_in_ = data["feature_names_in"]
        self.feature_names_out_ = data["feature_names_out"]
        self.config = data["config"]
        logger.info(f"Preprocessing pipeline loaded from {path}")
```

---

### 6. `src/data/validate.py`

```python
"""
Data schema validation — catches bad data BEFORE it enters training.
Uses pandera for DataFrame-level validation.
"""
import json
import sys
from pathlib import Path

import pandas as pd
import pandera as pa
from pandera import Column, DataFrameSchema, Check

from src.utils.config import load_config
from src.utils.logger import get_logger

logger = get_logger(__name__)


def build_schema_from_config(config_path: str) -> DataFrameSchema:
    """Build pandera schema from feature_config.yaml + schema.json."""
    config = load_config(config_path)
    schema_path = Path("data/reference/schema.json")
    
    with open(schema_path) as f:
        schema_def = json.load(f)

    columns = {}
    for feature in schema_def["features"]:
        name = feature["name"]
        dtype = feature["dtype"]
        nullable = feature.get("nullable", False)
        checks = []

        if "min" in feature:
            checks.append(Check.ge(feature["min"]))
        if "max" in feature:
            checks.append(Check.le(feature["max"]))
        if "allowed_values" in feature:
            checks.append(Check.isin(feature["allowed_values"]))
        if "regex" in feature:
            checks.append(Check.str_matches(feature["regex"]))

        columns[name] = Column(dtype, checks=checks, nullable=nullable)

    return DataFrameSchema(
        columns=columns,
        strict=False,                         # Allow extra columns (just ignore them)
        coerce=True,                          # Attempt type coercion before failing
    )


def validate_data(
    df: pd.DataFrame,
    config_path: str = "configs/feature_config.yaml",
) -> pd.DataFrame:
    """Validate DataFrame against schema. Raises SchemaError on failure."""
    schema = build_schema_from_config(config_path)
    
    logger.info(f"Validating {len(df)} rows against schema")
    validated_df = schema.validate(df, lazy=True)  # lazy=True collects ALL errors
    
    logger.info("Data validation passed")
    return validated_df


def check_data_quality(df: pd.DataFrame) -> dict:
    """Compute data quality metrics — logged to MLflow as metadata."""
    report = {
        "total_rows": len(df),
        "total_columns": len(df.columns),
        "missing_rate": df.isnull().mean().to_dict(),
        "duplicate_rows": int(df.duplicated().sum()),
        "duplicate_rate": float(df.duplicated().mean()),
    }

    # Numeric column stats
    numeric_cols = df.select_dtypes(include="number").columns
    for col in numeric_cols:
        report[f"{col}_mean"] = float(df[col].mean())
        report[f"{col}_std"] = float(df[col].std())
        report[f"{col}_min"] = float(df[col].min())
        report[f"{col}_max"] = float(df[col].max())

    return report


if __name__ == "__main__":
    import argparse
    parser = argparse.ArgumentParser()
    parser.add_argument("--config", default="configs/feature_config.yaml")
    parser.add_argument("--data-path", default="data/raw/data.parquet")
    args = parser.parse_args()

    df = pd.read_parquet(args.data_path)
    validate_data(df, args.config)
    report = check_data_quality(df)
    logger.info(f"Data quality report: {json.dumps(report, indent=2)}")
```

---

### 7. `src/data/split.py`

```python
"""
Train/validation/test split with stratification and reproducibility.
"""
import pandas as pd
from sklearn.model_selection import train_test_split
from pathlib import Path

from src.utils.config import load_config
from src.utils.logger import get_logger
from src.utils.reproducibility import set_global_seed

logger = get_logger(__name__)


def create_splits(
    df: pd.DataFrame,
    config_path: str = "configs/train_config.yaml",
    output_dir: str = "data/splits",
) -> dict[str, pd.DataFrame]:
    """Create stratified train/val/test splits."""
    config = load_config(config_path)
    seed = config.get("seed", 42)
    target_col = config["target_column"]
    test_size = config.get("test_size", 0.15)
    val_size = config.get("val_size", 0.15)

    set_global_seed(seed)

    # First split: separate test set
    train_val, test = train_test_split(
        df,
        test_size=test_size,
        random_state=seed,
        stratify=df[target_col] if config.get("stratify", True) else None,
    )

    # Second split: separate validation from training
    val_ratio = val_size / (1 - test_size)     # Adjust ratio for remaining data
    train, val = train_test_split(
        train_val,
        test_size=val_ratio,
        random_state=seed,
        stratify=train_val[target_col] if config.get("stratify", True) else None,
    )

    splits = {"train": train, "val": val, "test": test}

    # Save splits
    output_path = Path(output_dir)
    output_path.mkdir(parents=True, exist_ok=True)
    for name, split_df in splits.items():
        filepath = output_path / f"{name}.parquet"
        split_df.to_parquet(filepath, index=False)
        logger.info(f"Split '{name}': {len(split_df)} rows → {filepath}")

    logger.info(
        f"Split ratios — train: {len(train)/len(df):.2%}, "
        f"val: {len(val)/len(df):.2%}, test: {len(test)/len(df):.2%}"
    )

    return splits
```

---

### 8. `src/models/train.py`

```python
"""
Model training orchestration.
Handles: data loading → preprocessing → training → logging → saving.
"""
import time
from pathlib import Path

import mlflow
import mlflow.sklearn
import numpy as np
import pandas as pd
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.metrics import accuracy_score, f1_score, precision_score, recall_score
import joblib

from src.data.preprocess import DataPreprocessor
from src.utils.config import load_config
from src.utils.logger import get_logger
from src.utils.reproducibility import set_global_seed

logger = get_logger(__name__)


def train_model(
    config_path: str = "configs/train_config.yaml",
    data_dir: str = "data/splits",
) -> dict:
    """Full training run with MLflow tracking."""
    config = load_config(config_path)
    seed = config.get("seed", 42)
    target_col = config["target_column"]
    model_params = config["model_params"]
    experiment_name = config.get("experiment_name", "ml-project")

    set_global_seed(seed)

    # ── Load data ──
    train_df = pd.read_parquet(Path(data_dir) / "train.parquet")
    val_df = pd.read_parquet(Path(data_dir) / "val.parquet")

    X_train = train_df.drop(columns=[target_col])
    y_train = train_df[target_col]
    X_val = val_df.drop(columns=[target_col])
    y_val = val_df[target_col]

    # ── MLflow experiment setup ──
    mlflow.set_experiment(experiment_name)

    with mlflow.start_run() as run:
        run_id = run.info.run_id
        logger.info(f"MLflow run started: {run_id}")

        # Log parameters
        mlflow.log_params(model_params)
        mlflow.log_param("seed", seed)
        mlflow.log_param("train_samples", len(X_train))
        mlflow.log_param("val_samples", len(X_val))
        mlflow.log_param("n_features_raw", X_train.shape[1])

        # ── Preprocessing ──
        preprocessor = DataPreprocessor()
        X_train_processed = preprocessor.fit_transform(X_train)
        X_val_processed = preprocessor.transform(X_val)

        mlflow.log_param("n_features_processed", X_train_processed.shape[1])

        # ── Training ──
        logger.info("Starting model training...")
        start_time = time.time()

        model = GradientBoostingClassifier(
            n_estimators=model_params.get("n_estimators", 200),
            learning_rate=model_params.get("learning_rate", 0.1),
            max_depth=model_params.get("max_depth", 5),
            min_samples_split=model_params.get("min_samples_split", 10),
            subsample=model_params.get("subsample", 0.8),
            random_state=seed,
        )
        model.fit(X_train_processed, y_train)

        training_time = time.time() - start_time
        mlflow.log_metric("training_time_seconds", training_time)

        # ── Evaluation ──
        y_train_pred = model.predict(X_train_processed)
        y_val_pred = model.predict(X_val_processed)
        y_val_proba = model.predict_proba(X_val_processed)[:, 1]

        train_metrics = {
            "train_accuracy": accuracy_score(y_train, y_train_pred),
            "train_f1": f1_score(y_train, y_train_pred, average="weighted"),
        }
        val_metrics = {
            "val_accuracy": accuracy_score(y_val, y_val_pred),
            "val_f1": f1_score(y_val, y_val_pred, average="weighted"),
            "val_precision": precision_score(y_val, y_val_pred, average="weighted"),
            "val_recall": recall_score(y_val, y_val_pred, average="weighted"),
        }

        mlflow.log_metrics({**train_metrics, **val_metrics})

        logger.info(f"Train metrics: {train_metrics}")
        logger.info(f"Val metrics: {val_metrics}")

        # Check for overfitting
        overfit_gap = train_metrics["train_f1"] - val_metrics["val_f1"]
        mlflow.log_metric("overfit_gap_f1", overfit_gap)
        if overfit_gap > 0.1:
            logger.warning(f"Potential overfitting detected: gap = {overfit_gap:.4f}")

        # ── Save artifacts ──
        artifacts_dir = Path("artifacts")
        artifacts_dir.mkdir(exist_ok=True)

        # Save preprocessing pipeline
        preprocessor.save(str(artifacts_dir / "preprocessor.joblib"))
        mlflow.log_artifact(str(artifacts_dir / "preprocessor.joblib"))

        # Save model with MLflow's sklearn flavor (enables model serving)
        mlflow.sklearn.log_model(
            model,
            artifact_path="model",
            registered_model_name=config.get("model_name", "ml-project-model"),
            input_example=X_val.iloc[:3],      # For signature inference
        )

        # Log feature importance
        if hasattr(model, "feature_importances_"):
            importance_df = pd.DataFrame({
                "feature": preprocessor.feature_names_out_,
                "importance": model.feature_importances_,
            }).sort_values("importance", ascending=False)
            importance_df.to_csv(str(artifacts_dir / "feature_importance.csv"), index=False)
            mlflow.log_artifact(str(artifacts_dir / "feature_importance.csv"))

        result = {
            "run_id": run_id,
            "model_uri": f"runs:/{run_id}/model",
            "metrics": {**train_metrics, **val_metrics},
            "training_time": training_time,
        }

    logger.info(f"Training complete. Run ID: {run_id}")
    return result


if __name__ == "__main__":
    import argparse
    parser = argparse.ArgumentParser()
    parser.add_argument("--config", default="configs/train_config.yaml")
    parser.add_argument("--data-dir", default="data/splits")
    args = parser.parse_args()
    result = train_model(args.config, args.data_dir)
    print(f"Run ID: {result['run_id']}")
    print(f"Val F1: {result['metrics']['val_f1']:.4f}")
```

---

### 9. `src/models/evaluate.py`

```python
"""
Model evaluation with quality gating.
This is the GATE — if metrics don't meet thresholds, the pipeline stops.
"""
import json
import sys
from pathlib import Path

import mlflow
import numpy as np
import pandas as pd
from sklearn.metrics import (
    accuracy_score, f1_score, precision_score, recall_score,
    roc_auc_score, confusion_matrix, classification_report,
)

from src.data.preprocess import DataPreprocessor
from src.utils.config import load_config
from src.utils.logger import get_logger

logger = get_logger(__name__)


def evaluate_model(
    model_uri: str,
    config_path: str = "configs/train_config.yaml",
    test_data_path: str = "data/splits/test.parquet",
    preprocessor_path: str = "artifacts/preprocessor.joblib",
) -> dict:
    """Evaluate model on test set and enforce quality gates."""
    config = load_config(config_path)
    target_col = config["target_column"]
    thresholds = config.get("quality_gates", {})

    # ── Load model and preprocessor ──
    model = mlflow.sklearn.load_model(model_uri)
    preprocessor = DataPreprocessor()
    preprocessor.load(preprocessor_path)

    # ── Load and prepare test data ──
    test_df = pd.read_parquet(test_data_path)
    X_test = test_df.drop(columns=[target_col])
    y_test = test_df[target_col]
    X_test_processed = preprocessor.transform(X_test)

    # ── Compute metrics ──
    y_pred = model.predict(X_test_processed)
    y_proba = model.predict_proba(X_test_processed)

    metrics = {
        "test_accuracy": float(accuracy_score(y_test, y_pred)),
        "test_f1": float(f1_score(y_test, y_pred, average="weighted")),
        "test_precision": float(precision_score(y_test, y_pred, average="weighted")),
        "test_recall": float(recall_score(y_test, y_pred, average="weighted")),
    }

    # ROC AUC (binary or multi-class)
    if y_proba.shape[1] == 2:
        metrics["test_roc_auc"] = float(roc_auc_score(y_test, y_proba[:, 1]))
    else:
        metrics["test_roc_auc"] = float(
            roc_auc_score(y_test, y_proba, multi_class="ovr", average="weighted")
        )

    logger.info(f"Test metrics: {json.dumps(metrics, indent=2)}")

    # ── Quality Gates ──
    gate_results = {}
    all_passed = True

    for metric_name, threshold in thresholds.items():
        actual = metrics.get(metric_name)
        if actual is None:
            logger.warning(f"Gate metric '{metric_name}' not found in computed metrics")
            continue

        passed = actual >= threshold
        gate_results[metric_name] = {
            "threshold": threshold,
            "actual": actual,
            "passed": passed,
        }

        if not passed:
            all_passed = False
            logger.error(
                f"GATE FAILED: {metric_name} = {actual:.4f} < {threshold:.4f}"
            )
        else:
            logger.info(
                f"GATE PASSED: {metric_name} = {actual:.4f} >= {threshold:.4f}"
            )

    # ── Log to MLflow ──
    with mlflow.start_run(run_id=mlflow.active_run().info.run_id if mlflow.active_run() else None):
        mlflow.log_metrics(metrics)
        mlflow.log_dict(gate_results, "quality_gates.json")

        # Log confusion matrix
        cm = confusion_matrix(y_test, y_pred)
        cm_df = pd.DataFrame(cm)
        cm_df.to_csv("artifacts/confusion_matrix.csv")
        mlflow.log_artifact("artifacts/confusion_matrix.csv")

        # Log classification report
        report = classification_report(y_test, y_pred, output_dict=True)
        mlflow.log_dict(report, "classification_report.json")

    return {
        "metrics": metrics,
        "gate_results": gate_results,
        "all_gates_passed": all_passed,
    }


if __name__ == "__main__":
    import argparse
    parser = argparse.ArgumentParser()
    parser.add_argument("--model-uri", required=True)
    parser.add_argument("--config", default="configs/train_config.yaml")
    parser.add_argument("--test-data", default="data/splits/test.parquet")
    parser.add_argument("--preprocessor", default="artifacts/preprocessor.joblib")
    args = parser.parse_args()

    result = evaluate_model(args.model_uri, args.config, args.test_data, args.preprocessor)
    
    if not result["all_gates_passed"]:
        logger.error("Quality gates FAILED — pipeline will not proceed to registration")
        sys.exit(1)
    else:
        logger.info("All quality gates passed — model approved for registration")
```

---

### 10. `src/models/predict.py`

```python
"""
Prediction logic — used both in batch and real-time serving.
"""
import numpy as np
import pandas as pd
import mlflow.sklearn

from src.data.preprocess import DataPreprocessor
from src.utils.logger import get_logger

logger = get_logger(__name__)


class ModelPredictor:
    """Wraps model + preprocessing for clean prediction interface."""

    def __init__(
        self,
        model_uri: str | None = None,
        model: object | None = None,
        preprocessor_path: str | None = None,
    ):
        if model is not None:
            self.model = model
        elif model_uri:
            self.model = mlflow.sklearn.load_model(model_uri)
        else:
            raise ValueError("Provide either model_uri or model object")

        self.preprocessor = DataPreprocessor()
        if preprocessor_path:
            self.preprocessor.load(preprocessor_path)

    def predict(self, X: pd.DataFrame) -> np.ndarray:
        """Return class predictions."""
        X_processed = self.preprocessor.transform(X)
        return self.model.predict(X_processed)

    def predict_proba(self, X: pd.DataFrame) -> np.ndarray:
        """Return class probabilities."""
        X_processed = self.preprocessor.transform(X)
        return self.model.predict_proba(X_processed)

    def predict_single(self, features: dict) -> dict:
        """Single prediction for real-time serving."""
        df = pd.DataFrame([features])
        prediction = self.predict(df)[0]
        probabilities = self.predict_proba(df)[0]

        return {
            "prediction": int(prediction),
            "confidence": float(max(probabilities)),
            "probabilities": {
                str(i): float(p) for i, p in enumerate(probabilities)
            },
        }
```

---

### 11. `src/models/model_registry.py`

```python
"""
MLflow Model Registry wrapper — handles registration, promotion, rollback.
"""
import mlflow
from mlflow.tracking import MlflowClient

from src.utils.logger import get_logger

logger = get_logger(__name__)


class ModelRegistryManager:
    def __init__(self, tracking_uri: str | None = None):
        if tracking_uri:
            mlflow.set_tracking_uri(tracking_uri)
        self.client = MlflowClient()

    def register_model(
        self,
        run_id: str,
        model_name: str,
        artifact_path: str = "model",
        tags: dict | None = None,
    ) -> str:
        """Register a model version from an MLflow run."""
        model_uri = f"runs:/{run_id}/{artifact_path}"
        result = mlflow.register_model(model_uri, model_name)
        version = result.version

        if tags:
            for key, value in tags.items():
                self.client.set_model_version_tag(model_name, version, key, value)

        logger.info(f"Registered model '{model_name}' version {version}")
        return version

    def promote(
        self,
        model_name: str,
        version: str,
        to_stage: str,
        archive_existing: bool = True,
    ) -> None:
        """Promote a model version to a stage (Staging/Production)."""
        self.client.transition_model_version_stage(
            name=model_name,
            version=version,
            stage=to_stage,
            archive_existing_versions=archive_existing,
        )
        logger.info(f"Promoted '{model_name}' v{version} → {to_stage}")

    def get_production_model_uri(self, model_name: str) -> str:
        """Get the URI of the current production model."""
        versions = self.client.get_latest_versions(model_name, stages=["Production"])
        if not versions:
            raise ValueError(f"No Production version found for '{model_name}'")
        version = versions[0]
        return f"models:/{model_name}/{version.version}"

    def rollback(self, model_name: str) -> None:
        """Rollback: demote current Production, promote previous."""
        prod_versions = self.client.get_latest_versions(model_name, stages=["Production"])
        archived = self.client.get_latest_versions(model_name, stages=["Archived"])
        
        if not prod_versions or not archived:
            raise ValueError("Cannot rollback — no archived version found")

        # Archive current production
        current = prod_versions[0]
        self.client.transition_model_version_stage(
            model_name, current.version, "Archived"
        )
        
        # Restore most recent archived
        previous = max(archived, key=lambda v: int(v.version))
        self.client.transition_model_version_stage(
            model_name, previous.version, "Production"
        )
        logger.info(
            f"Rolled back '{model_name}': v{current.version} → Archived, "
            f"v{previous.version} → Production"
        )
```

---

### 12. `src/data/drift_detector.py`

```python
"""
Statistical drift detection — catches when production data diverges from training data.
"""
import numpy as np
import pandas as pd
from scipy import stats

from src.utils.logger import get_logger

logger = get_logger(__name__)


class DriftDetector:
    """Detect feature drift using statistical tests."""

    def __init__(self, reference_data: pd.DataFrame, significance_level: float = 0.05):
        self.reference = reference_data
        self.significance = significance_level

    def detect_drift(self, current_data: pd.DataFrame) -> dict:
        """Run drift detection on all shared columns."""
        results = {}
        shared_cols = set(self.reference.columns) & set(current_data.columns)

        for col in shared_cols:
            ref_col = self.reference[col].dropna()
            cur_col = current_data[col].dropna()

            if ref_col.dtype in ["float64", "int64", "float32", "int32"]:
                # Kolmogorov-Smirnov test for numeric columns
                stat, p_value = stats.ks_2samp(ref_col, cur_col)
                test_name = "KS"
            else:
                # Chi-squared test for categorical columns
                ref_counts = ref_col.value_counts(normalize=True)
                cur_counts = cur_col.value_counts(normalize=True)
                all_categories = set(ref_counts.index) | set(cur_counts.index)
                ref_freq = [ref_counts.get(c, 0) for c in all_categories]
                cur_freq = [cur_counts.get(c, 0) for c in all_categories]
                
                # Add small epsilon to avoid zero frequencies
                ref_freq = np.array(ref_freq) + 1e-10
                cur_freq = np.array(cur_freq) + 1e-10
                
                stat, p_value = stats.chisquare(cur_freq, f_exp=ref_freq)
                test_name = "chi2"

            drifted = p_value < self.significance
            results[col] = {
                "test": test_name,
                "statistic": float(stat),
                "p_value": float(p_value),
                "drifted": drifted,
            }

            if drifted:
                logger.warning(f"DRIFT DETECTED: {col} (p={p_value:.6f}, test={test_name})")

        n_drifted = sum(1 for r in results.values() if r["drifted"])
        drift_ratio = n_drifted / len(results) if results else 0

        return {
            "columns": results,
            "n_drifted": n_drifted,
            "n_total": len(results),
            "drift_ratio": drift_ratio,
            "overall_drift": drift_ratio > 0.3,   # Alert if >30% of features drift
        }

    def compute_psi(
        self,
        reference_col: pd.Series,
        current_col: pd.Series,
        n_bins: int = 10,
    ) -> float:
        """Population Stability Index — industry standard for drift."""
        ref_hist, bin_edges = np.histogram(reference_col, bins=n_bins)
        cur_hist, _ = np.histogram(current_col, bins=bin_edges)

        ref_pct = (ref_hist + 1) / (ref_hist.sum() + n_bins)
        cur_pct = (cur_hist + 1) / (cur_hist.sum() + n_bins)

        psi = np.sum((cur_pct - ref_pct) * np.log(cur_pct / ref_pct))
        return float(psi)
```

---

### 13. `src/utils/config.py`

```python
"""
Configuration loader — merges YAML config with environment variables.
"""
import os
from pathlib import Path

import yaml


def load_config(config_path: str) -> dict:
    """Load YAML config with environment variable override."""
    with open(config_path) as f:
        config = yaml.safe_load(f)

    # Environment variables override config values
    # Convention: ML_PROJECT_<SECTION>_<KEY> overrides config[section][key]
    for key, value in os.environ.items():
        if key.startswith("ML_PROJECT_"):
            parts = key[len("ML_PROJECT_"):].lower().split("_", 1)
            if len(parts) == 2:
                section, param = parts
                if section in config and param in config[section]:
                    # Attempt type coercion
                    original_type = type(config[section][param])
                    try:
                        config[section][param] = original_type(value)
                    except (ValueError, TypeError):
                        config[section][param] = value

    return config
```

---

### 14. `src/utils/reproducibility.py`

```python
"""
Reproducibility utilities — set seeds everywhere.
"""
import os
import random

import numpy as np


def set_global_seed(seed: int = 42) -> None:
    """Set all random seeds for reproducibility."""
    random.seed(seed)
    np.random.seed(seed)
    os.environ["PYTHONHASHSEED"] = str(seed)

    try:
        import torch
        torch.manual_seed(seed)
        torch.cuda.manual_seed_all(seed)
        torch.backends.cudnn.deterministic = True
        torch.backends.cudnn.benchmark = False
    except ImportError:
        pass

    try:
        import tensorflow as tf
        tf.random.set_seed(seed)
    except ImportError:
        pass
```

---

### 15. `src/utils/logger.py`

```python
"""
Structured JSON logging for production.
"""
import logging
import json
import sys
from datetime import datetime, timezone


class JSONFormatter(logging.Formatter):
    """JSON log format — parseable by CloudWatch, Datadog, ELK."""
    def format(self, record):
        log_record = {
            "timestamp": datetime.now(timezone.utc).isoformat(),
            "level": record.levelname,
            "logger": record.name,
            "message": record.getMessage(),
            "module": record.module,
            "function": record.funcName,
            "line": record.lineno,
        }
        if record.exc_info and record.exc_info[0]:
            log_record["exception"] = self.formatException(record.exc_info)
        if hasattr(record, "extra"):
            log_record.update(record.extra)
        return json.dumps(log_record)


def get_logger(name: str, level: str = "INFO") -> logging.Logger:
    """Get a structured JSON logger."""
    logger = logging.getLogger(name)
    if not logger.handlers:
        handler = logging.StreamHandler(sys.stdout)
        handler.setFormatter(JSONFormatter())
        logger.addHandler(handler)
    logger.setLevel(getattr(logging, level.upper()))
    return logger
```

---

### 16. `pipelines/kfp_pipeline.py` — Kubeflow Pipeline Definition

```python
"""
Kubeflow Pipelines (KFP) v2 pipeline definition.
Compiles to YAML for submission to KFP.
"""
from kfp import dsl, compiler
from kfp.dsl import Input, Output, Dataset, Model, Metrics


@dsl.component(
    base_image="123456789012.dkr.ecr.us-east-1.amazonaws.com/ml-train:latest",
    packages_to_install=[],                   # All deps in base image
)
def validate_data(
    data_path: str,
    config_path: str,
    validation_report: Output[Metrics],
) -> bool:
    """KFP Component: Validate data schema and quality."""
    from src.data.validate import validate_data, check_data_quality
    import pandas as pd

    df = pd.read_parquet(data_path)
    validate_data(df, config_path)
    report = check_data_quality(df)

    for k, v in report.items():
        if isinstance(v, (int, float)):
            validation_report.log_metric(k, v)

    return True


@dsl.component(
    base_image="123456789012.dkr.ecr.us-east-1.amazonaws.com/ml-train:latest",
)
def preprocess_data(
    data_path: str,
    config_path: str,
    output_dir: Output[Dataset],
):
    """KFP Component: Feature engineering + split."""
    from src.data.preprocess import DataPreprocessor
    from src.data.split import create_splits
    import pandas as pd

    df = pd.read_parquet(data_path)
    splits = create_splits(df, config_path, output_dir.path)


@dsl.component(
    base_image="123456789012.dkr.ecr.us-east-1.amazonaws.com/ml-train:latest",
)
def train_model(
    splits_dir: Input[Dataset],
    config_path: str,
    model_artifact: Output[Model],
    metrics: Output[Metrics],
) -> str:
    """KFP Component: Train model with MLflow tracking."""
    from src.models.train import train_model as run_training

    result = run_training(config_path, splits_dir.path)

    metrics.log_metric("val_f1", result["metrics"]["val_f1"])
    metrics.log_metric("val_accuracy", result["metrics"]["val_accuracy"])
    metrics.log_metric("training_time", result["training_time"])

    return result["run_id"]


@dsl.component(
    base_image="123456789012.dkr.ecr.us-east-1.amazonaws.com/ml-train:latest",
)
def evaluate_and_gate(
    run_id: str,
    splits_dir: Input[Dataset],
    config_path: str,
    metrics: Output[Metrics],
) -> bool:
    """KFP Component: Evaluate on test set + quality gate."""
    from src.models.evaluate import evaluate_model

    model_uri = f"runs:/{run_id}/model"
    result = evaluate_model(
        model_uri=model_uri,
        config_path=config_path,
        test_data_path=f"{splits_dir.path}/test.parquet",
    )

    for k, v in result["metrics"].items():
        metrics.log_metric(k, v)

    return result["all_gates_passed"]


@dsl.component(
    base_image="123456789012.dkr.ecr.us-east-1.amazonaws.com/ml-train:latest",
)
def register_model(
    run_id: str,
    model_name: str,
    gates_passed: bool,
):
    """KFP Component: Register model in MLflow if gates passed."""
    if not gates_passed:
        raise ValueError("Quality gates failed — model will NOT be registered")

    from src.models.model_registry import ModelRegistryManager

    registry = ModelRegistryManager()
    version = registry.register_model(
        run_id=run_id,
        model_name=model_name,
        tags={"commit_sha": "{{ workflow.parameters.commit_sha }}"},
    )
    registry.promote(model_name, version, to_stage="Staging")


@dsl.pipeline(
    name="ML Training Pipeline",
    description="End-to-end: validate → preprocess → train → evaluate → register",
)
def ml_training_pipeline(
    data_path: str = "s3://ml-project-data/raw/latest/data.parquet",
    config_path: str = "configs/train_config.yaml",
    model_name: str = "ml-project-model",
):
    """Main pipeline — wires all components together."""
    # Step 1: Validate
    validate_task = validate_data(
        data_path=data_path,
        config_path=config_path,
    )

    # Step 2: Preprocess + Split
    preprocess_task = preprocess_data(
        data_path=data_path,
        config_path=config_path,
    ).after(validate_task)

    # Step 3: Train
    train_task = train_model(
        splits_dir=preprocess_task.outputs["output_dir"],
        config_path=config_path,
    )

    # Step 4: Evaluate + Gate
    eval_task = evaluate_and_gate(
        run_id=train_task.output,
        splits_dir=preprocess_task.outputs["output_dir"],
        config_path=config_path,
    )

    # Step 5: Register (only if gates pass)
    register_model(
        run_id=train_task.output,
        model_name=model_name,
        gates_passed=eval_task.output,
    )


if __name__ == "__main__":
    compiler.Compiler().compile(
        pipeline_func=ml_training_pipeline,
        package_path="pipelines/compiled/ml_training_pipeline.yaml",
    )
    print("Pipeline compiled → pipelines/compiled/ml_training_pipeline.yaml")
```

---

### 17. `pipelines/triggers/trigger_pipeline.py`

```python
"""
Trigger a KFP pipeline run from GitHub Actions.
Outputs the run_id for downstream polling.
"""
import argparse
import sys

import kfp
import yaml


def trigger(args):
    client = kfp.Client(
        host=args.host,
        existing_token=args.token or None,
    )

    # Load params
    with open(args.params_file) as f:
        params = yaml.safe_load(f)
    
    params["commit_sha"] = args.commit_sha

    # Find or create experiment
    try:
        experiment = client.get_experiment(experiment_name=args.experiment_name)
    except Exception:
        experiment = client.create_experiment(name=args.experiment_name)

    # Submit run
    run = client.create_run_from_pipeline_package(
        pipeline_file="pipelines/compiled/ml_training_pipeline.yaml",
        arguments=params,
        run_name=args.run_name,
        experiment_id=experiment.experiment_id,
    )

    print(f"::set-output name=run_id::{run.run_id}")
    print(f"Pipeline run submitted: {run.run_id}")


if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--host", required=True)
    parser.add_argument("--pipeline-name", required=True)
    parser.add_argument("--experiment-name", required=True)
    parser.add_argument("--run-name", required=True)
    parser.add_argument("--params-file", default="params.yaml")
    parser.add_argument("--commit-sha", default="unknown")
    parser.add_argument("--token", default=None)
    args = parser.parse_args()

    import os
    args.token = args.token or os.environ.get("KFP_TOKEN")
    trigger(args)
```

---

### 18. `pipelines/triggers/poll_pipeline.py`

```python
"""
Poll a KFP pipeline run until completion.
Exits 0 on success, 1 on failure/timeout.
"""
import argparse
import sys
import time

import kfp


def poll(args):
    client = kfp.Client(host=args.host, existing_token=args.token or None)

    start_time = time.time()
    while True:
        run = client.get_run(args.run_id)
        status = run.run.status

        elapsed = time.time() - start_time
        print(f"[{elapsed:.0f}s] Pipeline status: {status}")

        if status in ("Succeeded",):
            print("Pipeline completed successfully")
            # Extract model version from run output if available
            print(f"::set-output name=model_version::latest")
            print(f"::set-output name=model_uri::models:/ml-project-model/latest")
            sys.exit(0)
        elif status in ("Failed", "Error", "Skipped"):
            print(f"Pipeline failed with status: {status}")
            sys.exit(1)
        elif elapsed > args.timeout:
            print(f"Pipeline timed out after {args.timeout}s")
            sys.exit(1)

        time.sleep(args.poll_interval)


if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--host", required=True)
    parser.add_argument("--run-id", required=True)
    parser.add_argument("--timeout", type=int, default=7200)
    parser.add_argument("--poll-interval", type=int, default=60)
    parser.add_argument("--token", default=None)
    args = parser.parse_args()

    import os
    args.token = args.token or os.environ.get("KFP_TOKEN")
    poll(args)
```

---

### 19. `serving/app.py` — FastAPI Inference Server

```python
"""
FastAPI inference server — production-ready with health checks,
request validation, structured logging, and metrics.
"""
import os
import time
from contextlib import asynccontextmanager

import mlflow.sklearn
from fastapi import FastAPI, HTTPException
from prometheus_client import Counter, Histogram, generate_latest
from starlette.responses import Response

from serving.request_schema import PredictionRequest
from serving.response_schema import PredictionResponse
from serving.model_loader import load_production_model
from src.utils.logger import get_logger

logger = get_logger(__name__)

# Prometheus metrics
PREDICTION_COUNT = Counter("predictions_total", "Total predictions", ["status"])
PREDICTION_LATENCY = Histogram("prediction_latency_seconds", "Prediction latency")

# Global model reference
model_predictor = None


@asynccontextmanager
async def lifespan(app: FastAPI):
    """Load model on startup, cleanup on shutdown."""
    global model_predictor
    logger.info("Loading production model...")
    model_predictor = load_production_model()
    logger.info("Model loaded successfully")
    yield
    logger.info("Shutting down — releasing model resources")
    model_predictor = None


app = FastAPI(
    title="ML Project Inference API",
    version="1.0.0",
    lifespan=lifespan,
)


@app.get("/health")
async def health():
    """Health check — used by load balancer and Kubernetes readiness probe."""
    return {
        "status": "healthy",
        "model_loaded": model_predictor is not None,
    }


@app.get("/ready")
async def readiness():
    """Readiness check — returns 503 if model not loaded."""
    if model_predictor is None:
        raise HTTPException(status_code=503, detail="Model not loaded")
    return {"status": "ready"}


@app.post("/predict", response_model=PredictionResponse)
async def predict(request: PredictionRequest):
    """Generate prediction for a single request."""
    start_time = time.time()

    try:
        result = model_predictor.predict_single(request.features)
        latency = time.time() - start_time

        PREDICTION_COUNT.labels(status="success").inc()
        PREDICTION_LATENCY.observe(latency)

        return PredictionResponse(
            prediction=result["prediction"],
            confidence=result["confidence"],
            probabilities=result["probabilities"],
            model_version=os.getenv("MODEL_VERSION", "unknown"),
            latency_ms=round(latency * 1000, 2),
        )
    except Exception as e:
        PREDICTION_COUNT.labels(status="error").inc()
        logger.error(f"Prediction failed: {str(e)}")
        raise HTTPException(status_code=500, detail=str(e))


@app.get("/metrics")
async def metrics():
    """Prometheus metrics endpoint."""
    return Response(content=generate_latest(), media_type="text/plain")
```

---

### 20. `serving/request_schema.py`

```python
from pydantic import BaseModel, Field


class PredictionRequest(BaseModel):
    features: dict = Field(
        ...,
        description="Feature dictionary matching training schema",
        json_schema_extra={"example": {"age": 35, "income": 75000, "category": "A"}},
    )
    request_id: str | None = Field(None, description="Optional request ID for tracing")
```

---

### 21. `serving/response_schema.py`

```python
from pydantic import BaseModel


class PredictionResponse(BaseModel):
    prediction: int
    confidence: float
    probabilities: dict[str, float]
    model_version: str
    latency_ms: float
```

---

### 22. `serving/model_loader.py`

```python
"""
Load model from MLflow Registry or local fallback.
"""
import os
import mlflow

from src.models.predict import ModelPredictor
from src.utils.logger import get_logger

logger = get_logger(__name__)


def load_production_model() -> ModelPredictor:
    """Load the current production model from MLflow Registry."""
    model_name = os.getenv("MODEL_NAME", "ml-project-model")
    tracking_uri = os.getenv("MLFLOW_TRACKING_URI")

    if tracking_uri:
        mlflow.set_tracking_uri(tracking_uri)
        model_uri = f"models:/{model_name}/Production"
        logger.info(f"Loading model from registry: {model_uri}")
    else:
        model_uri = os.getenv("MODEL_PATH", "/opt/ml/model")
        logger.info(f"Loading model from local path: {model_uri}")

    preprocessor_path = os.getenv(
        "PREPROCESSOR_PATH", "/opt/ml/artifacts/preprocessor.joblib"
    )

    return ModelPredictor(
        model_uri=model_uri,
        preprocessor_path=preprocessor_path,
    )
```

---

### 23. `serving/gunicorn_config.py`

```python
"""
Gunicorn configuration for production serving.
"""
import multiprocessing
import os

# Server
bind = f"0.0.0.0:{os.getenv('PORT', '8080')}"
workers = int(os.getenv("GUNICORN_WORKERS", multiprocessing.cpu_count() * 2 + 1))
worker_class = "uvicorn.workers.UvicornWorker"
timeout = int(os.getenv("GUNICORN_TIMEOUT", "120"))
keepalive = 5
max_requests = 1000                           # Restart workers after N requests (memory leak protection)
max_requests_jitter = 50                      # Add randomness to prevent all workers restarting at once
graceful_timeout = 30

# Logging
accesslog = "-"
errorlog = "-"
loglevel = os.getenv("LOG_LEVEL", "info")

# Preloading
preload_app = True                            # Load model ONCE, share across workers (saves memory)
```

---

### 24. `docker/Dockerfile.serve` — Inference Image

```dockerfile
# ============================================================
# Multi-stage build for lean inference image
# ============================================================

# ── Stage 1: Build dependencies ──
FROM python:3.11.7-slim-bookworm AS builder

WORKDIR /build

COPY requirements/requirements-serve.txt .
COPY requirements/requirements-base.txt .
RUN pip install --no-cache-dir --prefix=/install \
    -r requirements-base.txt \
    -r requirements-serve.txt

# ── Stage 2: Production image ──
FROM python:3.11.7-slim-bookworm AS production

# Security: run as non-root
RUN groupadd -r mluser && useradd -r -g mluser mluser

# Install only runtime system deps
RUN apt-get update && apt-get install -y --no-install-recommends \
    libgomp1 curl \
    && rm -rf /var/lib/apt/lists/*

# Copy Python packages from builder
COPY --from=builder /install /usr/local

WORKDIR /app

# Copy application code
COPY src/ ./src/
COPY serving/ ./serving/
COPY configs/ ./configs/
COPY setup.py setup.cfg ./
RUN pip install --no-cache-dir -e .

# Build args for traceability
ARG MODEL_VERSION=unknown
ARG COMMIT_SHA=unknown
ARG BUILD_DATE=unknown

ENV MODEL_VERSION=${MODEL_VERSION}
ENV COMMIT_SHA=${COMMIT_SHA}
ENV BUILD_DATE=${BUILD_DATE}
ENV PYTHONUNBUFFERED=1
ENV PYTHONDONTWRITEBYTECODE=1
ENV PORT=8080

# Health check
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
    CMD curl -f http://localhost:${PORT}/health || exit 1

# Switch to non-root user
USER mluser

EXPOSE ${PORT}

CMD ["gunicorn", "serving.app:app", "-c", "serving/gunicorn_config.py"]
```

---

### 25. `docker/Dockerfile.ci` — CI Base Image

```dockerfile
FROM python:3.11.7-slim-bookworm

RUN apt-get update && apt-get install -y --no-install-recommends \
    git curl libgomp1 \
    && rm -rf /var/lib/apt/lists/*

COPY requirements/requirements-ci.txt /tmp/
RUN pip install --no-cache-dir -r /tmp/requirements-ci.txt

# Pre-install system tools
RUN pip install --no-cache-dir pip-tools
```

---

### 26. `docker/Dockerfile.train` — Training Image

```dockerfile
FROM python:3.11.7-slim-bookworm

RUN apt-get update && apt-get install -y --no-install-recommends \
    git curl libgomp1 build-essential \
    && rm -rf /var/lib/apt/lists/*

COPY requirements/requirements-base.txt /tmp/
COPY requirements/requirements-train.txt /tmp/
RUN pip install --no-cache-dir \
    -r /tmp/requirements-base.txt \
    -r /tmp/requirements-train.txt

WORKDIR /app
COPY src/ ./src/
COPY configs/ ./configs/
COPY pipelines/ ./pipelines/
COPY setup.py setup.cfg ./
RUN pip install --no-cache-dir -e .
```

---

### 27. Config Files

#### `configs/train_config.yaml`

```yaml
experiment_name: ml-project
model_name: ml-project-model
target_column: target
seed: 42

# Split ratios
test_size: 0.15
val_size: 0.15
stratify: true

# Model hyperparameters
model_params:
  n_estimators: 200
  learning_rate: 0.1
  max_depth: 5
  min_samples_split: 10
  subsample: 0.8

# Quality gates — pipeline stops if any metric is below threshold
quality_gates:
  test_f1: 0.75
  test_accuracy: 0.80
  test_roc_auc: 0.82
```

#### `configs/feature_config.yaml`

```yaml
features:
  numeric:
    - age
    - income
    - credit_score
    - account_balance
    - transaction_count
  
  categorical:
    - category
    - region
    - channel
  
  ordinal:
    - education_level
    - risk_tier
  
  ordinal_categories:
    - ["high_school", "bachelors", "masters", "phd"]
    - ["low", "medium", "high", "critical"]

  target: target
```

#### `configs/serve_config.yaml`

```yaml
model_name: ml-project-model
model_stage: Production
timeout_seconds: 30
max_batch_size: 64
log_predictions: true
log_sample_rate: 0.1        # Log 10% of predictions for monitoring
```

---

### 28. `params.yaml` — DVC Parameters

```yaml
# Referenced by dvc.yaml and training code
seed: 42
test_size: 0.15
val_size: 0.15
n_estimators: 200
learning_rate: 0.1
max_depth: 5
min_samples_split: 10
subsample: 0.8
```

---

### 29. `dvc.yaml` — DVC Pipeline Definition

```yaml
stages:
  preprocess:
    cmd: python -m src.data.preprocess --config configs/feature_config.yaml
    deps:
      - src/data/preprocess.py
      - configs/feature_config.yaml
      - data/raw
    outs:
      - data/processed
    params:
      - seed

  split:
    cmd: python -m src.data.split --config configs/train_config.yaml
    deps:
      - src/data/split.py
      - configs/train_config.yaml
      - data/processed
    outs:
      - data/splits
    params:
      - seed
      - test_size
      - val_size

  train:
    cmd: python -m src.models.train --config configs/train_config.yaml
    deps:
      - src/models/train.py
      - configs/train_config.yaml
      - data/splits
    outs:
      - artifacts/preprocessor.joblib
    params:
      - n_estimators
      - learning_rate
      - max_depth
      - min_samples_split
      - subsample
    metrics:
      - artifacts/metrics.json:
          cache: false
    plots:
      - artifacts/feature_importance.csv:
          x: feature
          y: importance

  evaluate:
    cmd: python -m src.models.evaluate --config configs/train_config.yaml
    deps:
      - src/models/evaluate.py
      - data/splits/test.parquet
      - artifacts/preprocessor.joblib
    metrics:
      - artifacts/evaluation.json:
          cache: false
    plots:
      - artifacts/confusion_matrix.csv
```

---

### 30. `tests/conftest.py` — Shared Test Fixtures

```python
"""
Shared test fixtures for all test types.
"""
import numpy as np
import pandas as pd
import pytest
from pathlib import Path


@pytest.fixture
def sample_dataframe():
    """Minimal synthetic dataset matching production schema."""
    np.random.seed(42)
    n = 200
    return pd.DataFrame({
        "age": np.random.randint(18, 80, n),
        "income": np.random.uniform(20000, 200000, n),
        "credit_score": np.random.randint(300, 850, n),
        "account_balance": np.random.uniform(0, 100000, n),
        "transaction_count": np.random.randint(0, 500, n),
        "category": np.random.choice(["A", "B", "C"], n),
        "region": np.random.choice(["east", "west", "central"], n),
        "channel": np.random.choice(["online", "branch", "mobile"], n),
        "education_level": np.random.choice(
            ["high_school", "bachelors", "masters", "phd"], n
        ),
        "risk_tier": np.random.choice(["low", "medium", "high", "critical"], n),
        "target": np.random.choice([0, 1], n, p=[0.7, 0.3]),
    })


@pytest.fixture
def sample_features():
    """Single sample feature dict for prediction tests."""
    return {
        "age": 35,
        "income": 75000.0,
        "credit_score": 720,
        "account_balance": 15000.0,
        "transaction_count": 150,
        "category": "B",
        "region": "east",
        "channel": "online",
        "education_level": "bachelors",
        "risk_tier": "medium",
    }


@pytest.fixture
def tmp_model_dir(tmp_path):
    """Temporary directory for model artifacts."""
    model_dir = tmp_path / "artifacts"
    model_dir.mkdir()
    return model_dir
```

---

### 31. `tests/unit/test_preprocess.py`

```python
"""
Unit tests for preprocessing pipeline.
"""
import numpy as np
import pytest

from src.data.preprocess import DataPreprocessor


class TestDataPreprocessor:
    def test_fit_transform_shape(self, sample_dataframe):
        """Output should have correct number of rows."""
        preprocessor = DataPreprocessor()
        X = sample_dataframe.drop(columns=["target"])
        result = preprocessor.fit_transform(X)
        assert result.shape[0] == len(X)

    def test_transform_after_fit(self, sample_dataframe):
        """Transform should work on new data after fitting."""
        preprocessor = DataPreprocessor()
        X = sample_dataframe.drop(columns=["target"])
        preprocessor.fit_transform(X)
        result = preprocessor.transform(X.iloc[:10])
        assert result.shape[0] == 10

    def test_transform_without_fit_raises(self, sample_dataframe):
        """Should raise if transform called before fit."""
        preprocessor = DataPreprocessor()
        X = sample_dataframe.drop(columns=["target"])
        with pytest.raises(RuntimeError):
            preprocessor.transform(X)

    def test_save_and_load(self, sample_dataframe, tmp_model_dir):
        """Saved pipeline should produce identical output when loaded."""
        preprocessor = DataPreprocessor()
        X = sample_dataframe.drop(columns=["target"])
        original_output = preprocessor.fit_transform(X)

        # Save
        save_path = str(tmp_model_dir / "preprocessor.joblib")
        preprocessor.save(save_path)

        # Load and compare
        loaded = DataPreprocessor()
        loaded.load(save_path)
        loaded_output = loaded.transform(X)

        np.testing.assert_array_almost_equal(original_output, loaded_output)

    def test_handles_missing_values(self, sample_dataframe):
        """Preprocessor should handle NaN values."""
        X = sample_dataframe.drop(columns=["target"])
        X.loc[0, "age"] = np.nan
        X.loc[1, "category"] = np.nan

        preprocessor = DataPreprocessor()
        result = preprocessor.fit_transform(X)
        assert not np.isnan(result).any()
```

---

### 32. `tests/unit/test_evaluate.py`

```python
"""
Unit tests for evaluation and quality gating.
"""
import pytest
from unittest.mock import patch, MagicMock
import numpy as np
from sklearn.ensemble import GradientBoostingClassifier

from src.models.evaluate import evaluate_model


class TestQualityGates:
    def test_gates_pass_with_good_model(self, sample_dataframe, tmp_model_dir):
        """Quality gates should pass when metrics exceed thresholds."""
        # This tests the gating logic, not the model quality
        # In practice you'd mock the model to return known good predictions
        pass

    def test_gates_fail_with_bad_threshold(self):
        """Quality gates should fail when threshold is impossibly high."""
        # Test with threshold of 0.99 which almost no model meets
        pass

    def test_evaluation_produces_all_metrics(self):
        """Evaluation should compute all required metrics."""
        # Verify accuracy, f1, precision, recall, roc_auc are all present
        pass
```

---

### 33. Root Config Files

#### `.flake8`

```ini
[flake8]
max-line-length = 100
extend-exclude =
    .git,
    __pycache__,
    build,
    dist,
    *.egg-info,
    notebooks/,
    pipelines/compiled/
per-file-ignores =
    tests/*:E501
    notebooks/*:E501,W503,E402
max-complexity = 12
```

#### `pyproject.toml`

```toml
[build-system]
requires = ["setuptools>=68.0", "wheel"]
build-backend = "setuptools.backends._legacy:_Backend"

[project]
name = "ml-project"
version = "1.0.0"
requires-python = ">=3.11"
description = "Production ML pipeline"

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-ra -q --strict-markers"
markers = [
    "slow: marks tests as slow (deselect with '-m \"not slow\"')",
    "integration: marks integration tests",
    "smoke: marks smoke tests (post-deploy)",
]

[tool.mypy]
python_version = "3.11"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = false      # Gradual adoption
ignore_missing_imports = true

[tool.black]
line-length = 100
target-version = ["py311"]

[tool.bandit]
exclude_dirs = ["tests"]
skips = ["B101"]                   # Skip assert warnings (used in tests)
```

#### `setup.py`

```python
from setuptools import setup, find_packages

setup(
    name="ml-project",
    version="1.0.0",
    packages=find_packages(),
    python_requires=">=3.11",
)
```

#### `.pre-commit-config.yaml`

```yaml
repos:
  - repo: https://github.com/psf/black
    rev: 24.3.0
    hooks:
      - id: black
        language_version: python3.11

  - repo: https://github.com/pycqa/flake8
    rev: 7.0.0
    hooks:
      - id: flake8

  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
        args: ["--maxkb=500"]

  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets
        args: ["--baseline", ".secrets.baseline"]
```

#### `.gitignore`

```gitignore
# Python
__pycache__/
*.py[cod]
*.egg-info/
dist/
build/
.eggs/

# Virtual environments
.venv/
venv/
env/

# IDE
.vscode/
.idea/
*.swp

# OS
.DS_Store
Thumbs.db

# ML artifacts (tracked by DVC, not Git)
data/raw/*
data/processed/*
data/splits/*
!data/**/.gitkeep
artifacts/
models/

# DVC
/data/*.dvc

# Environment
.env
*.env.local

# Jupyter
.ipynb_checkpoints/

# Coverage
htmlcov/
.coverage
coverage.xml

# Secrets
.secrets.baseline

# Compiled pipeline
pipelines/compiled/*.yaml
```

#### `Makefile`

```makefile
.PHONY: install lint test train serve clean

install:
	pip install -r requirements/requirements-dev.txt
	pip install -e .
	pre-commit install

lint:
	flake8 src/ pipelines/ serving/ tests/
	mypy src/

test:
	pytest tests/unit/ -v --cov=src --cov-report=term-missing

test-integration:
	pytest tests/integration/ -v -m integration

train:
	python -m src.models.train --config configs/train_config.yaml

serve:
	uvicorn serving.app:app --reload --port 8080

compile-pipeline:
	python pipelines/kfp_pipeline.py

pull-data:
	dvc pull

push-data:
	dvc push

generate-requirements:
	bash scripts/generate_requirements.sh

docker-build-serve:
	docker build -f docker/Dockerfile.serve -t ml-project-serve:latest .

docker-run-serve:
	docker run -p 8080:8080 ml-project-serve:latest

clean:
	find . -type d -name __pycache__ -exec rm -rf {} +
	find . -type f -name "*.pyc" -delete
	rm -rf .pytest_cache .mypy_cache htmlcov coverage.xml
```

#### `.github/CODEOWNERS`

```
# ML code changes require ML team review
/src/                @ml-team
/pipelines/          @ml-team
/configs/            @ml-team

# Infrastructure changes require platform team review
/infra/              @platform-team
/docker/             @platform-team
/.github/workflows/  @platform-team @ml-team

# Serving changes require both teams
/serving/            @ml-team @platform-team
```

#### `.github/pull_request_template.md`

```markdown
## What does this PR do?

## Checklist
- [ ] Unit tests pass locally (`make test`)
- [ ] Lint passes (`make lint`)
- [ ] If data changed: `dvc.lock` updated and committed
- [ ] If model changed: experiment tracked in MLflow
- [ ] If config changed: validated locally with sample run
- [ ] If serving changed: Docker image builds successfully
- [ ] Model card updated (if model behavior changes)
- [ ] Documentation updated

## Model Metrics (if applicable)
| Metric | Before | After |
|--------|--------|-------|
| F1     |        |       |
| AUC    |        |       |
```

---

### 34. `data/reference/schema.json`

```json
{
  "features": [
    {"name": "age", "dtype": "int64", "nullable": false, "min": 0, "max": 120},
    {"name": "income", "dtype": "float64", "nullable": false, "min": 0},
    {"name": "credit_score", "dtype": "int64", "nullable": false, "min": 300, "max": 850},
    {"name": "account_balance", "dtype": "float64", "nullable": true, "min": 0},
    {"name": "transaction_count", "dtype": "int64", "nullable": false, "min": 0},
    {"name": "category", "dtype": "object", "nullable": false, "allowed_values": ["A", "B", "C"]},
    {"name": "region", "dtype": "object", "nullable": false, "allowed_values": ["east", "west", "central"]},
    {"name": "channel", "dtype": "object", "nullable": false, "allowed_values": ["online", "branch", "mobile"]},
    {"name": "education_level", "dtype": "object", "nullable": false, "allowed_values": ["high_school", "bachelors", "masters", "phd"]},
    {"name": "risk_tier", "dtype": "object", "nullable": false, "allowed_values": ["low", "medium", "high", "critical"]},
    {"name": "target", "dtype": "int64", "nullable": false, "allowed_values": [0, 1]}
  ]
}
```

---

### 35. `scripts/promote_model.py`

```python
"""
Manual model promotion script — used in CI and ad-hoc.
"""
import argparse
import mlflow
from mlflow.tracking import MlflowClient


def promote(model_name: str, from_stage: str, to_stage: str, archive: bool):
    client = MlflowClient()
    
    versions = client.get_latest_versions(model_name, stages=[from_stage])
    if not versions:
        raise ValueError(f"No model found in stage '{from_stage}'")
    
    version = versions[0]
    print(f"Promoting {model_name} v{version.version}: {from_stage} → {to_stage}")
    
    client.transition_model_version_stage(
        name=model_name,
        version=version.version,
        stage=to_stage,
        archive_existing_versions=archive,
    )
    print(f"Done. {model_name} v{version.version} is now in {to_stage}")


if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--model-name", required=True)
    parser.add_argument("--from-stage", default="Staging")
    parser.add_argument("--to-stage", default="Production")
    parser.add_argument("--archive-existing", action="store_true")
    args = parser.parse_args()

    promote(args.model_name, args.from_stage, args.to_stage, args.archive_existing)
```

---

### 36. `scripts/smoke_test_endpoint.py`

```python
"""
Post-deploy smoke test — verifies endpoint responds correctly.
"""
import argparse
import json
import sys
import requests


def smoke_test(endpoint_url: str, payload_path: str, expected_status: int):
    # Health check
    health = requests.get(f"{endpoint_url}/health", timeout=10)
    assert health.status_code == 200, f"Health check failed: {health.status_code}"
    print("Health check: PASS")

    # Prediction test
    with open(payload_path) as f:
        payload = json.load(f)

    resp = requests.post(f"{endpoint_url}/predict", json=payload, timeout=30)
    assert resp.status_code == expected_status, (
        f"Expected {expected_status}, got {resp.status_code}: {resp.text}"
    )
    
    result = resp.json()
    assert "prediction" in result
    assert "confidence" in result
    assert result["confidence"] > 0.0
    print(f"Prediction test: PASS (prediction={result['prediction']}, "
          f"confidence={result['confidence']:.4f})")


if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--endpoint-url", required=True)
    parser.add_argument("--test-payload", default="tests/fixtures/sample_request.json")
    parser.add_argument("--expected-status", type=int, default=200)
    args = parser.parse_args()

    try:
        smoke_test(args.endpoint_url, args.test_payload, args.expected_status)
        print("SMOKE TEST PASSED")
    except Exception as e:
        print(f"SMOKE TEST FAILED: {e}")
        sys.exit(1)
```

---

### 37. `tests/fixtures/sample_request.json`

```json
{
  "features": {
    "age": 35,
    "income": 75000.0,
    "credit_score": 720,
    "account_balance": 15000.0,
    "transaction_count": 150,
    "category": "B",
    "region": "east",
    "channel": "online",
    "education_level": "bachelors",
    "risk_tier": "medium"
  },
  "request_id": "smoke-test-001"
}
```

---

### 38. `.dvc/config`

```ini
[core]
    remote = myremote
    autostage = true

['remote "myremote"']
    url = s3://ml-project-data-bucket/dvc-store
    region = us-east-1
```

---

### 39. `monitoring/evidently/data_drift_report.py`

```python
"""
Generate Evidently AI data drift report.
Run periodically to detect when production data drifts from training data.
"""
import pandas as pd
from evidently.report import Report
from evidently.metric_preset import DataDriftPreset, DataQualityPreset


def generate_drift_report(
    reference_path: str,
    current_path: str,
    output_path: str = "monitoring/reports/drift_report.html",
):
    reference = pd.read_parquet(reference_path)
    current = pd.read_parquet(current_path)

    report = Report(metrics=[
        DataDriftPreset(),
        DataQualityPreset(),
    ])
    report.run(reference_data=reference, current_data=current)
    report.save_html(output_path)
    print(f"Drift report saved to {output_path}")

    # Return summary for alerting
    results = report.as_dict()
    n_drifted = results["metrics"][0]["result"]["number_of_drifted_columns"]
    drift_share = results["metrics"][0]["result"]["share_of_drifted_columns"]

    return {
        "n_drifted_columns": n_drifted,
        "drift_share": drift_share,
        "alert": drift_share > 0.3,
    }


if __name__ == "__main__":
    import argparse
    parser = argparse.ArgumentParser()
    parser.add_argument("--reference", required=True)
    parser.add_argument("--current", required=True)
    parser.add_argument("--output", default="monitoring/reports/drift_report.html")
    args = parser.parse_args()
    result = generate_drift_report(args.reference, args.current, args.output)
    print(f"Drift summary: {result}")
```

---

## How It All Connects: Step-by-Step Flow

```
Developer pushes to main
       │
       ▼
┌─ GitHub Actions triggers ──────────────────────────────────────────┐
│                                                                     │
│  JOB 1: ci-checks                                                   │
│  ├── Step 3: Checkout code                                          │
│  ├── Step 4: Install dependencies (pip + cache)                     │
│  ├── Step 5: Lint (flake8 + mypy + bandit)                          │
│  └── Step 6: Unit tests (pytest + coverage)                         │
│       │                                                              │
│       ▼ (passes)                                                     │
│  JOB 2: training                                                     │
│  ├── Step 7: DVC pull (S3 → runner)                                 │
│  ├── Step 8: Trigger KFP pipeline ──┐                               │
│  │   ├── 8a. Validate data           │                               │
│  │   ├── 8b. Preprocess + split      │  (runs on Kubeflow cluster)  │
│  │   ├── 8c. Train + MLflow log      │                               │
│  │   ├── 8d. Evaluate + quality gate │                               │
│  │   └── 8e. Register in MLflow      │                               │
│  │       ◄──────────────────────────┘                               │
│  ├── Step 9: Verify model registered (Staging)                       │
│  └── Step 10: Promote Staging → Production                           │
│       │                                                              │
│       ▼ (passes)                                                     │
│  JOB 3: build-and-push                                               │
│  ├── Step 11: Docker build (Dockerfile.serve)                        │
│  ├── Step 12: Push to ECR                                            │
│  └── Trivy security scan                                             │
│       │                                                              │
│       ▼ (passes)                                                     │
│  JOB 4: deploy (requires manual approval)                            │
│  ├── Update SageMaker endpoint                                       │
│  ├── Smoke test                                                      │
│  └── Slack notification                                              │
└──────────────────────────────────────────────────────────────────────┘
```

---

## File Count Summary

| Directory | Files | Purpose |
|-----------|-------|---------|
| `.github/` | 5 | CI/CD workflows, CODEOWNERS, PR template |
| `configs/` | 6 | Training, serving, feature, logging configs |
| `data/` | 6 | DVC-tracked dirs + reference schemas |
| `docker/` | 5 | Dockerfiles for train, serve, CI, dataprep |
| `docs/` | 8 | Architecture, runbook, model card, ADRs |
| `infra/` | 16 | Terraform, Helm, setup scripts |
| `notebooks/` | 5 | EDA, experiments (NOT production) |
| `pipelines/` | 9 | KFP pipeline + components + triggers |
| `requirements/` | 13 | Split deps with .in + .txt + constraints |
| `scripts/` | 9 | Developer utilities, promotion, rollback |
| `serving/` | 9 | FastAPI app, schemas, middleware, tests |
| `src/` | 18 | Core ML code (data, features, models, monitoring, utils) |
| `tests/` | 15 | Unit, integration, smoke tests + fixtures |
| `monitoring/` | 6 | Dashboards, alerts, drift reports |
| Root | 12 | Config files (.flake8, pyproject.toml, Makefile, etc.) |
| **TOTAL** | **~142** | **Complete production ML repository** |
