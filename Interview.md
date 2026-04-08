# INTERVIEW  

1. Schedule
2. Screening
3. Technical Rounds
4. Manager Round
5. HR Discussion
   
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

- I am working on Daimler Project which comes under automotive domain/Industry, their i am dealing with Battery Fault Classification. The project aims to classify battery cell faults in real time using sensor data from the Battery Management System (BMS). This helps detect issues like overheating, overcharging, internal short circuits, and cell degradation early — before they become safety-critical.  
-  The main purpose is to improve EV battery safety, and extend battery lifespan.  

## Interview Schedule:

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


## Screening Round:
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


## Technical Rounds:
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

6. if we give a opportunity to handle a team how you will handle
1. Listen and understand:
 - Having 1-on-1s with every team member — understanding their strengths, career goals, frustrations, and what's blocking them
2. Set clear goals and ownership
3. Build a culture of trust, not micromanagement
 
### CHALLENGES
**Silent Feature Drift in Production — Battery Fault Classifier**
**Situation:** 
when we working on ev battery fault classifier model which is running in production for about three months, performing well. Accuracy and latency dashboards on Grafana looked stable. But then the battery domain engineers started reporting something strange — the number of degraded classifications had quietly increased by 30% over the past few weeks. No alerts had fired. No errors in logs.  
Why it was dangerous: This is what I call a silent failure — the model wasn't crashing, it was confidently making wrong predictions. the model is missing real faults. 

**Root cause analysis:**
1. Deep Monitoring Analysis
2. Checked model and pipeline — No code changes, no redeployment, same XGBoost model version in MLflow. Pipeline was clean. So the problem wasn't in our system.
3. Compared training vs live data distributions — This is where I found it. The battery supplier had made a small manufacturing change that shifted the internal resistance and temperature-under-load distributions.

**Actions I took:**
1.  Immediate fix — retrained with fresh data
2.  we added a drift detection layer : This was the bigger lesson. I implemented a proper feature drift monitoring system:
3.  Added prediction distribution monitoring
4.  Established a retraining policy
5.  Set up a simple communication channel with the battery supplier and procurement team. if Any material or process changes now get flagged to the ML team proactively.


## HR Round:
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


## Learnings from Interview  
Infosys:  
	> if we give a team how you will handle it?  
	> if team members not listining you, what would you do?  
	> How do you handle dead lines?  
	> when client is scolding/harsh/rude with you , how would you handle?  
	> day to day activities  
	> challenges  
	> once you get a new project how you handle or start it like distributing modules, sharing workload.    

## Team Handling	
**You don't 've prior experience how do you handle a team?**	
I haven’t had formal reporting management yet. But in my current role, Even though I've been an IC for 7+ years, I've naturally taken on informal leadership throughout my career — mentoring junior engineers, technical ownership, leading architecture discussions, and coordinating across teams with data scientists, which already requires many of the same skills.  

I believe people management is an extension of ownership and communication, and I’m confident I If given a formal team, i have a plan with the structured approach where.

### Step 1.First, I would understand the project goal, timelines, dependencies, and expectations clearly so I can give the team the right direction.  

### Step 2. First, I'd have one-on-one conversations with every team member to understand:
```  
→ What are their strengths and interests?
→ What are their skill gaps?
→ What motivates them?
→ What frustrates them?
→ where they blocking  
```
> "For example, one person might be strong in Kubernetes but weak in ML pipelines. Another might be great at monitoring but wants to learn deployment. Understanding this helps me assign the right work to the right person."

### Step 3 — Set Clear Expectations(Detailed)
The idea is simple: if people know exactly what's expected, most problems never happen. Confusion causes 80% of team issues — not laziness, not bad attitude.  
> I believe most team problems come from unclear expectations, not bad people. If everyone knows what's expected, 80% of issues don't happen.

**A. Roles — "Who owns what?**
Without clear roles, you get two problems:  
```
Problem 1: DUPLICATION — Two people work on the same thing
"I thought you were setting up Grafana dashboards"
"No, I thought YOU were doing it"

Problem 2: GAPS — Nobody works on something
"Who was supposed to configure the drift alerts?"
"I thought someone else was handling it"
```
How you fix it:  "In the first team meeting, I'd clearly define ownership areas and write them down:"  
```
Example for a 5-person MLOps team:

Ravi    → Training pipelines (Kubeflow, DVC)
             "You own everything from data ingestion to model registration"

Priya   → CI/CD and GitOps (GitHub Actions, ArgoCD)
             "You own every PR workflow, build pipeline, and deployment config"
```
> "Each person knows exactly what they're responsible for. No confusion, no overlap, no gaps."  
> **Important:** Ownership doesn't mean isolation. If Ravi needs help with a Kubernetes issue in his pipeline, Karthik helps. But Ravi is still the accountable owner for pipelines.

**B. Processes — "How do we work together?"**
"I'd define clear working agreements that everyone follows:"  
```
Code:
→ Every change goes through a PR — no direct push to main
→ Minimum 1 reviewer before merge
→ CI must pass before merge (GitHub Actions)
→ Write meaningful commit messages

Communication:
→ Blockers → Raise in daily standup, don't wait
→ Quick questions → Slack (team channel, not DMs)
→ Design decisions → Write a short doc, discuss in team meeting
→ Urgent production issues → Phone call + Slack alert channel

On-call:
→ Weekly rotation — each person takes 1 week
→ Runbooks for every known alert
→ Escalation path: on-call → team lead → manager
```
> "These aren't heavy rules — they're just agreements that prevent daily confusion. The team helps define them so they feel ownership, not enforcement."

**C. Standards — "What does good look like?"**
Without quality standards, you get inconsistency:  "I'd define minimum quality standards for our MLOps work:"
```
Code quality:
→ Every function must have unit tests
→ Code must pass linting (flake8/black for Python)
→ No hardcoded values — use config files or environment variables
→ Follow team naming conventions

Pipeline quality:
→ Every Kubeflow component must be independently testable
→ Every pipeline must have data validation step
→ Model evaluation must check class-level metrics, not just overall accuracy
→ All pipeline parameters must be configurable, not hardcoded

Deployment quality:
→ Every deployment must go through canary (no direct 100% rollout)
→ KServe manifests must be stored in Git (GitOps)
→ Rollback must be tested before going live
```
> "When everyone knows what 'good' looks like, you don't need to micromanage. People self-correct because the standard is clear."
  
### Step 4 — Create Ownership (Deep Level)
I wouldn't micromanage. Instead, I'd divide the MLOps platform into clear ownership areas:"  
This answers "you OWN this end-to-end — decisions, quality, improvements, on-call, everything."  
```
"Ravi, you OWN training pipelines. That means:
  → You decide the pipeline architecture
  → You decide when to refactor
  → You review all PRs in this area
  → You're the go-to person when it breaks
  → You present pipeline health in team meetings
  → You propose improvements proactively
  → You don't wait for me to tell you what to fix"
```
> "Ownership gives people pride and accountability. They're not just executing tasks — they're owning a piece of the platform."

### Step 5 — Regular Cadence
"I'd establish a regular cadence — daily standups to surface blockers immediately, weekly one-on-ones with each member to build trust and address personal concerns, biweekly retrospectives for the team to improve its own process, and monthly architecture reviews to address tech debt and plan improvements.   
```  
Meeting              Frequency    Duration    Purpose

Daily standup        Every day    15 min      Surface blockers
Weekly 1-on-1        Every week   30 min      Build trust, grow people
Biweekly retro       Every 2 wks  1 hour      Team improves its process
Monthly arch review  Every month  1-2 hours   Big picture and tech debt
```
### Step 6 — Shield the Team
"My job as a lead is to protect the team from distractions — unnecessary meetings, unclear requirements, last-minute scope changes. I handle the stakeholder communication so the team can focus on building."  

### Step 7 — Grow the Team
"Finally, I'd invest in their growth:"  
```  
→ Pair them on tasks outside their comfort zone
→ Encourage internal tech talks (each person presents their area)
→ Support certifications (AWS, Kubernetes, MLOps)
→ Give credit publicly, give feedback privately
```
**Put It All Together — Your Crisp Answer**

"If given a team, my approach would be: first understand each person — their strengths, gaps, and goals through one-on-ones. Then set clear expectations — who owns what, how we work, and what quality looks like. I'd divide the platform into clear ownership areas so each person owns a piece end-to-end rather than just executing tasks. I'd establish a regular cadence — daily standups, weekly one-on-ones, biweekly retros. My job as a lead would be to shield the team from distractions, unblock them when they're stuck, and grow them by giving stretch assignments and learning opportunities. I've been doing this informally as a senior IC for years — mentoring juniors, leading design discussions, coordinating with data science teams. A formal role would just make that official."  

### if team members not listining you, what would you do?  
If someone isn't listening, there's always a reason — They don't understand, disagree technically, Attitude problem, they might feel unheard, or they might be dealing with personal issues. So I'd start with a private one-on-one conversation — ask questions, listen, and understand their perspective. 

If Reason 1: They Don't Understand   -  i try to explain again in detail.  
If Reason 2: They Disagree Technically  - Your response: "Arun, I respect your opinion. Let's do this properly —  you present the case for blue-green, I'll present the case for canary, and we'll evaluate together based on our requirements.   
If Reason 3: They Feel Unheard  - "Sneha, I realize I've been pushing back on your monitoring proposals because of deadlines,  
If Reason 4: Personal Issues  - I just want to know , if you're okay and how I can support you."  
If Reason 5: Genuine Attitude Problem (Rare) - Direct conversation with clear expectations.   
If no improvement after 2 weeks:  
	- Involve your manager / HR
	- Put on a performance improvement plan (PIP) 
If still no improvement:  
	- This person may need to leave the team or role
