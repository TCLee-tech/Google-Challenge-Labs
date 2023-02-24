Step: Create IAM service account \
gcloud iam service-accounts create my-sa-123 --display-name "my service account"


Step: Bind Cloud Storage admin to service account \
gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID \
    --member serviceAccount:my-sa-123@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role roles/storage.admin


Step: Bind Big Query admin to service account \
gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID  \     
    --member serviceAccount:my-sa-123@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role roles/bigquery.admin


Step: Create Google Credentials key (Json file) for IAM service account \
gcloud iam service-accounts keys create key.json --iam-account=my-sa-123@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com


Step: Set Json key file as environmental variable in Cloud Shell Terminal \
export GOOGLE_APPLICATION_CREDENTIALS=key.json


Step: Copy analyze-images.py file from Google Cloud Storage into Cloud Shell Terminal. \
FYI: cp means copy, gs refers to Google Cloud Storage \
gsutil cp gs://$DEVSHELL_PROJECT_ID/analyze-images.py


Step: Run the analyze-images.py script. \
You need to upload the script into the Cloud Shell. Click on the 3-dots button for drop-down menu at the top right of Cloud Shell. Upload file option is there. \
python3 analyze-images.py $DEVSHELL_PROJECT_ID $DEVSHELL_PROJECT_ID

======================================================================

#References

#Getting started with authentication \
https://cloud.google.com/docs/authentication/getting-started
 - for commands to create service account, grant permissions/role to service account, generate key file and set environmental variable

#Qwiklab: Service Accounts and Roles: Fundamentals \
https://www.cloudskillsboost.google/focuses/1038?parent=catalog
 - more info on service account creation, granting of role to service account so that it has permission to complete specific tasks

#Google Cloud Vision ImageAnnotator class syntax \
https://googleapis.dev/python/vision/latest/vision_v1/image_annotator.html
 - class google.cloud.vision_v1.services.image_annotator.ImageAnnotatorClient()

#Google Cloud Vision API \
https://cloud.google.com/vision/docs/ocr
 - detect text in images (local files)
 - codes send contents of image as base64 encoded string in body of request to API

#Google Cloud Translation \
https://cloud.google.com/translate/docs/basic/translating-text?hl=en#translating_text
 - codes for translating input strings