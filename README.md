# Task API — Kubernetes Deployment with Terraform + Helm
  
## Prerequisites

Install the following tools before running anything:  
    - Docker Desktop  
    - Terraform  
    - k3d  
    - helm  
    - kubectl  
    - Python 3.11+  


### Create a folder 'Task-API-Service' in your local and navigate to that directory

cd Task-API-Service

### Clone the repository

git clone https://github.com/PoornimaN-Personal/Task-API-Service.git

## Run Tests Locally

### Install dependencies

pip install -r src/requirements.txt

### Run tests

pytest

This will resturn "Passed" status for the test cases. you can tweek the code and make the testcase fail if you want to check.

## Build & Run the API Using Docker

### Build the Docker image

docker build -t task-api:latest .

Check if the image created  
docker images  

###  Run the container

docker run -p 8000:8000 task-api:latest

If you want your application to run with custom port (for ex: 9000)  

docker run -p 9000:9000 -e APP_PORT=9000 task-api


### Validation
 http://localhost:8000/tasks

 ### Expected:
 
•	/tasks returns []

👉 http://localhost:8000/docs

•	/docs opens Swagger UI

You can test POST endpointss by posting few values and then try to see the values with GET endpoints

Now if you access 👉 http://localhost:8000/tasks , you can see the list which you have posted via Swagger UI

## GitHub CI Validation:

Assumption:
create one test repository and push the code from your local to GIT.   
As per CI.yaml file , the workflow will get triggered automatically upon push/pull request.  
git init  
git add .  
git commit -m "Initial Commit"    
git branch -m main  
git remote add origin https://github.com/<your-user-name>/<repo-name>/.git  
git push -u origin main  

This PUSH request will automatically trigger workflow and it will test, build and push the package to GHCR.  
Go to Actions --> You can see the workflow is running. Once it is finished    
Go to Package --> Chack if A new image with the name "task-api" is available  

### Note: Ensure that you have write permission to write package   

Settings --> Actions --> general --> Workflow Permission  
Enable Read & Write permissions 

## Terraform to deploy application 

Terraform configuration will do the following,

● Provision k3d Kubernetes cluster.
● Create a dedicated namespace for the application.
● Deploy the application using the Terraform Helm provider.

Navigate to terraform folder where we have the configuration files

cd Task-API-Servic/terraform

### Apply terraform

terraform init  
terraform apply --auto-approve

Terraform will:

✔ Create cluster 
✔ Create namespace task-api
✔ Deploy Helm chart
✔ Apply your image which you have pushed to GHCR already through workflow)

### Validation:

✅ 1. Validate the Kubernetes Cluster
Run:
kubectl cluster-info

kubectl get nodes

Expected:
•	1 server + 1 agent node (or whatever you configured)
•	Status: Ready

✅ 2. Confirm Namespace Exists
kubectl get ns
You should see:
task-api

✅ 3. Check Helm Release
helm list -n task-api
Expected output:
NAME      	NAMESPACE 	REVISION	STATUS  	CHART
taskapi   	task-api  	1       	deployed	taskapi-0.1.0

✅ 4. Validate Deployment
kubectl get deployments -n task-api
kubectl describe deployment taskapi -n task-api
Expected:
•	Deployment should show AVAILABLE = 1

✅ 5. Validate Pod is Running
kubectl get pods -n task-api
Expected:
taskapi-xxxxxx   Running
If Running → success 🎉

✅ 6. Validate Service
kubectl get svc -n task-api
You should see a ClusterIP service:
taskapi   ClusterIP   10.x.x.x   <none>        8000/TCP
Since it is ClusterIP, it is internal only.
To test the API, you must port-forward.

✅ 7. Test the API via Port Forward
Run:
kubectl port-forward svc/taskapi 8000:8000 -n task-api

To test it using different custom port,

kubectl port-forward svc/taskapi 9000:8000 -n task-api

## Helm Configuration Explanation

values.yaml contains:  

Key values in helm/values.yaml:
- replicaCount: number of API replicas
- image.repository: container image repository
- image.tag: image version tag
- service.port: port exposed by the Kubernetes Service
- 
Override with values.dev.yaml for local/dev environments.

You can override the image and port details by updating values.dev.yaml file

## Assumptions & Limitations
- Tasks are stored in memory only (no database persistence).
- Service is exposed internally (ClusterIP);
- ClusterIP service type is not externally accessible unless you port-forward.
- The container image is assumed to be publicly accessible (GHCR or Docker Hub).

## Design Decisions
 - FastAPI : high performancd, automatic data validation through pandatic , documentation genaration(Swagger UI)  
 - k3d cluster: Fast, simple and no cost 
 - GHCR: Easiest when compared to Docker Hub (No Extra accounts & No extra secrets needed. We can use the in-built ${{ secrets.GITHUB_TOKEN }}

## Use of AI








