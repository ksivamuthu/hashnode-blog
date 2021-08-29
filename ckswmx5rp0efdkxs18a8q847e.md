## Google Cloud Run - Build and Deploy a Flask App

Hello 👋 , In this blog post series, I'm going to walk you through the  [Google Cloud Run](https://cloud.google.com/run)  to develop and deploy highly scalable containerized applications on a fully managed serverless platform. Let's see how Google Cloud Run makes app development simpler and faster. So, you can write your application in your favorite language or containerized apps with your favorite tools and deploy them in seconds. Cloud Run abstracts all infrastructure management by automatically scaling up and down to zero and costs you for the time only when the app is running. Let's dive deep into it. 

Let's take a look at how to deploy a Flask app on Cloud Run, Set up continuous deployment of Flask App on Cloud from Github, protect, Add authentication to the app running on a Cloud Run using Google Cloud Endpoints and other advanced features.

In this first part of the series, we are going to see, 

1. Create a Flask App
2. Deploy a Flask App on Cloud Run
3. Set up a Continuous Deployment to Deploy.

## Create a Flask App

In this step, we are going to create a flask app and run it locally. Let's create a minimal flask application that looks something like this,

```python
## index.py
import os

from flask import Flask, jsonify, request

app = Flask(__name__)

@app.route("/")
def index():
    return jsonify({ 'status' : 'running'})

@app.route("/order", methods=["POST"])
def create_order():
    data = request.get_json()
    print('Request Data: ' + str(data))
    return jsonify({ 'msg' : 'Order Created Successfully'})

if __name__ == "__main__":
    app.run(debug=True, host="0.0.0.0", port=int(os.environ.get("PORT", 5000)))
```

Run this flask app locally.

```bash
export FLASK_APP=index
flask run
```

You can hit the API using the curl or postman.

```bash
➜  ~  curl --location --request GET 'http://127.0.0.1:5000/'
{"status":"running"}
```

To test the service locally and to easily integrate with the cloud run, you can use [Cloud Code](https://cloud.google.com/code) or Docker installed locally to run and test locally, including running locally with access to Google Cloud services.

## Deploy a Flask app in Cloud Run

You can deploy the app using two ways. 1. Create a docker container 2. Directly from the source. When you deploy the gcloud project from the source, the gcloud tool identifies the application type and sets up the cloud build to build the container and deploy in google Cloud Run.

Deploy from source using the following command.

```bash
gcloud run deploys order-service --source .
```

If prompted,

1. Source Location
2. Region to Deploy
3. Enable the Artifactory Registry - press `y` to set up the artifactory registry.
4. You will be prompted to allow unauthenticated invocations - press `y` for now
5. In few minutes, the build will be done, and it will deploy the service and provide you the URL

```bash
➜ /cloud-run-flask-demo ➜  gcloud run deploy order-service --source .
This command is equivalent to running "gcloud builds submit --pack image=[IMAGE] ." and "gcloud run deploy order-service --image [IMAGE]"

Building using Buildpacks and deploying the container to Cloud Run service [order-service] in project [*******] region [us-east1]
✓ Building and deploying... Done.
  ✓ Uploading sources...
  ✓ Building Container... Logs are available at [https://console.cloud.google.com/cloud-build/builds/...].
  ✓ Creating Revision...
  ✓ Routing traffic...
Done.
Service [order-service] revision [order-service-00002-rab] has been deployed and is serving 100 percent of traffic.
Service URL: https://order-service-......ue.a.run.app
```

What's going on here... The application does not have Dockerfile. Buildpacks automatically determines that this is a Python application. It know how to run the python application and how to start the python server from Procfile. Buildpacks build a docker container image for your application, so we don't need to worry about managing the base image - as it's all managed by Google and tooling to build the container image locally or how to containerize your app.

Now you can run the curl command in your service .run.app URL to check whether it's working.

```bash
➜  cloud-run-flask-demo  curl --location --request GET 'https://order-service-******-ue.a.run.app'
{"status":"running"}
```

## Set up a Continuous Deployment to Deploy.

Deploying from source code is great !! Now it's time to set up a continuous deployment to deploy whenever the code is pushed. By connecting your GitHub repositories to Cloud Run, you can configure builds and deploy your repositories without writing Dockerfiles or build files. To configure automated builds, while deploying a new Cloud Run service, choose “Continuously deploy new revisions from a source repository”, connect your repo.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630205333503/G02ozps_C.png)

Once you configured your repo, Fill the options in Build Configuration step. 

- *Branch* - indicates what source should be used when running the trigger. You can put the [regex](https://github.com/google/re2/wiki/Syntax) here. Matched branches are automatically verified.
- *Build Type*
    - If your repository should be built using Docker and it contains a Dockerfile, select **Dockerfile**. **Source location** indicates the location and name of the Dockerfile. This directory will be used as the Docker build context. All paths should be relative to the current directory.
    - Otherwise, select **Google Cloud Buildpacks**. Use **Buildpack context** to specify the directory and **Entrypoint** (optional) to provide the command to start the server.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630205344602/7sHOaNf7_.png)

Once you set up the continuous build, the push to the branch you configure will trigger the new version to deploy in the Cloud Run.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630205363532/gbao6bg0h.png)

## What's Next?

In this first part of the series, we see how to set up the Cloud Run from the source and how buildpack automatically converts your application code into a container image that you can then deploy to Cloud Run. And also, we see how to set up the continuous deployment from your source code in github to trigger the Cloud Build on push and run the deployment automatically. Source Code to Production in seconds !!

In the upcoming posts in this series, we will see how to test and debug locally, how to integrate with other Google services such as storage, authentication, and more.

I am going to write a lot about cloud, containers, IoT, and Devops. If you are interested in any of that, make sure to follow me if you haven’t already. Please follow me [@ksivamuthu](https://www.twitter.com/ksivamuthu) Twitter or check out my blogs at https://blogs.sivamuthukumar.com