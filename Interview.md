INTERVIEW


Interview Schedule:
—————————
1. What is your current CTC (Cost to Company)? 
 20,00,000/- Lakhs

2. Current Monthly Take Home?
1,26,000/- 

3.What is your expected CTC? 
32,00,000/- Lakhs
4. Expected Monthly Take Home?
2,08,767/-(after all deductions tax-56000/-, EPF-1800, ProfTAX-200)

5. Are you open to hybrid or on-site work, or do you prefer fully remote?
Hyderabad, remote

6. Why are you looking for a job change?
It’s been 7 years working in the same org, I feel it’s the right time to take new challenges, explore new technologies, and grow further in my career.

7. What is your total and relevant experience in MLOPS?
My Total IT  Experience and  relevant experience both are same 6.7 years
in that 7yrs(2011-2018) i'm into non-it(as a data driven marketing specialist in Geny Medium) and relevant experience on mlops is 6.7 years till date.

9. What is your current project?
Daimler (Mercedes-Benz)

10. Do you have any offers in hand?
Currently, I don’t have an offer, but I  have a few opportunities in the pipeline. I’m evaluating the right fit where I can contribute long-term, and your role looks very aligned.”

11. Payroll Details?
Larc Software Private Ltd (Bangalore)——1998
Address: Survey Nos. 10/2A 10/2B 3rd floor, Sunriver building, off Intermediate ring road EGL, Challaghatta, Domlur, Bengaluru, Karnataka 560071.
Phone Number: HR Manager (KEERTHI M) +91 84988 50932
Reporting Manager (SANJAY P) +91 84988 50932
Total Employs LARC SOFTWARE 320 above
www.larach.biz
machireddyvinodh@larach.biz
Mail ID Password:  Larc@1234$
EMP ID: LARC-6402

Ai vs ml
Docker image size reduce
Handling huge data - pyspark
Model tracking
Transformers
How you will confirm a good model in registry.
Feature store
ECS VS EKS


Screening Round:
————————
1. Tell me about yourself.

2. What is MLOPS?
Machine Learning Operations
Where we train the models with the data
Mlops is extension to DevOps in DevOps we  focuses on automating the lifecycle of traditional software/Application, were as in MLOps it automates the lifecycle of a machine learning model.

3. What is the main goal of MLOps?
The main goal of MLOps is to make machine learning models easy to build, deploy, Monitor, retrain and  keep running smoothly in production, with reliability, scalability, and automation."

4. Difference between DevOps and MLOps?
DevOps focuses on software applications. MLOps extends this to machine learning,

5. Can you explain the ML lifecycle?
Business Understanding → Data Acquisition → Data Versioning (raw) → Exploratory Data Analysis (EDA) → Data Cleaning & Preprocessing → Feature Engineering & Selection → Data Versioning (processed) → Model Training & Experiment Tracking → Model Evaluation & Tuning → Model Registry & Packaging → Model Deployment → Model Monitoring → Feedback & Retraining → Rollback & Retirement

6. What tools and technologies have you worked on?
MLflow, Kubeflow, Airflow, DVC, Python, Amazon Sagemaker, Azure ML, GCP Vertex AI, Git, gitHub Actions, Jenkins, Ansible, Docker, Kubernetes, Terraform, Prometheus, Grafana.

7. How do you handle versioning in ML projects?
I handle versioning at three levels — data, code, and models. 
- For data, I use tools like DVC to version raw and processed datasets so that every experiment is reproducible. 
- For code, I rely on Git/GitHub. 
- For models, I use MLflow or a model registry to track versions along with metadata, hyperparameters, and performance metrics. Together, this ensures traceability, reproducibility, and easy rollback if needed.”

8. How do you monitor ML models in production?
I monitor ML models in production by tracking model performance, data drift, and system metrics. By using tech stack prometheus, grafana and cloud ml monitoring.

9. What challenges do you face in deploying ML models?
Answer:
Common challenges are handling large data, reproducibility, dependency issues, scaling the model, monitoring for drift, and ensuring security and compliance.

10. If a model works offline but fails in production, what steps will you take?
Answer:
Check data differences between training and production, review feature preprocessing, validate environment dependencies, and analyse monitoring logs to find mismatches.

11. How do you automate model retraining?
Answer:
I use pipelines (Kubeflow, Airflow, or Jenkins) that trigger retraining when new data arrives, evaluate the new model, and deploy only if it performs better than the current one.

12. What are your strengths and weaknesses as a MLOPS engineer?
ANS: "My biggest strength as an MLOps engineer is building reliable and scalable ML pipelines that ensure smooth deployment, monitoring, and collaboration between data science and operations."
"Sometimes I spend too much time helping other teams, which slows down my own work. But I am improving by setting clear priorities and managing my time better."

13. Where do you see yourself in five years?
ANS: In five years, I see myself growing into a lead or architect role in MLOps, where I can design large-scale ML systems, mentor junior engineers, and make strategic decisions that help the company get real value from AI."
	Follow-up-Question: 
	   - how cognizant support long term career development for employees?

14. What do you know about our company? | Why do you want to join our company?

15. What is data drift vs concept drift?
Data drift: input data distribution changes over time.
Concept drift: the relationship between input and output changes.

17. What security practices do you follow in MLOps?
Scanning images with Trivy, using secrets management, access control, and encrypting data in transit and at rest.

18. Have you worked on CI/CD for ML pipelines? How did you set it up?
Yes, by using GitHub Actions/Jenkins for building, testing, containerizing, and deploying models automatically.

19. How do you collaborate with data scientists?
I help productionize their models by containerizing, automating training pipelines, and setting monitoring so they can focus on improving accuracy.

20. Tell me about a challenging ML project you worked on. How did you solve it?

21. How do you keep yourself updated with new ML/MLOps tools?
Reading blogs, following cloud providers’ updates, and practicing in hands-on projects.

22. How do you prioritize tasks when multiple teams need your support?
By aligning with business priorities, setting clear timelines, and balancing urgent vs important tasks.

23. Have you worked in agile/scrum environments?
Yes, I participate in sprint planning, daily standups, and retrospectives for delivering ML pipelines incrementally.

24. Why should we hire you for this MLOps role?
Because I have strong DevOps + ML background, hands-on with end-to-end pipelines, and can help scale your AI projects faster and more reliably.

25. Do you have any questions for us? (Always prepare 1–2 smart ones)
What MLOps tools are currently in use here?
How does the company measure success of ML projects in production?

26. DS vs MLOps responsibilities
Think of it like DS starts at problem → finishes at best model selection, then MLOps takes over for packaging → deployment → monitoring → automation, with some overlap in the middle.


27. Monitoring vs Observability
Definition: Collecting and tracking known metrics to check system/model health.
Focus:
System metrics: CPU, memory, latency, throughput.
ML metrics: accuracy, drift, precision/recall.
Example:
If model accuracy < 80%, send alert.
If API latency > 200ms, trigger alarm.

Observability
Definition: A deeper ability to understand why something went wrong, not just what went wrong.
Focus:
Logs, traces, metrics all combined.
Root cause analysis (debugging).
Simple Analogy
Monitoring = Thermometer → tells you the fever exists.
Observability = Doctor → investigates symptoms to find the cause.


Technical Rounds:
-----------------
- Walk me through the end-to-end ML lifecycle and where MLOps fits in.
- If a model’s accuracy suddenly drops in production, what’s your step-by-step approach to diagnose it?
- In your pipeline, where would you place data validation checks and how would you automate them?
- Tell me how you implemented Kubeflow Pipelines
- Can you create a GitHub Actions / Jenkins pipeline to train, evaluate, register, and deploy a model automatically?
- How do you deploy ML models on AWS SageMaker / Azure ML / GCP Vertex AI? Which is your preferred platform and why?
- How would you implement blue-green or canary deployment for ML models?
- How do you secure model endpoints in production? (Auth, API Gateway, rate limiting)
- How do you manage model secrets (DB passwords, API keys) in pipelines?
- What’s your approach to cost optimization for ML workloads in the cloud?
- Your model training job that usually takes 30 mins is now taking 2 hours. How would you troubleshoot?
- You deployed a model, but after 2 weeks, business KPIs dropped despite good accuracy. What could be wrong?
- The data team sends you a new dataset with extra columns. How will you integrate it without breaking the pipeline?
- How would you handle model rollback if the latest model causes bad predictions in production?
Give me a real example where you automated a repetitive ML process and saved significant time.
Special interview questions:
Explain how do you create and manage kubernetes cluster?
We use namespaces for logical isolation then we setup resource quota on namespace, so the teams are restricted to the compute power 
And we help the teams to enable resource requests and limits on each pod, so that each pod is restricted with compute and memory power





1. self
2. My Day-to-Day Activities as an MLOps Engineer
3. Tell me about your project
4. Describe some real-time challenges you have faced and how you resolved them.
5. Explain your ML pipeline in your current or previous projects
6. Mlops Best Pratices
7. cost-efficient


SELF:
-----
- My name is Vinodh Machireddy.
- I’m from Andhra Pradesh, Bharath.
- Currently, I’m working as a Senior MLOps Engineer at Larc Software Pvt Ltd.
I bring 6+ years of experience in automating end-to-end ML pipelines. focusing on scalable deployments, proactive monitoring, retraining strategies, and cloud-native infrastructure.
I’ve worked with MLflow, Kubeflow, and DVC for experiment tracking, Model Registry and versioning, used Docker and Kubernetes for containerisation and orchestration; set up monitoring Solutions with Prometheus and Grafana, and deployed ML solutions across multi-cloud platforms such as AWS, Azure, and GCP.
My goal is to simplify the Machine Learning lifecycle, ensure reproducibility, and apply MLOps practices to deliver scalable, reliable, and cost-efficient AI solutions in production.
That’s a quick introduction about me.
Thanks for the opportunity!


My Day-to-Day Activities as an MLOps Engineer:
————————————————————
1. Checking Pipelines and Models
Start the day by reviewing ML pipeline runs in Kubeflow/Airflow to ensure data ingestion, preprocessing, training, and deployment stages executed successfully.
I also check monitoring dashboards to see if any production model is showing performance drop or errors.

2. Versioning and Tracking
Pull latest code (Git), review/merge PRs.
I make sure all code, data, and models are versioned properly, so any experiment can be reproduced later.
I use tools like Git, MLflow, or DVC to track experiments and model versions.

3. Working with Data Scientists
I help data scientists move their models from notebooks to production by building pipelines, writing configs, and adding tests.
Together we review metrics to decide if a model is good enough to promote to production.

4. Deploying Models
I package models into Docker containers and deploy them on Kubernetes clusters.
I use safe rollout strategies like canary or A/B testing before making them live for all users.

5. Monitoring and Alerts
I monitor both system metrics (CPU, memory, latency) and model metrics (accuracy, drift, bias).
If performance drops, I investigate the issue and decide whether to retrain the model.

6. Automating Retraining
I set up automated retraining pipelines that run when new data arrives or when a model starts drifting.
Only better models get promoted to production automatically.

8. Team Work
I attend daily stand-ups with data scientists, QA, and cloud teams.
We discuss ongoing issues, improvements, and after any failure, we do a post-mortem to learn from it.


Tell me about your project:
---------------------------
1.Overall Project
2.business use case/purpose
3.Architecture
4.tech stack used
5.and Roles & Responsibility.

1. Overall Project
I am working on Daimler Project under automotive domain, their focusing on Traffic Sign Recognition (TSR). The project is part of Advanced Driver Assistance Systems (ADAS) and aims to detect and classify traffic signs from camera input in real time.
2. Business Use Case / Purpose
The main purpose is to enhance driver safety and compliance.
By recognising traffic signs such as speed limits, stop signs, or pedestrian crossings, the vehicle can alert the driver or adjust driving behaviour.
3. Architecture
The architecture has two main layers:
Data & Model Layer
Camera sensors capture road images.
Images are preprocessed and passed through a deep learning classification model (CNN-based).
The output is the predicted traffic sign category.
MLOps Pipeline Layer
Data Pipeline: Collects and versions training/validation images.
Training Pipeline: Handles preprocessing, augmentation, training, evaluation.
Deployment Pipeline: Packages the trained model into a container and deploys as a microservice.
Monitoring & Retraining: Tracks accuracy, latency, drift, and triggers retraining if needed.
4. Tech Stack Used: MLflow, DVC, Kubeflow, Docker, Kubernetes, CI/CD, Prometheus, Grafana, Trivy

5. Roles & Responsibilities (as Senior MLOps Engineer)
Designed end-to-end ML pipelines for data preprocessing, training, deployment, and monitoring.
Implemented model versioning and experiment tracking with MLflow and DVC.
Containerised models with Docker and deployed on Kubernetes clusters.
Set up real-time monitoring dashboards and alerts for model accuracy, latency, and drift.
Automated retraining workflows when new data or drift was detected.
Collaborated with data scientists to productionize research models into production pipelines.
Ensured reproducibility, security, and reliability of ML workflows across environments.



CHALLENGES
———————

Model Degraded due to Hidden Feature Drift in Production

We had a machine learning model running in production. For the first few months, it worked well and gave good accuracy. After some time, the predictions started becoming wrong more often.
At first, it was not obvious because normal monitoring (like uptime, latency, schema checks) showed everything healthy. But business teams noticed that the model was missing important cases (for example, not catching fraud or misreading traffic signs).
The tricky part was — data pipelines looked fine (no missing columns, no errors). But actually, the input data pattern had changed silently. That hidden change in the features made the model behave poorly.
What made it tricky
No clear alert initially – dashboards showed stable latency and throughput, only business KPIs hinted at an issue.
Data pipeline looked “healthy” – schema matched, no missing columns. But hidden drift existed.

Resolution:
When accuracy started dropping, first I stabilised the system by adding guardrails like request validation, circuit-breaker, and fallback to the last stable model. These safety checks ensured users were not badly affected while I investigated and fixed the root cause.

Deep Monitoring Analysis
Set up feature-level drift metrics (PSI, KS test) using Evidently AI.
Debugging the Root Cause

Retraining the Model
Pulled recent 2 months of production data with new encoding.

Safe Deployment
Used canary rollout on 10% traffic.

Long-Term Fix
Set alerting on PSI > 0.3 for top 10 features.


1. Model Drift in Production
Challenge:
We had a machine learning model deployed in production. Initially, the accuracy was good, but after a few months the predictions started degrading.

Resolution:
When accuracy started dropping, first I stabilised the system by adding guardrails like request validation, circuit-breaker, and fallback to the last stable model. These safety checks ensured users were not badly affected while I investigated and fixed the root cause.

Root cause: Data distribution in production had changed — the incoming features looked different compared to the data the model was originally trained.(data drift)

I set up Prometheus/Grafana to track live data and system health, and MLflow to log model metrics and versions.

I configured alerts so whenever drift goes above a threshold, it notifies us and triggers the retraining pipeline.

The pipeline pulls fresh data → validates it → preprocesses → retrains → evaluates. If the new model is better, it’s registered in MLflow Model Registry.

I used Kubernetes canary rollout: the new model first gets a small % of traffic. If it performs well, it’s promoted to full production; if not, it auto-rolls back.

Now the system is self-healing: whenever drift happens, monitoring + pipeline + canary ensure models are automatically retrained and deployed safely.
