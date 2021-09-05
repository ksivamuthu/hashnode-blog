## Google Cloud Run - Add Persistent Storage

Hello 👋 , In  [part 1](https://blog.sivamuthukumar.com/cloud-run-flask-app) of the  [Google Cloud Run series](https://blog.sivamuthukumar.com/series/google-cloud-run), we learned what Google Cloud Run is, how to deploy the source code or container to the Google Cloud Run, and set up the continuous deployment to deploy the application to Cloud Run using a few clicks set up in the portal. In this part 2 of the series, let's see the options available to add persistent storage to the Cloud Run applications. At the end of this blog post, you will learn to add persistent storage into the python flask application.

## Google Cloud Storage

Google provides various storage options that address various needs of your applications. This post will explain the three storage options to integrate into your application running in Google Cloud Run

1. Cloud SQL - Storing Structured Data
2. Cloud Firestore - Storing Unstructured Data
3. Cloud Storage - Storing files such as images and video

## Cloud SQL - Storing Structured Data

Cloud SQL is a fully managed database service that helps you set up, maintain, manage, and administer your relational databases in the cloud.  You can configure MySQL, PostgreSQL, SQL Server - RDS in the Cloud SQL. In this, we will create a MySQL instance that can be accessed from the Private IP address. And connect the Cloud SQL instance in the Google Cloud Run.

```bash
gcloud sql instances create cloud-sql-demo \
--database-version=MYSQL_8_0 \
--cpu=NUMBER_CPUS \
--memory=MEMORY_SIZE \
--region=REGION
--network # Private IP Address
```

Once the Cloud SQL is provisioned, you can add the connections to Cloud Run services and configure the VPC  Access connector in the Portal or using `gcloud` cli

```bash
gcloud compute networks vpc-access connectors create CONNECTOR_NAME \
--network VPC_NETWORK \
--region REGION \
--range IP_RANGE

gcloud run services update SERVICE --vpc-connector CONNECTOR_NAME
```

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630811595793/pOfy4gbfQ.png)

Please refer to [Connecting to Cloud SQL from Cloud Run (fully managed)](https://cloud.google.com/sql/docs/mysql/connect-run) for detailed documentation on how to set up managed Cloud SQL connection with Cloud Run.

Connect the CloudSQL Database providers using `sqlalchemy` library by configuring the database host, port, database and credentials.

```python
def init_tcp_connection_engine(db_config):
    db_user = os.environ["DB_USER"]
    db_pass = os.environ["DB_PASS"]
    db_name = os.environ["DB_NAME"]
    db_host = os.environ["DB_HOST"]

    # Extract host and port from db_host
    host_args = db_host.split(":")
    db_hostname, db_port = host_args[0], int(host_args[1])

    pool = sqlalchemy.create_engine(
        sqlalchemy.engine.url.URL.create(
            drivername="mysql+pymysql",
            username=db_user, 
            password=db_pass, 
            host=db_hostname,  
            port=db_port,
            database=db_name, 
        ),
        **db_config
    )
    return pool
```

Let's create order data and store the order table.

```python
@app.route("/order", methods=["POST"])
def create_order():
    data = request.get_json()
		item = data['item']
		requested_at = datetime.datetime.utcnow()

		stmt = sqlalchemy.text(
        "INSERT INTO orders (item, requested_at)" " VALUES (:item, :requested_at)"
    )
   try:
        with db.connect() as conn:
            conn.execute(stmt, item=item, requested_at=requested_at)
    except Exception as e:
        logger.exception(e)
        return Response(
            status=500,
            response=jsonify({ error: "Unable to successfully cast vote! Please check the "
            "application logs for more details."  }),
        )

    return Response(
        status=200,
        response=jsonify({ 'msg' : 'Order Created Successfully'}),
    )  
```

Google recommends that you use Secret Manager to store sensitive information such as SQL credentials. After setting up the secrets in the Secret Manager, you can update the env variables to update the secrets in Cloud Run.

```bash
gcloud run services update SERVICE_NAME \
--add-cloudsql-instances=CLOUD_SQL_CONNECTION_NAME
--update-env-vars=CLOUD_SQL_CONNECTION_NAME=CLOUD_SQL_CONNECTION_NAME_SECRET \
--update-secrets=DB_USER=DB_USER_SECRET:latest \
--update-secrets=DB_PASS=DB_PASS_SECRET:latest \
--update-secrets=DB_NAME=DB_NAME_SECRET:latest
```

## Cloud Firestore - Storing Unstructured Data

[Cloud Firestore](https://firebase.google.com/docs/firestore) is a flexible, scalable, NoSQL database for mobile, web, and server development from Firebase and Google Cloud. Following Cloud Firestore's NoSQL data model, you store data in documents that contain fields mapping to values. These documents are stored in collections, which are containers for your documents that you can use to organize your data and build queries. You can refer to this [quick start](https://firebase.google.com/docs/firestore/quickstart) to set up the Cloud Firestore.

Create a Cloud FireStore Client by initializing the app using `firebase_admin` package.

```python
import firebase_admin
from firebase_admin import credentials
from firebase_admin import firestore

cred = credentials.ApplicationDefault()
firebase_admin.initialize_app(cred, {
  'projectId': project_id,
})

db = firestore.client()
```

 Add the new order to the `orders` collection to store.

```python
@app.route("/order", methods=["POST"])
def create_order():
    data = request.get_json()
		item = data['item']
		requested_at = datetime.datetime.utcnow()

		db_col = db.collection(u'orders');
		db_col.add({
		    u'item': item,
		    u'requested_at': requested_at
		})

    return Response(
        status=200,
        response=jsonify({ 'msg' : 'Order Created Successfully'}),
    )  
```

## Cloud Storage - Storing files such as image and video

You can use [Google Cloud Storage](https://cloud.google.com/storage/docs) to store the files such as images, videos, and other static content. We are using the Google Cloud Client Library to store files in Google Cloud Storage. The file content of the request is saved into the Cloud Storage blob and writes to the Cloud Storage bucket. Once the file is uploaded to Cloud Storage, the pubic URL to this file is returned, which you can serve to the file directly from Cloud Storage. You can store this URL in your data storage too.

```python
import logging
import os

from flask import Flask, request
from google.cloud import storage

CLOUD_STORAGE_BUCKET = os.environ["cloud_storage_bucket"]

@app.route('/upload', methods=['POST'])
def upload():
    """Process the uploaded file and upload it to Google Cloud Storage."""
    uploaded_file = request.files.get('file')

    if not uploaded_file:
        return 'No file uploaded.', 400

    gcs = storage.Client()

    # Get the bucket that the file will be uploaded to.
    bucket = gcs.get_bucket(CLOUD_STORAGE_BUCKET)

    # Create a new blob and upload the file's content.
    blob = bucket.blob(uploaded_file.filename)

    blob.upload_from_string(
        uploaded_file.read(),
        content_type=uploaded_file.content_type
    )
    return blob.public_url
```

## Next...

In  [this second part](https://blog.sivamuthukumar.com/google-cloud-run-add-persistent-storage) of the series, we explored the options to add Persistent Storage in Google Cloud Run. In the next part of the series, we will explore more on how to orchestrate a Workflow using Cloud Run & Cloud Functions, and in the final part of the series, we will see how to add Cloud Endpoints to the Cloud Run applications and build developer API portal.

I'm Siva - working as Sr. Software Architect at [Computer Enterprises Inc](https://www.ceiamerica.com) from Orlando. I am going to write technical blogs on Cloud, Containers, IoT, and Devops. If you are interested in any of that, make sure to follow me if you haven’t already. Please follow me [@ksivamuthu](https://www.twitter.com/ksivamuthu) Twitter or check out my blogs at [https://blog.sivamuthukumar.com](https://blog.sivamuthukumar.com) Please drop a comment below or reach me out if you've any questions or comments.