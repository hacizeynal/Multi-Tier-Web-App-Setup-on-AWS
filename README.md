# Multi-Tier-Web-Application-Setup on AWS Cloud

## Introduction

Following technologies are used in this project

Nginx <br/>
Tomcat <br/>
RabbitMQ \
Memcached <br/>
MySQL <br/>


![](https://file%2B.vscode-resource.vscode-cdn.net/Users/zhajili/Desktop/DevOps/DevOps%20Projects/AWS%20for%20WebApp%20Setup%20Lift%20and%20Shift/Screenshot%202022-10-03%20at%2022.23.41.png?version%3D1666023926588)


The request will be coming from a client browser and traffic will be redirected to the Nginx server, Nginx will be used as Load Balancer, it will forward requests to the Tomcat Application Server, Apache will be our server which our Java Application will be running. We can use shared storage as NFS. Our application server will forward traffic to the RabbitMQ which will be used as Message Broker. Message Broker will forward requests to the Memcached for Database caching which will cache our queries for MySQL.

5 EC2 instance will be created on AWS Cloud and each EC2 will be created with respective user-data/script.

