
## MIGRATING APPLICATIONS/SOLUTIONS TO THE CLOUD USING DOCKER

### Containerization using **DOCKER**

Software packages and applications have to be deployed to make their usage possible, and example is in the **[Enterprise website deployment](https://github.com/Johnstx/DevOps_Projects/tree/main/Project_19)**, where an infrastructure was designed using terraform and ansible. This concept is simple and may be the use case for light workload/ requirement. 
But, imagine situations where multiple small apps need to be deployed - web front-end, - web-backend, - processing jobs, monitoring, logging solutions, etc. with unique OS/runtime/dependency requirements and also higher load requirements, scaling these apps will be tedious, exhausting and expensive to maintain.

A much better idea would be to bundle the application with its dependencies and requirements - libraries and configurations in a way it runs uniformly in different environments. This concept is called **Containerization**
#### DOCKER TECHNOLOGY

Docker is written in the [Go programming language](https://golang.org/) and takes advantage of several features of the Linux kernel to deliver its functionality. Docker uses a technology called namespaces to provide the isolated workspace called the container. When you run a container, Docker creates a set of namespaces for that container.

These namespaces provide a layer of isolation. Each aspect of a container runs in a separate namespace and its access is limited to that namespace.



This project will implement containerization using **[Docker](https://www.docker.com)** , the concept is known as Dockerization. 

#### LETS DOCKERIZE AN APPLICATION

Prerequisites:-

1. 
