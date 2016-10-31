# Let's Chat on Bluemix Overview

<p align="center">
  <img alt="Let's Chat" src="http://i.imgur.com/0a3l5VF.png" />
</p>
<p align="center"><strong>running on</strong></p>
<p align="center">
  <img alt="IBM Containers" src="https://encrypted-tbn2.gstatic.com/images?q=tbn:ANd9GcSwFL7rmMXcMDxIM0m0hiLPwJFeE23l3puGJj78bjPBYhQ78xvcZw" />
</p>


This repository will show you how to run the open-source [_Let's Chat_][lets_chat_url] application on [IBM Containers][ibm_containers_url] in [IBM Bluemix][bluemix_signup_url].  The provided Dockerfiles, scripts, and pipeline file will build & deploy the necessary Docker images for the Let's Chat application and link the running container to a MongoDB service instance in IBM Bluemix.

IBM Containers is an Enterprise-grade Docker container service, available on IBM Bluemix.  Provided here are all the necessary artifacts to build and deploy the sample application and deploy it on IBM Containers, leveraging additional IBM Bluemix services where appropriate.  As additional capabilities are made available through IBM Bluemix, this sample application repository will be updated to take advantage of them as appropriate.

## Deploy to Bluemix
**(requires [pre-req steps](#setup-bluemix-pre-requisites) to be run first)**  
[![Deploy to Bluemix](https://bluemix.net/deploy/button.png)](https://bluemix.net/deploy)

## How it works
I'll let the [SD Elements][lets_chat_url] team describe Let's Chat here:
> Let's Chat is a persistent messaging application that runs on Node.js and MongoDB. It's designed to be easily deployable and fits well with small, intimate teams.

Similar to Slack, it's quick to start and even simpler to use.  Once the application is deployed, follow the steps below to start collaborating with your teammates or audience immediately!

![Let's Chat Screenshot](http://i.imgur.com/C4uMD67.png)

1.  Once the Let's Chat environment is up and running, you will need to create users.  The easiest way to do this is to create a new user, by clicking on **I need an account**.  Enter the necessary information, making sure the email is unique, and click **Register**.  

2.  After creating an initial user, enter your registered user's credentials and click **Sign in**.

3.  You'll see all the rooms listed upon logging in, along with any other users who are currently logged into the system.  Enter an existing room or create a new room.  

4.  Begin chatting with your teammates!  

5.  Additional configuration documentation is available on the [Let's Chat Wiki][lets_chat_wiki_url], for direction on authentication, self-registration, and login throttling.

## Repository structure
**`.bluemix/`** - The exported YAML document describing the Advanced Pipeline below.  
**`Dockerfile`** - The Dockerfile necessary for building the Docker image for Let's Chat to run on IBM Containers.  
**`scripts/`** - The startup script which the Dockerfile uses as it's entrypoint executable.  It calls a [extract-vcap.py][extract_vcap_url] utility script to extract the Mongo credentials from the embedded VCAP_SERVICES.  
For more information on IBM Containers and the interaction with VCAP_SERVICES, you can read [this blog post][containers_bluemix_blog] on the Bluemix blog.  

## Running the app on Bluemix
Let's Chat is a straight-forward NodeJS application, requiring only a MongoDB as its sole datastore.  IBM Bluemix provides you with all the necessary services to run Let's Chat with minimal management overhead.  Instead of deploying and managing your own MongoDB server, this sample application leverages one of the available MongoDB services available on Bluemix today.  The provided build pipeline then dynamically links the running container instances with the available MongoDB service instance at deploy time, making this a very portable and repeatable deployment process.

To deploy the sample application, follow the steps below.  Integration with the **Deploy to Bluemix** button is being worked on and will be available soon.  This walkthrough will make use of the IBM Bluemix DevOps Services Delivery Pipeline capability to build the Docker images and deploy them to running container instances on IBM Containers.  You can also perform the same activities via the IBM Containers command line (_cf ic_) if desired.

### Setup Bluemix pre-requisites
1. Create a Bluemix Account  
[Sign up][bluemix_signup_url] for Bluemix, or use an existing account.
2. Enable your Bluemix Account for usage of IBM Containers  (_Optional_ - only required if not already completed)  
Select an existing Space or create a new one in the Bluemix Dashboard and click **START CONTAINERS**.  You will be prompted to create a private Docker registry in Bluemix.  
**Note:** This cannot be changed once it is created.  So keep it short and confined to something you wouldn't be embarrassed to share with your peers.
3.  Create the Bluemix bridge application  
 1.  This can be done from either the Bluemix UI or from the CloudFoundry CLI.
 2.  This walkthrough will use the name **lets-chat-bridge** for reference.  Make note of the name used, as it will be needed to bind the container instances to later on.
4.  Create the Compose MongoDB service instance  
 1.  In another browser tab, sign up for a *Free 30-Day Trial* at [Compose](https://compose.io)
 2.  Create a MongoDB deployment.  Make note of the host, port, user, and password for the Mongo Console, available under **Admin > Connection Strings**.
 3.  Back in the Bluemix Dashboard, create a new instance of the **MongoDB by Compose** service using the information from the Compose account you just created.
 4.  If you are prompted to restage your application, click **RESTAGE** and wait a few moments for your application to be running again.

### Create your Bluemix DevOps Services project
5.  Fork the [current repository][current_repo_url].
6.  Login to [Bluemix DevOps Services][devops_services_url] and create a new project.
7.  Enter a name for your new project.  **lets-chat-demo** will be used here for reference.
8.  Select **Link to an existing GitHub repository** and **Link to a Git repo on GitHub**.  Select the newly forked repository.  
9.  Ensure under **Make this a Bluemix Project**, that the space selected is the space you deployed your bridge application to above.  
10.  Click **CREATE** and wait for your Bluemix DevOps Services project to be created.  Now whenever any changes are made in your forked GitHub repository, they will flow into this project and kick off the Delivery Pipeline you will configure in the next step.

### Configure a Basic Delivery Pipeline
> Use this Basic Pipeline to deploy a single Let's Chat container with a public IP, accessed via http://{public_ip}:8080  

21.  Once created, go to the **BUILD & DEPLOY** tab of your project.  
22.  Create a **Build** stage & associated job with the following properties:  
  1.  **Builder Type:**  IBM Container Service  
  2.  **Space:**  Your container-enabled space configured in [Pre-req Step 2](#setup-bluemix-pre-requisites)  
  3.  **Image Name:**  lets-chat-bmx  
  3.  **Build Script:**  The default build script is acceptable.
23.  Create a **Deploy** stage & associated job with the following properties:  
  1.  **Deployer Type:** IBM Containers on Bluemix  
  2.  **Space:**  Your container-enabled space configured in [Pre-req Step 2](#setup-bluemix-pre-requisites)  
  3.  **Name:** lets-chat-single  
  4.  **Port:**  8080,5222  
  5.  **Deployer Script:**  The default deployer script is acceptable.  
  6.  **Environment Properties**  
    1.  **BIND_TO**  lets-chat-bridge (or your app name used in [Pre-req Step 3](#setup-bluemix-pre-requisites))  
	2.  **CONTAINER_SIZE**  tiny  
24.  This pipeline will now build whenever a commit is pushed to the forked repository.  Optionally, you can click the **Run Stage** button in the Build stage to kick off the delivery pipeline.  
25.  Once the Build and Deploy stages have completed successfully, you can access the running Let's Chat server by the public IP address assigned.  This is available through the log of the deploy stage, the Bluemix UI, or the `cf ic ip list` command.  

## API documentation
There is no API made available through this sample application.

### Useful links
#### IBM Bluemix
* [IBM Bluemix](https://bluemix.net/)  
* [IBM  Bluemix Documentation](https://www.ng.bluemix.net/docs/)  
* [IBM Bluemix Developers Community](http://developer.ibm.com/bluemix)

#### IBM Containers
* [IBM Containers][ibm_containers_url]
* [IBM Containers Service Overview video](https://www.youtube.com/watch?v=WMUiBE_7MoU)
* [IBM Containers and Bluemix Services blog post][containers_bluemix_blog]

_Let's Chat on Bluemix_ is a sample application created for the purpose of demonstrating a Docker application on IBM Containers. The program is provided as-is with no warranties of any kind, express or implied.

[bluemix_signup_url]: https://console.ng.bluemix.net/?cm_mmc=Display-GitHubReadMe-_-BluemixSampleApp-LetsChat-_-Docker-_-BM-Containers
[bluemix_catalog_url]: https://console.ng.bluemix.net/catalog/
[bluemix_dashboard_url]: https://console.ng.bluemix.net/?direct=classic/#/resources
[devops_services_url]: https://hub.jazz.net/
[cloud_foundry_url]: https://github.com/cloudfoundry/cli
[current_repo_url]: https://github.com/osowski/lets-chat-bluemix
[gist_lets_chat_build_url]: https://gist.github.com/osowski/8dca076ac07b5069aabe#file-lets-chat-build-sh
[gist_nginx_build_url]: https://gist.github.com/osowski/8dca076ac07b5069aabe#file-nginx-build-sh
[gist_lets_chat_deploy_url]: https://gist.github.com/osowski/8dca076ac07b5069aabe#file-lets-chat-deploy-cluster-member-sh
[gist_nginx_deploy_url]: https://gist.github.com/osowski/8dca076ac07b5069aabe#file-nginx-deploy-sh
[extract_vcap_url]: https://github.com/osowski/ibm-containers/blob/master/utils/docker-build/extract-vcap.py
[containers_bluemix_blog]: https://developer.ibm.com/bluemix/2015/07/06/simplifying-distributed-docker-applications/
[ibm_containers_url]: https://console.ng.bluemix.net/solutions/open-architecture/
[lets_chat_url]: http://sdelements.github.io/lets-chat/
[lets_chat_wiki_url]: https://github.com/sdelements/lets-chat/wiki
