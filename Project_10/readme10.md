
### LOAD BALANCER SOLUTION WITH NGINX AND SSL/TLS

To expand our experience on load balancing, we will employ the use of [Nginx](https://www.nginx.com/) load balancer solution.

We will also ensure that connections to the web solutions are secure and that informasion is [encrypted in transit](https://security.berkeley.edu/data-encryption-transit-guideline) - we will also cover connection over secured HTTP (HTTPS protocol) using[ SSL/TSL technologies](https://en.wikipedia.org/wiki/Secure_Sockets_Layer), this is employed to prevent attacks called [ Man-In-The-Middle (MIMT) attack.](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)


SSL and its newer version, TSL – is a security technology that protects connection from MITM attacks by creating an encrypted session between browser and Web server. Here we will refer this family of cryptographic protocols as SSL/TLS – even though SSL was replaced by TLS, the term is still being widely used.

SSL/TLS uses [digital certificates](https://en.wikipedia.org/wiki/Public_key_certificate) to identify and validate a Website. A browser reads the certificate issued by a [Certificate Authority (CA)](https://en.wikipedia.org/wiki/Certificate_authority) to make sure that the website is registered in the CA so it can be trusted to establish a secured connection.

There are different types of SSL/TLS certificates – you can learn more about them [here](https://blog.hubspot.com/marketing/what-is-ssl). You can also watch a tutorial on how SSL works [here](https://youtu.be/T4Df5_cojAs) or an additional resource [here](https://youtu.be/SJJmoDZ3il8)

In this project you will register your website with [LetsEnrcypt](https://letsencrypt.org/) Certificate Authority, to automate certificate issuance you will use a shell client recommended by LetsEncrypt – [certbot](https://certbot.eff.org/).

This project consists of two parts:

1. Configure Nginx as a Load Balancer
2. Register a new domain name and configure secured connection using SSL/TLS certificates

The Target architecture will look like this:

![Alt text](images/nginx_lb.png) 