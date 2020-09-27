# Cloud DevOps Capstone Project

This project is intended to set up a pipeline that allows us to lint the target code, build the correponding Docker container, and deploy it to a [kubernetes](https://kubernetes.io/) cluster. Blue/green deployment is also used.

## Table of Contents

* [Project Overview](#project-overview)
* [Getting Started](#getting-started)
* [Repository Files](#repository-files)
* [Contributing](#contributing)


## Project Overview

The aim of this project is to create a Jenkins pipeline. Blue/green deployment is used, and the chosen Docker application is a simple Nginx one.  These are the different steps that made up the pipeline:

* Test application code using linting.
* Build a Docker image that containerizes the application, a simple Nginx one.
* Deploy the containerized application using Docker.
* Create a configuration file for the kubectl cluster.
* Set the current kubctl context to the cluster.
* Create the blue replication controller with its Docker image.
* Create the green replication controller with its Docker image.
* Create the service in the Kubernetes cluster to the blue replication controller.
* Wait until the user gives the instruction to continue.
* Update the service to redirect to green by changing the selector to app=green.
* Check the application deployed in the cluster and its correct deployment.

Apart from the steps in the pipeline, and as already mentioned, it is necessary to previously configure Kubernetes and create a Kubernetes cluster.

## Getting Started

In this section, it can be seen how the pipeline works as designed.

### Creating the Kubernetes cluster

As a first  step, the Kubernetes cluster is created, as can be seen below. Firstly, in the pipeline . Secondly, in AWS CloudFormation. Finaly, in AWS Elastic Kubernetes Service.

![0-KubernetesClusterCreation](/projectscreenshots/1.KubernetesClusterDeploy.png)
![0-KubernetesClusterCreation2](/projectscreenshots/2.CloudformationStack.png)
![0-KubernetesClusterCreation3](/projectscreenshots/3.EKSCluster.png)

### Running the pipeline

From now on, the pipeline itself is run, and its stages are shown below.

* Test application code using linting. Below is successful Linting screenshot :-
![1-SuccessfulLintingScreenshot](/projectscreenshots/4.SuccesfulLinting.png)
* Build a Docker image that containerizes the application, a simple Nginx one.
![2-BuildtheDockerImage](/projectscreenshots/5.SuccessDockerimageBuild.png)
* Deploy the containerized application using Docker. Both the action in the pipeline and on DockerHub are shown.
![3-UploadtheImagetoDocker](/projectscreenshots/6.UploadDockerimagetoRepository.png)
* Create a configuration file for the kubectl cluster.
![6-Create&Settheconfigurationfileforkubectlcluster](/projectscreenshots/7.Configfileupdating.png)
* Create the blue replication controller with its Docker image.
![8-CreatethebluereplicationcontrollerwithitsDockerimage](/projectscreenshots/8.CreateBlueReplicationController.png)
* Create the green replication controller with its Docker image.
![9-CreatethegreenreplicationcontrollerwithitsDockerimage](/projectscreenshots/9.CreateGreenReplicationController.png)
* Create the service in the Kubernetes cluster to the blue replication controller.
![10-CreatetheserviceintheKubernetesclustertothebluereplicationcontroller](/projectscreenshots/10.CreateServiceBlue.png)
* Wait until the user gives the instruction to continue. The interaction with the user and the logged result are both shown.
![11-Waituntiltheusergivestheinstructiontocontinue](/projectscreenshots/11.ApprovalforGreenService.png)
* Update the service to redirect to green by changing the selector to app=green.
![12-Updatetheservicetoredirecttogreenbychangingtheselectortoapp=green](/projectscreenshots/12.UpdatedtheServicetoGreen.png)
* Check the application deployed in the cluster and its correct deployment. Firstly, we can see in the pipeline logs that the deployed application is running, as both pods are running. Secondly, with the obtained IP of the service (see at "LOAD_BALANCER_INGRESS + : + PORT"), we successfully access the application via the browser. Finally, a screenshot of the AWS EC2 page showing the newly created instances is also shown. Three instances can be seen just below the Jenkins masterbox. 
![13-CheckSuccessfulDeployment](/projectscreenshots/13.CheckApplication.png)
![13-CheckSuccessfulDeployment4](/projectscreenshots/14.WebApplicationUI.png)
![13-CheckSuccessfulDeployment3](/projectscreenshots/15.WorkerNodesinstance.png)

## Repository Files

In this section, the repository files are described:

* *blue-controller.json*: this file specifies the replication controller blue pod.
* *blue-service.json*: this file specifies the blue service.
* *Dockerfile*: contains all the commands to assemble the image.
* *green-controller.json*: this file specifies the replication controller green pod.
* *green-service.json*: this file specifies the green service.
* *index.html*: simple html file that makes up the application.
* *Jenkinsfile*: this file contains the pipeline which is the main goal of this project.
* */projectScreenshots*: this folder contains a number of screenshots to show the correct workings of the project.

## Contributing

This repository contains all the work that makes up the final capstone project.
