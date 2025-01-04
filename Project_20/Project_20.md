

## MIGRATING AN APPLICATION/ SOLUTION TO THE CLOUD THROUGH CONTAINERIZATION

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

As a Proof Of Concept (POC), will use a simple DevOps tooling PHP website using MySQL as a Database. *the db, which is part of the architecture will be deployed in a container*


#### [MySQL set-up]()
*We will use  a pre-built MySQL database **container**, configure it to our taste and ensure request from PHP application is possible.*

### 1. Pull MySQL Docker Image from [Docker Hub Registry](https://hub.docker.com/)

Pull the appropriate [docker image for MySQl](https://hub.docker.com/_/mysql). *Download a specific version or go for the latest release. (**note the tags**)*.

* To download the image, run - 
```
docker pull mysql/mysql-server:8.0
```

* Confirm the image download
```
docker images ls
```
![docker pull and image list](<images/1 - docker pull and image ls.jpg>)

### 2. Deploy the MySQL container

1. *After image pull is passed, next launch the container with -*

```
docker run --name toolingdb -e MYSQL_ROOT_PASSWORD=custompassword -d mysql/mysql-server:8.0
```
![MySQL container](<images/2 - docker run.jpg>)


2. *Confirm status of the container*

```
docker ps -a
```
![docker ps](<images/3 - docker ps.jpg>)


*You should see the newly created container listed in the output. It includes container details, one being the status of this virtual environment. The status changes from health: starting to healthy, once the setup is complete.*

**Step 3: Connecting to the MySQL Docker Container**
*Connection to the database container can be achieved through  two ways.*
***Approach 1**: Connecting directly to the container running the MySQL server*.

***Approach 2**: Use a second container as a MySQL client*



### **Approach 1**
Direct connection to the container running the MySQL server.
```
docker exec -it toolingdb mysql -uroot -p
```
![Connection to the DB](<images/4 - approach one - direct connection.jpg>)

Provide the root password when prompted. With that, you have connected the MySQL client to the server.
Finally, change the server root password to protect your database.

### **Approach 2**
First, create a network:

Creating a custom network is not necessary because even if we do not create a network, Docker will use the default network for all the containers you run. By default, the network we created above is of DRIVER Bridge. 
Verify by running the ```docker network ls``` command.

But there are use cases where this is necessary. For example, if there is a requirement to control the cidr range of the containers running the entire application stack. This will be an ideal situation to create a network and specify the --subnet.

For clarity's sake, we will create a network with a subnet dedicated for our project and use it for both MySQL and the application so that they can connect.

```
docker network create --subnet=172.18.0.0/24 tooling_app_network 
```

Run the MySQL Server container using the created network
But before that,  create an environment variable to store the root password:
```
export MYSQL_PW=<root-secret-password>
```
Then, pull the image and run the container, all in one command like below:

```
docker run --network tooling_app_network -h mysqlserverhost --name=mysql-server -e MYSQL_ROOT_PASSWORD=$MYSQL_PW  -d mysql/mysql-server:8.0
```
![deploying conainer within a docker network](<images/6 - mysql-server with network.jpg>)

Flags used

    * -d runs the container in detached mode
    * --network connects a container to a network
    * -h specifies a hostname
If the image is not found locally, it will be downloaded from the registry.

Run ``docker ps -a`` to view the created resource.

Connecting to the MySQL server.

It is best practice not to connect to the MySQL server remotely using the root user. 
Therefore, create an SQL script that will create a user we can use to connect remotely.

Create a file and name it ``create_user.sql`` and add the below code in the file:

```
CREATE USER 'staxx'@'%' IDENTIFIED BY 'movement';
GRANT ALL PRIVILEGES ON * . * TO 'staxx'@'%';
```
*NB*: Change <staxx> and <movement> to your unique name and password respectively
![MySQL script](<images/7 - script for mysql-server user.jpg>)


Run the script :
```
docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < ./create_user.sql
```

**NB**: If you see a warning like below, it is acceptable to ignore:

```
mysql: [Warning] Using a password on the command line interface can be insecure.

```
![alt text](<images/10 - dbconnect-php.jpg>)



### 3. Connecting to the MySQL server from a second container running the MySQL client utility

The upside about this approach is that you do not have to install any client tool on your laptop, and you do not need to connect directly to the container running the MySQL server.

Run the MySQL Client Container:

```
docker run --network tooling_app_network --name mysql-client -it --rm mysql mysql -h mysqlserverhost -u staxx -p
```


![run MySQL script](<images/8 - connecting to mysql server with a mysql client using the user in the script.jpg>)

Flags used:

* --name gives the container a name
* -it runs in interactive mode and Allocate a pseudo-TTY
* --rm automatically removes the container when it exits
* --network connects a container to a network
* -h a MySQL flag specifying the MySQL server Container hostname
* -u user created from the SQL script
* -p password specified for the user created from the SQL script



### 4. Prepare database schema
Now you need to prepare a database schema so that the Tooling application can connect to it.
First, clone the source code of the tooling app repo

1. Clone the repo from [here](https://github.com/Johnstx/tooling.git)

```
git clone https://github.com/Johnstx/tooling.git
```

2. Export the location of the SQL file

```
export tooling_db_schema=~/tooling/html/tooling_db_schema.sql
```
Find the ``tooling_db_schema.sql`` in the ``html`` folder of cloned ``repo.``

3. Use the SQL script to create the database and prepare the schema. With the``docker exec`` command, you can execute a command in a running container.

```
docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < $tooling_db_schema
```
4. Update the ``db_conn.php`` file with connection details to the database

```
$servername = "mysqlserverhost";
$username = "staxx";
$password = "movement";
$dbname = "toolingdb";
``` 

5. Run the Application (in a container).

Here we are implement the objective of the project, containerization.
*Containerization of an application begins with creating a file with a special name 'Dockerfile' (no extensions).*
*This 'Dockerfile' contains a configuration or processes that provides instructions to docker with ways to package the application into a container*.
Here, we will utilize a Dockerfile contained in the repo cloned earlier, however in other real life scenarios, a DevOps engineer should create one as per application use case.

To get more ideas on creating a ``Dockerfile`` and building a container from it, click [here](https://www.youtube.com/watch?v=hnxI-K10auY).

Find official Docker best practices for writing Dockerfiles [here](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

So a breakdown of steps to containerize a tooling application from  a repo (as per this case)

1. Check out the repo to your local machine which should have 'Docker Engine' running.
2. Build a 'docker image' which the tooling app will use - the 'Dockerfile' present in the repo will be used to build this 'image'.
3. Run he ``docker build`` command, specify a tag for the version of the image that will built.

```
docker build -t tooling:0.0.1 .
```
*NB: Ensure you are inside the folder that has the ```Dockerfile``` and build your container:*

*Notice the ```.``` at the end. This is important as that tells Docker to locate the Dockerfile in the current directory you are running the command. Otherwise, you would need to specify the absolute path to the Dockerfile.*

![alt text](<images/13 - docker build tooling 001.jpg>)

To confirm the build pass ok, run 
```
docker image ls
``` 
![alt text](<images/14  - docker image ls.jpg>)

4. Launch the container with ``docker run`` command.
```
docker run --network tooling_app_network -p 8085:80 -it tooling:0.0.1
```
Flags - 
 * specify the ```--network``` flag so that both the Tooling app and the database can easily connect on the same virtual network created earlier.
 * The ```-p``` flag is used to map the container port with the host port. Within the container, apache is the webserver running and, by default, it listens on ```port 80```. You can confirm this with the CMD ["start-apache"] section of the Dockerfile. But we cannot directly use ```port 80``` on our host machine because it is already in use.
The workaround is to use another port that is not used by the host machine. In our case, port 8085 is free, so we can map that to port 80 running in the container.

Run ```docker ps -a``` to confirm container runs ok

![alt text](<images/3 - docker ps.jpg>)

5. Access your application via port exposed from a container

If everything works, you can open the browser and type ```http://localhost:8085```

![alt text](<images/16 - tooling site.jpg>)

..and after log in ..

![alt text](<images/16 - tooling site 2.jpg>)

![alt text](<images/16 - tooling site devop tools.jpg>)

Migration is successful.

### Docker Registry

Docker registry is a repo where docker images are stored for easy collaborations/contributions within the teams.

In order to highlight the efficiency of containerization, an image of the application will be made availabe on the docker registry so that other members of the team  can  ```pull```, ```run``` and perform other intended task with the application. 

A ```jenkinsfile``` (in the repo) will be used to simulate the build job of the dockerfile and then push the created image to the docker registry.


### Deployment of the container with [Docker compose](https://docs.docker.com/compose/)

Docker Compose is a tool for defining and running multi-container applications. It is the key to unlocking a streamlined and efficient development and deployment experience.

Compose simplifies the control of your entire application stack, making it easy to manage services, networks, and volumes in a single, comprehensible YAML configuration file. Then, with a single command, you create and start all the services from your configuration file.

*NB: Docker compose is a neater and simpler method of migrating an app to docker. The method we used is provides an insight into the different units thats involved in containerization and its useful knowledge but in practice, I would suggest applying docker compose once you have a good understanding of it. Use link on the header above to look through its documentation.*

Lets refactor the tooling app POC to leverage on the use of docker compose - 

1. Install docker compose on your local workstation. [Click to install](https://docs.docker.com/compose/install/)
2. Create a file, name it ```tooling.yaml```
3. Start writing the Docker Compose definitions with YAML syntax. The YAML file is used for defining resources like services, networks, and volumes.

Here, ```tooling.yaml``` contents looks like this - 

```


version: "3.9"
services:
  tooling_frontend:
    build: .
    image: johnstx/tool
    ports:
      - "5000:80"
    volumes:
      - tooling_frontend:/var/www/html
    links:
      - db
  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_DATABASE: "toolingdb"
      MYSQL_USER: "staxx"
      MYSQL_PASSWORD: "movement"
      MYSQL_ROOT_PASSWORD: "password1"
    volumes:
      - db:/var/lib/mysql
volumes:
  tooling_frontend:
  db:
```

Then run the command to start the containers

```
 docker-compose -f tooling.yaml  up -d 
```
![alt text](<images/pipeline - 6 - compose tooling.jpg>)


To Verify that the compose is in the ```running``` status:

```
docker compose ls
``` 
![alt text](<images/pipeline - 7 - compose ls.jpg>)


### Push image to registry with jenkinsfile

A file named ```jenkinsfile`` should contain a pipeline to carry out these jobs.
For this, we used the jenkins pipline below:-

```
   pipeline {

        environment {
        registry = "johnstx/tooling" 
        registryCredential = 'dockerhub-login' // Jenkins credentials ID for Docker Hub
        image_name = "johnstx/tool"
        Docker_compose_file = "tooling.yaml"
               // BUILD_TAG = 'master-0.0.1'
        }
        
        agent any

        stages {


        stage('Clean Workspace') {
          steps {
                    cleanWs() // Cleans the workspace before running the pipeline
          }
        }


          stage ('Checkout SCM') {
            steps {
              git branch: 'compose' , url : 'https://github.com/Johnstx/Tooling-Docker-.git',   credentialsId: 'github-login'
            }
          }



          // stage('Docker Build image') {
          //   steps {
          //       script {
          //           //  docker.withRegistry( '' ,  'dockerhub-login' ) {
          //                 // Build the Docker image and assign it to dockerImage 
          //                 dockerImage = docker.build("${env.REGISTRY}:${env.BUILD_TAG}")
          //                  }
          //       }
          //   }


           stage('Docker Build image') {
            steps {
                script {
                    //  docker  build using docker-compose
                          sh "docker-compose -f ${Docker_compose_file} build"
                           }
                }
            }

          stage ('tag the docker image') {
            steps {
              script {
                sh "docker image tag ${image_name }:latest ${registry}:cmpse-0.0.1"
                 } 
                }
          }

          stage('Deploy docker image to docker hub') {
            steps {
              script {
                         // log in to docker hub
                      docker.withRegistry ( '', registryCredential) {
                            // dockerImage.push()
                            // push the tagged image
                            sh "docker push ${registry}:cmpse-0.0.1"
                 }
              }
            }
          }
      

        //   stage('Clean up'){
        //      steps{
        //         script {
        //             // sh "docker rmi $registry:latest"
        //             sh "docker rmi ${registry}:feature-0.0.1"
        //      }
        //   }
        // }
 stage('Clean up') {
            steps {
                script {
                    // Clean up local Docker images to save space
                    sh "docker rmi ${image_name }:latest"
                    sh "docker rmi ${registry}:cmpse-0.0.1"
                  }
             }
         }
        }
   }
```
**To view the full Lab, *click the link* [here](https://github.com/Johnstx/Tooling-Docker-.git)**


The pipeline should go through like below:

![alt text](<images/pipeline - 8 - docker-compose  - complete - .jpg>)

Image pushed to docker registry - a job specified in the pipeline.

![alt text](<images/pipeline - 8 - docker-compose  -registry.jpg>)


Ensure that the tooling site http endpoint is able to return status code ```200```. Any other code will be determined a stage failure.

![alt text](<images/pipeline - 7 - browser.jpg>)






Related Projects on Migrations
[migrating the PHP-Todo app into a containerized application]()