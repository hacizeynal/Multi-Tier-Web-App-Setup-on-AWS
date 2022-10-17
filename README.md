## Multi-Tier-Web-Application-Setup on AWS Cloud

### Introduction

We will deploy local WebApp to AWS with Lift and Shift Approach.

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

Before creating artifacts we will update application.properties in src/resources with new DNS name which we created on Route53 on AWS.

Following internal IP addresses with respective DNS names are created ,internal IP address will be changed when we create new EC2 instance.

Verification of Route53 DNS records

[![Screenshot-2022-10-17-at-19-33-30.png](https://i.postimg.cc/vm2CCD28/Screenshot-2022-10-17-at-19-33-30.png)](https://postimg.cc/94Tx7Xjs)


```
db.devops.com 172.18.161.112
rabbit.devops.com 172.18.174.49
memcache.devops.com 172.18.164.196
```
We will execute mvn -install command on the directory where our pom.xml exists and after successfull execution target directory will be created.

```
\King Julien$ ls -l target 

drwxr-xr-x   7 zhajili  staff       224 17 Oct 18:25 classes
drwxr-xr-x   3 zhajili  staff        96 17 Oct 19:28 generated-sources
drwxr-xr-x   3 zhajili  staff        96 17 Oct 19:28 generated-test-sources
-rw-r--r--   1 zhajili  staff     86178 17 Oct 19:28 jacoco.exec
drwxr-xr-x   3 zhajili  staff        96 17 Oct 19:28 maven-archiver
drwxr-xr-x   3 zhajili  staff        96 17 Oct 19:28 maven-status
drwxr-xr-x   3 zhajili  staff        96 17 Oct 19:28 site
drwxr-xr-x  10 zhajili  staff       320 17 Oct 19:28 surefire-reports
drwxr-xr-x   3 zhajili  staff        96 17 Oct 18:25 test-classes
drwxr-xr-x   5 zhajili  staff       160 17 Oct 19:28 vprofile-v2
-rw-r--r--   1 zhajili  staff  48451120 17 Oct 19:28 vprofile-v2.war
\King Julien$ 

```
After creating artifact we will use AWS CLI to upload artifact to AWS S3 bucket.

First we will create S3 bucket with below command and copy our artifact there.

```
\King Julien$ aws s3 mb s3://devops-project-storage
make_bucket: devops-project-storage

\King Julien$ aws s3 cp vprofile-v2.war s3://devops-project-storage/vprofile-v2.war
upload: ./vprofile-v2.war to s3://devops-project-storage/vprofile-v2.war

\King Julien$ aws s3 ls s3://devops-project-storage 
2022-10-17 21:13:51   48451120 vprofile-v2.war

```
In order to download artifact to Tomcat EC2 instance ,we will create a role which will be used by EC2.This role will have AmazonS3FullAccess access like below JSON format.

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "sts:AssumeRole"
            ],
            "Principal": {
                "Service": [
                    "ec2.amazonaws.com"
                ]
            }
        }
    ]
}

After assinging IAM role to EC2 instance we will login to EC2 instance and we will verify if tomcat is still running.

```
ubuntu@ip-172-18-167-220:~$ sudo -i
root@ip-172-18-167-220:~# systemctl status tomcat8
● tomcat8.service - LSB: Start Tomcat.
   Loaded: loaded (/etc/init.d/tomcat8; generated)
   Active: active (running) since Sun 2022-10-16 23:14:19 UTC; 20h ago
     Docs: man:systemd-sysv-generator(8)
    Tasks: 30 (limit: 1134)
   CGroup: /system.slice/tomcat8.service
           └─19884 /usr/lib/jvm/java-8-openjdk-amd64/bin/java -Djava.util.logging.config.file=/var/lib/tomcat8/conf/logging.prop
```
We will delete ROOT directory and will download artifact here ,we will need to install AWS CLI as well.
We will move artifact and rename it to ROOT.war in order to make it default application.


```

root@ip-172-18-167-220:~# cd /var/lib/tomcat8/
conf/    lib/     logs/    policy/  webapps/ work/    
root@ip-172-18-167-220:~# cd /var/lib/tomcat8/webapps/
root@ip-172-18-167-220:/var/lib/tomcat8/webapps# ls -la
drwxr-xr-x 3 root    root    4096 Oct 16 23:14 ROOT
root@ip-172-18-167-220:/var/lib/tomcat8/webapps# systemctl stop tomcat8
root@ip-172-18-167-220:/var/lib/tomcat8/webapps# rm -rf ROOT/

root@ip-172-18-167-220:/var/lib/tomcat8/webapps# apt install awscli -y
root@ip-172-18-167-220:/var/lib/tomcat8/webapps# aws s3 ls s3://devops-project-storage
2022-10-17 19:13:51   48451120 vprofile-v2.war


root@ip-172-18-167-220:/var/lib/tomcat8/webapps# aws s3 cp s3://devops-project-storage/vprofile-v2.war /tmp
download: s3://devops-project-storage/vprofile-v2.war to ../../../../tmp/vprofile-v2.war

root@ip-172-18-167-220:/var/lib/tomcat8/webapps# ls -l /tmp/

-rw-r--r-- 1 root    root    48451120 Oct 17 19:13 vprofile-v2.war

root@ip-172-18-167-220:/var/lib/tomcat8/webapps# cp /tmp/vprofile-v2.war /var/lib/tomcat8/webapps/ROOT.war
root@ip-172-18-167-220:/var/lib/tomcat8/webapps# systemctl start tomcat8


```
We can do small verification test to DB via telnet and check if we have reachibility.

```
root@ip-172-18-167-220:/var/lib/tomcat8/webapps/ROOT/WEB-INF/classes# telnet db.devops.com 3306
Trying 172.18.161.112...
Connected to db.devops.com.
Escape character is '^]'.
R
5.5.68-MariaDB-$jmb&1�+DU_\1}G(0YPmysql_native_password
```






