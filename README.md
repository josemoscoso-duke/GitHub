# Distributed Image Processing in Cloud Dataproc
This project is based on the [Distributed Image Processing in Cloud Dataproc](https://www.qwiklabs.com/focuses/5834?catalog_rank=%7B%22rank%22%3A8%2C%22num_filters%22%3A0%2C%22has_search%22%3Atrue%7D&parent=catalog&search_id=9364283) qwiklab from Google Cloud Self-paced labs.


## Overview
This repo shows how to implement an Apache Spark cluster with Cloud Dataproc service, with the goal of distributing a computationally intensive image processing task onto a cluster of machines. Google cloud services used in this demo are:

- Virtual machine (VM): Development machine to host services. I have used an ssh connection to log from my local (ubuntu) to this VM through the following command:

`gcloud compute ssh --zone "us-central1-a" "devhost" --project "cloud-dataproc-project3"`

 where devhost is the name of the GCP VM instance that host services.

- Cloud storage buckets: to upload and collect the results (output)
- Cloud dataproc cluster: setting a region (i.e. us-central1) to create a new cluster

In summary, the VM instance manages the interaction with the storage buckets to upload images, submits scala jobs and retrieve outlined faces also stored in the storage bucket.

### Dataflow diagram
![plot](img_files/plot_png_example.png)

Input image             |  Solarized Ocean
:-------------------------:|:-------------------------:
![](img_files/family_photo.jpeg)  |  ![](img_files/family_photo.jpeg)


## Prerequisites

- Review that your project is linked to an active, valid Cloud Billing account. You will not be able to use the products and services enabled in your project. This is true even if your project only uses Google Cloud services that are free.
- Confirm that the default compute Service Account {project-number}-compute@developer.gserviceaccount.com is present and has the editor role assigned.


## Guide to use the container


## Links to documentation

> Flask :  https://flask.palletsprojects.com/en/1.1.x/    
> Containers and Docker : https://www.docker.com/resources/what-container.
