# Serverless Cloud Run Development: Challenge Lab

#### Situational Overview
Pet Theory is a veterinary practice which is keen to utilize serverless architecture to update their existing systems.

#### Architecture
Pet Theory has nominated the existing monolithic Billing application to be reimagined using serverless.  
Over the course of this lab, you will be expected to implement this design update.  
![Serverless architecture for Billing application](https://github.com/TCLee-tech/Google-Challenge-Labs/blob/1e6f24202ad9bff67cc3c21ede453e377be04e31/Serverless%20Cloud%20Run%20Development%20Challenge%20Lab/Serverless%20architecture%20for%20billing%20application.jpg)  

The development team will provide either the code or an image to be deployed as part of the solution.

#### Developing a Minimal Viable Product (MVP)
You will build a prototype that meets the following high-level requirements:
| Ref | Definition of Done |
| --- | ---                |
| 1   | Deploy staging architecture |
| 2   | Deploy production architecture |
| 3   | Secure access between components in production architecture |

#### Provision the lab environment

**:point_right:^TO DO^**
1. Open Cloud Shell.  
2. Set default project in environment.  
`$(gcloud projects list --format='value(PROJECT_ID)' --filter='qwiklabs-gcp')`  
3. Set region.  
`gcloud config set run/region us-central1`  
4. Set Cloud Run platform.  
`gcloud config set run/platform managed`  
5. Clone Pet Theory repo.  
`git clone https://github.com/rosera/pet-theory.git && cd pet-theory/lab07`  

<Hr>

### Task 1: Enable a Public Service
1. Set up a Rest API for the billing service. Use the information in the table below:

| **FIELD** | **VALUE** |
| ---       | ---       |
| Billing Image | billing-staging-api:0.1 |
| Billing Service | Public billing service, e.g. public-billing-service-346 |
| Authentication | unauthenticated |
| Code | pet-theory/lab07/unit-api-billing |

#### Architecture
![Task 1 architecture](https://github.com/TCLee-tech/Google-Challenge-Labs/blob/5a784637ae711cfac95a76b3b88e63e3fcebc896/Serverless%20Cloud%20Run%20Development%20Challenge%20Lab/Serverless%20Cloud%20Run%20Dev%20Challenge%20Lab%20Task%201%20image.jpg)  
  
2. Using the available code, deploy the Billing Service.

#### Assessment
To complete this task successfully, you are required to implement the following:

  - Build an image using Cloud Build.
  - Deploy a Cloud Run service as an unauthenticated service.
  - Test service responds when the endpoint is accessed.

Note: Activity Tracking can take some time to register. Wait 30 seconds before retrying.

**:point_right:^TO DO^**
1. Change to sub-directory containing code for this task.  
`cd ~/pet-theory/lab07/unit-api-billing`
2. Build container image with tag. It will be uploaded to gcr.io.  
`gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/billing-staging-api:0.1`
3. Deploy image as running container on Cloud Run Service.  
  ```
  gcloud run deploy [Public billing service, e.g. public-billing-service-346] \
  --image gcr.io/$GOOGLE_CLOUD_PROJECT/billing-staging-api:0.1 \
  --allow-unauthenticated
  // Cloud Run region and platform have been preset during provision of lab environment before Task 1.  
  ```  
  

4. Test Cloud Run Service's URL endpoint.  
    - save URL of Public billing service in environmental variable **SERVICE_URL**.  
      `SERVICE_URL=$(gcloud run services describe [Public billing service] --format "value(status.url)")`  
    - display SERVICE_URL to verify.  
      `echo $SERVICE_URL`  
    - Make annoymous GET request to service URL.  
      `curl -X GET $SERVICE_URL`  
    - You should get a response ...

<Hr>

### Task 2. Deploy a Frontend Service
Set up a Frontend Service. Use the information in the table below:

| **FIELD** | **VALUE** |
| ---       | ---       |
| Image Name | frontend-staging:0.1 |
| Service Name | Frontend staging service, e.g. frontend-staging-service-756 |
| Authentication | unauthenticated |
| Code | pet-theory/lab07/staging-frontend-billing |

#### Architecture
![Task 2 architecture](https://github.com/TCLee-tech/Google/blob/864f971a0facf9162a0aa214fb239a6383c3f559/Serverless%20Cloud%20Run%20Development/Serverless%20Cloud%20Run%20Development%20Challenge%20Lab/Serverless%20Cloud%20Run%20Dev%20Challenge%20Lab%20Task%202%20image.jpg)  

#### Assessment
To complete this section successfully, you are required to implement the following tasks:

- Build an image using Cloud Build.
- Deploy the image to Cloud Run as an unauthenticated service.
- Service should respond when the endpoint is accessed.

**:point_right:^TO DO^**
1. Change to sub-directory containing codes for this task.  
`cd ~/pet-theory/lab07/staging-frontend-billing`  
2. Build container image with tag, and upload to gcr.io.  
`gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/frontend-staging:0.1`  
3. Deploy the image to Cloud Run.  
```
gcloud run deploy [Frontend staging service, e.g. frontend-staging-service-756] \
--image gcr.io/$GOOGLE_CLOUD_PROJECT/frontend-staging:0.1 \
--allow-unauthenticated
```
4. Test frontend-staging-service-xxx URL endpoint.  
    - save URL of service in environmental variable **SERVICE_URL_TASK2**  
      `SERVICE_URL_TASK2=$(gcloud run services describe [Frontend staging service] --format "value(status.url)")`  
    - display SERVICE_URL_TASK2 to verify.  
      `ECHO $SERVICE_URL_TASK2`  
    - make annoymous GET request to service URL.  
      `curl -X get $SERVICE_URL_TASK2`  
    - You should get a reponse ...  

<Hr>
  
  ### Task 3. Deploy a Private Service  
  The development team updated their application and would like this deployed to the staging environment:
  
| **FIELD** | **VALUE** |
| ---       | ---       |
| Image Name | billing-staging:0.2 |
| Service Name | Private billing service, e.g. private-billing-service-630 |
| Repository | gcr.io|
| Authentication | authenticated |
| Code | pet-theory/lab07/staging-api-billing |

#### Architecture
![Task 3 architecture](https://github.com/TCLee-tech/Google/blob/144bdd4409319b76b91fb1f2a3d9ff43a2ce681d/Serverless%20Cloud%20Run%20Development/Serverless%20Cloud%20Run%20Development%20Challenge%20Lab/Serverless%20Cloud%20Run%20Dev%20Challenge%20Lab%20Task%203%20image.jpg)  

#### Assessment: Cloud Run Development
To complete this section successfully, you are required to implement the following tasks:

- Delete the existing Billing Service.
- Build an image using Cloud Build.
- Deploy the image to Cloud Run requiring authentication.
- Assign the BILLING_URL to an environment variable.  

  Note: Replace PRIVATE_BILLING_SERVICE inside the code-block with private-billing-service-xxx  
```
  BILLING_URL=$(gcloud run services describe PRIVATE_BILLING_SERVICE \
  --platform managed \
  --region us-central1 \
  --format "value(status.url)")
  ```  
- Service should respond when the endpoint is accessed  
  `curl -X get -H "Authorization: Bearer $(gcloud auth print-identity-token)" $BILLING_URL`  

**:point_right:^TO DO^**  
A Public billing service was deployed in Task 1.  
  - [View a list of available Cloud Run Services](https://cloud.google.com/run/docs/managing/services#viewing_the_list_of_services_in_your_project) to confirm: `gcloud run services list`  
1. [Delete](https://cloud.google.com/sdk/gcloud/reference/run/services/delete) the existing Public billing service from Task 1.  
`gcloud run services delete [Public billing service from Task 1] --platform managed  --region us-central1`    
To verify Public billing service removed: `gcloud run services list`    
2. Change to sub-directory containing codes for task 3.  
`cd ~/pet-theory/lab07/staging-api-billing`  
3. Build tagged container image using codes in this sub-directory.  
`gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/billing-staging-api:0.2`  
4. Deploy image to [Cloud Run requiring authentication](https://cloud.google.com/sdk/gcloud/reference/run/deploy#--[no-]allow-unauthenticated).  
```
gcloud run deploy [Private billing service, e.g. private-billing-service-630] \
  --image gcr.io/$GOOGLE_CLOUD_PROJECT/billing-staging-api:0.2 \
  --no-allow-unauthenticated
```
5. Test Private billing service URL endpoint.  
    - assign URL of service to an environment variable **BILLING_URL**  
     ```
    BILLING_URL=$(gcloud run services describe [Private billing service, e.g. private-billing-service-630] \
      --platform managed \
      --region us-central1 \
      --format "value(status.url)")
    ```
    - display BILLING_URL to verify if assignment successful.  
      `echo $BILLING_URL`  
    - make GET request with authorization to this Private billing service endpoint.  
      `curl -X get -H "Authorization: Bearer $(gcloud auth print-identity-token)" $BILLING_URL`  
    - Service should response with `{"status":"Billing Service Rest API: Online"}`.  
 
<Hr>

### Task 4: Create a Billing Service Account
In preparation for the deployment to production, you will need to create a Service Account for the Billing Service:

| **FIELD** | **VALUE** |
| ---       | ---       |
| Service Account | Billing service account, e.g. billing-service-sa-524 |
| Display Name | Billing Service Cloud Run |
| Service Name | billing-service|
| Role | N/A |

#### Architecture
![Task 4 architecture](https://github.com/TCLee-tech/Google/blob/eb8df3b722b7cde693df563c19201bf718292dc9/Serverless%20Cloud%20Run%20Development/Serverless%20Cloud%20Run%20Development%20Challenge%20Lab/Serverless%20Cloud%20Run%20Dev%20Challenge%20Lab%20Task%204%20image)  

#### Assessment: Service Account
To complete this section successfully, you are required to create a Service Account.

**:point_right:^TO DO^**  
To create service account for the Billing Service on Cloud Run:  
`gcloud iam service-accounts create [Billing service account] --display-name "Billing Service Cloud Run"`  
[Create service account reference](https://cloud.google.com/iam/docs/service-accounts-create#iam-service-accounts-create-gcloud)

<Hr>

### Task 5: Deploy the Billing Service
Associate the Billing Service Account with Billing Service:

| **FIELD** | **VALUE** |
| ---       | ---       |
| Image Name | billing-prod-api:0.1 |
| Service Name | Billing production service, e.g. billing-prod-service-638 |
| Authentication | authenticated |
| Code | pet-theory/lab07/prod-api-billing |
| Service Account | Billing service account, e.g. billing-service-sa-524|

#### Architecture
![Task 5 Architecture](https://github.com/TCLee-tech/Google/blob/4e4822d85186d035aaa58a96884c64a39e3fa86d/Serverless%20Cloud%20Run%20Development/Serverless%20Cloud%20Run%20Development%20Challenge%20Lab/Serverless%20Cloud%20Run%20Dev%20Challenge%20Lab%20Task%205%20image)

#### Assessment: Cloud Run Development
To complete this section successfully, you are required to implement the following tasks:

- Deploy the image to Cloud Run.
- Enable Authentication.
- Enable the Service Account.
- Service should respond when the endpoint is accessed.
- Get the URL of the Billing Service:

Note: Replace PRIVATE_BILLING_SERVICE inside the code-block with [private billing service from Task 3, e.g. private-billing-service-630]
```
PROD_BILLING_URL=$(gcloud run services describe PRIVATE_BILLING_SERVICE \
--platform managed \
--region us-central1 \
--format "value(status.url)")
```
Access the deployed endpoint:
```
curl -X get -H "Authorization: Bearer \
$(gcloud auth print-identity-token)" \
$PROD_BILLING_URL
```


**:point_right:^TO DO^**
1. Change to sub-directory containing codes for Task 5.  
`cd ~/pet-theory/lab07/prod-api-billing`  
2. Build tagged container image of billing-prod-service-xxx using codes in this sub-directory.  
`gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/billing-prod-api:0.1`  
3. Deploy image to Cloud Run, requiring authentication.  
```
gcloud run deploy [Billing production service, e.g. billing-prod-service-638] \
--image gcr.io/$GOOGLE_CLOUD_PROJECT/billing-prod-api:0.1 \
--no-allow-unauthenticated
```
4. Test the billing-prod-service-xxx  endpoint
    - set PROD_BILLING_URL environment variable to private-billing-service-xxx endpoint (from Task 3).
    ```
    PROD_BILLING_URL=$(gcloud run services describe PRIVATE_BILLING_SERVICE \
    --platform managed \
    --region us-central1 \
    --format 'value(status.url)')
    ```
    - access the deployed endpoint.  
    `curl -X get -H "Authorization: Bearer $(gcloud auth print-identity-token)" $PROD_BILLING_URL`  
    You should get response `"status":"Billing Service Rest API: Online"}`.

<Hr>

### Task 6: Frontend Service Account
Create a new Service Account for the Frontend capable of invoking the Billing Service:

| **FIELD** | **VALUE** |
| ---       | ---       |
| Service Account | Frontend service account, e.g. frontend-service-sa-975 |
| Display Name | Billing Service Cloud Run Invoker |
| Service Name | frontend-prod-service |
| Role | run.invoker |

#### Architecture
![Task 6 Architecture](https://github.com/TCLee-tech/Google/blob/8a6290f5d56fdfdaaa185d23aadf84050f7d215c/Serverless%20Cloud%20Run%20Development/Serverless%20Cloud%20Run%20Development%20Challenge%20Lab/Serverless%20Cloud%20Run%20Dev%20Challenge%20Lab%20Task%206%20image)

#### Assessment
To complete this section successfully, you are required to implement the following tasks:

- Create a Service Account.
- Apply Service Account for Frontend Service.
- Give Service Account run.invoker permission.
- Bind Account to Service.

**:point_right:^TO DO^**
1. Create Frontend service account.  
`gcloud iam service-accounts create [Frontend service account] --display-name "Billing Service Cloud Run Invoker"`  
2. Apply Frontend service account to Frontend production service
```  
gcloud run services add-iam-policy-binding frontend-prod-service \
  --member=serviceAccount:[Frontend service account, e.g. frontend-service-sa-975]@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com \
  --role=roles/run.invoker \
  --region us-central1 \
  --platform managed
```

<Hr>

### Task 7: Redeploy the Frontend Service
Use the new Service Account and redeploy the Frontend Service:

| **FIELD** | **VALUE** |
| ---       | ---       |
| Image Name | frontend-prod:0.1 |
| Service Name | Frontend production service, e.g. frontend-prod-service-350 |
| Repository | gcr.io |
| Authentication | unauthenticated |
| Code | pet-theory/lab07/prod-frontend-billing |
| Service Account | Frontend service account, e.g. frontend-service-sa-975 |

#### Architecture
![Task 7 Architecture](https://github.com/TCLee-tech/Google/blob/262e0ba3aefcd4b641103e076ab48d91f187499f/Serverless%20Cloud%20Run%20Development/Serverless%20Cloud%20Run%20Development%20Challenge%20Lab/Serverless%20Cloud%20Run%20Dev%20Challenge%20Lab%20Task%207%20image)  

#### Assessment: Cloud Run Development
To complete this section successfully, you are required to implement the following tasks:

- Deploy the image to Cloud Run.
- Enable Authentication.
- Enable Service Account.
- Service should respond when the endpoint is accessed.

**:point_right:^TO DO^**
1. Change to sub-directory containing codes for Task 7.  
  `cd ~/pet-theory/lab07/prod-frontend-billing`
2. Build tagged container image of "Frontend production service" using codes in this sub-directory.  
  `gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/frontend-prod:0.1`
3. Deploy "Frontend production service" image to Cloud Run, no need authentication.
  ```
  gcloud run deploy [Frontend production service, e.g. frontend-prod-service-350] \
    --image gcr.io/$GOOGLE_CLOUD_PROJECT/frontend-prod:0.1 \
    --platform managed \
    --region us-central1 \
    --allow-unauthenticated 
  ```
4.  Frontend service account was applied to frontend-prod-service in Task 6. Re-apply Frontend service account to Frontend production service created in step 3 above.
  ```
  gcloud run services add-iam-policy-binding [frontend production service] \
    --member=serviceAccount:[Frontend service account]@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com \
    --role=roles/run.invoker \
    --region us-central1 \
    --platform managed
  ```
5. Since Frontend production service communicates with Billing production service, ping the Billing production service endpoint.  
  `curl -X get -H "Authorization: Bearer $(gcloud auth print-identity-token)" $PROD_BILLING_URL`  
  You should get a response `{"status":"Billing Service Rest API: Online"}`  
6. Access the production frontend service to display the user interface.  
    - save URL of production frontend service to an environment variable **FRONTEND_URL**:  
    ```
    FRONTEND_URL=$(gcloud run services describe [frontend production service] \
      --platform managed \
      --region us-central1 \
      --format "value (status.url)")
    ```
    - display FRONTEND_URL  
      `echo $FRONTEND_URL`  
    - make an annoymous unauthenticated GET request to frontend URL:  
      `curl -X get $FRONTEND_URL`  
    - You should see info on screen from the Billing Service.  

