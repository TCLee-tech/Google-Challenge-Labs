# Deploy and Manage Cloud Environments with Google Cloud: Challenge Lab

### Challenge scenario
You are a cloud engineer working for Cymbal Direct, an online direct-to-consumer footwear and apparel retailer. Cymbal Direct has been scaling rapidly, and as part of their scaling strategy they have acquired a new company, Antern. Antern is an existing ecommerce shop with a large amount of data that—if properly integrated into your existing cloud environment—will be incredibly useful to further scale the business and reach more customers. As part of this acquisition, Cymbal Direct wants to move Antern’s existing workloads and infrastructure from on-prem, into Google Cloud.

Antern has the following resources that need to be migrated, copied, or recreated in Cymbal Direct’s existing cloud environment:

- A PostgreSQL database (running on a virtual machine) that needs to be migrated to Cloud SQL for PostgreSQL  
- Containerized microservices application code to deploy on GKE (with reported reliability issues during testing that need to be troubleshooted)  
- A VPC network with two subnetworks and firewalls that need to be created to connect new resources together  
- IAM users across multiple projects that need to be granted the proper permissions and roles on specific resources  

You are tasked with helping Cymbal Direct achieve these goals.  

<br>

### Task 1: Migrate a stand-alone PostgreSQL database to a Cloud SQL for PostgreSQL instance
Antern has been using a PostgreSQL database running on an on-prem VM to store ecommerce shop orders. As part of the acquisition strategy, Cymbal has requested that this database be migrated to Cloud SQL for PostgreSQL using [Database Migration Service](https://cloud.google.com/database-migration). Having the database running on Cloud SQL allows Cymbal to get all the operational benefits of PostgreSQL with added enterprise availability, stability, and security.

In this task you must migrate the stand-alone PostgreSQL `orders` database running on the `antern-postgresql-vm` virtual machine to a Cloud SQL for PostgreSQL instance using a Database Migration Services continuous migration job and VPC Peering connectivity.

#### Prepare the stand-alone PostgreSQL database for migration
> Note: For the first task, you will need to log in to the **Antern Project** with the **Antern Owner** credentials.  

In this sub-task you must prepare the stand-alone PostgreSQL database so that it satisfies the requirements for migration by Database Migration Services.

To complete this sub-task you must complete the following steps:  

1. Enable the Google Cloud APIs required for Database Migration Services.  

Database Migration Services require the [Database Migration API](https://cloud.google.com/database-migration/docs/reference/rest) and the [Service Networking API](https://cloud.google.com/service-infrastructure/docs/service-networking/reference/rest) to be enabled in order to function. You must enable these APIs for your project.

:red_circle: :red_circle: **Solution to sub-task 1** :red_circle: :red_circle:   
  - Log in as **Antern Owner**.  
  - In the Search Bar at the top of the Google Cloud Console, search for **Database Migration API**. Select the **Database Migration API** option under Marketplace. Click on **ENABLE**.  
  - Repeat search for **Service Networking API** and enable it.

:red_circle: :red_circle: ================ :red_circle: :red_circle:   



2. Upgrade the target databases on the `antern-postgresql-vm` virtual machine with the `pglogical` database extension.  
  
You must install and configure the **pglogical** database extension on the stand-alone PostgreSQL database on the `antern-postgresql-vm` Compute Instance VM. The pglogical database extension package that you must install is named `postgresql-13-pglogical`.  

:red_circle: :red_circle: **Solution to sub-task 2a** :red_circle: :red_circle:    
  - SSH into the `antern-postgresql-vm`  

  - In the Google Cloud Console, on the **Navigation menu**, click **Compute Engine > VM instances**.

  - For the `antern-postgresql-vm` VM instance, select **SSH**.

  - If prompted, click **Authorize**.

  - In the SSH terminal, install the pglogical database extension:
```
sudo apt install postgresql-13-pglogical
```

:red_circle: :red_circle: ================ :red_circle: :red_circle:  

To complete the configuration of the **pglogical** database extension you must edit the PostgreSQL configuration file `/etc/postgresql/13/main/postgresql.conf` to enable the **pglogical** database extension and you must edit the `/etc/postgresql/13/main/pg_hba.conf` to allow access from all hosts.  

:red_circle: :red_circle: **Solution to sub-task 2b** :red_circle: :red_circle:    

In the SSH terminal,
```
sudo su - postgres -c "gsutil cp gs://cloud-training/gsp918/pg_hba_append.conf ."
sudo su - postgres -c "gsutil cp gs://cloud-training/gsp918/postgresql_append.conf ."
sudo su - postgres -c "cat pg_hba_append.conf >> /etc/postgresql/13/main/pg_hba.conf"
sudo su - postgres -c "cat postgresql_append.conf >> /etc/postgresql/13/main/postgresql.conf"
sudo systemctl restart postgresql@13-main
```
Explanation: 
- Above commands are from [Migrate to Cloud SQL for PostgreSQL using Database Migration Service](https://www.cloudskillsboost.google/focuses/22792?parent=catalog) quest.
- `sudo` is acronym for superuser do. Allows you to issue a single command as the root user or another user without a need to change login identity.
- `su` is acronym for switch user or substitute user.
- `su - postgres` means to switch to postgres user.
- `-c` is flag to issue command
- [gsutil](https://cloud.google.com/storage/docs/gsutil) is Google's legacy Cloud Storage CLI tool. Deals with buckets and objects.
- `cp` is copy command
- `gsutil cp gs://cloud-training/gsp918/pg_hba_append.conf .` is a command to copy configuration file from google storage to current location.
- `cat` is command to concatenate (join).
- `cat pg_hba_append.conf >> /etc/postgresql/13/main/pg_hba.conf` is a command to join/add pg_hba_append.conf to the end of /etc/postgresql/13/main/pg_hba.conf file.
- whenever you make changes to the config files on a Linux system, you need to restart the services (in this case, postgresql) so that they re-read the config files and apply the changes.
- [pglogical](https://github.com/2ndQuadrant/pglogical) is a PostgreSQL extension that provides logical streaming replication, using a pub/sub model.

:red_circle: :red_circle: ================ :red_circle: :red_circle:  

3. Create a dedicated user for database migration on the stand-alone database.
The new user that you create on the stand-alone PostgreSQL installation on the `antern-postgresql-vm` virtual machine must be configured using the following user name and password:

    - **Migration user name** : `Postgres_Migration_Username, e.g. migration_admin`

    - **Migration user password** : `DMS_1s_cool!`

:red_circle: :red_circle: **Solution to sub-task 3** :red_circle: :red_circle:  

In the SSH terminal for `antern-postgresql-vm`, launch the **psql** command-line tool:
```
sudo su - postgres
psql
```
In **psql**, create a new user with the replication role:

```
CREATE USER [Postgre_Migration_Username] PASSWORD 'DMS_1s_cool!';   //change [Postgre_Migration_Username] to that assigned during lab
ALTER DATABASE orders OWNER TO [Postgre_Migration_Username];
ALTER ROLE [Postgre_Migration_Username] WITH REPLICATION;
```

:red_circle: :red_circle: ================ :red_circle: :red_circle:  

4. Grant that user the required privileges and permissions for databases to be migrated.
Database Migration Services require that the migration user has privileges to specific schemata and relations of the target databases for migration, in this case that is the `orders` and `postgres` databases.

:red_circle: :red_circle: **Solution to sub-task 4** :red_circle: :red_circle:  
In **psql**, add the `pglogical` database extension to the `orders` and `postgres` databases.  
```
\c orders;
CREATE EXTENSION pglogical;
\c postgres;
CREATE EXTENSION pglogical;
```
List the PostgreSQL databases on the `antern-postgresql-vm` VM using `\l` to confirm that there are only `orders` and `postgres` databases to migrate.    

In **psql**, grant permissions to the `pglogical` schema and tables for the `orders` database:
```
\c orders;
GRANT USAGE ON SCHEMA pglogical TO [Postgre_Migration_Username];
GRANT ALL ON SCHEMA pglogical TO [Postgre_Migration_Username];
GRANT SELECT ON pglogical.tables TO [Postgre_Migration_Username];
GRANT SELECT ON pglogical.depend TO [Postgre_Migration_Username];
GRANT SELECT ON pglogical.local_node TO [Postgre_Migration_Username];
GRANT SELECT ON pglogical.local_sync_status TO [Postgre_Migration_Username];
GRANT SELECT ON pglogical.node TO [Postgre_Migration_Username];
GRANT SELECT ON pglogical.node_interface TO [Postgre_Migration_Username];
GRANT SELECT ON pglogical.queue TO [Postgre_Migration_Username];
GRANT SELECT ON pglogical.replication_set TO [Postgre_Migration_Username];
GRANT SELECT ON pglogical.replication_set_seq TO [Postgre_Migration_Username];
GRANT SELECT ON pglogical.replication_set_table TO [Postgre_Migration_Username];
GRANT SELECT ON pglogical.sequence_state TO [Postgre_Migration_Username];
GRANT SELECT ON pglogical.subscription TO [Postgre_Migration_Username];
```
In **psql**, grant permissions to the `pglogical` schema and tables for the `postgres` database.   
  - you only need to change "\c orders;" to "\c postgres;" in the above command block and re-use it.
```
\c postgres;
GRANT USAGE ON SCHEMA pglogical TO [Postgre_Migration_Username];
GRANT ALL ON SCHEMA pglogical TO [Postgre_Migration_Username];
GRANT SELECT ON pglogical.tables TO [Postgre_Migration_Username];
GRANT SELECT ON pglogical.depend TO [Postgre_Migration_Username];
GRANT SELECT ON pglogical.local_node TO [Postgre_Migration_Username];
GRANT SELECT ON pglogical.local_sync_status TO [Postgre_Migration_Username];
GRANT SELECT ON pglogical.node TO [Postgre_Migration_Username];
GRANT SELECT ON pglogical.node_interface TO [Postgre_Migration_Username];
GRANT SELECT ON pglogical.queue TO [Postgre_Migration_Username];
GRANT SELECT ON pglogical.replication_set TO [Postgre_Migration_Username];
GRANT SELECT ON pglogical.replication_set_seq TO [Postgre_Migration_Username];
GRANT SELECT ON pglogical.replication_set_table TO [Postgre_Migration_Username];
GRANT SELECT ON pglogical.sequence_state TO [Postgre_Migration_Username];
GRANT SELECT ON pglogical.subscription TO [Postgre_Migration_Username];
```
In **psql**, grant permissions to the `public` schema and tables for the `orders` database:
```
GRANT USAGE ON SCHEMA public TO [Postgre_Migration_Username];
GRANT ALL ON SCHEMA public TO [Postgre_Migration_Username];
```

:red_circle: :red_circle: ================ :red_circle: :red_circle:  

5. The Database Migration Service requires all tables to be migrated to have a primary key. Once you have granted the user the required privileges, run the following to add a primary key to the `inventory_items` table and exit psql.
```
ALTER TABLE public.inventory_items ADD PRIMARY KEY(id);
\q 
exit
```
> Note: The detailed prerequisites for migrating a stand-alone PostgreSQL database to Cloud SQL for PostgreSQL are provided in the suggestion links in the Cloud Console GUI for Database Migration Services. Should you need some help on the detailed steps you must take, you may refer to that documentation, or you can look at the detailed steps in the migration lab that is part of this quest.

:red_circle: :red_circle: **Solution to sub-task 5** :red_circle: :red_circle:   

Make the [Postgre_Migration_Username, e.g. migration_admin] user the owner of the tables in the `orders` database, so that you can edit the source data:  

```
\c orders;
\dt
ALTER TABLE public.distribution_centers OWNER TO [Postgre_Migration_Username];
ALTER TABLE public.inventory_items OWNER TO [Postgre_Migration_Username];
ALTER TABLE public.order_items OWNER TO [Postgre_Migration_Username];
ALTER TABLE public.products OWNER TO [Postgre_Migration_Username];
ALTER TABLE public.users OWNER TO [Postgre_Migration_Username];
\dt
```

Then add a primary key to the `inventory_items` table and exit psql:
```
ALTER TABLE public.inventory_items ADD PRIMARY KEY(id);
\q 
exit
```

- `\q` is to quit **psql**
- `exit` is to exit postgres user session

:red_circle: :red_circle: ================ :red_circle: :red_circle:  

#### Migrate the stand-alone PostgreSQL database to a Cloud SQL for PostgreSQL instance

In this sub-task you must perform the migration using Database Migration Services.

To complete this sub-task you must complete the following steps:

1. Create a new Database Migration Service connection profile for the stand-alone PostgreSQL database, using the credentials of the `Postgres Migration User` migration user you created earlier.
    - **Username** : `Postgres_Migration_Username`
    - **Password** : `DMS_1s_cool!`  

You must configure the connection profile using the internal ip-address of the source compute instance.

:red_circle: :red_circle: **Solution to sub-task 1** :red_circle: :red_circle:  
A connection profile is a configuration object that stores info about the source database instance (the PostgreSQL running on `antern-postgressql-vm`) and is used by Database Migration Service to migrate data.  

First, obtain the internal IP of the `antern-postgresql-vm` VM hosting the source database instance:  
  - In Google Cloud Console, **Navigation menu** > **Compute ENgine** > **VM instances**.   
  - Copy the **internal IP** value for `antern-postgresql-vm`.  

Second, create the connection profile:
  - In the Google Cloud Console, **Navigation menu** > **Database Migration** > **Connection profiles**.
  - Click **+ Create Profile**.
  - For **Database engine**, select **PostgreSQL**.
  - For **Connection profile name**, enter **postgres-vm**.
  - For **Hostname or IP address**, enter the internal IP for the PostgreSQL source instance that you copied in the previous task (e.g., 10.138.0.2)
  - For **Port**, leave as **5432**.
  - For **Username**, enter **Postgres_Migration_Username**.
  - For **Password**, enter **DMS_1s_cool!**.
  - For **Region** select [region - during lab, copy from Task 2 below].
  - For all other values leave the defaults.
  - Click **Create**.
A new connection profile named **postgres-vm** will appear in the Connections profile list.

:red_circle: :red_circle: ================ :red_circle: :red_circle:  

<br>

2. Create a new continuous Database Migration Service job.
As part of the migration job configuration, make sure that you specify the following properties for the destination Cloud SQL instance:

  - The **Destination Instance ID** must be set to `Migrated Cloud SQL for PostgreSQL Instance ID`
  - The **Password** for the migrated instance must be set to `supersecret!`
  - **Database version** must be set to **Cloud SQL for PostgreSQL 13**
  - **Region** must be set to [filled in at lab start]
  - For **Connections** both **Public IP** and **Private IP** must be set
  - Select a standard machine type with **1 vCPU**
  - For **Storage** type, use **SSD**
  - Set the storage capacity to **10 GB**
  - For the **Connectivity Method**, you must use **VPC peering** with the **default** VPC network.

:red_circle: :red_circle: **Solution to sub-task 2** :red_circle: :red_circle:  
5 steps -  
    i. create a new continuous migration job  
    ii. define the source database instance  
    iii. create a new destination database instance  
    iv. define connectivity method  
    v. edit source instance `pg_hba.conf` file to allow Database Migration Service to access from VPC IP address  

(i) Create a new continuous migration job
  - In the Google Cloud Console, **Navigation menu** > **Database Migration** > **Migration jobs**.
  - Click **+ Create Migration Job**.
  - **For Migration job name**, enter **vm-to-cloudsql**.
  - For **Source database engine**, select **PostgreSQL**.
  - For **Destination region**, select [region].
  - For **Destination database engine**, select **Cloud SQL for PostgreSQL**.
  - For **Migration job type**, select **Continuous**.
  - Leave the defaults for the other settings.
  - Click **Save & Continue**.

(ii) Define the source database instance
  - For **Source connection profile**, select **postgres-vm**.
  - Click **Save & Continue**.

(iii) Create a new destination database instance
  - For **Destination Instance ID**, enter `Migrated Cloud SQL for PostgreSQL Instance ID - assigned at start of lab`.
  - For **Root password**, enter **supersecret!**.
  - For **Database version**, select **Cloud SQL for PostgreSQL 13**.
  - In **Choose region and zone** section, select **Single zone** for Zonal availability and leave Primary zone as any.
  - For **Connections**, select **Private IP** and **Public IP**.
  - Leave **Associated networking VPC to peer** as **default**
  - Leave **Use an automatically allocated IP range** selected.
  - Click **Allocate & Connect**.

  - For **Machine shapes**, check **1 vCPU, 3.75 GB**
  - For **Storage type**, leave **SSD** selected
  - For **Storage capacity**, select **10 GB**
  - Click **Create & Continue**.
  - If prompted to confirm, click **CREATE DESTINATION AND CONTINUE**.

(iv) Define connectivity method
  - For **Connectivity method**, select **VPC peering**.
  - For **VPC**, leave as **default** selected.  
  > If you see "The destination instance creation must finish before you complete this step. This can take a few minutes. In the meantime, choose how to connect or save and exit and return later." message, the **Configure & Continue** button is disabled. Wait.
  - When you see an updated message that "Instance was created. Define the connectivity method, then Configure & Continue.", click **Configure & Continue**.

(v) Edit source `pg_hba.conf` PostgreSQL configuration file to allow Database Migration Service to access from VPC IP address.
  - In the Google Cloud Console, **Navigation menu** > **VPC network** > right-click **VPC network peering** to open in a new tab.
  - Click on the `servicenetworking-googleapis-com` entry.
  - In the **Imported routes** tab, select and copy the `Destination IP range` (e.g. 10.107.176.0/24).
  - In a SSH terminal for the `antern-postgresql-vm` instance, edit the `pg_hba.conf` file:
  ```
  sudo nano /etc/postgresql/13/main/pg_hba.conf
  ```
  - on the last line of the file, replace the IP range in "host all all 0.0.0.0/0 md5" with the IP range copied from the `Destination IP range`. You should get "host all all `Destination IP range` md5".
  - Save and exit the nano editor with **Ctrl-O, Enter, Ctrl-X**
  - Restart the PostgreSQL service for the config changes to take effect. In the VM instance SSH Terminal session:
  ```
  sudo systemctl start postgresql@13-main
  ```
:red_circle: :red_circle: ================ :red_circle: :red_circle:  

3. Test and then start the continuous migration job.
> **Note**: If you do not correctly prepare the source PostgreSQL environment, the migration might fail completely, or it might fail to migrate some individual tables. If some tables are missing, even though the migration appears to be working otherwise, check that you have correctly configured all of the source database tables.

:red_circle: :red_circle: **Solution to sub-task 3** :red_circle: :red_circle:  

  - In the **Database Migration Service** tab you open earlier, review the details of the migration job.
  - Click **Test Job**.
  - If you get a tick mark and "Your migration job test was successful", click **Create & Start Job**.
  - If prompted to confirm, click **Create & Start**.
  - To verify and review the status of the continuous migration job,
      - In the Google Cloud Console, **Navigation menu** > **Database Migration** > **Migration jobs**.
  - Click the migration job **vm-to-cloudsql** to see the details page.
  - Review the migration job status.
    - If you have not started the job yet, the status will show as **Not started**.
    - If the job has just started, status will show as **Starting**, and then transition to **Running Full dump in progress** to indicate that the initial database dump is in progress.
    - After the initial database dump has been completed, status will transition to **Running CDC in progress** to indicate that the continuous migration is in progress.
    - A **Completed** status is shown only after the destination database has been promoted to a stand-alone database for reading and writing data. Migration job is completed.

:red_circle: :red_circle: ================ :red_circle: :red_circle:  

#### Promote a Cloud SQL to be a stand-alone instance for reading and writing data
In this task, you must complete the migration by promoting the Cloud SQL for PostgreSQL instance to a stand-alone instance.

:red_circle: :red_circle: **Solution to sub-task** :red_circle: :red_circle:  

- Google Cloud Console > **Navigation menu** > **Database Migration** > **Migration jobs**.
- Click the migration job name **vm-to-cloudsql** to see the details page.
- Select **PROMOTE** at top of page.
- If prompted to confirm, click **Promote**.
- to verify, Google Cloud Console > **Navigation menu** > **Databases** > **SQL**. Notice that `Migrated Cloud SQL for PostgreSQL Instance ID` is a PostgreSQL external primary instance.

:red_circle: :red_circle: ================ :red_circle: :red_circle:  

<hr>

### Task 2: Update permissions and add IAM roles to users

Now that the database has been migrated to a Cloud SQL for PostgreSQL instance, you will need to update user roles via IAM for different members on the Antern and Cymbal teams. Specifically, you want to grant the **Antern Editor** user access to the Cloud SQL database, the **Cymbal Owner** admin access to have full control of Cloud SQL resources, and the **Cymbal Editor** to have editor permissions on the project.

**Note**: For this task, you will need to log in to the **Antern Project** with the **Antern Owner** credentials.

1. Grant the **Antern Editor** user the **Cloud SQL Instance User** role for the CloudSQL database. Their username is: `Antern Editor username`.  
    - Navigate to the Cloud SQL database you just created. In the **Users** section, add the **Antern Editor** user account to the database you created. Use **Cloud IAM authentication** and for the principal use their username above.
2. Grant the **Cymbal Owner** user the **Cloud SQL Admin** role for the CloudSQL database. Their username is: `Cymbal Owner username`.
    - Navigate to the Cloud SQL database you just created. In the **Users** section, add the **Cymbal Owner** user account to the database you created. Use **Cloud IAM authentication** and for the principal use their username above.
3. Change the **Cymbal Editor** user role from **Viewer** to **Editor**. Their username is `Cymbal Editor username`.

:red_circle: :red_circle: **Solution to Task 2** :red_circle: :red_circle:  

  - **Navigation menu** > **SQL** > click on the `Destination Instance ID, e.g. corp-postgres29`  click on **Users** on the menu on the left. 
    - click on **ADD USER ACCOUNT** > select **Cloud IAM** and for **Principal**, paste in the username, e.g. student-00-2ff78e7fdff7@qwiklabs.net, assigned to **Antern Editor** during lab. Click **ADD**.
    - click on **ADD USER ACCOUNT** > select **Cloud IAM** and for **Principal**, paste in the username, e.g. student-02-9b295853bcee@qwiklabs.net, assigned to **Cymbal Owner** during lab. Click **ADD**.
  
  - **Navigation menu** > **IAM & Admin** > **IAM**. 
    - Look for the Principal that match the username of **Cymbal Editor**. The role should be displayed as **Viewer**. Click on the pencil icon to edit the role. Change the role to **Editor**. Click **Save**.
    - Look for the Principal that match the username of **Cymbal Owner**. There are 2 roles - **Viewer** and **Cloud SQL Instance User**. Click on the pencil icon to edit. Click on **Cloud SQL Instance User**, type **Cloud SQL Admin** in the filter and select it. Click **Save**.

:red_circle: :red_circle: ================ :red_circle: :red_circle:  

<hr>

### Task 3: Create networks and firewalls

As part of the acquisition strategy, a VPC network to connect resources internally needs to be recreated in the Cymbal project. Specifically, you will need to create a VPC network with two subnets and firewalls to open connections between resources. Additionally, on this network your team will need to be able to connect to Linux and Windows machines using SSH and RDP, as well as diagnose network communication issues via ICMP.

In this task, you will create a VPC network and firewall rules that satisfy these requirements.

> **Note**: For this task, you will need to log in to the **Cymbal Project** with the **Cymbal Owner** credentials.

#### Create a VPC network with two subnetworks

Create a VPC network named [network name] with two subnets: [subnet a name] and [subnet b name]. Use a **Regional** dynamic routing mode.

For [subnet a name] set the region to [network region 1].

  - Set the **IP stack type** to **IPv4 (single-stack)**
  - Set IPv4 range to `10.10.10.0/24`

For [subnet b name] set the region to [network region 2].

  - Set the **IP stack type** to **IPv4 (single-stack)**
  - Set IPv4 range to `10.10.20.0/24`


:red_circle: :red_circle: **Solution to Create a VPC network with two subnetworks** :red_circle: :red_circle:  

1. Create a custom network
```
gcloud compute networks create [network name] \
  --subnet-mode custom
```
  --bgp-routing-mode controls the behavior of Cloud Router. Choice of regional (default) or global.

2. Create [subnet a name]
```
gcloud compute networks subnets create [subnet a name] \
   --network [network name] \
   --region [network region 1] \
   --range 10.10.10.0/24
```
  - IPv4 single-stack type is default

3. Create [subnet b name]
```
gcloud compute networks subnets create [subnet b name] \
  --network [network name] \
  --region [network region 2] \
  --range 10.10.20.0/24
  ```
4. To verify, list your sub-networks:
```
gcloud compute networks subnets list \
--network [network name]
```  

:red_circle: :red_circle: ================ :red_circle: :red_circle:  

  #### Create firewall rules for the VPC network

1. Create a firewall rule named [firewall rule 1].

    - For the network, use [network name].
    - Set the priority to **65535**, the traffic to **Ingress** and action to **Allow**
    - The targets should be set to **all instances in the network** and the IP ranges should include **all IPv4 ranges**
    - Set the Protocol to **TCP** and port to **22**

2. Create a firewall rule named [firewall rule 2].

    - For the network, use [network name].
    - Set the priority to **65535**, the traffic to **Ingress** and action to **Allow**
    - The targets should be set to **all instances in the network** and the IP ranges should include **all IPv4 ranges**
    - Set the Protocol to **TCP** and port to **3389**

3. Create a firewall rule named [firewall rule 3].

    - For the network, use [network name].
    - Set the priority to **65535**, the traffic to **Ingress** and action to **Allow**
    - The targets should be set to **all instances in the network** and the IP ranges should include **all IPv4 ranges**
    - Set the Protocol to **icmp**

:red_circle: :red_circle: **Solution for Create firewall rules for the VPC network** :red_circle: :red_circle:  

```
gcloud compute firewall-rules create [firewall rule 1] \
  --network [network name] \
  --priority 65535 \
  --allow tcp:22

gcloud compute firewall-rules create [firewall rule 2] \
  --network [network name] \
  --priority 65535 \
  --allow tcp:3389

gcloud compute firewall-rules create [firewall rule 3] \
  --network [network name] \
  --priority 65535 \
  --allow icmp
```
To verify,
```
gcloud compute firewall-rules list \
  --filter network=[network name]
```

Explanation:
- [VPC firewall rules](https://cloud.google.com/firewall/docs/using-firewalls#:~:text=Permissions%20required%20for%20this%20task%201%20In%20the,on%20match%2C%20choose%20allow%20or%20deny.%20More%20items)
- `--direction ingress` is default
- by ommiting `--target-tags` and `--target-service-accounts`, firewall rule should apply to all targets (instances) in network.
- by ommiting `--destination-ranges`, firewall rule apply to entire IPv4 range.

:red_circle: :red_circle: ================ :red_circle: :red_circle:  

<hr>

### Task 4: Troubleshoot and fix a broken GKE cluster

> **Note**: For this task, you will need to log in to the **Cymbal Project** with the **Cymbal Owner** credentials.

After deploying the e-commerce website GKE cluster, your team has notified you that there are a few known issues with the GKE cluster that need to be addressed. They have found three bugs that need to be fixed:

  - **Bug #1**: Too much latency of the frontend service
  - **Bug #2**: Ratings are become stale
  - **Bug #3**: Crashing bug in the recommendation service

As part of the acquisition strategy, you have been assigned to fix `bug number`. Other engineers working on your team have provided some additional information for each of the issues they have found, which you can use to troubleshoot the issue.

**Hints:**

**Bug #1**: Visit the external IP of the demo application to see if there are any visible changes. You can also use monitoring dashboards to see metrics associated with each service.

**Bug #2**: Product ratings are managed by the "rating service", hosted on Google AppEngine. The rating data is kept up-to-date by periodically calling an API endpoint that collects all recently sent new rating scores for each product and calculates the new rating. Try to check if the rating service operates normally by inspecting the logs from the AppEngine service. Another team member mentioned this might have to do with an issue in the main.py file.

**Bug #3**: Browse your website until you encounter an issue, and use Cloud Logging to view logs exported by each service. Another team member mentioned this crashing bug might be due to an integer conversion stage in the service.

#### Create a BigQuery log sink

Before fixing the underlying issue, you have been requested to create a log sink to send out the errors associated with the broken service. You will then need use IAM to give users from Antern different levels of access to BigQuery so they can view and interact with the dataset.

1. Use the **Logs Explorer** to investigate your running GKE app and investigate the service has errors. Hint: you should be looking for logs with severity `ERROR`.

:red_circle: :red_circle: **Solution for sub-task 1** :red_circle: :red_circle:  

  - In the Cloud Console, select **navigation menu** > **Logging** > **Logs Explorer**.
  - In the filter box to "Search fields and values" under **Log fields** on the left of the screen, type **error** and select on the "Error" option displayed.
  - Look at the list of log entries under "Query results".
  - To expand a log entry, click on the **>** arrow on the left.

![Log Explorer](https://github.com/TCLee-tech/Google-Challenge-Labs/blob/5723697eb13bb6b8bb353e90fa9ad093f81b4edd/Deploy%20and%20Manage%20Cloud%20Environments%20with%20Google%20Cloud/Log%20Explorer.jpg)

![Custom summary fields under Edit of Log Explorer](https://github.com/TCLee-tech/Google-Challenge-Labs/blob/5723697eb13bb6b8bb353e90fa9ad093f81b4edd/Deploy%20and%20Manage%20Cloud%20Environments%20with%20Google%20Cloud/custom%20summary%20fields.jpg)

:red_circle: :red_circle: ================ :red_circle: :red_circle:  

2. Once you have identified the service error logs, create a sink to send the logs out to BigQuery.

    - Name the sink `sink name`
    - For the destination, create a BigQuery dataset named `gke_app_errors_sink` with a location of **us (multiple regions in United States)**.
    - Click on **EDIT** in the headings for **Query results**. In the `Custom summary fields`, include: `resource.type`, `resource.labels.module_id`, and `protoPayload.line.severity`.
  
  :red_circle: :red_circle: **Solution for sub-task 2** :red_circle: :red_circle:  

  - Click **Create sink** from the **More actions** drop-down.
  - Fill in the fields as follows:
    - Sink name: `sink name` and click **NEXT**.
    - Select sink service: **BigQuery dataset**
    - For **Select Bigquery dataset**, select **Create new BigQuery dataset**
    - In the "Create dataset" slide-out, enter `gke_app_errors_sink` for **Dataset ID**.
    - Leave **Location type** as **Multi-region**, **US (multiple regions in United States)**.
    - Click **CREATE DATASET**.
    - Click **NEXT**.
    - in the **Choose logs to include in sink**, type "severity=ERROR" in the filter box.
    - Click **NEXT**.
    - Leave the rest of the options at the default settings.
    - Click **CREATE SINK**.

Reference: 
  - [Logging query language](https://cloud.google.com/logging/docs/view/logging-query-language)
  - [Configure log sinks - route logs to supported destinations](https://cloud.google.com/logging/docs/export/configure_export_v2)

![Create sink from More Actions in Log Explorer](https://github.com/TCLee-tech/Google-Challenge-Labs/blob/5723697eb13bb6b8bb353e90fa9ad093f81b4edd/Deploy%20and%20Manage%20Cloud%20Environments%20with%20Google%20Cloud/Create%20sink%20from%20More%20actions%20drop-down%20menu.jpg)

:red_circle: :red_circle: ================ :red_circle: :red_circle:  

3. Grant the **Antern Editor** user the **BigQuery Data Viewer** role for this project. Their username is: `Antern Editor username`.

4. Grant the **Antern Owner** user the **BigQuery Admin** role for this project. Their username is: `Antern Owner username`.

:red_circle: :red_circle: **Solution for sub-tasks 3 & 4** :red_circle: :red_circle:  

```
gcloud projects add-iam-policy-binding [PROJECT_ID] --member="user:EMAIL_ADDRESS of Antern Editor" --role=roles/bigquery.dataViewer
gcloud projects add-iam-policy-binding [PROJECT_ID] --member="user:EMAIL_ADDRESS of Antern Owner" --role=roles/bigquery.admin
```
  - copy the PROJECT_ID from the drop-down list at the top of the Cloud Console.

Reference: [BigQuery IAM](https://cloud.google.com/bigquery/docs/access-control#bigquery)

:red_circle: :red_circle: ================ :red_circle: :red_circle:  

#### Fix the GKE cluster

Now that you have created a log sink in BigQuery for the errors in the service, some engineers on your team took a look and figured out the correct steps to fix the issue. In this task you will download the solution code and run it to fix the service in your GKE cluster.

1. Connect to the **cloud-ops-sandbox** GKE cluster and run the following commands to remediate the issue. Answer the verification questions when prompted.
```
git clone --depth 1 --branch csb_1220 https://github.com/GoogleCloudPlatform/cloud-ops-sandbox.git
cd cloud-ops-sandbox/sre-recipes
./sandboxctl sre-recipes restore recipe number
./sandboxctl sre-recipes verify recipe number
```

2. Verify the e-commerce shop is properly working.

:red_circle: :red_circle: **Solution to Fix the GKE cluster** :red_circle: :red_circle:  

  - check that you are in the windows tab for **Cymbal Project** as **Cymbal Owner**.
  - run the remediation commands:
```
git clone --depth 1 --branch csb_1220 https://github.com/GoogleCloudPlatform/cloud-ops-sandbox.git
cd cloud-ops-sandbox/sre-recipes
./sandboxctl sre-recipes restore recipe number
./sandboxctl sre-recipes verify recipe number
```
  - answers to quiz:  
    - Which service has an issue? Rating    
    - What was the cause of the issue? Scheduler job that sends recollect request to rating service does not work.  
  - to verify that the e-commerce shop is working properly:  
    - **Navigation menu** > Kubernetes Engines > Services and Ingress > Status should show tick mark for all services.  
    - Click on the IP Endpoint for **frontend-external** (Type: External load balancer) to access front page of application. Check that it is working properly.  
  
:red_circle: :red_circle: ================ :red_circle: :red_circle:  

![Task 4 Kubernetes services ok](https://github.com/TCLee-tech/Google-Challenge-Labs/blob/5723697eb13bb6b8bb353e90fa9ad093f81b4edd/Deploy%20and%20Manage%20Cloud%20Environments%20with%20Google%20Cloud/Task4%20all%20Kubernetes%20services%20ok.jpg)

![Quest complete confetti](https://github.com/TCLee-tech/Google-Challenge-Labs/blob/5723697eb13bb6b8bb353e90fa9ad093f81b4edd/Deploy%20and%20Manage%20Cloud%20Environments%20with%20Google%20Cloud/Deploy%20and%20Manage%20Cloud%20Environments%20with%20Google%20Cloud%20completion%20confetti.jpg)
