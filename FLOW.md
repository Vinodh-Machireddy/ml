# FLOW

1. Project Initiation & Tech Stack ✅ Done  
2. What the Data Scientists hand over — brief context, just enough to show you understand the ML side
3. Training Pipeline (KFP) — how you productionized their notebook into a repeatable pipeline (this is YOUR territory)
4. Experiment Tracking & Model Registry (MLflow) — how models are versioned and promoted (YOUR territory)
5. CI/CD Pipeline (GitHub Actions + ArgoCD + DVC) — how code and model changes flow to production (YOUR territory)
6. Model Serving (KServe) — real-time inference architecture (YOUR territory)
7. Monitoring & Observability (Prometheus + Grafana) — drift detection, alerting (YOUR territory)
8. Retraining Strategy — when and how retraining is triggered (YOUR territory)

> **Team Flow:** Platform Engineering → Data Engineering → Data Science → MLOps Engineering

**as a Senior MLOps Engineer.** my job is not to build the model from scratch — we have data scientists for that. My responsibility starts from the point where we need to productionize that model. That means — how do we take this model from a Jupyter notebook, build a proper training pipeline around it, package it, deploy it to production, serve it in real time, monitor it, and retrain it.   

**Data ingestion from Vehical to s3**
The raw telemetry data lands in S3 via the ingestion pipeline from platform engineering teams. Now the data science team pull this raw data from s3, did their EDA, cleaned it, and future engineering steps and pushes the processed training dataset back into S3 as **Parquet files. i.e train.parquet**.
> so, s3 contains raw data, processed data, model artifacts, etc...    

**I received three essentially things from Data Scientists:**  
- A Jupyter notebook — with the full training code, preprocessing logic, feature engineering, model training, and evaluation.  
- A requirements file — the Python dependencies.  
- The training dataset in S3 — versioned with DVC.

My work starts from S3 onwards. My KFP pipeline picks up data from this processed S3 path as its input. we take this notebook and turn it into a production-grade, automated, repeatable, auditable training pipeline. 

"So the first major thing I did was, I decomposed the monolithic Jupyter notebook into a Kubeflow Pipeline — KFP."  
```
Step 1: Data Ingestion                         → one container
Step 2: Data Validation                        → one container  
Step 3: Preprocessing & Feature Engineering    → one container
Step 4: Model Training                         → one container
Step 5: Model Evaluation                       → one container
Step 6: Model Registration                     → one container
```


"Now why KFP specifically? Because our infrastructure is on Kubernetes (EKS on AWS), and KFP runs natively on Kubernetes. Each step of the pipeline runs as a separate container inside a Kubernetes pod. That gives us isolation, scalability, and reproducibility out of the box."


