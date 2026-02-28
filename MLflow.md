

## What is Experiment Tracking?
When Data Scientist trains a model, they try many things(Different algorithms, hyperparameters, feature sets, datasets, evaluation metrics), we record every ML experiment details properly.  If we don’t track properly:  
- We don’t know which model gave best accuracy  
- We can’t reproduce results  
- We can’t compare experiments

### So experiment tracking stores:
- Parameters (learning rate, epochs, batch size)
- Metrics (accuracy, precision, recall, F1)
- Model artifacts
- Dataset version
- Code version
- Timestamp
- Who ran the experiment
## What is MLflow?
MLflow is an open-source platform used to manage the ML lifecycle. it mainly has 4 components:  
- Tracking → Log parameters, metrics, artifacts  
- Projects → Package ML code  
- Models → Standard model format  
- Model Registry → Manage model versions and stages  

## Who is Responsible? DS or MLOps?
✅ Data Scientist Responsibility:
- Logging parameters & metrics
- Running experiments
- Comparing models
- Selecting best model
- Registering model (sometimes)  

✅ MLOps Engineer Responsibility:
- Setting up MLflow server
- Configuring backend store (DB)
- Configuring artifact store (S3)
- Access control & security
- Automating model promotion
- CI/CD integration
- Governance & compliance
- Production deployment

### Who handles experiment tracking?  
“In our project, Data Scientists log experiments using MLflow inside Kubeflow Pipelines. As an MLOps Engineer, I set up and managed the MLflow tracking server, configured S3 as artifact store, integrated it with CI/CD, and implemented model promotion workflow from staging to production.”  

Components:
MLflow → Runs inside EKS (Deployment)  
RDS PostgreSQL → Backend store  
S3 → Artifact store  
IAM Role → Secure S3 access  
ALB / Ingress → Access MLflow UI  

MLflow deployed as Deployment in EKS 
🔹 Step 1: Create RDS PostgreSQL  
🔹 Step 2: Create S3 Bucket  
🔹 Step 3: Configure IAM Role for EKS  
🔹 Step 4: Create Docker Image for MLflow  
🔹 Step 5: Create Kubernetes Deployment YAML  
🔹 Step 6: Create Service  
🔹 Step 7: Expose via Ingress (ALB)  
🔹 Step 8: Integrate with Kubeflow  
🔹 Step 9: Manage via ArgoCD (GitOps)


