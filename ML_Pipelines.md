# First, Understand the Big Picture. The entire ML pipeline has 4 major phases:  
```
Phase 1: CI (Continuous Integration)    → Code push → Test → Build
Phase 2: CT (Continuous Training)       → Train → Evaluate → Register
Phase 3: CD (Continuous Deployment)     → Deploy model to production
Phase 4: CM (Continuous Monitoring)     → Monitor → Detect drift → Retrain
```
## The Complete Flow
```
MLE pushes code to GitHub
        │
        ↓
┌─────────────────────────────────────────────────┐
│           PHASE 1: CI (GitHub Actions)           │
│                                                  │
│  Step 1:  Git push triggers GitHub Actions       │
│  Step 2:  Linting (flake8) — code quality check  │
│  Step 3:  Unit tests (pytest) — logic check      │
│  Step 4:  DVC pull — get versioned data from S3  │
│  Step 5:  Docker build — containerise components │
│  Step 6:  Trivy scan — security vulnerability    │
│  Step 7:  Push images to ECR                     │
│  Step 8:  Compile pipeline YAML                  │
└─────────────────────┬───────────────────────────┘
                      │
                      ↓
┌─────────────────────────────────────────────────┐
│       PHASE 2: CT (Kubeflow Pipelines)           │
│                                                  │
│  Step 9:  Ingest data from S3                    │
│  Step 10: Validate data quality                  │
│  Step 11: Feature engineering                    │
│  Step 12: Train model                            │
│  Step 13: Evaluate model                         │
│  Step 14: Quality gate (accuracy >= 90%)         │
│  Step 15: Register model in MLflow               │
│  Step 16: Human approval (if needed)             │
└─────────────────────┬───────────────────────────┘
                      │
                      ↓
┌─────────────────────────────────────────────────┐
│        PHASE 3: CD (ArgoCD + KServe)             │
│                                                  │
│  Step 17: Update model URI in inference.yaml     │
│  Step 18: Push to ml-infra-repo (GitOps)         │
│  Step 19: ArgoCD detects change, syncs           │
│  Step 20: KServe deploys new InferenceService    │
│  Step 21: Canary/Rolling update (zero downtime)  │
└─────────────────────┬───────────────────────────┘
                      │
                      ↓
┌─────────────────────────────────────────────────┐
│      PHASE 4: CM (Prometheus + Grafana)          │
│                                                  │
│  Step 22: Prometheus scrapes ML metrics          │
│  Step 23: Grafana dashboard shows health         │
│  Step 24: Drift CronJob checks data drift        │
│  Step 25: Alert fires if drift/accuracy drop     │
│  Step 26: Alertmanager triggers retraining       │
│           → Goes back to Phase 2 (Step 9)        │
└─────────────────────────────────────────────────┘
```
