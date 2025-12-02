# Task API — Kubernetes Deployment with Terraform + Helm

## Assumptions & Limitations

- **Tasks are stored in memory only**    
   - No database persistence is implemented. All tasks will be lost when the application restarts.  
- **Service exposure**  
    - The application is exposed internally using a ClusterIP service type.  
    - ClusterIP services are not externally accessible unless you use `kubectl port-forward` or configure an ingress/load balancer.  
- **Container image availability**  
    - The Docker image is assumed to be publicly accessible (via GHCR).  
    - Private registry authentication is not covered in this setup.  
- **Terraform state management**  
    - Terraform state files are stored locally.  
    - No remote backend configuration (e.g., S3 + DynamoDB) is included in this project.  
    - This means state is not shared across team members and must be managed manually.  

## Design Decisions
 - **FastAPI:**
     - High Performance
     - Automatic data validation through Pydantic models
     - Interactive documentation genaration(Swagger UI)  
 - **EKS cluster:**
     - Amazon EKS is a fully managed, production‑ready Kubernetes service.  
     - Though EKS incurs charges, I chose EKS cluster over a k3d cluster because:  
         - The k3d Terraform provider is not actively maintained and outdated, leading to issues.  
         - Running k3d with Terraform on Windows typically requires WSL2 setup for proper integration, which adds extra complexity and environment dependencies.  
 - **GHCR:**
     - Easiest when compared to Docker Hub
     - No Extra accounts or Secrets needed.
     - Built-in authentication using ${{ secrets.GITHUB_TOKEN }}
       
## Prerequisites

 - Before you begin, make sure the following tools are installed and available:  
    - Docker Desktop - **Must be installed and Running**
    - Git
    - AWS Account
    - IAM User with Access Key and Secret Key -**admin‑level or equivalent permissions for EKS + VPC + IAM + Helm**
    - AWS CLI  
    - Terraform  
    - Helm  
    - kubectl  
    - Python 3.11+
> [!WARNING]
> **Do not use the AWS root account. Always use IAM user with required permissions.**


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
 - Verify that the Docker Desktop is running
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

  #### API Testing with FastAPI SwaggerUI
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

  #### To run the application on a custom port, use 
```
docker run -p <port>:<port> -e APP_PORT=<port> task-api
```
> You should be able to access the application using `http://localhost:<port>/tasks`
>
> Repeat the API testing at `http://localhost:<port>/docs`

> [!Important]
> After this step, stop and remove the container with `docker stop` and `docker rm`, then remove the image with `docker rmi`.


## GitHub CI Validation:

-  Create a test repository in your Git account
-  Push this project’s code into that repository using below git commands
```
git remote set-url origin https://github.com/<your-username>/<repo-name>.git 
git branch -M main
git push -u origin main
```
> This PUSH request to the main branch will automatically trigger workflow.  
> It will:  
>  ✅ Run unit tests with pytest  
>  ✅ Build the Docker image  
>  ✅ Publish the image to GHCR  
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

## Helm Configuration Explanation

Key values in helm/values.yaml:

| Key    | Purpose |
| -------- | ------- |
| replicaCount  | Number of API replicas    |
| image.repository | Container image repository   |
| image.tag    | Image version tag  |
| service.port  | Port exposed by the Kubernetes Service |

You can override these details by updating values.dev.yaml file  

## Terraform to deploy app  

Terraform configuration will do the following,  

● Provision EKS Kubernetes cluster.  
● Create a dedicated namespace for the application.  
● Deploy the application using the Terraform Helm provider.  

- Navigate to terraform folder where we have the configuration files  
```
  cd terraform
```
- Configure AWS Account on local machine:
    - Install AWS CLI  
    - Open terminal(linux/mac)/command prompt(windows)  
    - Run `aws configure`  
    - Provide the access key, secret key and region as requested   
      
> [!Important]
> The IAM account used must have permissions to create and manage AWS EKS clusters, VPCs, subnets, IAM roles/policies, and to deploy applications via the Helm provider.

- Review and Update `terraform.tfvars` file - 
    - This file contains project‑specific variables such as  `aws_region`   `cluster_name`   `namespace`
    - Please update `aws_region` with the same value which you used while running  `aws configure`
    -  `cluster_name`  and  `namespace` - If you want you can update it with different values or else leave it as is.
      
- Review and Update `values_override.yaml.tpl` file - **Optional**
    - By default it will pick up the image from the repository `ghcr.io/poorniman-personal/task-api`
    - You can update `${image_repo}` and `${image_tag}`  with the image which we published to GHCR in the [GitHub CI Validation](#github-ci-validation) section
        - For Example `ghcr.io/<your-git-username>/task-api`
- Initialize Terraform  
```
terraform init
```
> ✔️ Downloads required providers and initializes the working directory.
- Review the execution plan  
```
terraform plan
```
> ✔️ Shows what resources will be created/modified

- Apply the configuration
```
terraform apply --auto-approve
```
> ✔️ This provisions the AWS EKS cluster and deploys the application.
- Configure kubectl for EKS Cluster  
   Once the EKS cluster is successfully created, you need to update your local kubeconfig so that `kubectl` can connect to it:
  
    ```
    aws eks update-kubeconfig --name taskapi-cluster --region <region>
    ```

### Validation:
After configuring kubeconfig and deploying the application with Helm, run the following commands to validate the setup:  

✅ Check the Cluster Info
```
kubectl cluster-info
```
> ✔️ Confirms that your `kubectl` is connected to the correct cluster.

✅ List nodes
```
kubectl get nodes
```
> ✔️ Worker nodes should appear with status `Ready`.

✅ List Namespaces 
```
kubectl get ns
```
> ✔️ Confirms that the `task-api` namespace exists.

✅ Check Helm Release
```
helm list -n task-api
```
> ✔️ Shows the Helm release deployed in the `task-api` namespace.

✅ List Pods 
```
kubectl get pods -n task-api
```
>  ✔️ Pods should be in `Running` state.

✅ List Services
```
kubectl get svc -n task-api
```
>  ✔️ Confirms that the Kubernetes Service is created and exposing the application using `ClusterIP`

> [!Note]
  > Since it is ClusterIP, it is internal only.  
  > Use  `kubectl port-forward ` to expose the service locally and test endpoints configuring ingress.  

✅ Test the API via Port Forward
```
kubectl port-forward svc/task-api 8000:8000 -n task-api
```
> ✔️ This maps port 8000 on your local machine to port 8000 inside the cluster


### Follow the [Test the application](#test-the-application) section to do the complete validation.

✅ To Test using different port 
```
kubectl port-forward svc/task-api <port>:8000 -n task-api
```

### Terraform destroy
 - After completing validation and testing, it’s important to clean up resources to avoid unnecessary costs.
```
terraform state rm kubernetes_namespace.taskapi
```
```
terraform destroy
```
- Type yes when prompted.  
> ✔️ This command will remove all AWS resources created by your Terraform configuration (EKS cluster, VPC, subnets,helm deployment etc.)

## Use of AI
 - Copilot and ChatGPT were used to assist with FastAPI, Pytest, and Helm chart code and and troubleshooting issues.  
   - **Reason for use**  
      - These technologies were new to me, so I leveraged AI for learning and to generate a base scaffolding of code and configuration  
      - However, I did not completely depend on AI — instead, I first studied the concepts through official documentation and YouTube tutorials, then refined and updated            the AI‑generated output.  
   - **What went well?**  
      - Saved time by quickly identifying root causes of errors such as EKS authentication and kubeconfig issues, provider configuration errors.  
      - Provided structured explanations with the reason for the error and step‑by‑step fixes that reduced trial‑and‑error time.  
   - **Manual Changes Required?**  
      - Although AI provided initial drafts, all output required manual refinement, including:  
          - The GET /tasks endpoint was updated to include an ID field for each task, based on insights from a YouTube tutorial.  
          - Updated Helm chart paths and value file and yaml files references to match the out project  requirement.  
    - **How did you verify the AI output?**  
         - Verified the output by running commands like `pytest` `terraform` `kubectl` locally  
         - Validated the cluster health and deployments by follwing the steps mentioned in [Validation](#validation)  

## References:
- [FastAPI Document](https://fastapi.tiangolo.com/)  
- [FastAPI Tutorial](https://www.youtube.com/watch?v=rvFsGRvj9jo)  
- [Pytest](https://docs.pytest.org/en/stable/example/)  
- [Pytest Tutorial](https://www.youtube.com/watch?v=7dgQRVqF1N0&list=LL&index=1)  
- [GithubCI](https://docs.github.co)  
- [TerraformEKS](https://registry.terraform.io/modules/terraform-aws-modules/eks/aws/latest)   
- [TerraformKubernetes](https://registry.terraform.io/providers/hashicorp/kubernetes/latest/docs)  
- [TerraformHelm](https://registry.terraform.io/providers/hashicorp/kubernetes/latest/docs)  












