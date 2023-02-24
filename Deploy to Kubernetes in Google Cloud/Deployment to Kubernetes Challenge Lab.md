## Deploy to Kubernetes in Google Cloud: Challenge Lab

#### Challenge scenario
You have just completed training on containers and their creation and management and now you need to demonstrate to the Jooli Inc. development team your new skills. You have to help with some of their initial work on a new project around an application environment utilizing Kubernetes. Some of the work was already done for you, but other parts require your expert skills.

You are expected to create container images, store the images in a repository, and configure a Jenkins CI/CD pipeline to automate the build for the product. Your know that Kurt, your supervisor, will ask you to complete these tasks:

- Create a Docker image and store the Dockerfile.
- Test the created Docker image.
- Push the Docker image into the Container Repository.
- Use the image to create and expose a deployment in Kubernetes
- Update the image and push a change to the deployment.
- Create a pipeline in Jenkins to deploy a new version of your image when the source code changes.

Some Jooli Inc. standards you should follow:

- Create all resources in the `us-east1` region and `us-east1-b` zone, unless otherwise directed.

- Use the project VPCs.

- Naming is normally *team-resource*, e.g. an instance could be named **kraken-webserver1**.

- Allocate cost effective resource sizes. Projects are monitored and excessive resource use will result in the containing project's termination (and possibly yours), so beware. This is the guidance the monitoring team is willing to share: unless directed, use `n1-standard-1`.

<hr>

##### Task 1. Create a Docker image and store the Dockerfile  

1. Open Cloud Shell and run source <(gsutil cat gs://cloud-training/gsp318/marking/setup_marking_v2.sh). This command will install marking scripts you can use to help check your progress.  
2. Use Cloud Shell to clone the valkyrie-app source code repository (it is in your project).  
  The app source code is in valkyrie-app/source.

> `cd ..` to switch out of /marking sub-directory  
> `gcloud source repos clone valkyrie-app`  
> To verify, `ls` to show: marking  README-cloudshell.txt  valkyrie-app  
> `cd valkyrie-app` to change into valkyrie-app sub-directory.

3. Create valkyrie-app/Dockerfile and add the configuration below:  
```
FROM golang:1.10
WORKDIR /go/src/app
COPY source .
RUN go install -v
ENTRYPOINT ["app","-single=true","-port=8080"]
```

> Solution:
> ```
> cat > Dockerfile <<EOF
> FROM golang:1.10
> WORKDIR /go/src/app
> COPY source .
> RUN go install -v
> ENTRYPOINT ["app","-single=true","-port=8080"]
> EOF
> ```
> To verify, `cat Dockerfile` should show content of Dockerfile as above.

4. Use valkyrie-app/Dockerfile to create a Docker image called `Image Name` with the tag `Tag Name`.  

>`docker build -t [Image Name]:[Tag Name] .`  
>  In my lab, `Image Name` was `valkyrie-dev` and `Tag Name` was `v0.0.3`  
>  Run `docker images` to verify images built. Should see `valkyrie-dev` custom image and `golang` base image.

5. Once you have created the Docker image, and before clicking Check my progress, run `step1_v2.sh` to perform the local check of your work.  

>`cd..`  
>`cd marking`  
>`step1_v2.sh`  

After you get a successful response from the local marking you can check your progress.  
Click Check my progress to verify the objective.

<hr>

##### Task 2. Test the created Docker image

1. Launch a container using the image `Image with Tag`:
  - You need to map the host’s port 8080 to port 8080 on the container.
  - Add & to the end of the command to cause the container to run in the background.

> `cd ~/valkyrie-app`  
> `docker run -p 8080:8080 [Image Name]:[Tag name] &`

When your container is running you will see the page by Web Preview.

> Extra (but unnecessary) checks:  
> `curl http://localhost:8080` can also be used to check if the application within container can be reached at localhost port 8080.  
> `docker ps` should list the running container  
> `docker logs [container_id]` will show activity log of that container.  

2. Once you have your container running, and before clicking Check my progress, run `step2_v2.sh` to perform the local check of your work. After you get a successful response from the local marking you can check your progress.  
> `cd..`  
> `cd marking`  
> `step1_v2.sh`  

Click Check my progress to verify the objective.

<hr>

##### Task 3. Push the Docker image to the Container Repository  

1. Push the Docker image `Image with Tag` into the Container Registry.
2. Make sure you re-tag the container to `gcr.io/GCP Project ID/Image with Tag`.

> `cd ~/valkyrie-app`  
>  
>  To list GCP project ID: `gcloud config list project`  
>  `docker tag [Image Name]:[Tag name] gcr.io/GCP Project ID/[Image Name]:[Tag name]`  
>  To verify: `docker images`  
>  `docker push gcr.io/GCP Project ID/[Image Name]:[Tag name]`  
>  To verify image exists in GCR, navigate via GCP console to **Navigation Menu > Container Registry**. Or visit http://gcr.io/GCP Project ID/[Image Name]  

<hr>

##### Task 4. Create and expose a deployment in Kubernetes  
Kurt created the `deployment.yaml` and `service.yaml` to deploy your new container image to a Kubernetes cluster (called valkyrie-dev). The two files are in `valkyrie-app/k8s`.

1. Remember you need to get the Kubernetes credentials before you deploy the image onto the Kubernetes cluster.

> Change to `k8s` sub-directory  
> `cd k8s`  
>  
> Set default compute region to `us-east1`  
> `gcloud config set compute/region us-east1`  
> Set default compute zone to `us-east1-b`  
> `gcloud config set compute/zone us-east1-b`  
>   
> To verify that a cluster is running  
> `gcloud container clusters list`  
> Authenticate with cluster and get credentials to interact with it  
> `gcloud container clusters get-credentials valkyrie-dev`  
> To verify KE connection with cluster  
> `kubectl cluster-info`  

2. Before you create the deployments make sure you check the `deployment.yaml` and `service.yaml` files. Kurt thinks they need some values set (he thinks he left some placeholder values).

> `cat deployment.yaml` to view the file  
> `vi deployment.yaml` to open file with vi, the Unix text editor.  
> `i` to enter text entry mode  
> Replace placeholder values with needed values  
> Press 'ESC', followed by `:wq <Enter>` to write, then quit  
> `cat deployment.yaml` again to check if placeholder valued replaced.  
>  
> `cat service.yaml` to view the file  
> Should see no changes needed.  
>   
> `kubectl create` deployment object from configuration file  
> `kubectl create -f deployment.yaml`  
> To verify: `kubectl get deployments`. Should get 1/1 READY, 1 AVAILABLE.  
>  
> `kubectl create` service object from configuration file  
> `kubectl create -f service.yaml`  
> To verify: `kubectl get services`. Wait for External-IP generation.  

3. You can check the load balancer once it’s available.

> Open a new browser tab and enter the following address: `http://[external-IP]:8080`  
> Or `curl http://[external-IP]:8080`  

Click Check my progress to verify the objective.

<hr>
  
##### Task 5. Update the deployment with a new version of valkyrie-app
1. Before deploying the new code, increase the replicas from 1 to `Replicas count` to ensure you don't cause an outage.

Click Check my progress to verify the objective.

Kurt made changes to the source code (he put the changes in a branch called kurt-dev).

2. You need to merge kurt-dev into master (you should use git merge origin/kurt-dev).

> `cd ..` to change directory to valkyrie-app  
> `git merge origin/kurt-dev`  

3. Build the new code as version `Updated Version` of valkyrie-app, push the updated image to the Container Repository, and then redeploy to the valkyrie-dev cluster. You will know you have the new `Updated Version` version because the titles for the cards will be green.

> `docker build -t gcr.io/GCP Project ID/[Image Name]:[Updated Version] .`  
> To verify: `docker images`  
> `docker push gcr.io/GCP Project ID/[Image Name]:[Updated Version]`  
> To verify new version of image exists in GCR, navigate via GCP console to **Navigation Menu > Container Registry**. Or visit http://gcr.io/GCP Project ID/[Image Name]  
>   
> Update replica count and image version in deployment object  
> `kubectl edit deployment [deployment name]`  
> `i` to enter text entry mode.  
> Update the replica count and version of image  
> Press 'ESC", then `:wq` to write, save then quit  
> Once updated deployment object saved to cluster, Kubernetes will begin rolling update.  
> To verify: `kubectl get replicaset`. `kubectl rollout history deployment/[deployment name]` will show new entry.  

Click Check my progress to verify the objective.

<hr> 

##### Task 6. Create a pipeline in Jenkins to deploy your app
This process of building the container and pushing to the container repository can be automated using Jenkins. There is a Jenkins deployment in your `valkyrie-dev` cluster - connect to Jenkins and configure a job to build when you push a change to the source code.

1. Remember with Jenkins:
  - Get the password with `printf $(kubectl get secret cd-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo`

  - Connect to the Jenkins console using the commands below (but make sure you don't have a running container `docker ps`; if you do, kill it):
```
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/component=jenkins-master" -l "app.kubernetes.io/instance=cd" -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward $POD_NAME 8080:8080 >> /dev/null &
```
> Run `docker ps` to list all running containers  
> To stop running containers, `docker stop/kill [container_id]`  
> To remove containers, `docker rm [container_id]`  
>    
> Check if there is a Jenkins pod in the RUNNING state and if the container is READY.  
> `kubectl get pods`  
>  
> Setup port-forwarding from Cloud Shell to Jenkins UI to access Jenkins console.  
> ```  
> export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/component=jenkins-master" -l "app.kubernetes.io/instance=cd" -o jsonpath="{.items[0].metadata.name}")
> kubectl port-forward $POD_NAME 8080:8080 >> /dev/null &
> ```  
> Get the password 
> `printf $(kubectl get secret cd-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo`
> Note the password output to terminal

- Setup your credentials to use **Google Service Account from metadata**.

> This is to allow Jenkins to access Code Source Repository using cluster's service account credentials.  
> Step 1: Open Cloud Shell Preview at port 8080. Log in to Jenkins as 'admin' with password from terminal output in previous step   
> In Jenkins user interface, click **Manage Jenkins** in the left navigation then click **Manage Credentials**.  
> Step 2: Click **Jenkins**.  
> Step 3: Click **Global credentials (unrestricted**).  
> Step 4: Click **Add Credentials** in the left navigation.  
> Step 5: Select **Google Service Account from metadata** from the **Kind** drop-down and click **OK**.  
>The global credentials has been added. The name of the credential is the `Project ID` found in the `Connection Details` section of the lab.  
 
- Configure Jenkins Cloud for Kubernetes
> Step 1: In the Jenkins user interface, select **Manage Jenkins > Manage nodes and clouds**.  
> Step 2: Click **Configure Clouds** in the left navigation pane.  
> Step 3: Click **Add a new cloud** and select **Kubernetes**.  
> Step 4: Click **Kubernetes Cloud Details**.  
> Step 5: In the **Jenkins URL** field, enter the following value: http://cd-jenkins:8080  
> Step 6: In the **Jenkins tunnel** field, enter the following value: cd-jenkins-agent:50000  
> Step 7: Click Save.  

- Create a pipeline job that points to your */master branch on your source code.
> Navigate to your Jenkins user interface and follow these steps to configure a Pipeline job.  
> Step 1: Click **Dashboard > New Item** in the left panel.   
> Step 2: Name the project **valkyrie-app**, then choose the **Pipeline** option and click **OK**.  
> Step 3: On the next page, under Pipeline Definition, select **Pipeline script from SCM** and select **Git** for SCM.  
> Step 4: Add the source repo URL (find using command: gcloud source repos list)  
> Step 5: From the Credentials drop-down, select the qwiklabs accunt.  
> Step 6: Leave Branches to build - Branch Specifier as ***/master**  
> Step 7: Click Save leaving all other options with their defaults.  

2. Make two changes to your files before you commit and build:

- Edit valkyrie-app/Jenkinsfile and change YOUR_PROJECT to your actual project id.

> Check that you are in valkyrie-app folder.  
> `vi Jenkinsfile` to open Jenkinsfile using vi text editor, then `i` to insert text   
> Replace "YOUR-PROJECT' with Project ID from Connection Panel.  
> Press 'ESC' then `:wq`.  

- Edit valkyrie-app/source/html.go and change the two occurrences of green to orange.
> cd valkyrie-app/source  
> `cd source`  
> `vi html.go`, then `i`  
> Change the two instance of green to orange.  
> Press 'ESC', then `:wq`  

Use git to:
- Add all the changes then commit those changes to the master branch.
> `cd ..` to switch back to valkyrie-dev directory  
> `git config --global user.name "abc"`  
> `git config --global user.email "abc@yahoo.com"`  
> `git add .`  
> `git commit -m "color change green to orange"`  
- Push the changes back to the repository.
> `git push`  

When you are ready, manually trigger a build (the initial build will take some time, so just monitor the process). The build will replace the running containers with containers with different tags; you will see orange colored headings.

> In Jenkins console, in Pipeline valkyrie-app, click **Build Now**. Can click on **Console Output** and **Status** to view progress.   
>  
> If pipeline is not building, it may be an issue with [kubectl proxy](https://kubernetes.io/docs/tasks/extend-kubernetes/http-proxy-access-api/). The proxy authenticates itself with the Kubernetes API and proxies requests from your local machine to the service in the cluster without exposing your service to the Internet.   
> Start the proxy in the background `kubectl proxy &`  
> 
> Video:  
> https://drive.google.com/file/d/1dFNaCIajzYRIMZVMoCEUJIqKWnDch7dX/view
