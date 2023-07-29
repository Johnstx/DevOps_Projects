### PROJECT 8 LOAD BALANCER SOLUTION WITH APACHE

In the previous project, 3 identical web servers were created to share same conten of tooling website to a DevOps team. This concept/practice is neccessary to avoid crash and overload especially when a single web server is expected to feed heavy traffic requests. This concept however, will not be easily achieved, considering that these 3 web servers have their different IPs and any client trying to access the content would have to memorize these IPs. This is very complex when applied in real life scanario. 
However, Load Balancing technology allows us access uniform content from more than 1 web servers connected to a remote database and NFS server.
A Load Balancer (LB) distributes clients’ requests among underlying Web Servers and makes sure that the load is distributed in an optimal way. 

Read about different [Load Balancing](https://www.nginx.com/resources/glossary/load-balancing/) concepts and difference between [L4 Network LB](https://www.nginx.com/resources/glossary/layer-4-load-balancing/) and [L7 Application LB](https://www.nginx.com/resources/glossary/layer-7-load-balancing/).

Let us take a look at the updated solution architecture with an LB added on top of Web Servers (for simplicity let us assume it is a software L7 Application LB, for example – [Apache](https://httpd.apache.org/docs/2.4/mod/mod_proxy_balancer.html), [NGINX](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/) or [HAProxy](http://www.haproxy.org/))

![Alt text](images/Tooling-Website-Infrastructure-wLB.png)

In this project we will enhance our Tooling Website solution by adding a Load Balancer to disctribute traffic between Web Servers and allow users to access our website using a single URL.
