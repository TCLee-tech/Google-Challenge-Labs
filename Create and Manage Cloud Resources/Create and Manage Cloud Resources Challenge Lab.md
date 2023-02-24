# Create and Manage Cloud Resources: Challenge Lab

### Challenge scenario
You have started a new role as a Junior Cloud Engineer for Jooli, Inc. You are expected to help manage the infrastructure at Jooli. Common tasks include provisioning resources for projects.

You are expected to have the skills and knowledge for these tasks, so step-by-step guides are not provided.

Some Jooli, Inc. standards you should follow:

**Create all resources in the default region or zone**, unless otherwise directed.

Naming normally uses the format team-resource; for example, an instance could be named **nucleus-webserver1**.

Allocate cost-effective resource sizes. Projects are monitored, and excessive resource use will result in the containing project's termination (and possibly yours), so plan carefully. This is the guidance the monitoring team is willing to share: unless directed, use **f1-micro** for small Linux VMs, and use **n1-standard-1** for Windows or other applications, such as Kubernetes nodes.

Your challenge
As soon as you sit down at your desk and open your new laptop, you receive several requests from the Nucleus team. Read through each description, and then create the resources.

### Task 1. Create a project jumphost instance
You will use this instance to perform maintenance for the project.

Requirements:

Name the instance `Instance name`.  
Use an `f1-micro` machine type.  
Use the default image type (Debian Linux).  

**Solution**
1. Set default region and zone. This is a best practice.  
  `gcloud config set compute/region [region, e.g. us-central1]`  
  `gcloud config set compute/zone [zone, e.g. us-central1-a]`  
    - check the default region and zone for this Challenge Lab from Task 2
    - If you leave out this step, Google Cloud may use the region/zone nearest to your geographical location. These may not be the ones needed for progress in this Challenge Lab.    
2. To verify, `gcloud config get-value compute/zone`  
3. [gcloud compute instance create](https://cloud.google.com/sdk/gcloud/reference/compute/instances/create)  
`gcloud compute instances create [Instance-name-assigned-during-lab] --machine-type=f1-micro`   
    - can configure using Google Cloud Console and get gcloud command line from Console

<hr>

### Task 2. Create a Kubernetes service cluster
The team is building an application that will use a service running on Kubernetes. You need to:

* Create a zonal cluster using [zone-given-at-start-of-lab].
* Use the Docker container hello-app (`gcr.io/google-samples/hello-app:2.0`) as a placeholder; the team will replace the container with their own work later.
* Expose the app on port `App port number`.

**Solution**
1. [Create GKE cluster](https://cloud.google.com/sdk/gcloud/reference/container/clusters/create)  
`gcloud container clusters create nucleus-cluster --machine-type=n1-standard-1 --zone=[assigned-at-start-of-lab]`  
2. [Get authentication credentials and endpoint](https://cloud.google.com/sdk/gcloud/reference/container/clusters/get-credentials) to interact with cluster.  
`gcloud container clusters get-credentials nucleus-cluster --zone=[assigned-at-start-of-lab]`  
3. [Deploy](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) application  image to container using [kubectl create](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-deployment-em-).  
`kubectl create deployment hello-app  --image=gcr.io/google-samples/hello-app:2.0`  
To verify, `kubectl get deployment`  
4. Create a [Kubernetes Service](https://kubernetes.io/docs/concepts/services-networking/service/) using [kubectl expose](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#expose) to expose application to external traffic.  
`kubectl expose deployment hello-app --type=LoadBalancer --port=[app-port-number-assigned-during-lab]`  
To verify, `kubectl get services`.   
From a new browser window, visit http://[EXTERNAL-IP]:[port]   

<hr>

### Task 3. Set up an HTTP load balancer
You will serve the site via nginx web servers, but you want to ensure that the environment is fault-tolerant. Create an HTTP load balancer with a managed instance group of **2 nginx web servers**. Use the following code to configure the web servers; the team will replace this with their own configuration later.
```
cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF
```
You need to:
- Create an instance template.
- Create a target pool. 
- Create a managed instance group.
- Create a firewall rule named as `Firewall rule` to allow traffic (80/tcp).
- Create a health check.
- Create a backend service, and attach the managed instance group with named port (http:80).
- Create a URL map, and target the proxy to route requests to your URL map.
- Create a forwarding rule.

**Solution**
1. Create startup script `startup.sh`
```
cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF
```
2. [Create an instance template](https://cloud.google.com/sdk/gcloud/reference/compute/instance-templates/create)
```
gcloud compute instance-templates create nucleus-backend-template \
  --region=[get-from-Task2-during-lab] \
  --network=default \
  --machine-type=f1-micro \
  --metadata-from-file startup-script=startup.sh
```
3. Create target pool.  
`gcloud compute target-pools create nginx-pool`

4. Create a [Managed Instance Group](https://cloud.google.com/compute/docs/instance-groups)  
`gcloud compute instance-groups managed create nucleus-backend-group --template=nucleus-backend-template --size=2 --region=[allocated-at-start-of-lab]`

[Issue](https://cloud.google.com/load-balancing/docs/backend-service?&_ga=2.124613263.-405048375.1673775149#named_ports): For each instance group backend, must configure named ports using flag: `--named-ports=[key]:[value]`
  - mapping is for each instance group backend individually
  - different instance groups can have same port name, but different port numbers
  - when backend service communicates with different instance groups, will use different port number mapped
  - on the backend service, you specify a single named port using just the port name (`--port-name=xxxx`)
  - to make changes to created instance group backend: `gcloud compute instance-groups managed set-named-ports nucleus-backend-group --named-ports=http:80 --region=[assigned-during-lab]`
  - if need to update backend service with `--port-name`: `gcloud compute backend-services update nucleus-backend-service --port-name=http`

5. Create firewall rule  
   - source-ranges: from Google Cloud health checking system
```
gcloud compute firewall-rules create [name-of-firewall-rule-assigned-during-lab] \
  --network=default \
  --action=allow \
  --direction=ingress \
  --source-ranges=130.211.0.0/22,35.191.0.0/16 \
  --rules=tcp:80
```

6. Create [health check](https://cloud.google.com/load-balancing/docs/health-checks) for load balancer and VMs.  
`gcloud compute health-checks create http http-basic-check`

7. [Create a backend service](https://cloud.google.com/sdk/gcloud/reference/compute/backend-services/create).
```
gcloud compute backend-services create nucleus-backend-service \
  --protocol=HTTP \
  --port-name=http \
  --health-checks=http-basic-check \
  --global
```
8. Add Instance Group as backend to this backend service.
```
gcloud compute backend-services add-backend nucleus-backend-service \
  --instance-group=nucleus-backend-group \
  --instance-group-region=[assigned-during-lab] \
  --global
```
9. Create a [URL map](https://cloud.google.com/load-balancing/docs/url-map-concepts) to route incoming requests to correct backend service/bucket based on host and path rules.  
`gcloud compute url-maps create web-map-http --default-service=nucleus-backend-service`  
10. Create target HTTP proxy that will route requests based on URL map  
`gcloud compute target-http-proxies create http-nucleus-proxy --url-map=web-map-http`  
11. Create a [global forwarding rule](https://cloud.google.com/load-balancing/docs/using-forwarding-rules) to route incoming traffic to this proxy.
```
gcloud compute forwarding-rules create http-content-rule \
  --global \
  --target-http-proxy=http-nucleus-proxy \
  --ports=80
```
```
gcloud compute forwarding-rules create [firewall-rule-name-given-during-lab] \
  --global \
  --target-http-proxy=http-nucleus-proxy \
  --ports=80
```
To verify, `gcloud compute forwarding-rules list`

12. To verify entire Level 7 HTTP(S) Load Balancer, 
  - go to Cloud Console > `Navigation Menu` > `Network Services` > `Load balancing`.
  - in `Backend` section, click on the name of the backend and confirm that the VMs are `Healthy`.
  - Get the load balancer's IP address and test in a new browser tab http://IP_ADDRESS



