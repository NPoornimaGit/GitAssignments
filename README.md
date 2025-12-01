# Task API — Kubernetes Deployment with Terraform + Helm
  
## Prerequisites

 - Before you begin, make sure the following tools are installed and available:  
    - Docker Desktop - **Must be installed and Running**
    - Git
    - AWS Account
    - IAM User with Access Key and Secret Key
    - AWS CLI  
    - Terraform  
    - Helm  
    - kubectl  
    - Python 3.11+  

## Run Tests Locally
 - Open terminal
 - Clone the repository
```
git clone https://github.com/PoornimaN-Personal/Task-API-Service.git
```
 - Move into project folder
```
cd Task-API-Service
```
 - Install dependencies
```
pip install -r src/requirements.txt
```
 - Run Test
```
pytest -v
```
> ✔️ All tests should pass, and you’ll see a summary showing `3 passed` in the terminal.


## Build & Run the container Using Docker

 - Build the Docker image
```
docker build -t task-api:latest .
```
>  ✔️ This creates a Docker image named `task-api:latest` using the Dockerfile in the project root.
> 
>  ✔️   Run `docker images` to confirm the image was created. 
- Run the container
```
docker run -p 8000:8000 task-api:latest
```
> ✔️ This starts the container and maps port `8000` on your machine to port `8000` inside the container.
- Verify the container is running
```
docker ps
```
> ✔️ You should see `task-api` listed as an active container

## Test the application

- Open your browser and access the below url
```
http://localhost:8000/tasks
```
- Expected response:
```
[]
```
> ✔️ Once the app is running, open `http://localhost:8000/docs` to access the FastAPI Swagger UI. Use it to test the GET and POST endpoints interactively.

### API Testing with FastAPI SwaggerUI
 - Navigate to
```
   http://localhost:8000/docs
```
 - Test the GET endpoint
      - Expand the `GET /tasks` operation.
      - Click **Try it out → Execute**
      - ✔️ You should see a response with the current list of tasks (initially empty)
  - Test the POST endpoint
      - Expand the `POST /tasks` operation.
      - Click **Try it out** and provide a JSON body, for example:
      ```
        {
            "title": "Sample Task",
            "done": "false"
        }
      ```
      - Click **Execute.**
      - ✔️ You should see the created task returned in the response.
- Verify GET again
    - Re‑run the `GET /tasks` request.
    - ✔️ The list should now include the task you just created.

### To run the application on a custom port, use 
```
docker run -p <port>:<port> -e APP_PORT=<port> task-api
```
> You should be able to access the application using `http://localhost:8000/tasks`
>
> Repeat the API testing at `http://localhost:<port>/docs`

> [!Note]
> After testing, stop and remove the container with `docker stop` and `docker rm`, then remove the image with `docker rmi` to free up space.


## GitHub CI Validation:

-  Create a test repository in your Git account
-  Push this project’s code into that repository using below git commands
```
git remote set-url origin https://github.com/<your-username>/<repo-name>.git  
git branch -M main  
git push -u origin main  
```
> This PUSH request to the main brabch will automatically trigger workflow.
> It will:
> - ✅ Run unit tests with pytest
> - ✅ Build the Docker image
> - ✅ Publish the image to GHCR
- Viewing Results  
  - Go to the **Actions** tab in your GitHub repository.
  - Select the latest workflow run to see logs for each step.
  - > ✔️ You will see tests passed, image built, and published to GHCR
  - Go to **Package** tab on you GitHub account
  - > ✔️ You should see your Docker image with the name `task-api` listed  

> [!Note]
> In case if workflow fails, then please enssure you have write permission to write package  
> - Go to **Settings --> Actions --> General --> Workflow Permission**    
> - [x] **Enable Read & Write permissions**

## Terraform to deploy application 

Terraform configuration will do the following,

● Provision eks Kubernetes cluster.
● Create a dedicated namespace for the application.
● Deploy the application using the Terraform Helm provider.

Navigate to terraform folder where we have the configuration files

cd Task-API-Servic/terraform

### COnfigure AWS Account 
aws configure

give your access key and secret key details

### Apply terraform

terraform init  

When Terraform loads providers (Kubernetes/Helm), it needs kubeconfig at init time.  
But the kubeconfig file is only created after the cluster exists.  
So we should follow 2-phase terraform apply
1. Phase 1 → cluster comes up & kubeconfig file is created  
Phase 2 → providers can use the kubeconfig

terraform apply -target="k3d_cluster.taskapi" -target="data.k3d_cluster.taskapi" -target="local_file.kubeconfig"  

terraform apply --auto-approve

Terraform will:
Phase 1
✔ Create cluster 
✔ read the cluster
✔ write the kubeconfig file
Phase 2
✔ Create namespace task-api
✔ Deploy Helm chart
✔ Apply your image which you have pushed to GHCR already through workflow)

### Validation:
Once cluster is created

aws eks update-kubeconfig --name taskapi-cluster --region ap-southeast-2 
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
kubectl port-forward svc/task-api 8000:8000 -n task-api

To test it using different custom port,

kubectl port-forward svc/task-api 9000:8000 -n task-api


## Terraform destroy

terraform destroy

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








