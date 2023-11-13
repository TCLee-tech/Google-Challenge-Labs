## Cloud Architecture: Design, Implement, and Manage
### Migrate a MySQL Database to Google Cloud SQL

### Challenge scenario
Your WordPress blog is running on a server that is no longer suitable. As the first part of a complete migration exercise, you are migrating the locally hosted database used by the blog to Cloud SQL.

The existing WordPress installation is installed in the `/var/www/html/wordpress` directory in the instance called `blog` that is already running in the lab. You can access the blog by opening a web browser and pointing to the external IP address of the blog instance.

The existing database for the blog is provided by MySQL running on the same server. The existing MySQL database is called `wordpress` and the user called **blogadmin** with password __Password1*__ , which provides full access to that database.

### Your challenge

1. You need to create a new Cloud SQL instance to host the migrated database.
Once you have created the new database and configured it, you can then create a database dump of the existing database and import it into Cloud SQL.
2. When the data has been migrated, you will then reconfigure the blog software to use the migrated database.
3. For this lab, the WordPress site configuration file is located here: `/var/www/html/wordpress/wp-config.php`.

To sum it all up, your challenge is to migrate the database to Cloud SQL and then reconfigure the application so that it no longer relies on the local MySQL database. Good luck!

**Note:** Your lab activity tracking score will initially report a score of 20 points because your blog is running. If you reconfigure the blog application to use Cloud SQL database successfully, those points will remain in your grand total.
If the database has been incorrectly migrated, the "blog is running" test will fail, reducing your score by 20 points.

**Note:** Use the following values for the zone and region where applicable Zone: `Filled in at lab startup`. Region: `Filled in at lab startup`.

### Tips and tricks
**Google Cloud SQL - How-To Guides:** The Cloud SQL documentation includes a set of [How-to guides](https://cloud.google.com/sql/docs/mysql/introduction) that provide guidance on how to create instances and databases, and how to connect applications to those databases.

**WordPress Installation and Migration:** The [WordPress Codex](https://developer.wordpress.org/advanced-administration/before-install/howto-install/) provides information on how to install, configure, and migrate WordPress sites. You will find the instructions on how to create and prepare databases for use with WordPress [here](https://developer.wordpress.org/advanced-administration/before-install/creating-database/).

<hr>

Solution 👇👇👇 

### 1. Set default compute region and zone in local client (Cloud Shell) to that for Kubernetes cluster.
In Cloud Shell,
```
gcloud config set compute/region [Region]
gcloud config set compute/zone [Zone]
```
To verify,
```
gcloud config get-value compute/region
gcloud config get-value compute/zone
```
Reference:  
[gcloud compute defaults](https://cloud.google.com/compute/docs/gcloud-compute)  

### 2. Create a Cloud SQL instance

The VM instance named `blog` hosting the WordPress blog has internal ephemeral IP and static external IP addresses. It is in the `default` network.

From Google Cloud console,
- Navigation menu > SQL > Click **Create instance**.
- Click **MySQL**.
- In the **Instance ID** field, enter `cloudSQL-instance`.
- In the **Password** field, enter __Password1*__ for the root user. Remember this password for future use.
- For **Choose a Cloud SQL edition**, select **Enterprise**
- For **Choose a preset for this edition. Presets can be customized later as needed.**, select **Development** from the drop down list.
- Under **Choose region and zonal availability**, select `Region` specified by lab. For **Zonal availability** > Single zone > **Primary zone** > select `Zone` specified by lab.
- Leave the rest of settings as default.
- Click **CREATE INSTANCE**.

Alternatively, to use Cloud Shell,
```
gcloud sql instances create sql-instance --database-version=MYSQL_8_0 --cpu=4 --memory=16GB --zone=[ZONE] --root-password=Password1* --authorized-networks=[external_IP of `blog` VM, with structure xx.xx.xxx.0/24]
```

 - --edition flag default is `enterprise`
 - --assign-ip default is to assign public IP address


References:  
[Cloud SQL Quickstart: Connect from Compute Engine](https://cloud.google.com/sql/docs/mysql/connect-instance-compute-engine)   
[gcloud sql instances create](https://cloud.google.com/sdk/gcloud/reference/sql/instances/create)  
[Cloud SQL - configure public IP](https://cloud.google.com/sql/docs/mysql/configure-ip)

### 3. Create a database instance
From Google Cloud console,
- Navigation menu > SQL > click on name of SQL instance `sql-instance`. 
- On the left-hand menu, select **Databases** tab.
- Click **+ CREATE DATABASE**.
- In the `Create a databse` slide-out menu, enter `wordpress` as **Database Name** and press **CREATE**.

Alternatively, to use Cloud Shell,
```
gcloud sql databases create wordpress --instance=sql-instance
```
Reference:  
[gcloud sql databases create](https://cloud.google.com/sdk/gcloud/reference/sql/databases/create)  

### 4. Create a user
From Google Cloud console,
- Navigation menu > SQL > click on `sql-instance` instance name.
- On the left-hand SQL navigation menu, select **Users** tab.
- Click **+ ADD USER ACCOUNT**.
- In the `Add a user account to instance sql-instance` slide-out menu:
    - under `Built-in authentication`, enter **blogadmin** for `User name` and __Password1*__ for `Password`. 
    - under`Host name`, select `Restrict host by IP address or address range` and enter External-IP address of `blog` VM instance. Change the last octet to 0 with 24 mask, e.g. 34.82.147.67 becomes 34.82.147.**0/24**
    - Less secure alternative is to leave as default `Allow any host`. This option is not allowed for progress in this lab.
    - Click **ADD**.

Alternatively, to use Cloud Shell,
```
gcloud sql users create blogadmin --instance=sql-instance --password=Password1*
```   
Reference:  
[gcloud sql users create](https://cloud.google.com/sdk/gcloud/reference/sql/users/create)


### 5. Export existing `wordpress` database.

SSH into the `blog` VM instance.  

Run `mysqldump` to create a SQL dump file.   
```
mysqldump --databases wordpress --host=localhost --user=blogadmin --password=Password1* --hex-blob --skip-triggers --single-transaction --default-character-set=utf8mb4 >backup.sql
```
There should not be any error messages.

References:  
[Syntax for mysqldump](https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html#mysqldump-syntax)   
[Exporting data from on-premise MySQL database for import into Cloud SQL database](https://cloud.google.com/sql/docs/mysql/import-export/import-export-sql#export-mysqldump)   

Create a Cloud Storage bucket to keep the SQL dump file.  
```
gcloud storage buckets create gs://[Bucket_name, e.g. Project_ID]
```
Upload SQL dump file from `blog` VM to newly created Cloud Storage bucket
```
gcloud storage cp ~/backup.sql gs://[Bucket_name]
```
To verify, in Cloud console > Cloud Storage > click on [Project_ID] bucket > should see `backup.sql` object.   
  - Check the size of `backup.sql`. It should not be 0 KB (empty file, indicating a failure with the mysqldump command). It should be ~ 47.6 KB.  
  - Copy the gsutil URI for the next step (import SQL dump file into Cloud SQL).     

### 6. Importing SQL dump file into Cloud SQL   

a. Describe the SQL instance you are importing into
```
gcloud sql instances describe sql-instance
```
  - Syntax: gcloud sql instances describe SQL_INSTANCE_NAME  

b. Copy the serviceAccountEmailAddres field (e.g. p267356582507-sbccwa@gcp-sa-cloud-sql.iam.gserviceaccount.com)

c. Grant the `storage.objectAdmin` IAM role to the SQL instance's service account. So that the service account can read the Cloud Storage bucket. 
```
gsutil iam ch serviceAccount:[SERVICE-ACCOUNT]:objectAdmin \
gs://[Bucket_name]
```
d. Import the SQL dump file from Cloud Storage bucket into Cloud SQL
```
gcloud sql import sql sql-instance gs://[Bucket_name]/backup.sql --database=wordpress
```
  - Syntax: `gcloud sql import sql SQL_INSTANCE_NAME gs://BUCKET_NAME/IMPORT_FILE_NAME --database=DATABASE_NAME`

Reference:     
[Import a SQL dump file to Cloud SQL for MySQL](https://cloud.google.com/sql/docs/mysql/import-export/import-export-sql#import_a_sql_dump_file_to)   
[Syntax for gcloud sql import sql](https://cloud.google.com/sdk/gcloud/reference/sql/import/sql)   

### 7. Reconfigure blog software to use Cloud SQL database
 - SSH into VM instance named`blog`
 - Change to directory with WordPress config file. `cd /var/www/html/wordpress/`
 - `ls` to list directory content
 - stop the MySQL server. 
     - `sudo service mgsql stop`
     - `sudo service mysql status` to verify
 - Edit wp-config.php `sudo nano wp-config.php`
 - For **/ ** MySQL hostname */ define('DB_HOST', 'localhost');**, replace `localhost` with the Public IP address of the Cloud SQL instance.
 - Press `CTRL + X` to exit. `y + ENTER` to save.
 - Restart the webserver.
     - `sudo service apache2 restart`
     - `sudo service apache2 status` to verify  

 To verify, copy the External IP of the `blog` VM into a new browser tab **using http://[External_IP]**. The blog should load, even though the MySQL service on the `blog` VM was stopped.
  - do not just click on the External_IP of the `blog` VM from the Cloud console, or copy-and-paste the External_IP into the address bar of a new browser tab. It will run as **https**://[Enternal_IP]. The blog is not configured to handle https and give return an error "Hmmm… can't reach this page 35.229.50.22 refused to connect."
