# INTERVIEW  

1. self Intro  
2. My Day-to-Day Activities as an MLOps Engineer  
3. Tell me about your project  
4. Describe some real-time challenges you have faced and how you resolved them.  
5. Explain your ML pipeline in your current or previous projects  
6. Mlops Best Pratices  
7. cost-efficient  


## SELF:  
 
- My name is Vinodh Machireddy.  
- I'm originally from Andhra Pradesh, Bharath.  
- Currently, I’m working as a Senior MLOps Engineer at Larc Software Pvt Ltd.  
I bring over 7+ years of overall experience in designing, automating, and managing end-to-end machine learning lifecycle systems.  
- I’ve worked with ML pipeline orchestration using Kubeflow Pipelines, handled experiment tracking and model registry with   MLflow, and managed production-grade deployment and serving using ArgoCD and KServe. I have also set up monitoring and alerting solutions using Prometheus and Grafana, and deployed ML solutions on AWS.  
- At the end of the day, I just want to make sure ML models work smoothly in production
- That’s a quick introduction about me.  
- Thanks for the opportunity!  


## My Day-to-Day Activities as an MLOps Engineer: 

1. Morning — Monitoring & Health Checks  

My day starts with checking monitoring dashboards to see any overnight alerts is showing performance drop. 
and Quick check on Kubernetes cluster health — are all model serving pods running fine?  

2. CI, CD, CM steps  

3. Working with Data Scientists  
I help data scientists move their models from notebooks to production by building pipelines.
Together we review metrics to decide if a model is good enough to promote to production.

4. Team Work
I attend daily stand-ups with data scientists, QA, and cloud teams.
We discuss ongoing issues, improvements, and after any failure, we do a post-mortem to learn from it.  


## Tell me about your project:

1.Overall Project  
2.business use case/purpose  
3.Architecture  
4.tech stack used  
5.and Roles & Responsibility.  


- I am working on Daimler Project which comes under automotive domain/Industry, their i am dealing with Battery Fault Classification. The project aims to classify battery cell faults in real time using sensor data from the Battery Management System (BMS). This helps detect issues like overheating, overcharging, internal short circuits, and cell degradation early — before they become safety-critical.  
-  The main purpose is to improve EV battery safety, and extend battery lifespan.  


## HR Round Discussion:
**1. What are your strengths? and weaknesses?**
My strength and weakness are actually two sides of the same coin. 
- When I encounter something new, I take a little more time, because I don't like surface-level understanding. That initial slowness is my weakness, once I grasp a concept fully, I take **complete end-to-end ownership** of it.
- Another strong suit of mine is an automation-first mindset. whenever I start a new task, my first instinct is — **Can this be automated?** 

**Are you comfortable working from office / hybrid model?**
Ans: Yes, I’m open to hybrid or work-from-office based on project and business requirements.

**Are you okay with relocation?**
Ans: I’m not okay with relocation. At the moment, I have certain personal commitments that make immediate relocation difficult. However, I’m open to discussing this based on project requirements and timelines, and I can plan accordingly if needed.

**What is your current CTC? and expectations**  
My current fixed CTC is ₹20 LPA 
My monthly in-hand is approximately ₹1.5 lakh

**What is your notice period?**  
“My official notice period is __60_ days.”  
If negotiable:  
“My official notice period is ___ days, but I can try to discuss early release depending on transition planning.”(work handover is completed, they may release me earlier.)

**How do you handle pressure or tight deadlines?**
I handle pressure by bringing structure to the situation — first by identifying critical priorities, then breaking work into manageable parts, and communicating clearly with stakeholders about dependencies and risks.

**Where do you see yourself in 5 years?**
Ans: based on my skill set and automation first troubleshooting mindset , definitely i will be in the architect or manager level. 

**are you currently interviewing elsewhere?**
I have a few conversations in progress, but Infosys is my priority given the role and scale.  

**Have you worked with clients directly?**
Yes, I’ve worked in collaborative environments where interaction with cross-functional stakeholders was important, including discussing deployment requirements, release coordination, issue resolution, and production support expectations.

**Why do you want to join Infosys?**
1. opportunity to develope model from scratch
2. Antropic claude co-work intergation with infosys
3. long-term growth.

**why should we hire you**
I’m not limited to one tool in mlops space . i can handle take end-to-end ownership.
Automation first mindset
Strong troubleshooting mindset


## Manager Round
1. Walk me through a project you've owned end-to-end.
Ans: explain daimler end to end.

2. Tell me about a conflict with a teammate or stakeholder. How did you resolve it?
Ans: we mostly collaberate with data scientis teams. 
-  While building the ML pipeline for our battery fault classification project, there was a disagreement between me and a data scientist on the team. They wanted to retrain and deploy models manually using notebooks, while I was pushing for a fully automated CI/CD pipeline using GitHub Actions and MLflow for experiment tracking and model registry.
- Instead of escalating or insisting,I created a shor demo and I scheduled a short demo.  I showed how the automated pipeline would actually save them time. no manual artifact uploads, automatic versioning in MLflow, and one-click promotion to staging on KServe. I also incorporated their feedback.

3. How do you handle model drift in production?
Ans: Already Know.

4. What makes a Senior MLOps Engineer different from a Mid-level one?
Ans:
- A Mid-level MLOps Engineer is usually strong in implementing pipelines, deployments, and automation tasks within a defined scope. A Senior MLOps Engineer, however, is expected to take end-to-end ownership of the ML platform    
- A senior doesn't just fix incidents — they run postmortems, identify systemic gaps, and push for long-term fixes rather than patches.
- decision-making
- mentor team members
> **ownership, architecture, production thinking, and business impact.**

5. Tell me about a time you mentored someone or helped a teammate grow.
Ans: entire team i mentored to understand new tool . previously we are using Fast API for modal serving now we switched to kserve on project req. i explained to team how kserve works. and intergrations with the cloud. etc



## CHALLENGES
**Silent Feature Drift in Production — Battery Fault Classifier**
Situation: 
when we working on ev battery fault classifier model which is running in production for about three months, performing well. Accuracy and latency dashboards on Grafana looked stable. But then the battery domain engineers started reporting something strange — the number of degraded classifications had quietly increased by 30% over the past few weeks. No alerts had fired. No errors in logs.  
Why it was dangerous: This is what I call a silent failure — the model wasn't crashing, it was confidently making wrong predictions. the model is missing real faults. 

**Root cause analysis:**
1. Checked model and pipeline — No code changes, no redeployment, same XGBoost model version in MLflow. Pipeline was clean. So the problem wasn't in our system.
2. Compared training vs live data distributions — This is where I found it. The battery supplier had made a small manufacturing change that shifted the internal resistance and temperature-under-load distributions.

**Actions I took:**
1.  Immediate fix — retrained with fresh data
2.  we added a drift detection layer : This was the bigger lesson. I implemented a proper feature drift monitoring system:
3.  Added prediction distribution monitoring
4.  Established a retraining policy
5.  Set up a simple communication channel with the battery supplier and procurement team. if Any material or process changes now get flagged to the ML team proactively.






**Model Degraded due to Hidden Feature Drift in Production**

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
It’s been 7 years working in the same org, I feel it’s the right time to take new challenges, explore new technologies. not only for compenstation mainly looking for change.

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

