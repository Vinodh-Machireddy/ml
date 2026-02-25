<div align="center">
  
  # AWS
</div>

Interview Topics:  
- scalling   
- ingress   
- alb  
- Architecture explanation  
- Cost optimisation   
- Security best practices  
- monitoring  
- integration  
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


