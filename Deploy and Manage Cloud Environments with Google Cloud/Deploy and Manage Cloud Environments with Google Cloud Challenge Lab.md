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

:red_circle: :red_circle: **Solution** :red_circle: :red_circle:  
**Navigation menu > APIs & Services >** Search for **Database Migration API >** Click on **Enable**  
Repeat search for API. This time for **Service Networking API** and enable it.

2. Upgrade the target databases on the `antern-postgresql-vm` virtual machine with the `pglogical` database extension.  
  
You must install and configure the **pglogical** database extension on the stand-alone PostgreSQL database on the `antern-postgresql-vm` Compute Instance VM. The pglogical database extension package that you must install is named `postgresql-13-pglogical`.

To complete the configuration of the **pglogical** database extension you must edit the PostgreSQL configuration file `/etc/postgresql/13/main/postgresql.conf` to enable the **pglogical** database extension and you must edit the `/etc/postgresql/13/main/pg_hba.conf` to allow access from all hosts.

:red_circle: :red_circle: **Solution** :red_circle: :red_circle:    

```
sudo su - postgres -c "gsutil cp gs://cloud-training/gsp918/pg_hba_append.conf ."
sudo su - postgres -c "gsutil cp gs://cloud-training/gsp918/postgresql_append.conf ."
sudo su - postgres -c "cat pg_hba_append.conf >> /etc/postgresql/13/main/pg_hba.conf"
sudo su - postgres -c "cat postgresql_append.conf >> /etc/postgresql/13/main/postgresql.conf"
sudo systemctl restart postgresql@13-main
```
Note: 
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
