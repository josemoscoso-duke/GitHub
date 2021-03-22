# Distributed Image Processing in Cloud Dataproc
This project is based on the [Distributed Image Processing in Cloud Dataproc](https://www.qwiklabs.com/focuses/5834?catalog_rank=%7B%22rank%22%3A8%2C%22num_filters%22%3A0%2C%22has_search%22%3Atrue%7D&parent=catalog&search_id=9364283) qwiklab from Google Cloud Self-paced labs.

In summary, the VM instance manages the interaction with the storage buckets to upload images, submits scala jobs and retrieve outlined faces that are also stored in the cloud storage bucket.


## Overview
This repo shows how to implement an Apache Spark cluster with Cloud Dataproc service, with the goal of distributing a computationally intensive image processing task onto a cluster of machines. Google cloud services used in this demo are:

- Virtual machine (VM): Development machine to host services. I have used an ssh connection to log from my local (ubuntu) to this VM through the following command:

`gcloud compute ssh --zone "us-central1-a" "devhost" --project "cloud-dataproc-project3"`

 where devhost is the name of the GCP VM instance that host services.

- Cloud storage buckets: to upload and collect the results (output)
- Cloud dataproc cluster: setting a region (i.e. us-central1) to create a new cluster


### Dataflow diagram
<!--- [](img_files/GCP_dataproc.png) --->
<img src=img_files/GCP_dataproc.png width="500" height="300">

### Image processing example
Input image             |  Output image
:-------------------------:|:-------------------------:
![](img_files/family_photo.jpeg)  |  ![](img_files/out_family_photo.jpeg)


## Prerequisites

- Review that your project is linked to an active, valid Cloud Billing account. You will not be able to use the products and services enabled in your project. This is true even if your project only uses Google Cloud services that are free.
- Confirm that the default compute Service Account {project-number}-compute@developer.gserviceaccount.com is present and has the editor role assigned.

Replace `{project-number}` with your project number.
For **Role**, select **Project (or Basic)** > **Editor**. Click **Save**.

## Guide to deploy the services (vm, cloud storage and cloud dataproc cluster)

1. Create a development machine in Compute Engine  
  - **Compute Engine** > **VM Instances** > **Create**  
  - Use the following parameters, as detailed in the referenced [qwiklab](https://www.qwiklabs.com/focuses/5834?catalog_rank=%7B%22rank%22%3A8%2C%22num_filters%22%3A0%2C%22has_search%22%3Atrue%7D&parent=catalog&search_id=9364283)    
      * Name: devhost  
      * Series: N1  
      * Machine Type: 2 vCPUs (n1-standard-2 instance)  
      * Identity and API Access: Allow full access to all Cloud APIs.  

Click **Create**. This will serve as your development host.  

2. Install software (sbt and scala) to submit jobs to the cluster  
  - `sudo apt-get install -y dirmngr unzip`  
  - `sudo apt-get update`  
  - `sudo apt-get install -y apt-transport-https`  
  - `echo "deb https://dl.bintray.com/sbt/debian /" | \
       sudo tee -a /etc/apt/sources.list.d/sbt.list`  
  - `sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 recv 642AC823`  
  - `sudo apt-get update`  
  - `sudo apt-get install -y bc scala sbt`  

3. Download the feature detector files and build the JAR
  - `sudo apt-get update`
  - `gsutil cp gs://spls/gsp124/cloud-dataproc.zip .
     unzip cloud-dataproc.zip`
  - `cd cloud-dataproc/codelabs/opencv-haarcascade`

4. Launch the build: `sbt assembly`

5. Create a cloud Storage bucket
  - `GCP_PROJECT=$(gcloud config get-value core/project)`
  - `MYBUCKET="${USER//google}-image-${RANDOM}"`
  - `echo MYBUCKET=${MYBUCKET}`
  - `gsutil mb gs://${MYBUCKET}`

6. Download images and upload them into the GCP Bucket
  - This is only an example. I uploaded a different set of images:  
    `curl https://www.publicdomainpictures.net/pictures/10000/velka/296-1246658839vCW7.jpg | gsutil cp - gs://${MYBUCKET}/imgs/classroom.jpg`
  - Check buckets content:  
    `gsutil ls -R gs://${MYBUCKET}`

7. Create a Cloud Dataproc cluster
  - `MYCLUSTER="${USER/_/-}-qwiklab"`
  - `echo MYCLUSTER=${MYCLUSTER}`
  - Set a global Compute Engine region to use and create a new cluster:  
    * `gcloud config set dataproc/region us-central1`
    * `gcloud dataproc clusters create ${MYCLUSTER} --bucket=${MYBUCKET} --worker-machine-type=n1-standard-2 --master-machine-type=n1-standard-2 --initialization-actions=gs://spls/gsp010/install-libgtk.sh --image-version=2.0`  

8. Submit a job to Cloud dataproc  
  - Load face detection configuration file:  
  `curl https://raw.githubusercontent.com/opencv/opencv/master/data/haarcascades/haarcascade_frontalface_default.xml | gsutil cp - gs://${MYBUCKET}/haarcascade_frontalface_default.xml`
  - Set of images uploaded into the imgs directory in your Cloud Storage bucket as input to your Feature Detector:  
    * `cd ~/cloud-dataproc/codelabs/opencv-haarcascade`
    * `gcloud dataproc jobs submit spark \
    --cluster ${MYCLUSTER} \
    --jar target/scala-2.12/feature_detector-assembly-1.0.jar -- \
    gs://${MYBUCKET}/haarcascade_frontalface_default.xml \
    gs://${MYBUCKET}/imgs/ \
    gs://${MYBUCKET}/out/`

9. Monitor job and check performance
  - To monitor job, go to **Navigation menu** > **Dataproc** > **Jobs**  
  - To check performance, go to the storage bucket, to **Navigation menu** > **Storage** and locate the bucket. Images with outlined faces will be in the out/ directory
