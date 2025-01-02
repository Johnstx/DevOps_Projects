
## MIGRATING APPLICATIONS/SOLUTIONS TO THE CLOUD WITH CONTAINERIZATION

### Containerization using **DOCKER**

This project provides a walkthrough for deploying an application in a container, a concept called containerization.
This technology which has been widely adopted because it has simplified the building and deployments in environments where mulitple applications are required to run to provide a business solution.

### Benefits of its use.
If you are a DevOps engineer, developers will often send you applications to deploy, your environment has to be configured and set up like the developers's if the application must run, if not it won't and the dev will most times say "but it runs on my machine". 
Containerization enables the dev and the team to deploy application so it can run in a container by packaging the application in a [dockerfile](https://docs.docker.com/reference/dockerfile/) and then building it into a [docker image](https://docs.docker.com/reference/cli/docker/image/).
This will be of benefit to the team in numerous ways.


This project  will be used as Proof of Concept (POC) and will highlight the key steps in achieving a successful migration of an app or apps.
we will migrate a custom application, **[a tooling application](https://github.com/Johnstx/DevOps_Projects/tree/main/Project_19)** which was previously deployed via a VM based environment to the cloud employing using Terraform (IAC), Ansible and Jenkins.

This project will be run on a local workstation, another option would be to spin up an EC2 instance and run the subsequent and whole project in the cloud infrastructure like AWS.

### Technologies and Tools used
* [Docker Engine](https://docs.docker.com/engine/install/)
* [An account in Docker registry](https://hub.docker.com/)
* [Jenkins](https://www.jenkins.io/doc/)

###  STEPS

1. Install the client tools - [Docker Engine](https://docs.docker.com/engine/), a client-server application that contains: 
    * A server with a long-running daemon process dockerd.
    * APIs that specify interfaces that programs can use to talk to and instruct the Docker daemon.
    * A command-line interface (CLI) client docker.

**NB:** The tooling application used here as POC is an app deployed by the tech team to apply DevOps concepts  in their work environment. 

2. [Pull MySQL Docker Image from Docker Hub Registry](https://github.com/Johnstx/DevOps_Projects/blob/main/Project_20).
3. [Deploy the MySQL Container to your Docker Engine](https://github.com/Johnstx/DevOps_Projects/blob/main/Project_20).
4. [Connecting to the MySQL Docker Container]()
5. [Prepare database schema]()
6. [Clone the Tooling-app repository]()
7. [cd into the folder containing the dockerfile]()
8. [Build the docker image]()
9. [Run the container]()
10. [Redo above using a more simplified method, Docker compose]()