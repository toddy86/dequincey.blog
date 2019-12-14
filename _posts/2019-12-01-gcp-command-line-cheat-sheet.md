---
layout: post
title: GCP Command Line Cheat Sheet
categories: resources
---

A library of commonly used GCP commands for Cloud Storage, Airflow (Composer) etc.

<!-- more -->

## Google Cloud Storage (GCS)

#### Create a bucket
```
gsutil mb -p [PROJECT_NAME] -c [STORAGE_CLASS] -l [BUCKET_LOCATION] gs://[BUCKET_NAME]/
```

#### Copying files
The object location can either be a local file, or another GCS file.
```
gsutil cp [OBJECT_LOCATION] gs://[DESTINATION_BUCKET_NAME]/
```

## Composer
#### Set server location env variable
```
gcloud config set composer/location LOCATION  (e.g. europe-west1)
```

#### Trigger a DAG
Set the location env variable before running the below command.
```
# Trigger the dag for a specific date
gcloud composer environments run ENVIRONMENT_NAME  \trigger_dag -- DAG_NAME \--exec_date="2019-11-14"


# Trigger a specific dag run
gcloud composer environments run ENVIRONMENT_NAME  \trigger_dag -- DAG_NAME \--run_id=36474

# Set a DAG state
gcloud composer environments run ENVIRONMENT_NAME \dag_state -- DAG_NAME execution_date=2019-11-29


dag_state
gcloud composer environments run ENVIRONMENT_NAME  \trigger_dag -- DAG_NAME \--exec_date="2019-11-14"
```