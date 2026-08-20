Project Explanation
For this project, I implemented a CI/CD pipeline using Jenkins, Docker, Git, Kubernetes, and Argo CD.
I used two separate Git repositories:
•	Application/CI Repository – contains:
o	main.go
o	Dockerfile
o	Jenkinsfile
•	Kubernetes Repository – contains:
o	deploy.yaml
o	svc.yaml
o	ingress.yaml
1.	Application Repository
First, I created the application repository containing the main.go file, Dockerfile, and Jenkinsfile. The Dockerfile defines how the Go application is packaged into a Docker image, while the Jenkinsfile defines the CI pipeline.
2.	Jenkins Configuration
In Jenkins, I created a new Pipeline item.
I configured the following options:
•	Do not allow concurrent builds
•	Poll SCM: H/5 * * * *
•	Pipeline script from SCM
•	SCM: Git
•	Added the Git repository URL
•	Added Git credentials
•	Branch: */main
•	Script Path: Jenkinsfile
I used Poll SCM with H/5 * * * * so that Jenkins checks the Git repository approximately every five minutes for new changes. If a new commit is detected, Jenkins automatically starts the pipeline. I disabled concurrent builds because I do not want multiple builds of the same pipeline running at the same time.
3.	Building the Docker Image
When a new change is pushed to the application repository, Jenkins checks out the latest source code. The pipeline then uses the Dockerfile and main.go to build a new Docker image.
The image is given a new version/tag so that every application update can be identified separately.
4.	Pushing the Image to Docker Hub
After successfully building the Docker image, Jenkins logs in to Docker Hub and pushes the newly created image.
For example:
golang-app:0.0.15
This makes the new application image available for deployment.

 
5.	Updating the Kubernetes Repository
After pushing the Docker image, Jenkins clones my second Git repository, which is the Kubernetes/GitOps repository.
This repository contains the Kubernetes configuration files:
•	deploy.yaml
•	svc.yaml
•	ingress.yaml
Jenkins updates the image tag inside deploy.yaml.
For example, if the previous deployment was:
image: username/golang-app:0.0.14
Jenkins changes it to:
image: username/golang-app:0.0.15
Jenkins then commits and pushes this change back to the Git repository.
6.	Argo CD Deployment
This is where GitOps comes into the process.
Argo CD continuously monitors the Kubernetes Git repository.
When Jenkins pushes the updated deploy.yaml, Argo CD detects that the desired state in Git has changed. Argo CD then synchronizes the Kubernetes cluster with the updated Git repository and deploys the new Docker image into the production environment.
 
Overall Workflow
The complete workflow is:
Developer pushes code → Git → Jenkins detects change → Jenkins builds Docker image → Docker image pushed to Docker Hub → Jenkins updates Kubernetes deploy.yaml → Jenkins pushes the change to Git → Argo CD detects the change → Argo CD synchronizes Kubernetes → New version deployed to production.
Main Goal of the Project
The main goal of this project is to automate the complete application deployment process.
Whenever a new update is pushed to the application repository, the pipeline automatically:
1.	Builds a new Docker image.
2.	Pushes the image to Docker Hub.
3.	Updates the Kubernetes deployment with the new image tag.
4.	Pushes the updated Kubernetes configuration to Git.
5.	Allows Argo CD to detect the Git change.
6.	Argo CD synchronizes the change with Kubernetes.
7.	The new application version is deployed to production.
This approach combines CI through Jenkins with GitOps-based CD through Argo CD, making the deployment process automated, repeatable, and version-controlled.
 
 
Fig1: Architecture diagram
 
Fig2: Jenkins pipeline build history
 
Fig3: ArgoCD 

 
Fig4: output  

