## Cloud Architecture: Design, Implement, and Manage
### Scale Out and Update a Containerized Application on a Kubernetes Cluster

### Challenge scenario
You are taking over ownership of a test environment and have been given an updated version of a containerized test application to deploy. Your systems' architecture team has started adopting a containerized microservice architecture. You are responsible for managing the containerized test web applications. You will first deploy the initial version of a test application, called echo-app to a Kubernetes cluster called `echo-cluster` in a deployment called `echo-web`.

1. Before you get started, open the navigation menu and select **Cloud Storage**.

2. Verify the echo-web-v2.tar.gz file is in the gs://[PROJECT_ID] bucket.

![Scale out and update a containerized app on a Kubernetes Cluster image 1](https://github.com/TCLee-tech/Google-Challenge-Labs/blob/b1a8406c2a9f9885b19fba3eebf5144718e4d9e4/Cloud%20Architecture%20-%20Design%2C%20Implement%20and%20Manage/Scale%20out%20and%20update%20a%20containerized%20app%20on%20a%20Kubernetes%20Cluster%20image%201.png)

3. Check to make sure your GKE cluster has been created before continuing.

4. Open the navigation menu and select **Kuberntes Engine > Clusters**.

Continue when you see a green checkmark next to echo-cluster:

![Scale out and update a containerized app on a Kubernetes Cluster image 2](https://github.com/TCLee-tech/Google-Challenge-Labs/blob/b1a8406c2a9f9885b19fba3eebf5144718e4d9e4/Cloud%20Architecture%20-%20Design%2C%20Implement%20and%20Manage/Scale%20out%20and%20update%20a%20containerized%20app%20on%20a%20Kubernetes%20Cluster%20image%202.png)

5. To deploy your first version of the application, run the following commands in Cloud Shell to get up and running:
```
gcloud container clusters get-credentials echo-cluster --zone=Placeholder value.
```
```
kubectl create deployment echo-web --image=gcr.io/qwiklabs-resources/echo-app:v1
```
```
kubectl expose deployment echo-web --type=LoadBalancer --port 80 --target-port 8000
```
#### Your challenge
You need to update the running `echo-app` application in the `echo-web` deployment from the v1 to the v2 code you have been provided. You must also scale out the application to 2 instances (Qwiklab typo: should be 2 **replicas**) and confirm that they are all running.  

<hr>

### Task 1. Build and deploy the updated application with a new tag
The updated sample application, including the Dockerfile and the application context files, are contained in an archive called `echo-web-v2.tar.gz`. The archive has been copied to a Cloud Storage bucket in your lab project called gs://[PROJECT_ID]. V2 of the application adds a version number to the output of the application.

Solution 👇👇👇 

1. Follow the instructions in the Challenge Scenario.

2. Set default compute region and zone in local client (Cloud Shell) to that for Kubernetes cluster.
In Cloud Shell,
```
gcloud config set compute/region [Region]
gcloud config set compute/zone [Zone]
```
To verify,
```
gcloud config get-value compute/region
gcloud config get-value compute/zone
```
Reference:  
[gcloud compute defaults](https://cloud.google.com/compute/docs/gcloud-compute)

3. Download a copy of `echo-web-v2.tar.gz` from Cloud Storage bucket to Cloud Shell
```
gcloud storage cp --recursive gs://[Project_ID]/echo-web-v2.tar.gz . 
```

Reference:  
[gcloud storage](https://cloud.google.com/sdk/gcloud/reference/storage)  
[gcloud storage cp command](https://cloud.google.com/sdk/gcloud/reference/storage/cp)   

4. Unzip archive of application context files and Dockerfile into your current working directory
```
tar -xf  echo-web-v2.tar.gz 
```

5. Build the Docker image for version 2 (`v2`) of the `echo-web` application
```
docker build . --tag gcr.io/[Project_ID]/echo-app:v2
```
  - syntax: docker build . -t gcr.io/[Project_ID]/[Image Name]:[Tag]
  - the `.` in the command specifies that the PATH is the local directory
  - command must be executed from directory containing source code and Dockerfile

To verify, execute `docker images` to check for custom image. 

References:  
[Docker build](https://docs.docker.com/engine/reference/commandline/build/#examples)

<hr>

### Task 2. Push the image to the Container Registry
Your organization uses the Container Registry to host Docker images for deployments, and uses the `gcr.io` Container Registry hostname for all projects. You must push the updated image to the Container Registry before deploying it.

Solution 👇👇👇

1. Configure Docker (a 3rd-party client) to authenticate with Container Registry. Method: credential helper for gcloud. 
```
gcloud auth configure-docker
```
Command adds credentials for all GCR repositories. When asked "Do you want to continue (Y/n)?", enter `y`.   
In production, grant credentials for specific repo.

Reference:
[Authentication of third-party client with Container Registry](https://cloud.google.com/container-registry/docs/advanced-authentication#gcloud-helper)  

2. Push `v2`-taggged Docker image to gcr.io Container Registry
```
docker push gcr.io/[Project_ID]/echo-app:v2
```
To verify, in Cloud console > Container Registry > confirm that `echo-app` container image has been uploaded in [Project_ID] repo.

3. Update echo-web Kubernetes pods to use `echo-app:v2` image instead of `echo-app:v1` image
```
kubectl set image deployment/echo-web echo-app=gcr.io/[Project_ID]/echo-app:v2
```
- Syntax: kubectl set image deployment/[deployment name] [container/app]=[new container image]:[new tag]
- alternative approach is to edit deployment yaml `kubectl edit deployment/[deployment name]`

Verify that the rollout of `v2` of echo-app was sucessful by checking rollout status`kubectl rollout status deployment/echo-we`. Confirm with `kubectl get deployments` - check that `Pod Template.Containers.Image` refers to ` gcr.io/qwiklabs-gcp-xx-xxxx/echo-app:v2` and `Events` history reflects scale up of new image and scale down of old image.

Reference:  
[Kubernetes - Update a Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment)  

4. Upsize (scale) Deployment to 2 Replicas
```
kubectl scale deployment/echo-web --replicas=2
```
To verify, `kubectl get deployments`

Reference:   
[Kubernetes - Scaling a Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment)  

<hr>

### Troubleshooting

**Receiving a 504, Gateway timeout error:** This might just indicate that the application hasn't quite initialized yet, but it could also be caused by a mismatch between the default port that is set in the Dockerfile (TCP port 8000) and:
- The choice of application port you configured when deploying the application image, or
- When you configured external access.  
