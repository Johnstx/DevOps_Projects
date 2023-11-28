
## Project 15 - 

### AWS Cloud Solution For 2 Company Websites Using A Reverse Proxy Technology

Planned Network infrastructure for **wiska.store**
![Alt text](<images/Network infrastructure.png>)


#### *Some pre-requisites for this job* -
1. Configuration of AWS account and Organization Unit - *[watch video here](https://youtu.be/9PQYCc_20-Q)*
- AWS Master account was created. (Also known as Root Account)
- Within the Root account, a sub-account was created and named **DevOps**. (An email address is required here)
- Within the Root account, An AWS **Organization Unit (OU)** was created. Named it Dev. (We will launch **Dev** resources in there)
- **DevOps** account is moved into the **Dev** *OU*.
- Login to the newly created AWS account using the new email address.

2. A free domain name for wiska.store is created, using namecheap domain register. [Learn more](https://www.freenom.com/)

3. Create a AWS hosted zone and map to the free domain. A hosted zone is created on AWS and mapped to the new domain (wiska.store) - [Learn how to](https://youtu.be/IjcHp94Hq8A)


### 1. Set Up a Virtual Private Network (VPC)

    
            Tag all resources created apprioprately - 
            - Project: <Give your project a name>
            - Environment: <dev>
            - Automated: <No> (If you create a recource using an automation tool, it would be <Yes>)

1. Create a [VPC](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html).
In the VPC, click ``Actions`` and enable DNS Hostnames.
![Alt text](<images/1 vpc .jpg>)

2. Create an [Internet Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html). And attache the ``IG`` to the crated ``VPC`` 

3. Create subnets as per the architecture 
    **2 Public** Subnets and **4 Private** subnets
    ![Alt text](<images/3b subnet list.jpg>)

4. Create a (public) route table and associate it with public subnets.

5. Create a (private) route table and associate it with private subnets.
![Alt text](<images/6 associate like subnets to like RT(private to private and public to public).jpg>)
![Alt text](<images/8 edit routes.jpg>)

6. Edit a route in public route table, and associate it with the Internet Gateway. (This is what allows a public subnet to be accessible from the Internet). 

        Destination is set as 0.0.0.0/0 while the Target is set as the "Internet Gateway"

    ![Alt text](<images/8 edit routes.jpg>)

7. Create an [Elastic IPs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html)

8. Create a [Nat Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) and assign the Elastic IP to it. 
![Alt text](<images/7b NAT gateway creation before private RT edit.jpg>)

9. Edit the route in private route table, and associate it with the NAT Gateway. 
![Alt text](<images/7 a edit routes 2.jpg>)

#### SECURITY GROUPS 
10. Create a Security Group for:

    - ``Application Load Balancers``: Based on the architecture, we have external ALB and Internal ALB, So we create security group for both as they have unique roles in their layers.
    **ext-ALB** - Security group will have inbound rules from the internet ONLY (HTTPS & HTTP) and outbound rules set for all traffic.
    **int-ALB** - Allow inbound rules from the NGINX only, through HTTPS and HTTP
    - ``Nginx Servers``: Access to Nginx should only be allowed from a External application Load balancer (ALB) and SSH from the Bastion ONLY
    - ``Bastion Servers``: Access to the Bastion servers should be allowed only from workstations that need to SSH into the bastion servers. Hence, you can use your workstation public IP address. 


    

    - ``Webservers``: Access to Webservers should only be allowed from the internal Application loadbalancer (HTTPS & HTTP) and Bastion (SSH) ONLY.

    - ``Data Layer``: Access to the Data layer, which is comprised of Amazon Relational Database Service (RDS) and Amazon Elastic File System (EFS) must be carefully desinged - webservers and Bastion (mySQL) should be able to connect to RDS, while webservers ONLY will have access to EFS Mountpoint. 

      ![Alt text](<images/10 security groups.jpg>)
        Security groups created


### TLS CERTIFICATES - 

*Recall we created the domain name earlier, now we need to attach a certiticate to the domain name for further use in setting up the load balancer.*
DNS resolution with domain.
    ![Alt text](<images/11 web host.jpg>)

TLS Certificates From [Amazon Certificate Manager (ACM)](https://aws.amazon.com/certificate-manager/)
1. Navigate to AWS ACM
2. Request a public wildcard certificate for the YOUR domain name you registered in a domain register site.
3. Use DNS to validate the domain name
4. Tag the resource

    ![Alt text](<images/11b cert 2.jpg>)
    ![Alt text](<images/11c certificate for wiska.jpg>)

**Setup EFS**
[Amazon Elastic File System (Amazon EFS)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AmazonEFS.html) provides a simple, scalable, fully managed elastic Network File System (NFS) for use with AWS Cloud services and on-premises resources. 

**STEPS**
1. Create an EFS filesystem
2. Create an EFS mount target per AZ in the VPC, associate it with both subnets dedicated for data layer, in this case the private subnet-1 and private subnet-2 (where the webservers reside)
3. Associate the Security groups created earlier for data layer.
4. Create an EFS access point. (Give it a name and leave all other settings as default).
EFS Accesspoints provides a way to manages access to the EFS
Accesspoints are created for
- Tooling  
and 
- Wordpress

**Filesystem created**
    ![Alt text](<images/12a efs.jpg>)

**Accesspoint created**
   ![Alt text](<images/12b efs display accesspoints etc.jpg>)


**RDS SET-UP**-
**Pre-requisite**:
1. Create a KMS key from [Key Management Service (KMS)](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html) to be used to encrypt the database instance.
KMS Key set up
- A symmetric key (key that has an encrypt and decrypt capabilities) using KMS is created
- An alias, description and tag is inputed
- select key admins
- Assign key usage permissions to self only. Finish.


2. 2nd pe-requisite is creation of **subnet groups** under the **RDS**
- Create a **DB subnet group**
- Name it, Choose VPC, select AZ and select **subnets CIDR** of the datalayer (subnets are 3&4).
- finish

No we go ahead and create the RDS database.

- Select MySQL engine
- Select latest version
- Select the templates. To satisfy our architectural diagram, you will need to select either Dev/Test or Production Sample Template. But to minimize AWS cost, you can select the Do not create a standby instance option under Availability & durability sample template (*The production template will enable Multi-AZ deployment)*
- Configure other settings accordingly (*For test purposes, most of the default settings are good to go*). In the real world, you will need to size the database appropriately. You will need to get some information about the usage. If it is a highly transactional database that grows at 10GB weekly, you must bear that in mind while configuring the initial storage allocation, storage autoscaling, and maximum storage threshold.
- Configure VPC and security (*ensure the database is not available from the Internet*)
- Configure backups and retention
- Encrypt the database using the KMS key created earlier
- Enable [CloudWatch](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/monitoring-cloudwatch.html) monitoring and export ``Error`` and ``Slow Query`` logs (for production, also include ``Audit``)
**Note** This service is an expensinve one. Ensure to review the monthly cost before creating. (DO NOT LEAVE ANY SERVICE RUNNING FOR LONG)

### COMPUTE RESOURCES
Compute resources wil be set up and configured inside the VPC.The recources related to compute are:

- [EC2 Instances](https://www.amazonaws.cn/en/ec2/instance-types/)
- [Launch Templates](https://docs.aws.amazon.com/autoscaling/ec2/userguide/LaunchTemplates.html)
- [Target Groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html)
- [Autoscaling Groups](https://docs.aws.amazon.com/autoscaling/ec2/userguide/AutoScalingGroup.html)
- [TLS Certificates](https://en.wikipedia.org/wiki/Transport_Layer_Security)
-[ Application Load Balancers (ALB)](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html)


Compute resources is set up using CentOS for **Nginx**, **Bastion** and **Webservers**. 

### **Set Up Compute Resources for Nginx**

**Provision EC2 Instances for Nginx**


1. An EC2 Instance based on CentOS Amazon Machine is created.

2. The following softwares were installed

    * python
    * ntp
    * net-tools
    * vim
    * wget
    * telnet
    * epel-release
    * htop

For **NGINX AMI** installation, run the following - 

    yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
    yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
    yum install wget vim python3 telnet htop git mysql net-tools chrony -y
    systemctl start chronyd
    systemctl enable chronyd
    setsebool -P httpd_can_network_connect=1

**Selinux Policy**

    setsebool -P httpd_can_network_connect_db=1
    setsebool -P httpd_execmem=1
    setsebool -P httpd_use_nfs 1

**this section will instasll amazon efs utils for mounting the target on the Elastic file system**    

    git clone https://github.com/aws/efs-utils
    cd efs-utils
    yum install -y make
    yum install -y rpm-build
    make rpm 
    yum install -y  ./build/amazon-efs-utils*rpm

**Setting up self-signed certificate for the NGINX instance**

    sudo mkdir /etc/ssl/private
    sudo chmod 700 /etc/ssl/private
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/ACS.key -out /etc/ssl/certs/ACS.crt
    sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048



An NGINX ``AMI`` is created from this instance.

**BASTION ami installation**

    yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm 
    yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm 
    yum install wget vim python3 telnet htop git mysql net-tools chrony -y 
    systemctl start chronyd 
    systemctl enable chronyd

A Bastion `AMI` is created from this instance.


**WEBSERVER ami installation**

    yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
    yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
    yum install wget vim python3 telnet htop git mysql net-tools chrony -y
    systemctl start chronyd
    systemctl enable chronyd

**configure selinux policies for the webservers**

    setsebool -P httpd_can_network_connect=1
    setsebool -P httpd_can_network_connect_db=1
    setsebool -P httpd_execmem=1
    setsebool -P httpd_use_nfs 1

**Install efs-utils**

    git clone https://github.com/aws/efs-utils
    cd efs-utils
    yum install -y make
    yum install -y rpm-build
    make rpm 
    yum install -y  ./build/amazon-efs-utils*rpm

**setting up self-signed certificate for the apache webserver instance**

    yum install -y mod_ssl
    openssl req -newkey rsa:2048 -nodes -keyout /etc/pki/tls/private/ACS.key -x509 -days 365 -out /etc/pki/tls/certs/ACS.crt
    vi /etc/httpd/conf.d/ssl.conf

`AMI` is created for webserver from this instance.

### Target Groups
Target groups is created for Nginx and webservers (wordpress and tooling)

**Configure Target Groups**

- Select Instances as the target type
- Ensure the protocol HTTPS on secure TLS port 443
- Ensure that the health check path is /healthstatus
- Register **Nginx Instances** as targets for nginx target and same is done for **wordpress** and **tooling**- so we have `nginx-target`, `wordpress-target` & `tooling-target`
- Ensure that health check passes for the target group.


### Application Load Balancer (ALB) Set UP

1. Application Load Balancer To Route Traffic To NGINX

- An **Internet facing ALB** is created
- set to listen on HTTPS protocol (TCP port 443)
- ALB is created within the appropriate **VPC** | **AZ** | **Subnets** (public subnet 1 & 2)
- Choose the Certificate from **ACM** (automatically selected)
- Selected external load balancer Security Group
- Selected ``nginx-target`` as the target group


2. Application Load Balancer To Route Traffic To webservers

- Create an **Internal ALB**
- Ensure that it listens on HTTPS protocol (TCP port 443)
- Ensure the ALB is created within the appropriate VPC | AZ | Subnets
- Choose the Certificate from ACM
- Select Security Group
- Select 1 webserver (wordpress) as the default target group and create a listener rule for the tooling site webserver
- Ensure that health check passes for the target group

### Launch Template

**For Bastion**
1. Make use of the AMI to set up a launch template
2. Ensure the Instances are launched into a public subnet
3. Assign appropriate security group (bastion security group created earlier)
4. Configure ``Userdata`` to update ``yum`` package repository and install ``mysql``, ``Ansible`` and ``git``. See [Bastion-Userdata](https://github.com/Johnstx/DevOps_Projects/blob/main/Project_15/bastion-userdata.md), paste this script on the user data box.

5. Create launch template.

**For Nginx**
1. Make use of the AMI to set up a launch template
2. Ensure the Instances are launched into a public subnet
3. Assign appropriate security group
4. Configure Userdata to update yum package repository and install nginx. See the [nginx-userdata](https://github.com/Johnstx/DevOps_Projects/blob/main/Project_15/NGINX-userdata.md).
*Ensure the accesspoint are set to reflect the internal load balancer dns address*
5. Create Launch template.

**For Webserver - wordpress**
1. Make use of the AMI to set up a launch template
2. Ensure the Instances are launched into a private subnet(private subnet 1)
3. Assign appropriate security group
4. Configure Userdata to update yum package repository and install wordpress. See [wordpress-userdata](https://github.com/Johnstx/DevOps_Projects/blob/34511bd6b963b719b0d58a3a5e08a35ded1c3d59/Project_15/wordpress-userdata.md)
*Ensure to update the userdata with the efs accesspoint for wordpress*.
*Also, ensure the rds database endpoint is updated in the userdata*.
5. Create the template.


**For Tooling**
1. Make use of the AMI to set up a launch template
2. Ensure the Instances are launched into a private subnet(private subnet 1)
3. Assign appropriate security group
4. Configure Userdata to update 
*Ensure to update the userdata with the efs accesspoint for tooling*
*Also, ensure the rds database endpoint is updated in the userdata*.
5. Create the template.


### Create Autoscaling Group
**For Nginx**

1. Select the `nginx launch template`
2. Select the VPC
3. Select both public subnets
4. Enable Application Load Balancer for the AutoScalingGroup (ASG)
5. Select the ``nginx target group`` created earlier.
6. Ensure health checks for both EC2 and ALB
7. The desired capacity is 2
8. Minimum capacity is 2
9. Maximum capacity is 4
10. Set scale out if CPU utilization reaches 90%
11. Ensure there is an SNS topic to send scaling notifications.

**For Bastion**

1. Select the `Bastion launch template`
2. Select the VPC
3. Select both public subnets
4. Enable Application Load Balancer for the AutoScalingGroup (ASG)
5. Select the `bastion target group` created earlier 
6. Ensure that you have health checks for both EC2 and ALB
7. The desired capacity is 2
8. Minimum capacity is 2
9. Maximum capacity is 4
10. Set scale out if CPU utilization reaches 90%
11. Ensure there is an SNS topic to send scaling notifications


**For Wordpress**

1. Select the `wordpress launch template`
2. Select the VPC
3. Select private subnets 1 & 2
4. Enable Application Load Balancer for the AutoScalingGroup (ASG)
5. Select the `wordpress target group` created earlier
6. Ensure that you have health checks for both EC2 and ALB
7. The desired capacity is 2
8. Minimum capacity is 2
9. Maximum capacity is 4
10. Set scale out if CPU utilization reaches 90%
11. Ensure there is an SNS topic to send scaling notifications

**For Tooling**

1. Select the `tooling launch template`
2. Select the VPC
3. Select private subnets 1 & 2
4. Enable Application Load Balancer for the AutoScalingGroup (ASG)
5. Select the `tooling target group` created earlier
6. Ensure that you have health checks for both EC2 and ALB
7. The desired capacity is 2
8. Minimum capacity is 2
9. Maximum capacity is 4
10. Set scale out if CPU utilization reaches 90%
11. Ensure there is an SNS topic to send scaling notifications

### Configuring DNS with Route53
Earlier in this project you registered a free domain with Freenom and configured a hosted zone in Route53. But that is not all that needs to be done as far as DNS configuration is concerned.
You need to ensure that the main domain for the WordPress website can be reached, and the subdomain for Tooling website can also be reached using a browser.
Create other records such as CNAME, alias and A records.
NOTE: You can use either CNAME or alias records to achieve the same thing. But alias record has better functionality because it is a faster to resolve DNS record, and can coexist with other records on that name. Read here to get to know more about the differences.

Create an alias record for the root domain and direct its traffic to the ALB DNS name.
Create an alias record for tooling.<yourdomain>.com and direct its traffic to the ALB DNS name.


Check the healthchecks to ensure they are healthy

![Alt text](<images/healthchecks nginx.jpg>)
![Alt text](<images/healthchecks wp.jpg>)

Copy the domain names and test on the browser and check the site availabilty, if successful both tooling and wordpress sites should go live!

![Alt text](<images/wp webpage.jpg>)
![Alt text](<images/tooling web page.jpg>)

.. Job Done. 
