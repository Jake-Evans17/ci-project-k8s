# Continuous Integration Project

## 0. Project Definition

To deploy an application that allows users to create an account, verify that account with an email address and login; and to set up continuous integration to automate the process of deploying the app when changes are made to the source code.

## 1. Architecture

Micro-Services:
* Gateway
* 2 clients for front-end web pages
* 10 services - back-end APIs
* Database - mongo

## How to Run

### Prerequisites:

* Docker
* Docker-Compose
* Git
* GCP

### Steps:

1. Create a Kubernetes cluster

    `gcloud container clusters create default`
    
    `gcloud container clusters get-credentials default`

2. Git clone this repository in the cloud shell

    https://github.com/Jake-Evans17/ci-project-k8s

3. Export DOCKER_USER environment variable, setting it to your DockerHub username

4. Apply the .yaml files in services folder to create services

    `kubectl apply -f ./`

5. Change .yaml files in deployments folder

In the 11-authentication-deployment.yaml file, change the ACTIVATION_LINK environemnt variable so that it contains theExternal IP of load balancer created in step 4. This IP can be found using the following command.

    `kubectl get services`

6. Apply .yaml files in deployments folder to create pods using the same command as in step 4.

7. Git clone the following repository

    https://github.com/Jake-Evans17/jenkins-docker-gcp-setup

8. Run docker-compose.yaml file with the same command as in steps 6 and 4.

9. Login to Jenkins through a browser

The administrator password for Jenkins can be found in the logs for the newly created Jenkins pod, printed using the following command

    `kubectl logs JENKINS_POD_NAME`

10. Create a new Jenkins Project

Configure Source Code Management with a link to the Git repository and set GitHub hook trigger for GITScm polling as a build trigger. Add a Build Step to execute a shell command containing the following  script:

    `export host_ip=
    docker compose-build
    docker compose push`

where host_ip is the external IP of the gateway load balancer. Also include the following command, repeating it for each image and change accordingly:

    `kubectl --record deployment.apps/authentication-client set image deployment.v1.apps/authentication-client authentication-client=docker.io/jakeevans17/authentication-client:${BUILD_NUMBER}`

where BUILD_NUMBER is an environment variable referring to the current build.

### Common Errors:

* Jenkins pod status stuck on pending

Resize the cluster with the following command:

    `gcloud container clusters resize CLUSTER_NAME --node-pool POOL_NAME --num-nodes NUM_NODES`

* Jenkins - No valid crumb was included in the request:

Continue as admin then navigate to Manage Jenkins, Configure Global Security.
Under CSRF Protection, check 'Enable proxy compatibility' and apply changes.

## Appendix

#### Environment Variables

Variable Name| Definition
------------ | -------------
DOCKER_USER | Username for DockerHub
GMAIL_USER | Gmail address to send confirmation emails for registration
GMAIL_PASS | Password for gmail account
SERVICE_NAME | Application name
BUILD_NUMBER | Build number for the Jenkins project
ACTIVATION_LINK | Link sent to users to confirm email
