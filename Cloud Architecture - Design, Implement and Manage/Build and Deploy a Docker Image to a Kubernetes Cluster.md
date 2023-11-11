## Cloud Architecture: Design, Implement, and Manage
### Build and Deploy a Docker Image to a Kubernetes Cluster

### Challenge scenario
Your development team is interested in adopting a containerized microservices approach to application architecture. You need to test a sample application they have provided for you to make sure that it can be deployed to a Google Kubernetes container. The development group provided a simple Go application called `echo-web` with a Dockerfile and the associated context that allows you to build a Docker image immediately.

#### Your challenge
To test the deployment, you need to download the sample application, then build the Docker container image using a tag that allows it to be stored on the Container Registry. Once the image has been built, you'll push it out to the Container Registry before you can deploy it.

With the image prepared you can then create a Kubernetes cluster, then deploy the sample application to the cluster.

**Note:** In order to ensure accurate lab activity tracking you must use `echo-app` as the container repository image name, call your Kubernetes cluster `echo-cluster`, create the Kubernetes cluster in `filled in at lab start` zone and use `echo-web` for the deployment name.

<hr>

### Task 1. Create a Kubernetes Cluster
Your test environment is limited in capacity, so you should limit the test Kubernetes cluster you are creating to just two `e2-standard-2` instances. You must call your cluster `echo-cluster`.

Solution 👇👇👇 

1. Set default compute region and zone in local client (Cloud Shell) to that for Kubernetes cluster.
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

2. Create zonal Kubernetes cluster
```
gcloud container clusters create echo-cluster \
--machine-type=e2-standard-2 \
--num-nodes=2 \
```
    - note: a node is a VM instance. 2 VM instances = 2 nodes.  

To verify, `gcloud container clusters list`

Refeerence:  
[Create a zonal cluster](https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-zonal-cluster)  
[gcloud container clusters create command](https://cloud.google.com/sdk/gcloud/reference/container/clusters/create#--zone)  
[gcloud container clusters list command](https://cloud.google.com/sdk/gcloud/reference/container/clusters/list)  

<hr>

### Task 2. Build a tagged Docker Image
The sample application, including the Dockerfile and the application context files, are contained in an archive called `echo-web.tar.gz`. The archive has been copied to a Cloud Storage bucket belonging to your lab project called gs://[PROJECT_ID].

You must deploy this with a tag called `v1`.

Solution 👇👇👇 

1. Copy archive from Cloud Storage bucket to Cloud Shell
```
gcloud storage cp --recursive gs://[Project_ID]/echo-web.tar.gz . 
```

Reference:  
[gcloud storage](https://cloud.google.com/sdk/gcloud/reference/storage)  
[gcloud storage cp command](https://cloud.google.com/sdk/gcloud/reference/storage/cp)   

2. Unzip archive and extract application files into the current working directory
```
tar -xf  echo-web.tar.gz 
```

3. Build Docker image
```
docker build . --tag gcr.io/[Project_ID]/echo-app:v1
```
  - syntax: docker build . -t gcr.io/[Project_ID]/[Image Name]:[Tag]
  - the `.` in the command specifies that the PATH is the local directory
  - command must be executed from directory containing source code and Dockerfile

To verify, execute `docker images` to check for custom image. 

References:  
[Docker build](https://docs.docker.com/engine/reference/commandline/build/#examples)

<hr>

### Task 3. Push the image to the Google Container Registry
Your organization has decided that it will always use the `gcr.io` Container Registry hostname for all projects. The sample application is a simple web application that reports some data describing the configuration of the system where the application is running. It is configured to use TCP port 8000 by default.

Solution 👇👇👇 
1. Configure Docker (a 3rd-party client) to authenticate with Container Registry. Method: credential helper for gcloud. 
```
gcloud auth configure-docker
```
Command adds credentials for all GCR repositories. When asked "Do you want to continue (Y/n)?", enter `y`. In production, grant credentials for specific repo.

Reference:
[Authentication of third-party client with Container Registry](https://cloud.google.com/container-registry/docs/advanced-authentication#gcloud-helper)  

2. Push taggged Docker image to gcr.io Container Registry
```
docker push gcr.io/[Project_ID]/echo-app:v1
```
To verify, in Cloud console > Container Registry > confirm that `echo-app` container image has been uploaded in [Project_ID] repo.

<hr>

### Task 4. Deploy the application to the Kubernetes Cluster
Even though the application is configured to respond to HTTP requests on port 8000, you must configure the service to respond to normal web requests on port 80. When configuring the cluster for your sample application, call your deployment `echo-web`.

Solution 👇👇👇   

1. Create deployment
```
kubectl create deployment echo-web --image=gcr.io/[Project_ID]/echo-app:v1 --port=8000
```
To get the status of the deployment, `kubectl get deployments`

Reference:  
[kubectl create deployment](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-deployment-em-)  
[Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)  

2. Expose the deployment with a new Kubernetes service that is a single point of access for ephemeral pods.
```
kubectl expose deployment echo-web --type=LoadBalancer --port=80 --target-port=8000
```
To verify, `kubectl get services`  

Reference:  
[kubectl expose](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#expose)   
[Services](https://kubernetes.io/docs/concepts/services-networking/service/)   

3. Copy the external_IP address and open it in a new browser tab to view the application

<hr>

### Troubleshooting
**Receiving a 504, Gateway timeout error:** This might just indicate that the application hasn't quite initialized yet, but it could also be caused by a mismatch between the default port that is set in the Dockerfile (TCP port 8000) and the choice of application port you configured when deploying the application image, or when you configured external access.

**Not receiving assessment score for the last three objectives:** This might just indicate that you have created your Kubernetes cluster in the different zone rather than `Placeholder value`. zone which is expected in the lab.