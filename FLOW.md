# FLOW

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
```  
Step 1: Data Ingestion                         → one container
Step 2: Data Validation                        → one container  
Step 3: Preprocessing & Feature Engineering    → one container
Step 4: Model Training                         → one container
Step 5: Model Evaluation                       → one container
Step 6: Model Registration                     → one container
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
```  
1. git clone https://github.com/daimler/bms-ml.git     ← get the repo
2. git checkout abc123  (if specific version)          ← go to that point
3. dvc pull                                            ← reads .dvc pointer file & downloads matching data from S3   
4. Data is now available inside the container          ← ready for next step
   > Step 2: Data Validation (receives data from Step 1) --> Step 3: Preprocessing --> ..... and so on                       
   > This step runs internally when KFP pipeline triggers. it asks us to pass commit id during run time. 
```





