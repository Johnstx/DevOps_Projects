#### LOAD BALANCER SOLUTION WITH NGINX AND SSL/TLS

Features of this task include:
1. Nginx
2. SSL/TLS certificate configurations

### STEP 1 - CONFIGURE NGINX AS A LOAD BALANCER

1. Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it ``Nginx LB`` (do not forget to open TCP port 80 for HTTP connections, also open TCP port **443** – this port is used for secured HTTPS connections)

2. Update ``/etc/hosts`` file for local DNS with Web Servers’ names (e.g. ``Web1`` and ``Web2``) and their local IP addresses 

3. Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers

Update the instance and Install Nginx

```
sudo apt update
sudo apt install nginx
```
![Alt text](images/6.jpg)

Configure Nginx LB using Web Servers’ names defined in ``/etc/hosts``

**Hint**: Read this blog to read about ``/etc/host``

Open the default nginx configuration file

```
sudo vi /etc/nginx/nginx.conf
```

```
#insert following configuration into http section

 upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
  }

server {
    listen 80;
    server_name www.domain.com;
    location / {
      proxy_pass http://myproject;
    }
  }

#comment out this line
#       include /etc/nginx/sites-enabled/*;
```
![Alt text](images/8.jpg)

Restart Nginx and make sure the service is up and running

```
sudo systemctl restart nginx
sudo systemctl status nginx
```
![Alt text](images/9.jpg)

### STEP 2 - REGISTER A NEW DOMAIN NAME AND CONFIGURE SECURED CONNECTION USING SSL/TLS CERTIFICATES

Now, lets make all neccessary configurations to encrypt the connection to the Tooling Web Solution.

In order to get a valid SSL certificate – you need to register a new domain name, you can do it using any Domain name registrar – a company that manages reservations of domain names. Some examples - [godaddy.com](https://godaddy.com/), [domain.com](https://www.domain.com/), [namecheap.com](https://www.namecheap.com/)

1. Register a new domain name with any registrar of your choice in any domain zone (e.g. ``.com``, ``.net``, ``.org``, ``.edu``, ``.info``, ``.xyz`` or any other)

2. Assign an Elastic IP to your Nginx LB server and associate your domain name with this Elastic IP

You might have noticed, that every time you restart or stop/start your EC2 instance – you get a new public IP address. When you want to associate your domain name – it is better to have a static IP address that does not change after reboot. Elastic IP is the solution for this problem, learn how to allocate an Elastic IP and associate it with an EC2 server [on this page](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html).

![Alt text](images/15.jpg)

3. Update[ A record](https://www.cloudflare.com/learning/dns/dns-records/dns-a-record/) in your registrar to point to Nginx LB using Elastic IP address

![Alt text](images/16.jpg)

Learn how associate your domain name to your Elastic IP on [this page](https://medium.com/progress-on-ios-development/connecting-an-ec2-instance-with-a-godaddy-domain-e74ff190c233).

Check that your Web Servers can be reached from your browser using new domain name using HTTP protocol – ``http://<your-domain-name.com>``

4. Configure Nginx to recognize your new domain name

Update your ``nginx.conf`` with ``server_name www.<your-domain-name.com>`` instead of ``server_name www.domain.com``
![Alt text](images/12.jpg)

5. Install [certbot](https://certbot.eff.org/) and request for an SSL/TLS certificate

Make the [snapd](https://snapcraft.io/snapd) service is active and running

```
sudo systemctl status snapd
```
![Alt text](images/17.jpg)

Install certbot

```
sudo snap install --classic certbot
```
![Alt text](images/18i.jpg)

Request your certificate (just follow the certbot instructions – you will need to choose which domain you want your certificate to be issued for, domain name will be looked up from ``nginx.conf`` file so make sure you have updated it on step 4).

```
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
```

Test secured access to your Web Solution by trying to reach ``https://<your-domain-name.com>``

You shall be able to access your website by using HTTPS protocol (that uses TCP port 443) and see a padlock pictogram in your browser’s search string.
Click on the padlock icon and you can see the details of the certificate issued for your website.

![Alt text](images/20.jpg)

![Alt text](images/22.jpg)

6. Set up periodical renewal of your SSL/TLS certificate

By default, LetsEncrypt certificate is valid for 90 days, so it is recommended to renew it at least every 60 days or more frequently.

You can test renewal command in ``dry-run`` mode

``sudo certbot renew --dry-run``

Best practice is to have a scheduled job that to run ``renew`` command periodically. Let us configure a ``cronjob`` to run the command twice a day.

To do so, lets edit the crontab file with the following command:

```
crontab -e
```
Add following line:

```
* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1
```

You can always change the interval of this cronjob if twice a day is too often by adjusting schedule expression.

Nginx Load Balancing Web Solution with secured HTTPS connection with periodically updated SSL/TLS certificates HAS JUST BEEN IMPLEMENTED. 

Merci!!