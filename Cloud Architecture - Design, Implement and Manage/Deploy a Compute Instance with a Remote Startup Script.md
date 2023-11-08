# Deploy a Compute Instance with a Remote Startup Script

### Challenge scenario
You have been given the responsibility of managing the configuration of your organization's Google Cloud virtual machines. You have decided to make some changes to the framework used for managing the deployment and configuration machines - you want to make it easier to modify the startup scripts used to initialize a number of the compute instances. Instead of storing startup scripts directly in the instances' metadata, you have decided to store the scripts in a Cloud Storage bucket and then configure the virtual machines to point to the relevant script file in the bucket.

A basic bash script that installs the Apache web server software called `install-web.sh` has been provided for you as a sample startup script. You can download this from the Student Resources links on the left side of the page.

```
#!/bin/bash
apt-get update
apt-get install -y apache2
```

### Your challenge
Configure a Linux Compute Engine instance that installs the Apache web server software using a remote startup script. In order to confirm that a compute instance Apache has successfully installed, the Compute Engine instance must be accessible via HTTP from the internet.

**Note:** In order to ensure accurate activity tracking you should not modify or change any of the pre-created lab resources, in particular the lab-monitor Compute Engine instance.

GCP Zone to Use: Placeholder value.

### Tips and Tricks
- **Configure Instance Metadata.** The [Running Startup Scripts](https://cloud.google.com/compute/docs/instances/startup-scripts) documentation page explains how Compute Engine instance metadata can be used to configure startup scripts.
- **Check if your Compute Engine instance is executing the startup script.** Use the Serial Console for the running virtual machine to look at the startup events to make sure that the startup script is being executed.
- **Check permissions.** Your Compute Engine instance might not have the correct permissions required to read the startup script from the storage bucket. The virtual machine needs to be given permissions that align with the storage permissions.
- **Check firewalls.** If the startup script has installed the software you may be unable to connect if a firewall has not been correctly configured.
- **Check the URL and address.** You will be unable to connect to the Apache web server if you are trying to access the Compute Engine instance using an HTTPS address rather than HTTP; or you are using the incorrect IP address. Check that your URL is http://[EXTERNAL_IP] rather than https://[EXTERNAL_IP] or http://[INTERNAL_IP]

<hr>

### Set default compute region and zone in local client (Cloud Shell)
```
gcloud config set compute/region [REGION, e.g. us-west1]
gcloud config set compute/zone [ZONE, e.g. us-west1-c]
```

### 1. Create a Cloud Storage bucket to store the virtual machine (VM) startup script
Activate Cloud Shell. In Cloud Shell,
```
gcloud storage buckets create gs://[Project ID]
```
Verify bucket creation from Cloud console > Cloud Storage > Buckets.

References:
[gcloud CLI tool](https://cloud.google.com/storage/docs/discover-object-storage-gcloud)
[gcloud commands](https://cloud.google.com/sdk/gcloud/reference/storage)

### 2. Store `install-web.sh` on local machine or in Cloud Shell.
To create a bash file (.sh) in Cloud Shell,
```
nano install-web.sh
```
Cut and paste content of script. `CTRL + X` to exit nano editor, then `Y` to save.
To verify, `cat install-web.sh`.

### Upload `install-web.sh` to Cloud Storage bucket.
syntax: gcloud storage cp [src] [dest]
```
gcloud storage cp install-web.sh gs://[Project ID]
```
To verify, Cloud console > Cloud Storage > Buckets > click on [Project ID] bucket and confirm that there is a `install-web.sh` object.
Click on the `install-web.sh` object to get its "gsutil URI" under LIVE OBJECT. "gsutil URI" syntax is `gs://[Project ID]/install-web.sh`

### Create VM instance
```
gcloud compute instances create [VM name] \
  --image-project=debian-cloud \
  --image-family=debian-11 \    //default
  --tags=http-server \   // to apply network firewall rules and routes to specific VMs
  --scopes=storage-ro \
  --metadata=startup-script-url=[cloud_storage_url]
```

gcloud compute instances create [VM-name] \
    --project=[Project ID] \
    --zone=us-west1-c \
    --machine-type=e2-medium \
    --network-interface=network-tier=PREMIUM,stack-type=IPV4_ONLY,subnet=default \
    --metadata=startup-script-url=gs://qwiklabs-gcp-02-dbe9b38c009c/install-web.sh,enable-oslogin=true \
    --maintenance-policy=MIGRATE \
    --provisioning-model=STANDARD \
    --service-account=470345287476-compute@developer.gserviceaccount.com \
    --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append \
    --tags=http-server \
    --create-disk=auto-delete=yes,boot=yes,device-name=instance-2,image=projects/debian-cloud/global/images/debian-11-bullseye-v20231010,mode=rw,size=10,type=projects/qwiklabs-gcp-02-dbe9b38c009c/zones/us-west1-c/diskTypes/pd-balanced \
    --no-shielded-secure-boot \
    --shielded-vtpm \
    --shielded-integrity-monitoring \
    --labels=goog-ec-src=vm_add-gcloud \
    --reservation-affinity=any

To verify, Cloud console > Compute Engine > VM instances > check that [VM name] has been created.
To make sure that the startup script was executed, view the serial port output
```
gcloud compute instances get-serial-port-output [VM name] \
  --zone [ZONE]
```
Reference:
[gcloud compute instances create command](https://cloud.google.com/sdk/gcloud/reference/compute/instances/create#--scopes)
[Viewing serial port output](https://cloud.google.com/compute/docs/troubleshooting/viewing-serial-port-output#viewing_serial_port_output)

### Allow HTTP ingress to VM instance.
- Every network has 2 implied firewall rules - block all ingress & allow all egress traffic.
- To access VM by HTTP from internet, we need to configure firewall to allow HTTP ingress to specific VM.
```
gcloud compute firewall-rules create [rule_name] \
  --action=ALLOW \
  --rules=tcp:80 \  
  --target-tags http-server  // best practice to define target VMs
```
- the follow flags are default and can be excluded
> --network=default
> --direction=INGRESS
> --source-ranges default='0.0.0.0/0'
> --destination-ranges default='0.0.0.0/0'

Reference:
[Firewall rules overvieew](https://cloud.google.com/firewall/docs/firewalls)
[Firewall rules commands](https://cloud.google.com/firewall/docs/using-firewalls)
[Firewall rules syntax](https://cloud.google.com/sdk/gcloud/reference/compute/firewall-rules/create)

For this lab, I used the default Compute Engine service account, which has the basic Editor role. It is a best practice, according to the principle of least privilege, to modify this service account's role or create a user-managed service account and restrict it to the IAM role for Cloud Storage access.

Reference:
[What is a service account?](https://cloud.google.com/compute/docs/access/service-accounts#accesscopesiam)


gcloud storage buckets add-iam-policy-binding gs://[project ID] --member=serviceAccount:[email address] --role=roles/storage.objectViewer

[gcloud storage buckets add-iam-policy-binding](https://cloud.google.com/sdk/gcloud/reference/storage/buckets/add-iam-policy-binding)