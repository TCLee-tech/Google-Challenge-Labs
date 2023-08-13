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
* Enable monitoring of the cluster via stackdriver
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

> Solution:  
> Create a custom mode VPC. Only customs mode VPCs start with no subnets, giving you full control. Auto mode VPCs are created with one subnet per region.  
> `gcloud compute networks create griffin-dev-vpc --subnet-mode custom`  
> `gcloud compute networks subnets create griffin-dev-wp --network=griffin-dev-vpc --region=us-east1 --range=192.168.16.0/20`  
> `gcloud compute networks subnets create griffin-dev-mgmt --network=griffin-dev-vpc --region=us-east1 --range=192.168.32.0/20`  
> To verify,
>   * `gcloud compute networks list`
>   * `gcloud compute networks subnets list --sort-by=NETWORK`.
>   * In the Cloud console, nagivate to **Navigation menu > VPC network > VPC networks**  

<hr>

### Task 2. Create production VPC manually
Create a VPC called *griffin-prod-vpc* with the following subnets only:

* *griffin-prod-wp*
  * IP address block: 192.168.48.0/20
* *griffin-prod-mgmt*
  * IP address block: 192.168.64.0/20

> Solution:  
> Create a custom mode VPC.  
> `gcloud compute networks create griffin-prod-vpc --subnet-mode custom`  
> `gcloud compute networks subnets create griffin-prod-wp --network=griffin-prod-vpc --region=us-east1 --range=192.168.48.0/20`  
> `gcloud compute networks subnets create griffin-prod-mgmt --network=griffin-prod-vpc --region=us-east1 --range=192.168.64.0/20`  
> To verify,  
>   * `gcloud compute networks list`  
>   * `gcloud compute networks subnets list --sort-by=NETWORK`.  
>   * In the Cloud console, nagivate to **Navigation menu > VPC network > VPC networks**  

<hr>

### Task 3. Create bastion host
Create a bastion host with two network interfaces, one connected to *griffin-dev-mgmt* and the other connected to *griffin-prod-mgmt*. Make sure you can SSH to the host.

