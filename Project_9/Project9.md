### PROJECT 9 - TOOLING WEBSITE DEPLOYMENT AUTOMATION WITH CONTINUOUS INTEGRATION. INTRODUCTION TO JENKINS

#### Task
Enhance the architecture prepared in Project 8 by adding a Jenkins server, configure a job to automatically deploy source codes changes from Git to NFS server.

Here is how the updated architecture will look like upon competition of this project:
![Alt text](images/add_jenkins.png)


### STEP 1 - Install Jenkins server

**NB**: Jenkins is a Java based application, so we must install some java dependencies so that Jenkins can run effectively.

1. Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"

2. Install [JDK](https://en.wikipedia.org/wiki/Java_Development_Kit) (since Jenkins is a Java-based application)

```
sudo apt update
sudo apt install default-jdk-headless
```
3. Install Jenkins

```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins
```
![Alt text](images/2.jpg)

To ensure Jenkins is running successfully, run 
```
sudo systemctl status jenkins
```
![Alt text](images/3.jpg)

4. Jenkins server uses TCP port 8080 by default, therefor the inbound rules in security group should be set accordingly.

![Alt text](images/4.jpg)

5. Perform initial Jenkins setup. 
 - From the browser, input the Jenkins server public IP or Public DNS name then add ``:8080`` .

 ``http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080`` 

Then run the command below to retrive the password required to proceed with Jenkins set up
```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
``` 
![Alt text](images/5.jpg)

In the 'Customize Jenkins window', choose 'Install suggested plugins'
![Alt text](images/6.jpg)

After the plugins install, create and admin user get your Jenkins server address.

![Alt text](images/9.jpg)


### STEP 2 - Configure Jenkins to retrieve source codes from GitHub using Webhooks

In this part, we will configure a simple Jenkins job/project. This job will be triggered by GitHub [webhooks](https://en.wikipedia.org/wiki/Webhook) and will execute a ‘build’ task to retrieve codes from GitHub and store it locally on Jenkins server.

1. Firstly, enable webhooks in GitHub repository settings

![Alt text](images/webhook_github.gif)

2. Go to Jenkins web console, click “New Item” and create a “Freestyle project”

![Alt text](images/create_freestyle.png)

To connect your GitHub repository, you will need to provide its URL, you can copy from the repository itself

![Alt text](images/github_url.png)

In configuration of your Jenkins freestyle project choose Git repository, provide there the link to your Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.

![Alt text](images/github_add_jenkins.png)

Save the configuration and let us try to run the build. For now we can only do it manually.
Click “Build Now” button, if you have configured everything correctly, the build will be successfull and you will see it under ``#1``

![Alt text](images/16.jpg)

You can open the build and check in “Console Output” if it has run successfully.


If so – congratulations! You have just made your very first Jenkins build!

But this build does not produce anything and it runs only when we trigger it manually. Let us fix it.

3. Click “Configure” your job/project and add these two configurations
Configure triggering the job from GitHub webhook:

![Alt text](images/15.jpg)

Configure “Post-build Actions” to archive all the files – files resulted from a build are called “artifacts”.
![Alt text](images/archive_artifacts.gif)

Now, go ahead and make some change in any file in your GitHub repository (e.g. ``README.MD`` file) and push the changes to the main branch.

You will see that a new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on Jenkins server.

With that done successfully, you have now configured an automated Jenkins job that recieves files from GitHub by Webhook trigger (this method is consisdered as 'push' because the changes are being 'pushed' and files transfer is initiated by Github). There are also other methods: trigger one job (downstream) from another (upstream), poll GitHub periodically and others.

By default, the artifacts are stored on Jenkins serve locally.
```
ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/

```
![Alt text](images/17i.jpg)

### STEP 3 – Configure Jenkins to copy files to NFS server via SSH

Now we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to ``/mnt/apps`` directory.

Jenkins is a highly extendable application and there are 1400+ plugins available. We will need a plugin that is called [“Publish Over SSH”](https://plugins.jenkins.io/publish-over-ssh/).

1. Install "Publish over SSH" plugin.

On main dashboard select “Manage Jenkins” and choose “Manage Plugins” menu item.

On “Available” tab search for “Publish Over SSH” plugin and install it 

![Alt text](images/18.jpg)

2. Configure the job/project to copy artifacts over to NFS server.
On main dashboard select “Manage Jenkins” and choose “Configure System” menu item.

Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server:

1. Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)
2. Arbitrary name
3. Hostname – can be ``private IP address`` of your NFS server
4. Username – ``ec2-user`` (since NFS server is based on EC2 with RHEL 8)
5. Remote directory – ``/mnt/apps`` since our Web Servers use it as a mointing point to retrieve files from the NFS server
Test the configuration and make sure the connection returns ``Success``. Remember, that TCP port 22 on NFS server must be open to receive SSH connections.

![Alt text](images/19.jpg)

Save the configuration, open your Jenkins job/project configuration page and add another one “Post-build Action”

![Alt text](images/send_build.png)

Configure it to send all files produced by the build into our previously define remote directory. In our case we want to copy all files and directories – so we use ``**``.
If you want to apply some particular pattern to define which files to send – [use this syntax.](http://ant.apache.org/manual/dirtasks.html#patterns)

![Alt text](images/send_build1.png)

Save this configuration and go ahead, change something in ``README.MD`` file in your GitHub Tooling repository.

Webhook will trigger a new job and in the “Console Output” of the job you will find something like this:

![Alt text](images/20.jpg)

To make sure that the files in ``/mnt/apps`` have been updated – connect via SSH/Putty to your NFS server and check README.MD file

```
cat /mnt/apps/README.md
```


If you see the changes you had previously made in your GitHub – the job works as expected.

![Alt text](images/21.jpg)
The changes as expected.

Implementation of Continuous Integration solution using Jenkins CI was successful.