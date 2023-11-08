## Cloud Architecture: Design, Implement, and Manage
### Google Cloud Essential Skills: Challenge Lab

#### Challenge scenario
Your company is ready to launch a brand new product! Because you are entering a totally new space, you have decided to deploy a new website as part of the product launch. The new site is complete, but the person who built the new site left the company before they could deploy it.

#### Your challenge
Your challenge is to deploy the site in the public cloud by completing the tasks below. You will use a simple Apache web server as a placeholder for the new site in this exercise. Good luck!

#### Running a basic Apache web server
A virtual machine instance on Compute Engine can be controlled like any standard Linux server. Deploy a simple Apache web server (a placeholder for the new product site) to learn the basics of running a server on a virtual machine instance.

<hr>

### Task 1. Create a Linux VM instance
Create a Linux virtual machine, name it `Instance name` and specify the zone as `Compute zone`.  
👇👇👇  
```
gcloud compute instances create [Instance name] --zone [Compute zone] --tags=http-server
```
To verify, from Cloud console > Compute Engine > VM instances > check for [Instance name] entry.

### Task 2. Enable public access to VM instance
While creating the Linux instance, make sure to apply the appropriate firewall rules so that potential customers can find your new product.   
👇👇👇   
```
gcloud compute firewall-rules create [FW-name] --action=ALLOW --rules=tcp:80 --target-tags http-server
```
To verify, from Cloud console > VPC network > Firewall


### Task 3. Running a basic Apache Web Server
A virtual machine instance on Compute Engine can be controlled like any standard Linux server.

Deploy a simple Apache web server (a placeholder for the new product site) to learn the basics of running a server on a virtual machine instance.   
👇👇👇   
1. Create a startup script and store it as a local file. 
```
nano startup-script.sh
```
Paste the following into `startup-script.sh`
```
#! /bin/bash
apt update
apt -y install apache2
cat <<EOF > /var/www/html/index.html
<html><body><p>"Hello World!"</p></body></html>
EOF
```
`CTRL + X` to exit. `Y + ENTER` to save.

To verify, `cat startup-script.sh`.

2. Pass the startup script as metadata to VM.
```
gcloud compute instances add-metadata [Instance name] --zone [Compute zone] --metadata-from-file startup-script=[File-Path]
```

Reference:  
[How to pass Linux Startup script from local file](https://cloud.google.com/compute/docs/instances/startup-scripts/linux#passing-local)  

3. Stop and restart the VM instance
- From Cloud console > Compute Engine > VM instances > click the 3 vertical dots at the end of the [Instance name] for "More actions".
- Select "Stop"
- After the VM has stopped, select "Start / Resume".

### Task 4. Test your server
Test that your instance is serving traffic on its external IP.
You should see the "Hello World!" page (a placeholder for the new product site).   
👇👇👇   
Copy the "External IP" of the VM instance from the Cloud console.
```
http://[External_IP of VM]
```

<hr>

#### Troubleshooting
**Receiving a Connection Refused error:**
- Your VM instance is not publicly accessible because the VM instance does not have the proper tag that allows Compute Engine to apply the appropriate firewall rules, or your project does not have a firewall rule that allows traffic to your instance's external IP address.
- You are trying to access the VM using an https address. Check that your URL is `http:// EXTERNAL_IP` and not `https:// EXTERNAL_IP`.
