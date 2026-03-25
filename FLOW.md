# FLOW

1. Project Initiation & Tech Stack ✅ Done  
2. What the Data Scientists hand over — brief context, just enough to show you understand the ML side
3. Training Pipeline (KFP) — how you productionized their notebook into a repeatable pipeline (this is YOUR territory)
4. Experiment Tracking & Model Registry (MLflow) — how models are versioned and promoted (YOUR territory)
5. CI/CD Pipeline (GitHub Actions + ArgoCD + DVC) — how code and model changes flow to production (YOUR territory)
6. Model Serving (KServe) — real-time inference architecture (YOUR territory)
7. Monitoring & Observability (Prometheus + Grafana) — drift detection, alerting (YOUR territory)
8. Retraining Strategy — when and how retraining is triggered (YOUR territory)


as a Senior MLOps Engineer. my job is not to build the model from scratch — we have data scientists for that. My responsibility starts from the point where we need to productionize that model. That means — how do we take this model from a Jupyter notebook, build a proper training pipeline around it, package it, deploy it to production, serve it in real time, monitor it, and retrain it.   
