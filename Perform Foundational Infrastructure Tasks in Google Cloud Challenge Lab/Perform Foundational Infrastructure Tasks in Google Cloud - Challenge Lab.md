# Perform Foundational Infrastructure Tasks in Google Cloud: Challenge Lab

### Challenge scenario
You are just starting your junior cloud engineer role with Jooli inc. So far you have been helping teams create and manage Google Cloud resources.

You are expected to have the skills and knowledge for these tasks so don’t expect step-by-step guides.

#### Your challenge
You are now asked to help a newly formed development team with some of their initial work on a new project around storing and organizing photographs, called memories. You have been asked to assist the memories team with initial configuration for their application development environment; you receive the following request to complete the following tasks:

- Create a bucket for storing the photographs.
- Create a Pub/Sub topic that will be used by a Cloud Function you create.
- Create a Cloud Function.
- Remove the previous cloud engineer’s access from the memories project.

Some Jooli Inc. standards you should follow:

- Create all resources in the `REGION` region and `ZONE` zone, unless otherwise directed.
- Use the project VPCs.
- Naming is normally team-resource, e.g. an instance could be named **kraken-webserver1**
- Allocate cost effective resource sizes. Projects are monitored and excessive resource use will result in the containing project's termination (and possibly yours), so beware. This is the guidance the monitoring team is willing to share; unless directed, use **e2-micro** for small Linux VMs and **e2-medium** for Windows or other applications such as Kubernetes nodes.
Each task is described in detail below, good luck!

<hr>

### Task 1. Create a bucket
You need to create a bucket called `Bucket Name` for the storage of the photographs. Ensure the resource is created in the `REGION` region and `ZONE` zone.

##### 🔴 Solution:
First, set default region and zone:
```
gcloud config set compute/region <REGION>
gcloud config set compute/zone <ZONE>
```
Create bucket:
```
gcloud storage buckets create gs://<Bucket Name> --location <REGION> 
```
To verify,
```
gcloud storage ls
```

To learn more:
- [Create buckets](https://cloud.google.com/storage/docs/creating-buckets#storage-create-bucket-cli)
- bucket location can only be narrowed down to a geographical region, not zone. [Bucket locations](https://cloud.google.com/storage/docs/locations)
- [List buckets](https://cloud.google.com/storage/docs/listing-buckets#gcloud-list-buckets)

<hr>

### Task 2. Create a Pub/Sub topic
Create a Pub/Sub topic called `Topic Name` for the Cloud Function to send messages.

##### 🔴 Solution:
```
gcloud pubsub topics create <Topic Name>
```
To verify,
```
gcloud pubsub topics list
```

<hr>

### Task 3. Create the thumbnail Cloud Function
Create a Cloud Function `Cloud Function Name` that will create a thumbnail from an image added to the `Bucket Name` bucket. Ensure the Cloud Function is using the 2nd Generation environment. Ensure the resource is created in the `REGION` region and `ZONE` zone.

1. Create a Cloud Function called `Cloud Function Name`
> **Note**: The Cloud Function is required to executes every time an object is created in the bucket created in Task 1. During the process Cloud Function may request permission to enable APIs. Please enable each of the required APIs as requested.

2. Make sure you set the **Entry point** (Function to execute) to `Cloud Function Name` and **Trigger** to `Cloud Storage`.

3. Add the following code to the `index.js`:

```
const functions = require('@google-cloud/functions-framework');
const crc32 = require("fast-crc32c");
const { Storage } = require('@google-cloud/storage');
const gcs = new Storage();
const { PubSub } = require('@google-cloud/pubsub');
const imagemagick = require("imagemagick-stream");
functions.cloudEvent('', cloudEvent => {
  const event = cloudEvent.data;
  console.log(`Event: ${event}`);
  console.log(`Hello ${event.bucket}`);
  const fileName = event.name;
  const bucketName = event.bucket;
  const size = "64x64"
  const bucket = gcs.bucket(bucketName);
  const topicName = "";
  const pubsub = new PubSub();
  if ( fileName.search("64x64_thumbnail") == -1 ){
    // doesn't have a thumbnail, get the filename extension
    var filename_split = fileName.split('.');
    var filename_ext = filename_split[filename_split.length - 1];
    var filename_without_ext = fileName.substring(0, fileName.length - filename_ext.length );
    if (filename_ext.toLowerCase() == 'png' || filename_ext.toLowerCase() == 'jpg'){
      // only support png and jpg at this point
      console.log(`Processing Original: gs://${bucketName}/${fileName}`);
      const gcsObject = bucket.file(fileName);
      let newFilename = filename_without_ext + size + '_thumbnail.' + filename_ext;
      let gcsNewObject = bucket.file(newFilename);
      let srcStream = gcsObject.createReadStream();
      let dstStream = gcsNewObject.createWriteStream();
      let resize = imagemagick().resize(size).quality(90);
      srcStream.pipe(resize).pipe(dstStream);
      return new Promise((resolve, reject) => {
        dstStream
          .on("error", (err) => {
            console.log(`Error: ${err}`);
            reject(err);
          })
          .on("finish", () => {
            console.log(`Success: ${fileName} → ${newFilename}`);
              // set the content-type
              gcsNewObject.setMetadata(
              {
                contentType: 'image/'+ filename_ext.toLowerCase()
              }, function(err, apiResponse) {});
              pubsub
                .topic(topicName)
                .publisher()
                .publish(Buffer.from(newFilename))
                .then(messageId => {
                  console.log(`Message ${messageId} published.`);
                })
                .catch(err => {
                  console.error('ERROR:', err);
                });
          });
      });
    }
    else {
      console.log(`gs://${bucketName}/${fileName} is not an image I can handle`);
    }
  }
  else {
    console.log(`gs://${bucketName}/${fileName} already has a thumbnail`);
  }
});
```
4. Add the following code to the `package.json`:
```
{
  "name": "thumbnails",
  "version": "1.0.0",
  "description": "Create Thumbnail of uploaded image",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "@google-cloud/functions-framework": "^3.0.0",
    "@google-cloud/pubsub": "^2.0.0",
    "@google-cloud/storage": "^5.0.0",
    "fast-crc32c": "1.0.4",
    "imagemagick-stream": "4.1.1"
  },
  "devDependencies": {},
  "engines": {
    "node": ">=4.3.2"
  }
}
```

##### 🔴 Solution:
1. Create a directory named "gcf" for Cloud Function code, then switch to that directory:
```
mkdir gcf
cd gcf
```
2. Create and open `index.js` for editing:
```
nano index.js
```
3. Copy the code meant for `index.js` from the lab instructions and paste it into the file.
4. Exit nano editor with saving:
**CTRL+ x**, followed by **y** and **ENTER**
5. Create and open `package.json` for editing:
```
nano package.json
```
6. Copy the code meant for `package.json` from the lab instructions and paste it into the file.
7. **CTRL+ x**, followed by **y** and **ENTER** to exit with saving.
8. Deploy function by running the following command in same directory containing the Node.js function codes:
```
gcloud functions deploy <Cloud Function Name> \
    --gen2 \
    --region <REGION> \
    --trigger-bucket <Bucket Name> \
    --runtime nodejs20
    
```
To learn more:
- [gcloud functions deploy](https://cloud.google.com/sdk/gcloud/reference/functions/deploy)
- [deploy 2nd gen function using CLI](https://cloud.google.com/functions/docs/create-deploy-gcloud)
- no need "--entry-point <Cloud Function Name>" because it defaults to the NAME/ID of the function
- If asked for permission to enable API(eventarc.googleapis.com) on project, enter **Y**.   

Alternative to step (8) using Google Cloud Console:
  - Navigation menu > **Cloud Functions** > **+ CREATE FUNCTION**
  - Environment: 2nd gen
  - Function name: `Cloud Function Name`
  - Region: < REGION >
  - **+ ADD TRIGGER**: Cloud Storage trigger
  - in side window to configure Cloud Storage trigger,
    - Event: google.cloud.storage.object.v1.finalized
    - Bucket: browse for qwiklabs-gcp-xxxxx-bucket and SELECT
    - click **GRANT** for all roles/permissions requested
      - Example 1: This trigger needs the role roles/eventarc.eventReceiver granted to service account 291777896377-compute@developer.gserviceaccount.com to receive events via Google sources.
      - Example 2: This trigger needs the role roles/pubsub.publisher granted to service account service-291777896377@gs-project-accounts.iam.gserviceaccount.com to receive events via Cloud Storage.
    - click **NEXT**
  - in the next screen, change **Entry point**: < Cloud Function Name >
  - For **Source code - Inline Editor**, replace with the codes for index.js and package.json
  - click **DEPLOY**  


To verify cloud function status:
```
gcloud functions describe <Cloud Function Name>
```
You should see "status: ACTIVE".

<hr>

### Task 4. Test the Infrastructure
You must upload one JPG or PNG image into the bucket

1. Upload a PNG or JPG image to `Bucket Name` bucket.
> **Note**: Alternatively, download this image https://storage.googleapis.com/cloud-training/gsp315/map.jpg to your machine. Then upload it to the bucket.
2. You will see a thumbnail image appear shortly afterwards (use **REFRESH** in the bucket details).

##### 🔴 Solution:
```
curl https://storage.googleapis.com/cloud-training/gsp315/map.jpg --output map.jpg
gcloud storage cp map.jpg gs://<Bucket Name>
```
In Google Cloud Console > Cloud Storage > Buckets > press **REFRESH** to check for thumbnail addition.

![thumbnail added by Cloud Function](https://github.com/TCLee-tech/Google-Challenge-Labs/blob/c263e28530e9418d0d8cf92e8f949725bac828c2/Perform%20Foundational%20Infrastructure%20Tasks%20in%20Google%20Cloud%20Challenge%20Lab/thumbnail%20created%20by%20Cloud%20Function.jpg)

To learn more: 
- [gcloud storage cp command](https://cloud.google.com/sdk/gcloud/reference/storage/cp)

<hr>

### Task 5. Remove the previous cloud engineer
You will see that there are two users defined in the project.

- One is your account (`Username 1` with the role of Owner).
- The other is the previous cloud engineer (`Username 2` with the role of Viewer).
1. Remove the previous cloud engineer’s access from the project.

##### 🔴 Solution:
From `Username 1` account, in Cloud Shell, run:
```
gcloud projects remove-iam-policy-binding <Project_ID> --member=user:<Username 2, e.g. student-01-xxxxxx@qwiklabs.net> --role=roles/viewer
```
To verify, Google Cloud Console > IAM & Admin > IAM. Under "VIEW BY PRINCIPALS", the entry for <Username 2> should be removed.

To learn more:
- [Revoke single role](https://cloud.google.com/iam/docs/granting-changing-revoking-access#iam-revoke-single-role-gcloud)
- [policy binding](https://cloud.google.com/iam/docs/reference/rest/v1/Policy#Binding)

