<div align="center">
  
  # AWS
</div>

Interview Topics:  
- scalling   
- ingress   
- loadbalancer  
- Architecture explanation  
- Cost optimisation   
- Security best practices  
- monitoring  
- troubleshoot/debug  
  
```GitHub → GitHub Actions → Kubeflow Pipeline → MLflow Tracking → MLflow Registry → Docker → ArgoCD → KServe → Prometheus → Grafana```
## How deployment happens from local → production
Step 1: Develop locally  
Step 2: Push code to GitHub  
Step 3: GitHub Actions builds Docker image  
Step 4: Push image to registry  
Step 5: ArgoCD deploys to cloud Kubernetes  
Step 6: KServe exposes public endpoint  
Step 7: Customers access model  
<details>
   <summary><b>Click to expand: Production MLOps project structure</b></summary>
  <img width="387" height="436" alt="Screenshot 2026-02-21 at 4 14 39 PM" src="https://github.com/user-attachments/assets/81fd56f3-68ec-4d3d-be49-5a977a87bd07" />
  <img width="370" height="553" alt="Screenshot 2026-02-21 at 4 15 15 PM" src="https://github.com/user-attachments/assets/194c3596-7661-46ff-9a3b-d92b73ae2028" />  
</details>

## How customers access ML model in real production
```
Customer Application
      ↓
Internet
      ↓
Public URL (example: api.company.com)
      ↓
Load Balancer / Ingress
      ↓
KServe Model Endpoint
      ↓
Model Container
      ↓
Prediction Response
```
Example:  
Customer sends request:  ```POST https://api.company.com/v1/models/fraud-detection:predict```  
Request body:  
```
{
  "instances": [1000, 45, 0.67]
}
```
Model returns:  
```
{
  "predictions": [0.92]
}
```
### Where this model is hosted in production
Usually on cloud Kubernetes:  
AWS EKS  
Azure AKS  
Google GKE  


# EKS
1. Control Plane/Management Layer/Master Node   
2. Data Plane/Worker Nodes  
   
If we setup kubernetes using kops, kube Admins. we need to manage entire control plane(master node), it is lenghty process.  
- kube-API server  
- etcd(pronounced et-see-dee)  database  
- kube scheduler
- kube controller manager  
- cloud-controller-manager  
That's why we going with aws EKS which is aws managed service which manages complete control plane.   

## EKS Setup
### Install aws CLI:
Def:- The AWS Command Line Interface (AWS CLI) is a unified tool to manage your AWS services. With just one tool to download and configure, you can control multiple AWS services from the command line and automate them through scripts.  
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version (or) /usr/local/bin/aws --version
sudo apt remove awscli
```
Note:- https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html  
Note:- https://github.com/aws/aws-cli/releases/tag/2.9.19  
     
### Install Eksctl: 
- eksctl is used to create, update, and delete EKS (Elastic Kubernetes Service) clusters on AWS.  
- eksctl is a command-line tool used to quickly provision and manage AWS EKS clusters without manually configuring networking, IAM, and node groups.  
Install Eksctl: ```https://docs.aws.amazon.com/eks/latest/eksctl/installation.html```  
Note:- KOPS / Kube Admin for self-managed Kubernetes k8s setup.

### Install kubectl:
Def:- The Kubernetes command-line tool, kubectl, allows you to run commands against Kubernetes clusters. You can use kubectl to deploy applications, inspect and manage cluster resources, and view logs.  

   - You must use a kubectl version that is within one minor version difference of your cluster.  
   For example, a v1.26 client can communicate with v1.25, v1.26, and v1.27 control planes.  
   Using the latest compatible version of kubectl helps avoid unforeseen issues.  
```
Note:-  https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/  
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"    
sudo chmod 700 kubectl
sudo mv kubectl /usr/local/bin/kubectl
sudo kubectl version (or) kubectl version --short/--client
```
## Fargate
- if any specific req like particular Linux distribution, only windows, specific config then we go for EC2 Instances, otherwise use Fargate for worker nodes.

- EKS manages control plane  
- aws Fargate is serverless compute for containers which manages worker nodes.   
Note:- for worker nodes we can use EC2 Instances also.but, we need to take care of High Availability.
  
Normal EKS (Without Fargate): 
In normal You must:  
- Create EC2 instances  
- Manage node groups  
- Scale nodes  
- Patch OS  
- Handle capacity  
- Pods run on EC2 worker nodes.  

  With Fargate in EKS:  
  - You run containers  
  - Without managing EC2 servers  
  - AWS manages infrastructure  
  -  No node groups. No instance selection.  
### Fargate Profile:
- Node Group: Group of EC2 instances. These EC2 instances act as Kubernetes worker nodes.   
- There are two types: 1.Managed Node Group (AWS manages EC2 lifecycle). 2.Self-managed Node Group (you manage EC2)  
- If you use AWS Fargate: There are no EC2 instances, No node groups, AWS directly runs pods on serverless compute. Instead, you create Fargate profiles.
  
### Pod
Pod is the smallest deployable unit in Kubernetes Cluster. It runs Your application container With its own IP Inside a Node.   
Pod gets internal IP Address which changes on restarts. Pod is temporary, Not exposed outside cluster by default.   
Problem:  
If pod restarts, IP changes. So how to access it reliably?  Because of this, direct Pod communication is not reliable.  
Answer → Service   
### Service
A Kubernetes Service is an abstraction layer over a set of Pods that exposes them through a fixed IP address and DNS name, automatically routes incoming traffic to available Pods, and handles load balancing across them.   

#### Core Components of Service
1. Selector
Matches Pod labels. This means: All Pods with label app=my-nginx, Become part of this Service. If label does not match → Service will not connect to Pod.
2. port
   ```port: 80```  
   This is the Service port. Inside cluster, you access
3. targetPort
   ```targetPort: 8080```
   This is container port inside Pod. Internal access only,  Not exposed outside cluster
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: my-nginx
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
```
Container runs on port 8080  
We want Service to expose it on port 80  

#### Types of Services:
1. ClusterIP (Default)
Internal communication only, Accessible with in the cluster Only.
2. NodePort
   Exposes service on each Node’s IP. Port range: 30000–32767 , Accessible from outside using: ```http://NodeIP:NodePort```
   Not recommended for production. Used mostly for: Testing, Development
3. Service Type: LoadBalancer
   When you create: ```type: LoadBalancer``` Kubernetes directly asks cloud provider (AWS) to create a Load Balancer. ```User → AWS Load Balancer → Service → Pods```
- Important Points:  
- Usually creates NLB in EKS  
- Created per Service  
- No path-based routing  
- One LoadBalancer per service (costly if many services)

##### In EKS We use ALB via Ingress:
Here you:
- Install AWS Load Balancer Controller
- Create Ingress resource
- ALB is created automatically

## Ingress
AWS Ingress (in the context of Kubernetes on AWS) refers to an Ingress resource that manages external HTTP/HTTPS traffic into a Kubernetes cluster.  
An Ingress is a set of rules that controls how outside users access services inside your Kubernetes cluster—think of it as a smart front door for your applications.  
- Suppose you have 3 services, If you create separate LoadBalancer for each , ❌ Very costly, ❌ Difficult to manage  
- Instead, use: One LoadBalancer, One Ingress, It Routes traffic based on URL path ```(/predict)``` or domain/Host name (```model.mycompany.com```)     

Without Ingress ```User → LoadBalancer → Service → Pod```  If you have 3 services → 3 LoadBalancers.  
With Ingress ```User → LoadBalancer → Ingress → Service → Pod```  Single LoadBalancer handles all services.  

### Ingress Resource
Ingress resource is:  
👉 A YAML configuration  
👉 That defines routing rules  
👉 For external traffic  
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ml-ingress
spec:
  rules:
  - host: mlapp.com
    http:
      paths:
      - path: /predict
        pathType: Prefix
        backend:
          service:
            name: model-service
            port:
              number: 80
      - path: /train
        pathType: Prefix
        backend:
          service:
            name: training-service
            port:
              number: 80
```
Important:  
Ingress itself does not route traffic. It needs something called:  
👉 Ingress Controller: Without controller → Ingress does nothing.  
### Ingress Controller:
Ingress Controller is actual component that:  
1. Reads Ingress resource  
2. Configures LoadBalancer  
3. Routes traffic  

Popular Controllers:  
1. NGINX Ingress Controller  
2. AWS Load Balancer Controller (for EKS)  

In AWS EKS: When using AWS Load Balancer Controller: Ingress creates ALB automatically and ALB routes traffic to services.  
Flow in EKS: ```User → AWS ALB → Ingress Controller → Service → Pod```   
Note:  
Pod → runs application  
Service → stable access to pods  
Ingress → smart router for HTTP traffic  
LoadBalancer → exposes to internet  

### IngressClass
IngressClass tells Kubernetes which Ingress Controller should handle a particular Ingress resource. If you have multiple Ingress Controllers in cluster,
IngressClass decides who will manage that Ingress.  It is a seperate yaml file, referenced inside the Ingress resource using ingressClassName.  
Example: If using AWS Load Balancer Controller.   
```
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: alb
spec:
  controller: ingress.k8s.aws/alb
```
## Load Balancer
A Load Balancer distributes incoming traffic across multiple servers to improve availability, Performance and scalability. In Kubernetes, it exposes services to the internet and forwards traffic to pods.  
## Scaling in Kubernetes
1. Manual Scaling
   ```kubectl scale deployment model --replicas=5```  Basic method. Not used in production automation.
2. HPA (Horizontal Pod Autoscaler)  
- HPA monitors CPU/memory of running Pods.  
- If usage crosses the threshold (e.g., CPU > 70%), HPA adds more Pods.  
- When load drops, HPA removes extra Pods.  
Example: If your app has 2 Pods and traffic spikes, HPA scales it to 5 Pods automatically.  

Example YAML:  
```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: model-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: model-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```
Means:
👉 Target CPU usage = 70%  , Now Kubernetes HPA controller automatically does:  
If average CPU > 70% → Increase replicas  
If average CPU < 70% → Decrease replicas  
Always stays between minReplicas and maxReplicas  

3. VPA (Vertical Pod Autoscaler)
   Scales the resource limits (CPU & memory) of existing Pods — makes Pods bigger or smaller, not more or fewer.  
Example: If a Pod needs more memory, VPA increases its memory limit automatically.  
Note:- Not commonly used with HPA together

4. CA — Cluster Autoscaler  
Scales the number of Nodes in the cluster itself.  
Example: If all Nodes are full and new Pods can't be scheduled, CA adds a new Node to the cluster (works well with AWS EC2 Auto Scaling Groups).
```High Traffic → HPA scales Pods → No Node Capacity → Cluster Autoscaler adds EC2 → Pods scheduled```

## Security
### 1. RBAC(Role-Based Access Control)
It is a Kubernetes resource to control:  
- Who can access cluster  
- What actions they can perform  
- On which resources  
#### Components of RBAC
1️⃣ Role
2️⃣ RoleBinding
3️⃣ ClusterRole
4️⃣ ClusterRoleBinding  

### 2. IRSA (IAM Roles for Service Accounts)
Instead of giving AWS permissions to EC2 nodes, we give permissions directly to Pods.  
Without IRSA:  
We attach IAM role to EC2 node  
All pods on that node get same permissions ❌  
Security risk  

With IRSA:  
Each pod gets its own IAM role ✅  
Fine-grained access control  
Follows least privilege principle  

```Pod → ServiceAccount → IAM Role → AWS Service (S3, ECR, etc.)```
### 3. OIDC (OpenID Connect)
OIDC allows one system to verify the identity of another system using secure tokens.  
OIDC is the foundation for IRSA. 
```Pod → ServiceAccount → OIDC Token → AWS STS → Temporary IAM Credentials```  

#### 4. ServiceAccounts
A ServiceAccount is a Kubernetes identity used by Pods to interact with the Kubernetes API or external services.  
When a Pod wants to:
- Access Kubernetes API
- Read secrets
- Call other services
- Access AWS (via IRSA)
It needs an identity. That identity is ServiceAccount.  

#### 5. IAM (Identity and Access Management)
##### Policy   
Defines permissions who to access, what actions they can perform, and which resource. Ex: ec2, s3, lambda....etc.    
          - Aws Managed Policys  
          - Custom Policys   
```
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::ml-model-bucket/*"
}
```
##### Roles 
Role is like a folder/directory which holds policies. A folder just stores files.  
- But an IAM Role:  
- Holds policies ✅  (What you can do)  
- Can be assumed by trusted entities ✅  (Who can assume it)  
- Generates temporary credentials ✅. AWS uses: STS (Security Token Service)to give temporary credentials.  
So Role is more than just a Folder.  

###### Example:  
Role is like:
👉 A temporary ID card with specific permissions.  
Policy = Rules written on that ID card.  
Role = The ID card itself.  
Service/User = Person wearing the ID card.  
When someone assumes the role: They “wear the ID card” and get those permissions.  

Policys are only attached to:  
- Users(for permanent Credientials)
- User Groups
- Roles
- Roles can't assign to a user or service, only we can Assume role  ```IAM User → Assume Role → Temporary Credentials → Access S3```
```
1. Below JSON Says: user vinodh is allowed to assume this role. 
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::123456789012:user/vinodh"
  },
  "Action": "sts:AssumeRole"
}
```
2. User Assumes Role (CLI Example)
```
aws sts assume-role \
  --role-arn arn:aws:iam::123456789012:role/s3-read-role \
  --role-session-name test-session
```
- AWS returns: Temporary AccessKey, SecretKey, SessionToken.  Now vinodh can access S3 using these temporary credentials.  
- A service(ec2, s3, ecs, lambda) also assumes a role like a user. ```EC2 → Instance Profile → IAM Role → Policies → AWS Services```
NOTE: Assume role means: Temporarily taking the permissions of an IAM Role. You don’t permanently own it. You borrow it for some time.

#### Service 
Service Uses permissions (A service assumes a role means The service temporarily takes the permissions defined in that IAM Role, And gets temporary AWS credentials)  
```EC2 → Assume Role → Get Temporary Credentials → Access S3```  





