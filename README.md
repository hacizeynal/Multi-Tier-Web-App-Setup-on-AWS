# Multi-Tier-Web-Application-Setup on AWS Cloud

## Introduction

Following technologies are used in this project

Nginx <br/>
Tomcat <br/>
RabbitMQ \
Memcached <br/>
MySQL <br/>



The request will be coming from a client browser and traffic will be redirected to the Nginx server, Nginx will be used as Load Balancer, it will forward requests to the Tomcat Application Server, Apache will be our server which our Java Application will be running. We can use shared storage as NFS. Our application server will forward traffic to the RabbitMQ which will be used as Message Broker. Message Broker will forward requests to the Memcached for Database caching which will cache our queries for MySQL.

5 EC2 instance will be created on AWS Cloud and each EC2 will be created with respective user-data/script.

Following AWS services will be used in this project

-- Route53 <br/>
-- IAM <br/>
-- ACM <br/>
-- VPC <br/>
-- EC2 <br/>


