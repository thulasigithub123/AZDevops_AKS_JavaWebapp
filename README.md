
## Azure Devops CI/CD


# 🚀 Azure DevOps CI/CD Pipeline for Maven-based Java Application with SonarQube + Trivy Scan and deployment on to Azure Kubernetes Multi-Node Cluster

This project demonstrates a complete CI/CD setup using **Azure DevOps**, integrating:
- Self-hosted agent on Azure VM
- Maven build automation
- SonarQube for code quality
- Trivy for vulnerability scanning
- Docker for containerization
- AKS (Azure Kubernetes Service) for deployment

---

## workflow

![alt text](image-88.png)

## 📁 Project Structure

```bash
.
├── azure-pipelines.yml          # CI pipeline for build, test, scan, and Docker push
├── kubernetes-manifests/       # AKS deployment and service manifests
├── src/                        # Java application source code
└── README.md
```

---

## 🛠️ Tools & Technologies

| Tool           | Purpose                         |
|----------------|----------------------------------|
| Azure DevOps   | CI/CD orchestration              |
| Maven          | Build automation                 |
| SonarQube      | Code quality analysis            |
| Trivy          | Container vulnerability scanning |
| Docker         | Image creation & containerization|
| AKS            | Application deployment platform  |
| Azure VM (Linux)| Self-hosted agent infrastructure |

---

## 🧪 CI Pipeline Overview

### Trigger
- Runs on every push to `main` branch

### Tasks
1. **Checkout Code**
2. **Build with Maven**
3. **SonarQube Analysis**
4. **Trivy Security Scan**
5. **Docker Build & Push to Docker Hub**
6. **Publish Artifact for CD pipeline**

---

## 📦 Artifact

- Location: `$(Pipeline.Workspace)/drop`
- Contains built `.jar` and `Dockerfile`

---

## 🚚 CD Pipeline Overview

1. **Trigger**: When artifact is published by CI
2. **Tasks**:
   - Pull Docker image from registry
   - Deploy to AKS using kubectl and YAML manifests

---

## 🧰 Self-Hosted Agent Setup

### 1. Provision VM
- Create Ubuntu VM on Azure
- Allow SSH (port 22) and custom ports like 9000 (for SonarQube)

### 2. Install Required Packages
```bash
sudo apt update
sudo apt install openjdk-17-jre-headless maven docker.io
sudo snap install trivy
```

### 3. Configure Docker Permissions
```bash
sudo usermod -aG docker $USER
```

### 4. Download Azure DevOps Agent
```bash
# Download and extract the agent
wget https://vstsagentpackage.azureedge.net/agent/<version>/vsts-agent-linux-x64-<version>.tar.gz
tar -xzf vsts-agent-linux-x64-<version>.tar.gz
./config.sh
./run.sh
```

> Use your Azure DevOps organization URL and generate a Personal Access Token (PAT) to authenticate.

---

## 🔍 SonarQube Setup

### Run Container
```bash
docker run -d --name sonar -p 9000:9000 mc1arke/sonarqube-with-community-branch-plugin
```

### Access
- URL: `http://<VM_Public_IP>:9000`

---

## 🛡️ Trivy Security Scan

Used in CI pipeline to scan Docker image:
```yaml
- script: |
    trivy image mydockeruser/myapp:$(Build.BuildId)
```

---

## ☸️ AKS Deployment

### 1. Create Cluster
```bash
az aks create --resource-group <group> --name <cluster-name> --node-count 2 --generate-ssh-keys
az aks get-credentials --name <cluster-name> --resource-group <group>
```

### 2. Apply Manifests
```bash
kubectl apply -f kubernetes-manifests/
```

---

## 🧪 End-to-End Test

1. Make a commit to Git repo
2. CI pipeline triggers: build, scan, analyze, push
3. CD pipeline triggers: pulls image and deploys to AKS
4. Application accessible via Ingress/LoadBalancer

---

## 🧹 Cleanup

```bash
# Stop and remove agent
./svc.sh stop
./svc.sh uninstall
rm -rf _work _diag

# Remove resources
az devops project delete --id <project-id>
az vm delete --name <vm> --resource-group <group> --yes
az aks delete --name <cluster> --resource-group <group> --yes
```

---

## ✅ Result

- CI/CD fully automated using Azure DevOps
- Code quality checks via SonarQube
- Vulnerability scan via Trivy
- Docker container built and pushed
- Deployed to Azure Kubernetes Service
- Verified running application

---

## 📸 Screenshots

> Refer to the original document for images on:
- VM setup
- Agent configuration
- CI/CD run outputs
- SonarQube portal
- Trivy scan results
- AKS deployment verification

---


## Hands-on
 
# Project creation


![alt text](image.png)

![alt text](image-1.png)


# Import the repository

![alt text](image-2.png)
![alt text](image-3.png)


# pipelines

Classic Pipeline 
YAML / Declarative Pipeline

![alt text](image-4.png)
![alt text](image-5.png)
![alt text](image-6.png)

Wanted to create a MAVEN based project

![alt text](image-7.png)

![alt text](image-10.png)

![alt text](image-9.png)
![alt text](image-8.png)


##[error]No hosted parallelism has been purchased or granted. To request a free parallelism grant, please fill out the following form https://aka.ms/azpipelines-parallelism-request

because we dont have any agent machine to perform this pipeline task

![alt text](image-11.png)

hence, we can setup one VM


## create Azure vm

![alt text](image-12.png)
![alt text](image-13.png)
![alt text](image-14.png)


Use mobaXterm / remote desktop connection manager to connect to a vm

using public IP

![alt text](image-15.png)

![alt text](image-16.png)

in linux ( ubuntu) the package installation starts

`sudo apt update`
![alt text](image-17.png)


add the vm to our pipeline as agent

 - create the agent in the organization setting > pipeline > agent pool

![alt text](image-18.png)
![alt text](image-19.png)
![alt text](image-20.png)

execute the download cmdline in agent 

![alt text](image-22.png)

![alt text](image-23.png)
![alt text](image-24.png)

config.sh  - to make this VM as agent
run.sh  - to initialize and start the agent

![alt text](image-25.png)

add ur organization url

generate your personal access token

![alt text](image-26.png)

workfolder : 
![alt text](image-27.png)

![alt text](image-28.png)

agent is added but offline. so we are to start the agent service

![alt text](image-29.png)


![alt text](image-30.png)

agent is online Now

# --------------

# package installation in Agent server - Java, Maven, trivy, sonarqube
`sudo apt install openjdk-17-jre-headless`
`sudo apt install maven`
`sudo snap install trivy`
`sudo apt install docker.io`

![alt text](image-31.png)
![alt text](image-32.png)
![alt text](image-33.png)

# sonarqube - tool to perform code quality check and code coverage

source code > may have different issues, bugs, vulnerabilities...

in our project - we will use docker image ( free custom image with branch feature - code coverage )


![alt text](image-34.png)

![alt text](image-35.png)

provide docker access to the azureuser

![alt text](image-36.png)

`docker run -d --name sonar -p 9000:9000 mc1arke/sonarqube-with-community-branch-plugin`

![alt text](image-37.png)


check the container status

![alt text](image-38.png)

if you're unable to access the sonarqube portal

need to update the azure Network setting - nsg to allow port 9000 

![alt text](image-39.png)
![alt text](image-40.png)

![alt text](image-41.png)


![alt text](image-44.png)

![alt text](image-45.png)

we can also set some expectation for the agent to run about project task

![alt text](image-42.png)

like the agent should have java, maven, trivy, sonarqube

![alt text](image-43.png)


Start the agent service and test the basic pipeline function

![alt text](image-46.png)

atleast it is working

![alt text](image-47.png)
![alt text](image-48.png)

![alt text](image-49.png)
![alt text](image-50.png)


![alt text](image-51.png)

check the artifact 

![alt text](image-52.png)

# integrate sonarqube into pipeline using marketplace plugin / Extension



![alt text](image-53.png)


![alt text](image-54.png)

![alt text](image-55.png)


![alt text](image-56.png)

endpoint - our agent which runs the sonarqube container

![alt text](image-57.png)
![alt text](image-58.png)
![alt text](image-59.png)

cmd task to execute trivy scan

![alt text](image-60.png)

![alt text](image-61.png)


Run the CI pipeline and verify the result

 -  Maven test and Package status
 - artifact has been produced and dropped
 - sonar qube  project preparation and analysis
 - trivy scan completion
 - docker image creation and push to our registry


 ![alt text](image-62.png)
 ![alt text](image-63.png)
 ![alt text](image-64.png)

![alt text](image-66.png)
 ![alt text](image-65.png)


 Prepare the deployment environment - AKS

 cluster

 ![alt text](image-67.png)



 # Release Pipeline


 ![alt text](image-68.png)
 ![alt text](image-69.png)


 # End to End Test

 create a change in repo

 ![alt text](image-70.png)

 Test the CI trigger

 ![alt text](image-71.png)
 ![alt text](image-72.png)

 ![alt text](image-73.png)

 check the Continuous trigger on Release pipeline

 ![alt text](image-74.png)

 release is succeeded
 ![alt text](image-85.png)
 ![alt text](image-75.png)
 ![alt text](image-76.png)
 ![alt text](image-77.png)


 trivy report
![alt text](image-84.png)
 ![alt text](image-83.png)

 check if the pod is created in our AKS node

 ![alt text](image-78.png)

 2 pod replicas
 ![alt text](image-79.png)

 To access the application - service & Ingress

 ![alt text](image-80.png)

 Application is accessible and running fine

 ![alt text](image-81.png)
![alt text](image-82.png)


## destruction

Stop the agent
delete the pipeline
delete the repo
delete the PAT, secrets, service connections
delete the docker Hub repository
delete the Azure devops project
remove the agent pools
Clear up all your resources

![alt text](image-86.png)

