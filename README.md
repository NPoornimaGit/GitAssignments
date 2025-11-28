# Task API — Kubernetes Deployment with Terraform + Helm
  
### Prerequisites

Install the following tools before running anything:  
    1. Docker Desktop  
    2. Terraform  
    3. k3d  
    4. helm  
    5. kubectl  
    6. Python 3.11+  


### Create a folder 'Task-API-Service' in your local and navigate to that directory

cd Task-API-Service

### Clone the repository

git clone https://github.com/PoornimaN-Personal/Task-API-Service.git

### Run Tests Locally

## Install dependencies

pip install -r src/requirements.txt

## Run tests

pytest

This will resturn "Passed" status for the test cases. you can tweek the code and make the testcase fail if you want to check.

### Build & Run the API Using Docker

## Build the Docker image

docker build -t task-api:latest .

Check if the image created  
docker images  

## Run the container

docker run -p 9000:8000 task-api:latest

## Validation
 http://localhost:8000/tasks

 # Expected:
 
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

# Note: Ensure that you have write permission to write package   

Settings --> Actions --> general --> Workflow Permission  
Enable Read & Write permissions  



