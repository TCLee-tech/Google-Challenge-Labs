# Serverless Firebase Development: Challenge Lab

### Challenge scenario
In this lab you will create a frontend solution using a Rest API and Firestore database. Cloud Firestore is a NoSQL document database that is part of the Firebase platform where you can store, sync, and query data for your mobile and web apps at scale. Lab content is based on resolving a real world scenario through the use of Google Cloud serverless infrastructure.

You will build the following architecture:
![overall architectire](https://github.com/TCLee-tech/Google/blob/4aa0f37a518552df5ddad4589db4ca7e12a539e7/Serverless%20Firebase%20Development/Image%201.png)

#### Provision the environment
1. Link to the project:  
`gcloud config set project $(gcloud projects list --format='value(PROJECT_ID)' --filter='qwiklabs-gcp')`
2. Clone the repo:  
`git clone https://github.com/rosera/pet-theory.git`

### Task 1. Create a Firestore database
In this scenario, you create a Firestore Database in Google Cloud. The high level architecture diagram below summarizes the general architecture.
![Task 1 architecture](https://github.com/TCLee-tech/Google/blob/cc5e6c7f37e5feaee90a2e6ce742e7d61b2120c4/Serverless%20Firebase%20Development/Image%202.png)
Requirements:

|Field |	Value |
| ---  |     ---  |
|Cloud Firestore |	Native Mode |
|Location	| nam5 (United States) |

To complete this section successfully, you are required to implement the following:

- Cloud Firestore Database
- Use Firestore Native Mode
- Add location nam5 (United States)

### = Task 1 Solution =
[Getting Started with Cloud Firestore](https://firebase.google.com/docs/firestore/quickstart)  
1. In the Cloud Console, go to **Navigation menu** and select **Firestore**.
2.  Click the **Select Native Mode** button.
3. In the **Select a location** dropdown, choose **nam5** and then click **Create Database**.  

OR  
In Cloud Shell CLI, enter `gcloud firestore databases create --region=nam5`  [Reference](https://cloud.google.com/sdk/gcloud/reference/firestore/databases/create)  

### Task 2. Populate the Database
In this scenario, populate the database using test data.

A high level architecture diagram below summarizes the general architecture.
![Task 2 Image 3](https://github.com/TCLee-tech/Google/blob/d348c801d996e8e2092acf9f41ec974850117bc7/Serverless%20Firebase%20Development/Task%202%20Image%203.png)

Example Firestore schema:

|Collection |	Document |	Field |
| ---       | ---        | ---    |
|data |	70234439 |	[dataset] |

The [Netflix Shows Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows) includes the following information:

|Field |	Description |
| ---  | ---            |
|show_id: |	Unique ID for every Movie / Tv Show |
|type: |	Identifier - A Movie or TV Show |
|title:	| Title of the Movie / Tv Show |
|director: |	Director of the Movie |
|cast:	| Actors involved in the movie / show |
|country:	| Country where the movie / show was produced |
|date_added: |	Date it was added on Netflix |
|release_year: |	Actual Release year of the move / show |
|rating: |	TV Rating of the movie / show |
|duration:	| Total Duration - in minutes or number of seasons |

To complete this section successfully, you are required to implement the following tasks:

1. Use the sample code from `pet-theory/lab06/firebase-import-csv/solution`:  
`npm install`  
2. To import CSV use the node `pet-theory/lab06/firebase-import-csv/solution/index.js`:  
`node index.js netflix_titles_original.csv`  

Note: Verify the Firestore Database has been updated by viewing the data in the Firestore UI.  

### = Task 2 Solution =
1. `cd ~/pet-theory/lab06/firebase-import-csv/solution`
2. `npm install`
3. `node index.js netflix_titles_original.csv`
4. Refresh Firestore UI to check if data uploaded


### Task 3. Create a REST API
In this scenario, create an example REST API.

A high level architecture diagram below summarizes the general architecture.

![Task3 image](https://github.com/TCLee-tech/Google/blob/2db3b2912128b850d660feceb001c307b91fd0aa/Serverless%20Firebase%20Development/Task%203%20Image%204.png)

Cloud Run development
|Field |	Value |
| ---  | ---      |
|Container Registry Image |	rest-api:0.1 |
|Cloud Run Service |	Dataset-Service-Name, e.g. netflix-dataset-service-811 |
|Permission |	--allow-unauthenticated |

To complete this section successfully, you are required to implement the following tasks:

1. Access `pet-theory/lab06/firebase-rest-api/solution-01`.
2. Build and Deploy the code to Google Container Registry.
3. Deploy the image as a Cloud Run service.  
Note: Deploy your service with 1 max instance to ensure you do not exceed the max limit for Cloud Run instances.
4. Go to Cloud Run and click `Dataset-Service-Name` then copy the service URL:  
`SERVICE_URL=copy url from your Dataset-Service-Name`  
`curl -X GET $SERVICE_URL` should respond with: {"status":"Netflix Dataset! Make a query."}  

### = Task 3 Solution =
1. `cd ~/pet-theory/lab06/firebase-rest-api/solution-01`  
2. To build container with code and put it in Container Registry, `gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/rest-api:0.1`  
3. To verify, in Cloud console, go to **Navigation menu** > **Container Registry** > refresh to see `rest-api`  
4. Once the container has been built, deploy it: `gcloud run deploy [dataset-service-name, e.g. netflix-dataset-service-811] --image gcr.io/$GOOGLE_CLOUD_PROJECT/rest-api:0.1 --platform managed --region us-central1 --allow-unauthenticated --max-instances=1`  
5. In Cloud Console > `Cloud Run`, click `[Dataset-Service-Name, e.g. netflix-dataset-service-811]` and copy `service URL`  
![Image from Cloud Console for Cloud Run service URL](https://github.com/TCLee-tech/Google/blob/3a65ea21b493fed3c1ecbbe23edc0113c4c60385/Serverless%20Firebase%20Development/API%20service%20URL.jpg)
6. In Cloud Shell, `SERVICE_URL=url copied from Cloud Console`  
  `curl -X GET $SERVICE_URL` should respond with {"status":"Netflix Dataset!Make a query."}  

[Firestore data locations](https://cloud.google.com/firestore/docs/locations)

### Task 4. Firestore API access
In this scenario, deploy an updated revision of the code to access the Firestore DB.

Deploy Cloud Run revision 0.2

|Field |	Value |
| ---  | ---      |
|Container Registry Image |	rest-api:0.2 |
|Cloud Run Service |	Dataset-Service-Name, e.g. netflix-dataset-service-811 |
|Permission |	--allow-unauthenticated |

To complete this section successfully, you are required to implement the following tasks:

1. Access `pet-theory/lab06/firebase-rest-api/solution-02`.  
2. Build the updated application.  
3. Use Cloud Build to tag and deploy image revision to Container Registry.  
4. Deploy the new image as Cloud Run service.    
Note: Deploy your service with 1 max instance to ensure you do not exceed the max limit for Cloud Run instances.  
5. Go to Cloud Run and click `[Dataset Service Name]` then copy the `service URL`:  
`SERVICE_URL=copy url from your Dataset Service Name`  
`curl -X GET $SERVICE_URL/2019` should respond with json dataset.  

![Image of json dataset](https://github.com/TCLee-tech/Google/blob/8cb06230cd9a6aa7dd0eb8b9ca667d4bae02f36e/Serverless%20Firebase%20Development/Task%204%20solution.jpg)

### = Task 4 Solution =
1. `cd ~/pet-theory/lab06/firebase-rest-api/solution-02`  
2. To build a new container image of application, tag it and push to Container Registry, `gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/rest-api:0.2`  
3. To deploy updated image as Cloud Run service, `gcloud run deploy [dataset-service-name, e.g. netflix-dataset-service-811] --image gcr.io/$GOOGLE_CLOUD_PROJECT/rest-api:0.2 --platform managed --region us-central1 --allow-unauthenticated --max-instances=1`  
4. In Cloud Console > `Cloud Run`, click `[Dataset-Service-Name]` and copy `service URL`  
`SERVICE_URL=url copied from Cloud Console`  
`curl -X GET $SERVICE_URL/2019` should respond with json dataset   


### Task 5. Deploy the Staging Frontend
In this scenario, deploy the Staging Frontend.

A high level architecture diagram below summarizes the general architecture.
![Task 5 image](https://github.com/TCLee-tech/Google/blob/ec58a699341c7fd6348936983c0c28afb112f1d2/Serverless%20Firebase%20Development/Task%205%20Image%205.png)

Deploy Frontend

|Field |	Value |
| ---  | ---      |
|REST_API_SERVICE |	REST API SERVICE URL |
|Container Registry Image |	frontend-staging:0.1 |
|Cloud Run Service |	Frontend-Staging-Service-Name |

To complete this section successfully, you are required to implement the following tasks:

1. Access `pet-theory/lab06/firebase-frontend`.  
2. Build the frontend staging application.  
3. Use Cloud Build to tag and deploy image revision to Container Registry.  
4. Deploy the new image as a Cloud Run service.  
Note: Deploy your service with 1 max instance to ensure you do not exceed the max limit for Cloud Run instances.  
5. Frontend access to Rest API and Firestore Database.  
6. Access the Frontend Service URL.  
Note: It's using a demo dataset to provide the onscreen entries.  

### = Task 5 Solution =
For reference: https://github.com/rosera/pet-theory/tree/main/lab06/firebase-frontend
1. `cd ~/pet-theory/lab06/firebase-frontend`  
2. To build a container image of frontend-staging:0.1, tag it and push to Container Registry, `gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/frontend-staging:0.1`
3. To deploy updated image as Cloud Run service, `gcloud run deploy [Frontend-Staging-Service-Name, e.g. frontend-staging-service-244] --image gcr.io/$GOOGLE_CLOUD_PROJECT/frontend-staging:0.1 --platform managed --region us-central1 --allow-unauthenticated --max-instances=1`
4. When the deployment is completed, go to Cloud Console > `Cloud Run` > click `[Frontend-Staging-Service-Name, e.g. frontend-staging-service-244]` > copy `service URL` > paste in a new browser window to see frontend served by Cloud Run. 

![Staging Frontend served by Cloud Run](https://github.com/TCLee-tech/Google/blob/a471ff9a211a856469506497dfd44af1357ac2be/Serverless%20Firebase%20Development/staging%20frontend%20served%20by%20Cloud%20Run.jpg)

### Task 6. Deploy the Production Frontend
In this scenario, update the Staging Frontend to use the Firestore database.

Deploy Frontend

|Field |	Value |
| ---  | ---      |
|REST_API_SERVICE |	REST API SERVICE URL |
|Container Registry Image |	frontend-production:0.1 |
|Cloud Run Service |	Frontend-Production-Service-Name, e.g. frontend-production-service-647 |

To complete this section successfully, you are required to implement the following tasks:

1. Access `pet-theory/lab06/firebase-frontend/public`.
2. Update the frontend application i.e. app.js to use the REST API.
3. Don't forget to append the year to the SERVICE_URL.
4. Use Cloud Build to tag and deploy image revision to Container Registry.
5. Deploy the new image as Cloud Run service
Note: Deploy your service with 1 max instance to ensure you do not exceed the max limit for Cloud Run instances.
6. Frontend access to Rest API and Firestore Database.
Now that the services have been deployed you will be able to see the contents of the Firestore database using the frontend service.

### = Task 6 Solution =
For reference: https://github.com/rosera/pet-theory/tree/main/lab06/firebase-frontend/public
1. `cd ~/pet-theory/lab06/firebase-frontend/public`  
2. Open `app.js`: `nano app.js`
3. Delete the line `const REST_API_SERVICE = "data/netflix.json"` 
4. Uncomment `//const REST_API_SERVICE = "https://XXXX-SERVICE.run.app/2023" `, replace REST API SERVICE URL with [Dataset-Service-Name, e.g. netflix-dataset-service-811] from Task 4. Don't forget to append date at end of URL.
5. Save and exit: `CTRL X` > `Y` > `ENTER`
6. Go up one level to directory with Dockerfile, index.js and package.json `cd ..`
7. To build a container image of frontend-production:0.1, tag it and push to Container Registry, `gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/frontend-production:0.1`
8. To deploy image as Cloud Run service, `gcloud run deploy [Frontend-Production-Service-Name, e.g. frontend-production-service-647] --image gcr.io/$GOOGLE_CLOUD_PROJECT/frontend-production:0.1 --platform managed --region us-central1 --allow-unauthenticated --max-instances=1`
9. In Cloud Console > `Cloud Run`, click `[Frontend-Production-Service-Name, e.g. frontend-production-service-647]` and click on `service URL` to see frontend of production app. 

![Production Frontend served by Cloud Run](https://github.com/TCLee-tech/Google/blob/57f8af32ef0c94d67cb3604b2156c8bdfed37bf8/Serverless%20Firebase%20Development/production%20frontend%20served%20by%20Cloud%20Run.jpg)