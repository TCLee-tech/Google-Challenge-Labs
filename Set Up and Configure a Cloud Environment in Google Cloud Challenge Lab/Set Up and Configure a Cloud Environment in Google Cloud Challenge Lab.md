# Set Up and Configure a Cloud Environment in Google Cloud: Challenge Lab

### Challenge scenario
As a cloud engineer in Jooli Inc. and recently trained with Google Cloud and Kubernetes, you have been asked to help a new team (Griffin) set up their environment. The team has asked for your help and has done some work, but needs you to complete the work.

You are expected to have the skills and knowledge for these tasks so don’t expect step-by-step guides.

You need to complete the following tasks:

* Create a development VPC with three subnets manually
* Create a production VPC with three subnets manually
* Create a bastion that is connected to both VPCs
* Create a development Cloud SQL Instance and connect and prepare the WordPress environment
* Create a Kubernetes cluster in the development VPC for WordPress
* Prepare the Kubernetes cluster for the WordPress environment
* Create a WordPress deployment using the supplied configuration
* Enable monitoring of the cluster via Cloud Monitoring
* Provide access for an additional engineer

Some Jooli Inc. standards you should follow:

* Create all resources in the **us-east1** region and **us-east1-b** zone, unless otherwise directed.

* Use the project VPCs.

* Naming is normally *team-resource*, e.g. an instance could be named **kraken-webserver1**.

* Allocate cost effective resource sizes. Projects are monitored and excessive resource use will result in the containing project's termination (and possibly yours), so beware. This is the guidance the monitoring team is willing to share: unless directed, use **e2-medium**.

### Your challenge
You need to help the team with some of their initial work on a new project. They plan to use WordPress and need you to set up a development environment. Some of the work was already done for you, but other parts require your expert skills.

As soon as you sit down at your desk and open your new laptop you receive the following request to complete these tasks. Good luck!

##### Environment
![overview](https://github.com/TCLee-tech/Google-Challenge-Labs/blob/f9f78b3aa2ba9f7cd8cdee9e9a0c334448b4d1aa/Set%20Up%20and%20Configure%20a%20Cloud%20Environment%20in%20Google%20Cloud%20Challenge%20Lab/overview.png)

<hr>

### Task 1. Create development VPC manually
Create a VPC called *griffin-dev-vpc* with the following subnets only:

* *griffin-dev-wp*
  * IP address block: 192.168.16.0/20
* *griffin-dev-mgmt*
  * IP address block: 192.168.32.0/20

##### 🔴 Solution:  
Create a custom mode VPC. Only customs mode VPCs start with no subnets, giving you full control. Auto mode VPCs are created with one subnet per region.  
If needed, `gcloud config set project [GCP Project ID]`  
```
gcloud compute networks create griffin-dev-vpc --subnet-mode custom  
gcloud compute networks subnets create griffin-dev-wp --network=griffin-dev-vpc --region=us-east1 --range=192.168.16.0/20  
gcloud compute networks subnets create griffin-dev-mgmt --network=griffin-dev-vpc --region=us-east1 --range=192.168.32.0/20
```

To verify,
- `gcloud compute networks list`
- `gcloud compute networks subnets list --sort-by=NETWORK`.
- In the Cloud console, nagivate to **Navigation menu > VPC network > VPC networks**  

<hr>

### Task 2. Create production VPC manually
Create a VPC called *griffin-prod-vpc* with the following subnets only:

* *griffin-prod-wp*
  * IP address block: 192.168.48.0/20
* *griffin-prod-mgmt*
  * IP address block: 192.168.64.0/20

##### 🔴 Solution:  
Create a custom mode VPC.  
```
gcloud compute networks create griffin-prod-vpc --subnet-mode custom  
gcloud compute networks subnets create griffin-prod-wp --network=griffin-prod-vpc --region=us-east1 --range=192.168.48.0/20  
gcloud compute networks subnets create griffin-prod-mgmt --network=griffin-prod-vpc --region=us-east1 --range=192.168.64.0/20
```
  
To verify,  
- `gcloud compute networks list`  
- `gcloud compute networks subnets list --sort-by=NETWORK`  
- In the Cloud console, nagivate to **Navigation menu > VPC network > VPC networks**  

<hr>

### Task 3. Create bastion host
Create a bastion host with two network interfaces, one connected to *griffin-dev-mgmt* and the other connected to *griffin-prod-mgmt*. Make sure you can SSH to the host.

##### 🔴 Solution:
A bastion host is a server (VM) used to access internal private networks from outside. Connection to the bastion host is normally by SSH.
```
gcloud compute instances create griffin-bastion --zone=us-east1-b --machine-type=e2-medium --network-interface=network=griffin-dev-vpc,subnet=griffin-dev-mgmt --network-interface=network=griffin-prod-vpc,subnet=griffin-prod-mgmt
```
References:  
- [Google bastion host](https://cloud.google.com/compute/docs/connect/ssh-using-bastion-host#gcloud_1)   
- [gcloud compute instances create](https://cloud.google.com/sdk/gcloud/reference/compute/instances/create)  

Create a firewall rules to allow bastion host VM to accept SSH connections from internet:
```
gcloud compute firewall-rules create griffin-dev-allow-ssh --network griffin-dev-vpc --allow tcp:22
gcloud compute firewall-rules create griffin-prod-allow-ssh --network griffin-prod-vpc --allow tcp:22
```
References:  
- [gcloud compute firewall-rules create](https://cloud.google.com/sdk/gcloud/reference/compute/firewall-rules/create)  
- tcp protocol, port 22 is Google's default allow for ssh connections.  

<hr>

### Task 4. Create and configure Cloud SQL Instance
Create a **MySQL Cloud SQL Instance** called *griffin-dev-db* in *us-east1*.

Connect to the instance and run the following SQL commands to prepare the **WordPress** environment:

CREATE DATABASE wordpress;
CREATE USER "wp_user"@"%" IDENTIFIED BY "stormwind_rules";
GRANT ALL PRIVILEGES ON wordpress.* TO "wp_user"@"%";
FLUSH PRIVILEGES;

These SQL statements create the worpdress database and create a user with access to the wordpress database.

You will use the username and password in task 6.

##### 🔴  Solution (a): Create a Cloud SQL Instance
- In the Google Cloud Console, navigate to **Navigation menu > SQL**
- Click **CREATE INSTANCE > Choose MySQL**
- Enter Cloud SQL instance ID as **griffin-dev-db**
- Enter a secure password in the **Password** field (remmeber it!).
- Select the database version as **MySQL 8**
- To **Choose a Cloud SQL edition**, choose **Development** (4vCPU, 16 GB RAM, 100 GB Storage, single zone) for **Enterprise** edition.
- Select the **Multi zones (Highly available)** field for **us-east1 (South Carolina)** region.
- Click **CREATE INSTANCE**
- It takes a few minutes for the instance to be created. Once it is ready, you will see a green checkmark next to the instance name.
- Click on the Cloud SQL instance. The **SQL Overview** page opens.

If you are impatient, you can create the Kubernetes cluster (Task 5) in Cloud Shell while waiting for the Cloud SQL instance to be created.

##### 🔴 Solution (b): Create WordPress database and user
- Run the following command to connect to the Cloud SQL instance: `gcloud sql connect griffin-dev-db --user=root --quiet`
- When prompted, enter the root password.
- At the SQL prompt, run the following SQL commands to create the WordPress database and user:
```
CREATE DATABASE wordpress;
CREATE USER "wp_user"@"%" IDENTIFIED BY "stormwind_rules";
GRANT ALL PRIVILEGES ON wordpress.* TO "wp_user"@"%";
FLUSH PRIVILEGES;
```
- **exit** to close the SQL prompt.

<hr>

### Task 5. Create Kubernetes cluster
Create a 2 node cluster (e2-standard-4) called *griffin-dev*, in the *griffin-dev-wp* subnet, and in zone *us-east1-b*.

##### 🔴 Solution:
```
gcloud container clusters create griffin-dev --num-nodes=2 --machine-type=e2-standard-4 --network=griffin-dev-vpc --subnetwork=griffin-dev-wp --zone=us-east1-b
```
- To verify,
  - `gcloud container clusters list` or
  - **Navigation menu > Kubernetes Engine > Clusters**

- `gcloud container clusters get-credentials griffin-dev --zone us-east1-b`

<hr>

### Task 6. Prepare the Kubernetes cluster
Use Cloud Shell and copy all files from *gs://cloud-training/gsp321/wp-k8s*.
The **WordPress** server needs to access the MySQL database using the *username* and *password* you created in task 4.

You do this by setting the values as secrets. **WordPress** also needs to store its working files outside the container, so you need to create a volume.

Add the following secrets and volume to the cluster using *wp-env.yaml*.

Make sure you configure the *username* to *wp_user* and *password* to *stormwind_rules* before creating the configuration.

You also need to provide a key for a service account that was already set up. This service account provides access to the database for a sidecar container.

Use the command below to create the key, and then add the key to the Kubernetes environment:
```
gcloud iam service-accounts keys create key.json \
    --iam-account=cloud-sql-proxy@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
kubectl create secret generic cloudsql-instance-credentials \
    --from-file key.json
```

##### 🔴 Solution:
- `gsutil -m cp -r gs://cloud-training/gsp321/wp-k8s .`
- `cd wp-k8s`
- To configure username to *wp_user* and password to *stormwind_rules*, you can use sed command:
- ```
  sed -i s/username_goes_here/wp_user/g wp-env.yaml
  sed -i s/password_goes_here/stormwind_rules/g wp-env.yaml
  ```

- alternatively, you can use nano editor:
  - `nano wp-env.yaml`
  - replace *username_goes_here* with *wp-user* and *password_goes_here* with *stormwind_rules*
  - `CTRL+O` to save, then `CTRL+X` to exit.  

- or click on "Open Editor" in Cloud Shell, and edit wp-env.yaml

- If you want to verify changes made, `cat wp-env.yaml`
- `kubectl create -f wp-env.yaml`
- To create key for service account, and add to Kubernetes environment:
```
gcloud iam service-accounts keys create key.json \
    --iam-account=cloud-sql-proxy@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
kubectl create secret generic cloudsql-instance-credentials \
    --from-file key.json
```

Note:
- [sed stream editor commands](https://www.geeksforgeeks.org/sed-command-in-linux-unix-with-examples/)
- -i argument for in-place editing
- s/original string or regex/replacement [input file name]: substitute
- g: global flag, replace all occurences
- wp-env.yaml creates a PersistentVolumeClaim and a Secret object.

<hr>

### Task 7. Create a WordPress deployment
Now that you have provisioned the MySQL database, and set up the secrets and volume, you can create the deployment using *wp-deployment.yaml*.

Before you create the deployment you need to edit *wp-deployment.yaml*.

Replace **YOUR_SQL_INSTANCE** with griffin-dev-db's **Instance connection name**.

Get the **Instance connection name** from your Cloud SQL instance.

After you create your WordPress deployment, create the service with *wp-service.yaml*.

Once the Load Balancer is created, you can visit the site and ensure you see the WordPress site installer.
At this point the dev team will take over and complete the install and you move on to the next task.

##### 🔴 Solution:
```
sed -i s/YOUR_SQL_INSTANCE/$(gcloud sql instances describe griffin-dev-db --format="value(connectionName)")/g wp-deployment.yaml
kubectl create -f wp-deployment.yaml
kubectl create -f wp-service.yaml
```
To verify, 
- `kubectl get deployments`
- Either:
  - In Google Cloud Console, go to Kubernetes Engine -> Services and Ingress ->
copy wordpress External load balancer Endpoints' IP address.  
  - `kubectl get services` and get the EXTERNAL-IP for wordpress LoadBalancer.
  - Open the external IP address in a new browser tab and ensure you see the WordPress site installer.

Reference:
- [How to view info on Cloud SQL instance](https://cloud.google.com/sql/docs/mysql/instance-info)

<hr>

### Task 8. Enable monitoring
Create an uptime check for your WordPress development site.

##### 🔴 Solution:
An uptime check verifies that the WordPress development site is up and available.
- In Google Cloud console, navigate to **Navigation menu > Monitoring**
- In the left pane of Cloud Monitoring, click on **Uptime checks**, and then click **Create Uptime Check**.
- For **Protocol**, leave as **HTTP**.
- For **Resource Type**, leave as **URL**.
- For **Hostname**, paste the External-IP of the wordpress LoadBalancer.
- For **Path**, enter **/**.
- Click **Continue**.
- In Response Validation, accept the defaults and then click **Continue**.
- In Alert & Notification, accept the defaults, and then click **Continue**.
- For Title, type **WPress Uptime Check**.
- Click **Test** to verify that your uptime check can connect to the resource.
- When you see a green check mark everything can connect.
- Click **Create**.

<hr>

### Task 9. Provide access for an additional engineer
You have an additional engineer starting and you want to ensure they have access to the project, so please go ahead and grant them the editor role to the project.
The second user account for the lab represents the additional engineer.

##### 🔴 Solution:
- In the Google Cloud console, navigate to **Navigation menu > IAM & Admin > IAM**.
- Search for the second user (ID starts with student-xx-, allocated at start of lab) with "Viewer" role.
- Click on pencil icon at end of row to edit.
- In the pop-up to edit access to "qwiklabs-gcp-xxx" project for this Principal(student-xx-), select **Editor** Role under **Assign roles**.
- Click **Save**.
- Notice the change in role for this second user.
- If you sign in to the second user account in a new browser tab, you will be able to change states.
