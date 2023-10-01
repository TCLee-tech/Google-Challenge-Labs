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

Reference:
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
**CTRL+ x**, followed by **y**
5. Create and open `package.json` for editing:
```
nano package.json
```
6. Copy the code meant for `package.json` from the lab instructions and paste it into the file.
7. **CTRL+ x**, followed by **y** to exit with saving.
8. Deploy function by running the following command in same directory containing the Node.js function codes:
```
gcloud functions deploy <Cloud Function Name> \
    --gen2
    --region <REGION>
    --trigger-bucket <Bucket Name> \
    --runtime nodejs20
    
```
Reference:
- [gcloud functions deploy](https://cloud.google.com/sdk/gcloud/reference/functions/deploy)
- [deploy 2nd gen function using CLI](https://cloud.google.com/functions/docs/create-deploy-gcloud)
- no need "--entry-point <Cloud Function Name>" because it defaults to the NAME/ID of the function

To verify cloud function status:
```
gcloud functions describe <Cloud Function Name>
```
You should see "status: ACTIVE".

<hr>

### Task 4. Test the Infrastructure
You must upload one JPG or PNG image into the bucket

1. Upload a PNG or JPG image to `Bucket Name` bucket.
> **Note**: Alternatively Download this image https://storage.googleapis.com/cloud-training/gsp315/map.jpg to your machine. Then upload it to the bucket.
2. You will see a thumbnail image appear shortly afterwards (use **REFRESH** in the bucket details).

##### 🔴 Solution:
```
curl https://storage.googleapis.com/cloud-training/gsp315/map.jpg .
gcloud storage cp map.jpg gs://<Bucket Name>
```
In Google Cloud Console > Cloud Storage > Buckets > press **REFRESH** to check for thumbnail addition.

Reference: [gcloud storage cp command](https://cloud.google.com/sdk/gcloud/reference/storage/cp)

<hr>

### Task 5. Remove the previous cloud engineer
You will see that there are two users defined in the project.

- One is your account (`Username 1` with the role of Owner).
- The other is the previous cloud engineer (`Username 2` with the role of Viewer).
1. Remove the previous cloud engineer’s access from the project.

##### 🔴 Solution:
From `Username 1` account, in Cloud Shell, run:
```
gcloud iam roles delete Viewer --project <PROJECT_ID>
```
To verify, log in to `Username 2` and try to view content of storage bucket.

Reference:
[gcloud iam roles delete](https://cloud.google.com/sdk/gcloud/reference/iam/roles/delete)


