

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
5. Update the ``db_conn.php`` file with connection details to the database

```
$servername = "mysqlserverhost";
$username = "staxx";
$password = "movement";
$dbname = "toolingdb";
``` 