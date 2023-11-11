## Cloud Architecture: Design, Implement, and Manage
### Scale Out and Update a Containerized Application on a Kubernetes Cluster

### Challenge scenario
You are taking over ownership of a test environment and have been given an updated version of a containerized test application to deploy. Your systems' architecture team has started adopting a containerized microservice architecture. You are responsible for managing the containerized test web applications. You will first deploy the initial version of a test application, called echo-app to a Kubernetes cluster called `echo-cluster` in a deployment called `echo-web`.

1. Before you get started, open the navigation menu and select **Cloud Storage**.

2. Verify the echo-web-v2.tar.gz file is in the `gs://[PROJECT_ID]` bucket.

![Scale out and update a containerized app on a Kubernetes Cluster image 1](https://github.com/TCLee-tech/Google-Challenge-Labs/blob/b1a8406c2a9f9885b19fba3eebf5144718e4d9e4/Cloud%20Architecture%20-%20Design%2C%20Implement%20and%20Manage/Scale%20out%20and%20update%20a%20containerized%20app%20on%20a%20Kubernetes%20Cluster%20image%201.png)

3. Check to make sure your GKE cluster has been created before continuing.
4. Open the navigation menu and select **Kuberntes Engine > Clusters**.   
Continue when you see a green checkmark next to `echo-cluster`:

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
You need to update the running `echo-app` application in the `echo-web` deployment from the v1 to the v2 code you have been provided. You must also scale out the application to 2 instances (Qwiklab's typo: should be 2 **replicas**) and confirm that they are all running.  

<hr>

### Task 1. Build and deploy the updated application with a new tag
The updated sample application, including the Dockerfile and the application context files, are contained in an archive called `echo-web-v2.tar.gz`. The archive has been copied to a Cloud Storage bucket in your lab project called `gs://[PROJECT_ID]`. V2 of the application adds a version number to the output of the application.

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

To verify, execute `docker images` to check for custom image and base image. 

References:  
[Docker build](https://docs.docker.com/engine/reference/commandline/build/#examples)

<hr>

### Task 2. Push the image to the Container Registry
Your organization uses the Container Registry to host Docker images for deployments, and uses the `gcr.io` Container Registry hostname for all projects. You must push the updated image to the Container Registry before deploying it.

Solution 👇👇👇

1. Push `v2`-taggged Docker image to gcr.io Container Registry
```
docker push gcr.io/[Project_ID]/echo-app:v2
```
To verify, in Cloud console > Container Registry > confirm that `echo-app:v2` container image has been uploaded

2. Update echo-web Kubernetes pods to use `echo-app:v2` image instead of `echo-app:v1` image
```
kubectl set image deployment/echo-web echo-app=echo-app:v2
```
- Syntax: kubectl set image deployment/[deployment name] [container/app]=[container/app]:[new tag]
- alternative approach is to edit deployment yaml `kubectl edit deployment/[deployment name]`

To verify, check rollout status for `v2` of echo-app `kubectl rollout status deployment/echo-web`, and/or get details of updated deployment with `kubectl get deployments`  

Reference:  
[Kubernetes - Update a Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment)  

3. Upsize (scale) Deployment to 2 Replicas
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
