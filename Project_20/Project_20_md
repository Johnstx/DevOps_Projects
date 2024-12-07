

## MIGRATING APPLICATIONS/SOLUTIONS TO THE CLOUD USING DOCKER

### Containerization using **[DOCKER](https://www.docker.com)**

Software packages and applications have to be deployed to make their usage possible, an example is in the **[Enterprise website deployment](https://github.com/Johnstx/DevOps_Projects/tree/main/Project_19)**, where an infrastructure was designed using terraform and ansible and runs  in a VM/EC2 environment, which a single OS.
But, imagine situations where multiple small apps need to be deployed - web front-end, - web-backend, - processing jobs, monitoring, logging solutions, etc. with different OS/runtime/dependency requirements and also higher load requirements. Scaling these apps will be tedious, exhausting and expensive to maintain accross different OS environments.

A much better idea would be to bundle the application with its dependencies and requirements - libraries and configurations in a way it runs uniformly in different environments. This concept is called **Containerization**, and Docker uses **Dockerfiles** to bundle all the components of a software into a portable and executable **Docker image** ( - proces called Dockerization)


### DOCKER TECHNOLOGY

Docker is written in the [Go programming language](https://golang.org/) and takes advantage of several features of the Linux kernel to deliver its functionality. Docker uses a technology called namespaces to provide the isolated workspace called the container. When you run a container, Docker creates a set of namespaces for that container.

These namespaces provide a layer of isolation. Each aspect of a container runs in a separate namespace and its access is limited to that namespace.
This enables apps in a container to operate sufficiently within the isolation irrespective of the OS kernel of the infrastructure, as opposed to the VM environment where there has to be  OS - Ppplications compatibilty for the application to run. 

Apps deployed in containers can therefore run in any OS, this makes this implementation suitable  For lightweight, scalable, and cloud-native applications that don't require full OS isolation.



This project will implement containerization using **[Docker](https://www.docker.com)** 


### LETS DOCKERIZE AN APPLICATION

As a Proof Of Concept (POC), will use a simple DevOps tooling PHP website using MySQL as a Database. *both ends will make use of dockerization concept*


### MySQL set-up
*We will use  a pre-built MySQL database **container**, configure it to our taste and ensure request from PHP application is possible.*

!. 