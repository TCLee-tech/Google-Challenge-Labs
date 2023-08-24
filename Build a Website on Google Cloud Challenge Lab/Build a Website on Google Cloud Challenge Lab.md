# Build a Website on Google Cloud: Challenge Lab

This is the [challenge lab](https://www.cloudskillsboost.google/focuses/11765?parent=catalog) for the [Build a Website on Google Cloud](https://www.cloudskillsboost.google/quests/115) quest.

### Challenge scenario
You have just started a new role at FancyStore, Inc.

Your task is to take the company's existing monolithic e-commerce website and break it into a series of logically separated microservices. The existing monolith code is sitting in a GitHub repo, and you will be expected to containerize this app and then refactor it.

You are expected to have the skills and knowledge for these tasks, so don't expect step-by-step guides.

You have been asked to take the lead on this, after the last team suffered from monolith-related burnout and left for greener pastures (literally, they are running a lavender farm now). You will be tasked with pulling down the source code, building a container from it (one of the farmers left you a Dockerfile), and then pushing it out to GKE.

You should first build, deploy, and test the Monolith, just to make sure that the source code is sound. After that, you should break out the constituent services into their own microservice deployments.

Some FancyStore, Inc. standards you should follow:

- Create your cluster in REGION (eg. us-east4)

- Naming is normally *team-resource*, e.g. an instance could be named **fancystore-orderservice1**.

- Allocate cost effective resource sizes. Projects are monitored and excessive resource use will result in the containing project's termination.

- Use the *e2-medium* machine type unless directed otherwise.

<hr>

### Task 1. Download the monolith code and build your container
1. Log in to your new project and open up Cloud Shell.

2. First things first, you'll need to clone your team's [git repo](https://github.com/googlecodelabs/monolith-to-microservices). There's a *setup.sh* script in the root directory of the project that you'll need to run to get your monolith container built up.

3. After running the *setup.sh* script, ensure your Cloud Shell is running its latest version of nodeJS:

`nvm install --lts`

There will be a few different projects that can be built and pushed.

4. Push the monolith build (conveniently located in the *monolith* directory) up to the Google Container Registry. There's a Dockerfile located in the *~/monotlith-to-microservices/monolith* folder which you can use to build the application container.

5. You will have to run Cloud Build (in that monolith folder) to build it, then push it up to GCR.

6. Name your artifact as follows:  
    - GCR Repo: gcr.io/${GOOGLE_CLOUD_PROJECT}
    - Image name: *Monolith Identifier, e.g. fancy-monolith-571*
    - Image version: 1.0.0

Hint:

Make sure that you submit a build named *Monolith Identifier* with a version of "1.0.0".

##### 🔴 Solution:  

Set GOOGLE_CLOUD_PROJECT environment variable:
```
export GOOGLE_CLOUD_PROJECT=$(gcloud config list --format 'value(core.project)')
```
(just in case) Set default region:
```
gcloud config set compute/zone ZONE           // given in Task 2 instructions
```
(just in case) Enable Containers API so that you can use GKE:
```
gcloud services enable container.googleapis.com
```
In Cloud Shell, clone the git repository containing the source codes and run the startup script to build the monolith application:
```
git clone https://github.com/googlecodelabs/monolith-to-microservices
cd monolith-to-microservices
./setup.sh
```
Install the long-term stable version of nodeJS:
```
nvm install --lts
```
Build the monolith container image using Cloud Build and push to Google Container Registry (GCR):
```
cd ~/monolith-to-microservices/monolith
gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/Monolith Identifier:1.0.0 .
```
To verify, in Cloud console, **Navigation menu > CI/CD > Container Registry > Images**.  
To view build history, **Navigation menu > Cloud Build > History**. Click on build ID to see details of build. From the build details page, click on **Execution Details** tab to see build image.  

<hr>

### Task 2. Create a kubernetes cluster and deploy the application
Now that you have the image created and sitting in the container registry, it's time to create a cluster to deploy it to.

You've been told to deploy all of your resources in the *ZONE, e.g. us-east4-a* zone, so first you'll need to create a GKE cluster for it. Start with a 3 node cluster to begin with.

1. Create your cluster as follows:
    - Cluster name: *Cluster Name, e.g. fancy-cluster-494*
    - Region: *REGION, e.g. us-east4*
    - Node count: 3

**Hint:**

Make sure your cluster is named *Cluster Name, e.g. fancy-cluster-494*, and is in the running state in *ZONE, e.g. us-east4-a*.

Now that you've built up an image, and have a cluster up and running, it's time to deploy your application.

You'll need to deploy the image that you've built onto your cluster. This will get your application up and running, but it can't be accessed until you expose it to the outside world. Your team has told you that the application runs on port 8080, but you will need to expose this on a more consumer-friendly port 80.

2. Create and expose your deployment as follows:
    - Cluster name: *Cluster Name, e.g. fancy-cluster-494*
    - Container name: *Monolith Identifier, e.g. fancy-monolith-571*
    - Container version: 1.0.0
    - Application port: 8080
    - Externally accessible port: 80

Note: For purposes of this lab, exposure of the service has been simplified. Typically, you would use an API gateway to secure your public endpoints. Learn more about best practices in the [Best practices for microservices Guide](https://cloud.google.com/architecture/migrating-a-monolithic-app-to-microservices-gke#best_practices_for_microservices).

3. Make note of the IP address that is assigned in the expose deployment operation. You should now be able to visit this IP address from your browser!

You should see the following:

![Fancy Store web page](https://github.com/TCLee-tech/Google-Challenge-Labs/blob/b8a0d382a7735aa9631e1b8d6c7363379213bfe1/Build%20a%20Website%20on%20Google%20Cloud%20Challenge%20Lab/Fancy%20Store%20web%20page.png)

**Hint:**

Make sure your deployment is named *Monolith Identifier, e.g. fancy-monolith-571*, and that you have exposed the service on port 80, and mapped it to port 8080.

##### 🔴 Solution:  

Create a GKE cluster:
```
gcloud container clusters create CLUSTER_NAME --zone=ZONE --num-nodes=3
```
To verify VM nodes:
```
gcloud compute instances list
```
To verify cluster in Cloud Console, **Navigation menu > Kubernetes Engine > Clusters**.

Read more:
[gcloud container clusters create](https://cloud.google.com/sdk/gcloud/reference/container/clusters/create)

To deploy container image in GCR to GKE cluster:
```
kubectl create deployment fancy-monolith-xxx --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/Monolith Identifier:1.0.0
```
To verify deployment:
`kubectl get deployment`  

To access container in GKE cluster from outside, create a Service (in this case, a Load Balancer that routes traffic from external 80 port to internal 8080 port):  
```
kubectl expose deployment fancy-monolith-xxx --type=LoadBalancer --port 80 --target-port 8080
```
To get external IP assigned by GKE to Service resource:
```
kubectl get service fancy-monolith-xxx
```
or, **Navigation menu > Kubernetes Engine > Service and Ingress > Endpoints**

Cut and paste external IP in a new browser tab to access FancyStore monolithic website.

<hr>

### Migrate monolith to microservices

Now that you have your existing monolith website running on GKE, you can start breaking each service into a microservice. Typically, a planning effort should take place on which services to break into smaller chunks, typically around specific parts of the application like business domain.

For the purposes of this Challenge, fast forward a bit and pretend that you have successfully broken out the monolith into a series of microservices: Orders, Products, and Frontend. Your code is ready, so now you've got to deploy your services.

<hr>

### Task 3. Create new microservices
There are 3 services that need to be broken out into their own containers. Since you are moving all of the services into containers, you need to track the following information for each service:

- The root folder of the service (where you will build the container)
- The repository you will upload the container to
- The name & version of the container artifact

**Create a containerized version of your microservices**

Below is the set of services which need to be containerized.

1. Navigate to the source roots mentioned below, and upload the artifacts that are created to the Google Container Registry with the metadata indicated:

| Orders Microservice | Service root folder: ~/monolith-to-microservices/microservices/src/orders |
| --- | --- |
| -> | GCR Repo: gcr.io/${GOOGLE_CLOUD_PROJECT} |
| -> | Image name: Orders Identifier, e.g. fancy-orders-543 |
| -> | Image version: 1.0.0 |
| **Products Microservice** | Service root folder: ~/monolith-to-microservices/microservices/src/products |
| -> | GCR Repo: gcr.io/${GOOGLE_CLOUD_PROJECT} |
| -> | Image name: Products Identifier, e.g. fancy-products-352 |
| -> | Image version: 1.0.0 |

2. Once these microservices have been containerized, and their images uploaded to the GCR, you should deploy and expose these services.

Hint: Make sure that you submit a build named Orders Identifier (fancy-orders-xxx) with a version of "1.0.0", AND a build named Products Identifier (fancy-products-xxx) with a version of "1.0.0".

##### 🔴 Solution:  
Build Orders Microservice container image and push to GCR:
```
cd ~/monolith-to-microservices/microservices/src/orders  
gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/Orders Identifier:1.0.0 .
```
Deploy Orders Microservice container image to GKE cluster:
```
kubectl create deployment fancy-orders-xxx --image=gcr.io/${GOOGLE_CLOUD_PRODUCT}/Orders Identifier:1.0.0
```

Build Products Microservice container image and push to GCR:
```
cd ~/monolith-to-microservices/microservices/src/products
gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/Products Identifier:1.0.0 .
```
Deploy Products Microservice container image to GKE cluster:
```
kubectl create deployment fancy-products-xxx --image=gcr.io/${GOOGLE_CLOUD_PRODUCT}/Products Identifier:1.0.0
```
To verify, `kubectl get deployments`
Or, **Navigation menu > Kubernetes Engine > Workloads** and make sure there is a "tick" mark and OK under **Status** for each workload (deployed container).

![kubectl create deployments ok](https://github.com/TCLee-tech/Google-Challenge-Labs/blob/6059006bddafc244d77c29e03fb69a5bad79beef/Build%20a%20Website%20on%20Google%20Cloud%20Challenge%20Lab/gke%20deployments%20ok.jpg)

<hr>

### Task 4. Deploy the new microservices
Deploy these new containers following the same process that you followed for the Monolith Identifier monolith. Note that these services will be listening on different ports, so make note of the port mappings in the table below.

1. Create and expose your deployments as follows:

| Orders Microservice | Cluster name: Cluster Name, e.g. fancy-cluster-494 |
| --- | --- |
| -> | Container name: Orders Identifier, e.g. fancy-orders-543 | 
| -> | Container version: 1.0.0 |
| -> | Application port: 8081 |
| -> | Externally accessible port: 80 |
| **Products Microservice** | Cluster name: Cluster Name, e.g. fancy-cluster-494 |
| -> | Container name: Products Identifier, e.g. fancy-products-352 |
| -> | Container version: 1.0.0 |
| -> | Application port: 8082 |
| -> | Externally accessible port: 80 |

NOTE: Please make note of the IP address of both the Orders and Products services once they have been exposed, you will need them in future steps.

You can verify that the deployments were successful and that the services have been exposed by going to the following URLs in your browser:

http://ORDERS_EXTERNAL_IP/api/orders   
http://PRODUCTS_EXTERNAL_IP/api/products

You will see each service return a JSON string if the deployments were successful.

Hint: Make sure your deployments are named `Orders Identifier (fancy-orders-xxx)` and `Products Identifier (fancy-products-xxx)`, and that you see the services exposed on port 80.

##### 🔴 Solution:  
To expose Orders Microservice container:
```
kubectl expose deployment fancy-orders-xxx --type=LoadBalancer --port 80 --target-port 8081
```
To expose Products Microservice container:
```
kubectl expose deployment fancy-products-xxx --type=LoadBalancer --port 80 --target-port 8082
```
To verify, get the external IP of the microservices:
```
kubectl get services
```
Or, **Navigation menu > Kubernetes Engine > Services and Ingress** and make sure there is a "tick" mark and OK under **Status** for each external load balancer service. Copy the endpoints.

Subsitute into the following urls and test in new browser tabs:
```
http://ORDERS_EXTERNAL_IP/api/orders    
http://PRODUCTS_EXTERNAL_IP/api/products  
```

![Expose deployment ok](https://github.com/TCLee-tech/Google-Challenge-Labs/blob/6059006bddafc244d77c29e03fb69a5bad79beef/Build%20a%20Website%20on%20Google%20Cloud%20Challenge%20Lab/GKE%20services%20ok.jpg)

<hr>

### Task 5. Configure and deploy the Frontend microservice
Now that you have extracted both the Orders and Products microservice, you need to configure the Frontend service to point to them, and get it deployed.

#### Reconfigure Frontend
1. Use the nano editor to replace the local URL with the IP address of the new Orders and Products microservices:
```
cd ~/monolith-to-microservices/react-app
nano .env
```
When the editor opens, your file should look like this:  
REACT_APP_ORDERS_URL=http://localhost:8081/api/orders  
REACT_APP_PRODUCTS_URL=http://localhost:8082/api/products  

2. Replace the Orders and Product microservice IP addresses so they match below:
```
REACT_APP_ORDERS_URL=http://<ORDERS_IP_ADDRESS>/api/orders
REACT_APP_PRODUCTS_URL=http://<PRODUCTS_IP_ADDRESS>/api/products
```
3. Press **CTRL+O**, press **ENTER**, then **CTRL+X** to save the file in the nano editor.

4. Now rebuild the frontend app before containerizing it:
```
npm run build
```
##### 🔴 Solution:  
Follow Task 5 instructions.

If you want to verify, `cat .env`. Click on each displayed url link to open a new browser tab with the returned JSON string.

<hr>

### Task 6. Create a containerized version of the Frontend microservice
With the Orders and Products microservices now containerized and deployed, and the Frontend service configured to point to them, the final step is to containerize and deploy the Frontend.

Use Cloud Build to package up the contents of the Frontend service and push it up to the Google Container Registry.

- Service root folder: ~/monolith-to-microservices/microservices/src/frontend
- GCR Repo: gcr.io/${GOOGLE_CLOUD_PROJECT}
- Image name: Frontend Identifier, e.g. fancy-frontend-562
- Image version: 1.0.0

This process may take a few minutes, so be patient.

Hint: Make sure that you submit a build named Frontend Identifier with a version of "1.0.0".

##### 🔴 Solution:  
Build Frontend Microservice container image and push to GCR:
```
cd ~/monolith-to-microservices/microservices/src/frontend
gcloud builds submit --tag gcr.io/${GOOLGE_CLOUD_PROJECT}/Frontend Identifier:1.0.0 .
```
To verify, `gcloud container images list --repository=gcr.io/${GOOGLE_CLOUD_PROJECT}`

<HR>

### Task 7. Deploy the Frontend microservice
Deploy this container following the same process that you followed for the "Orders" and "Products" microservices.

1. Create and expose your deployment as follows:
    - Cluster name: Cluster Name, e.g. fancy-cluster-494
    - Container name: Frontend Identifier, e.g. fancy-frontend-562
    - Container version: 1.0.0
    - Application port: 8080
    - Externally accessible port: 80

2. You can verify that the deployment was successful and that the microservices have been properly exposed by hitting the IP address of the frontend service in your browser.

You will see the Fancy Store homepage, with links to the Products and Orders pages powered by your new microservices.

##### 🔴 Solution:  
To deploy Frontend Microservice container image to GKE cluster:
```
kubectl create deployment fancy-frontend-xxx --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/Frontend Identifier:1.0.0
```
To expose the Frontend Microservice container:
```
kubectl expose deployment fancy-frontend-xxx --type=LoadBalancer --port 80 --target-port 8080
```
To get the external IP assigned by GKE to the Frontend microservice:
```
kubectl get service fancy-frontend-xxx
```
Cut and paste the external IP address into a new browser tab to access the Frontend microservice. 

Or, **Navigation menu > Kubernetes Engine > Services and Ingress** and wait for a "tick" mark and OK under **Status** for fancy-frontend-xxx Service. Click on the Endpoint url link for the Frontend microservice.

From the Frontend microservice webpage, click on the "Products" and "Orders" links to access the Orders and Products microservices.

![Frontend microservice](https://github.com/TCLee-tech/Google-Challenge-Labs/blob/528fba425b8e796d3dd43a81ea43e36995e14f79/Build%20a%20Website%20on%20Google%20Cloud%20Challenge%20Lab/Frontend%20microservice.jpg)

![Orders microservice](https://github.com/TCLee-tech/Google-Challenge-Labs/blob/528fba425b8e796d3dd43a81ea43e36995e14f79/Build%20a%20Website%20on%20Google%20Cloud%20Challenge%20Lab/Orders%20microservice.jpg)

![Products microservice](https://github.com/TCLee-tech/Google-Challenge-Labs/blob/528fba425b8e796d3dd43a81ea43e36995e14f79/Build%20a%20Website%20on%20Google%20Cloud%20Challenge%20Lab/Products%20microservice.jpg)

