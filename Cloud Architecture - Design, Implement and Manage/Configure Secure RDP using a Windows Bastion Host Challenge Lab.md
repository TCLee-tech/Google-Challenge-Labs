## Cloud Architecture: Design, Implement, and Manage
### Configure Secure RDP using a Windows Bastion Host: Challenge Lab

### Challenge scenario
Your company has decided to deploy new application services in the cloud and your assignment is developing a secure framework for managing the Windows services that will be deployed. You will need to create a new VPC network environment for the secure production Windows servers.

Production servers must initially be completely isolated from external networks and cannot be directly accessed from, or be able to connect directly to, the internet. In order to configure and manage your first server in this environment, you will also need to deploy a bastion host, or jump box, that can be accessed from the internet using the Microsoft Remote Desktop Protocol (RDP). The bastion host should only be accessible via RDP from the internet, and should only be able to communicate with the other compute instances inside the VPC network using RDP.

Your company also has a monitoring system running from the default VPC network, so all compute instances must have a second network interface with an internal only connection to the default VPC network.

### Your challenge
Deploy the secure Windows machine that is not configured for external communication inside a new VPC subnet, then deploy the Microsoft Internet Information Server on that secure machine.

## Tasks:
The key tasks are listed below. Good luck!

- Create a new VPC network with a single subnet.
- Create a firewall rule that allows external RDP traffic to the bastion host system.
- Deploy two Windows servers that are connected to both the VPC network and the default network.
- Create a virtual machine that points to the startup script.
- Configure a firewall rule to allow HTTP access to the virtual machine.

<hr>

### Task 1. Create the VPC network
1. Create a new VPC network called `securenetwork`.

2. Then create a new VPC subnet inside `securenetwork`.

3. Once the network and subnet have been configured, configure a firewall rule that allows inbound RDP traffic (TCP port 3389) from the internet to the bastion host. This rule should be applied to the appropriate host using network tags.

Solution 👇👇👇  


- Set default compute region and zone in local client (Cloud Shell). Check [Region] and [placeholder zone] for lab from Task 2.
```
gcloud config set compute/region [Region]
gcloud config set compute/zone [placeholder zone]
```
To verify,
```
gcloud config get-value compute/region
gcloud config get-value compute/zone
```
Reference:  
[gcloud compute defaults](https://cloud.google.com/compute/docs/gcloud-compute)

- Create VPC network.
```
gcloud compute networks create securenetwork \
--subnet-mode=custom
```
To verify, `gcloud compute networks list`.  

- Create custom subnet in VPC network.  
```
gcloud compute networks subnets create securenetwork \
--network=securenetwork \
--range=20.0.0.0/20
```
To verify, `gloud compute networks subnetworks list`

References:
[Create and manage VPC networks](https://cloud.google.com/vpc/docs/create-modify-vpc-networks#add-subnets)

- Create firewall rule for inbound RDP traffic.
```
gcloud compute firewall-rules create [FW-rule-name] \
--action=ALLOW \
--direction=INGRESS \
--network=securenetwork \
--priority=1000 \
--rules=tcp:3389 \
--source-ranges=0.0.0.0/0 \
--target-tags bastion-host
```
To verify, Cloud console > VPC network > Firewall. Check for addition of [FW-rule-name] as configured.

[Allow ingress RDP connections to VMs](https://cloud.google.com/firewall/docs/using-firewalls#common-use-cases-allow-rdp)

<hr>

### Task 2. Deploy your Windows instances and configure user passwords
1. Deploy a Windows 2016 server instance called vm-securehost with two network interfaces.
2. Configure the first network interface with an internal only connection to the new VPC subnet, and the second network interface with an internal only connection to the default VPC network. This is the secure server.
3. Install a second Windows 2016 server instance called vm-bastionhost with two network interfaces.
4. Configure the first network interface to connect to the new VPC subnet with an ephemeral public (external NAT) address, and the second network interface with an internal only connection to the default VPC network. This is the jump box or bastion host.
5. After your Windows instances have been created, create a user account and reset the Windows passwords in order to connect to each instance.
6. The following gcloud command creates a new user called `app-admin` and resets the password for a host called `vm-bastionhost` located in the `placeholder zone`:
```
gcloud compute reset-windows-password vm-bastionhost --user app-admin --zone "placeholder"
```
7. Alternatively, you can force a password reset from the Compute Engine console. You will have to repeat this for the second host as the login credentials for that instance will be different.

Solution 👇👇👇   

- Create Windows Server VM instance `vm-securehost`
```
gcloud compute instances create vm-securehost \
--network-interface=network=securenetwork,subnet=securenetwork,no-address \
--network-interface=network=default,subnet=default,no-address \
--zone [placeholder zone] \
--image-project windows-cloud \
--image-family windows-2016
```

- Create Windows Server VM instance `vm-bastionhost`
```
gcloud compute instances create vm-bastionhost \
--network-interface=network=securenetwork,subnet=securenetwork \
--network-interface=network=default,subnet=default,no-address \
--zone [placeholder zone] \
--image-project windows-cloud \
--image-family windows-2016 \
--tags=bastion-host
```   
For `network-interface` flag, if `address` is not specified, defaults to "ephemeral IP".   
`tags` identify specific VM for firewall rule/route.      
To verify, check Cloud console > Compute Engine > VM instances for creation of both instances.    

- Create a user account and reset Windows password for `vm-securehost`
```
gcloud compute reset-windows-password vm-securehost --user app-admin --zone [placeholder zone]
```  
When prompted `Would you like to set or reset the password for [app-admin] (Y/n)?`, enter `y`.  
Note down the password generated. username: app-admin. This is needed for task 3.

- Create a user account and reset Windows password for `vm-bastionhost`
```
gcloud compute reset-windows-password vm-bastionhost --user app-admin --zone [placeholder zone]
```
When prompted `Would you like to set or reset the password for [app-admin] (Y/n)?`, enter `y`.  
Note down the password generated. username: app-admin. These are needed for task 3.  


Reference:  
[Create Windows Server instance](https://cloud.google.com/compute/docs/instances/windows/creating-managing-windows-instances#gcloud)
[Windows Server 2016 images](https://cloud.google.com/compute/docs/images/os-details#windows_server)

<hr>

### Task 3. Connect to the secure host and configure Internet Information Server
To connect to the secure host, you have to RDP into the bastion host first, and from there open a second RDP session to connect to the internal private network address of the secure host. A Windows Compute Instance with an external address can be connected to via RDP using the RDP button that appears next to Windows Compute instances in the Compute Instance summary page.

When connected to a Windows server, you can launch the Microsoft RDP client using the command `mstsc.exe`, or you can search for Remote Desktop Manager from the Start menu. This will allow you to connect from the bastion host to other compute instances on the same VPC even if those instances do not have a direct internet connection themselves.  

Solution 👇👇👇  
- In Cloud console > Compute Engine > VM instances > click on **RDP** to connect to `vm-bastionhost`
- Click on "Download the RDP file if you will be using a 3rd-party client." 
- Open Remote Desktop Connection, after downloading. Click "Connect".
- In the `Windows Security` screen to **Enter your credentials**, do not use the displayed username. Select **More choices** > **Use a different account**.  
- Enter the username `app-admin` and password for `vm-bastionhost` from `Task 2`.
- Clieck "Yes" to connect.
- In the Windows Server which is running on vm-bastionhost, type `mstsc.exe` or `remote desktop connection` in the seach bar at the bottom of the window, next to the Windows Start button. Select the `Remote Desktop Connection` app.
- In the `Windows Security` screen to **Enter your credentials**, do not use the displayed username. Select **More choices** > **Use a different account**.  
- Enter the username `app-admin` and password from `Task 2`.
- To configure Microsoft Internet Information Services (IIS) on vm-securehost:
    - Once connected to `vm-securehost`, in the `Server Manager` > `Dashboard`, click on `2. Add roles and features`.
    - Click `Next` for default `Installation type`
    - Click `Next` for `Server Selection` if vm-securehost is highlighted for `Select a server from the server pool`
    - **Tick the box** next to **Web Server (IIS)** and click `Add Features` in the pop-up `Add Roles and Features Wizard`.
    - Click `Next` for `Features`, `Web Server Role (IIS)` and `Role Services`.
    - Click `Install` on `Confirmation` page.
    - Click `Close` when feature installation is completed.

END OF LAB.


### Troubleshooting
**Unable to connect to the Bastion host:** Make sure you are attempting to connect to the external address of the bastion host. If the address is correct you may not be able to connect to the bastion host if the firewall rule is not correctly configured to allow TCP port 3389 (RDP) traffic from the internet, or your own system's public IP-address, to the network interface on the bastion host that has an external address. Finally, you might have issues connecting via RDP if your own network does not allow access to internet addresses via RDP. If everything else is definitely OK, you will need to talk to the owner of the network you are connected to the internet with to open up port 3389 or connect using a different network.   
**Unable to connect to the Secure Host from the Bastion host:** If you can successfully connect to the bastion host but are unable to make the internal RDP connection using Microsoft Remote Desktop Connection application, check that both instances are connected to the same VPC network.