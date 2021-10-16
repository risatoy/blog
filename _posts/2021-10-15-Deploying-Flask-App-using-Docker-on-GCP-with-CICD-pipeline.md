---
layout: post
title:  "Deploying Flask App using Docker on Google Cloud Platform with CI/CD pipeline"
date:   2021-10-15 23:30:00 +0900
categories: Flask
---

This blog details the process of deploying flask app using docker on Google Cloud Platform (Cloud Build, Cloud Container Registry and Cloud Run) with CI/CD pipeline setup.

##### 1) Create app.py
```
from flask import Flask, Response, jsonify, render_template, logging, request
app = Flask(__name__)

@app.route('/')
def home():
    return render_template('index.html')

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=80)
```

##### 2) Create index.html inside templates folder
``` templates/index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <h1>hello world! 2.0</h1>
</body>
</html>
```

##### 3) Create Dockerfile
```
FROM python:3.8

ENV PORT 80
ENV HOST 0.0.0.0

EXPOSE 80

RUN apt-get update -y && \
    apt-get install -y python3-pip

COPY ./requirements.txt /app/requirements.txt

WORKDIR /app

RUN pip install -r requirements.txt

COPY . /app


ENTRYPOINT ["python", "app.py"]
```

##### 4) Create requirements.txt file using ```pip freeze -r > requirements.txt```
```
Flask==1.1.2
```

##### 5) Run ```docker build . ``` to build docker image
> ``NOTE:`` Install [Docker](https://docs.docker.com/get-docker/) to use docker command

##### 6) Create a new project on GCP

##### 7) Create a new repo on Github

##### 8) Get ImageID using ```docker images```

##### 9) ```docker tag <imageid> gcr.io/<gcp-project-id>/<projectname>``` to create a tag to the docker image

##### 10) ```gcloud init``` to check if you are in the right GCP project
> ``NOTE:`` Install gcloud to use gcloud command

##### 11) ```gcloud auth configure-docker``` to add credentials

##### 12) Enable Container Registry on GCP

##### 13) ```docker push gcr.io/<gcp-project-id>/<projectname>``` to push the image that you built to GCP Container Registry
option: check Container Registry if the image is uploaded

##### 14) Enable Cloud Build on GCP

##### 15) On Cloud Run, Create service -> fill in service name, select region (my preference is us-west1) -> select container image (latest) -> allow unauthenticated invocations -> create

##### 16) On Cloud Run, Click "Edit & Deploy new revision" -> change the port number to 80 -> change autoscaling to maximum 1 -> deploy
option: When green icon appears, access the website from the url

### Setting Up Continuous Deployment

##### 17) Create cloudbuild.yaml (reference: https://cloud.google.com/build/docs/deploying-builds/deploy-cloud-run#building_and_deploying_a_container)
option: change title on index.html

##### 18) Push to GitHub

##### 19) GCP -> Cloud Build -> Create trigger -> filled in name "<your-app-name>-trigger" -> source repositry, select github repo -> Configuration, select Cloud Build configuration file -> create
  <- this trigger is listening for any time that we make a commit to the github repo
     option: try "run" -> check build history -> when failed, go to settings > enable cloud run admin and service accounts -> go back to triggers -> run again

##### 20) Check the url if its update is made!


