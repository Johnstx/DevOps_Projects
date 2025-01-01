
## MIGRATING APPLICATIONS/SOLUTIONS TO THE CLOUD USING DOCKER

### Containerization using **DOCKER**

This project provides a walkthrough for deploying an application in a container, a concept called containerization.
This technology which has been widely adopted because it has simplified the building and deployments in environments where mulitple applications are required to run to provide a business solution.

### Benefits of its use.
If you are a DevOps engineer, developers will often send you applications to deploy, your environment has to be configured and set up like the developers's if the application must run, if not it won't and the dev will most times say "but it runs on my machine". 
Containerization enables the dev and the team to deploy application so it can run in a container by packaging the application in a [dockerfile](https://docs.docker.com/reference/dockerfile/) and then building it into a [docker image](https://docs.docker.com/reference/cli/docker/image/).

This project highlights the key steps in achieving a successful migration of an app or apps.
This lab will migrate a custom application, **[Enterprise website deployment](https://github.com/Johnstx/DevOps_Projects/tree/main/Project_19)** which was previously deployed via a VM based environment to the cloud employing use of docker  container.

### Technologies and Tools applied :-
* [Docker Engine](https://docs.docker.com/engine/install/)
* [An account in Docker registry](https://hub.docker.com/)
* [Jenkins](https://www.jenkins.io/doc/)

### **Prerequisites:-**

1. Install [Docker Engine](https://docs.docker.com/engine/), a client-server application that contains: 
 * A server with a long-running daemon process dockerd.
 * APIs that specify interfaces that programs can use to talk to and instruct the Docker daemon.
 * A command-line interface (CLI) client docker.
Install docker engine [here](https://docs.docker.com/engine/install/)
