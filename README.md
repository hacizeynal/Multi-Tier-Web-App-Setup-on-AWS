## Multi-Tier-Web-Application-Setup on AWS Cloud

### Introduction

Following technologies are used in this project

Nginx {Elastic Load Balancer} <br/> 
Tomcat {Application server} <br/> 
RabbitMQ {Backend server} <br/> 
Memcached {Backend server} <br/>
MySQL {Backend server} <br/>

[![diagram.png](https://i.postimg.cc/25R2p2Hc/diagram.png)](https://postimg.cc/tskhWtGW)

The request will be coming from a client browser and traffic will be redirected to the Nginx server, Nginx will be used as Load Balancer, it will forward requests to the Tomcat Application Server, Apache will be our server which our Java Application will be running. We can use shared storage as NFS. Our application server will forward traffic to the RabbitMQ which will be used as Message Broker. Message Broker will forward requests to the Memcached for Database caching which will cache our queries for MySQL.

5 EC2 instance will be created on AWS Cloud and each EC2 will be created with respective user-data/script.

Following AWS services will be used in this project

-- Route53 <br/> 
-- IAM <br/>
-- ACM <br/>
-- VPC <br/>
-- EC2 <br/> 

Order of operations are described below

[![Screenshot-2022-10-17-at-18-43-19.png](https://i.postimg.cc/QdXQZC17/Screenshot-2022-10-17-at-18-43-19.png)](https://postimg.cc/F7Bdj9PF)

After spin up EC2 instances we will create artifact on our laptop ,in order to do that we should have following applications on our local laptop ,let's verify it.

```
\King Julien$ mvn -version

Apache Maven 3.8.5 (3599d3414f046de2324203b78ddcf9b5e4388aa0)
Maven home: /Users/zhajili/Applications/apache-maven-3.8.5
Java version: 1.8.0_332, vendor: Amazon.com Inc., runtime: /Users/zhajili/Library/Java/JavaVirtualMachines/corretto-1.8.0_332/Contents/Home/jre
Default locale: en_GB, platform encoding: UTF-8
OS name: "mac os x", version: "12.6", arch: "x86_64", family: "mac"

\King Julien$ java -version

openjdk version "1.8.0_332"
OpenJDK Runtime Environment Corretto-8.332.08.1 (build 1.8.0_332-b08)
OpenJDK 64-Bit Server VM Corretto-8.332.08.1 (build 25.332-b08, mixed mode)

```

We will execute mvn -install command on the directory where our pom.xml exists.

Before creating artifacts we will update application.properties in src/resources with new DNS name which we created on Route53 on AWS.

Following internal IP addresses with respective DNS names are created ,internal IP address will be changed when we create new EC2 instance.

```
db.devops.com 172.18.161.112
rabbit.devops.com 172.18.174.49
memcache.devops.com 172.18.164.196
```



