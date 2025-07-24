# vProfile Re-Architected Deployment using AWS Managed Services

This folder contains the re-architected deployment of the **vProfile Java web application** using AWS Platform-as-a-Service (PaaS) offerings, replacing the traditional EC2 setup with fully managed AWS services.

---

## ✅ Project Overview

The goal of this setup is to minimize operational overhead by leveraging AWS-managed services, while maintaining the multi-tier architecture of the application.

### 📦 Services Used:

* **AWS Elastic Beanstalk** for application hosting
* **Amazon MQ (RabbitMQ)** as the message broker
* **Amazon ElastiCache (Memcached)** for caching
* **Amazon RDS (MySQL)** as the database
* **Amazon S3** for artifact storage
* **Amazon CloudFront** as CDN
* **AWS ACM** for SSL
* **GoDaddy** for domain management (vprofile.in)

---

## 🏗️ Architecture Diagram

```
Users
  |
[CloudFront + ACM SSL + GoDaddy DNS]
  |
[AWS Application Load Balancer (ALB)]
  |
[Elastic Beanstalk (Tomcat Platform)]
  |
  |---> Amazon MQ (RabbitMQ)
  |---> Amazon ElastiCache (Memcached)
  |---> Amazon RDS (MySQL)
```

---

## 🔨 Steps to Deploy

### 1. 📦 Build and Upload WAR File

```bash
mvn clean install
aws s3 cp target/ROOT.war s3://<your-bucket-name>/ROOT.war
```

### 2. ☁️ Create Elastic Beanstalk Environment

* Platform: **Tomcat 8/Java 8 or 11**
* Environment: Web Server (Load Balanced, Auto Scaling)
* Upload the WAR file directly from local or S3

### 3. ⚙️ Environment Configuration

Set the following environment properties in **Application.properties** which is present at "*\src\main\resources\application.properties":

```properties
jdbc.url=jdbc:mysql://vprofile-rearch-rds.cghugysoox62.us-east-1.rds.amazonaws.com:3306/accounts?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull
jdbc.username=user
jdbc.password=passwd

#Memcached Configuration For Active and StandBy Host
#For Active Host
memcached.active.host=vprofile-rearch-cache.j3k1vz.cfg.use1.cache.amazonaws.com
memcached.active.port=11211
#For StandBy Host
memcached.standBy.host=127.0.0.2
memcached.standBy.port=11211

#RabbitMq Configuration
rabbitmq.address=b-e53bc5a2-b1b2-43dd-99f8-1608b4ab4698.mq.us-east-1.on.aws
rabbitmq.port=5671
rabbitmq.username=user
rabbitmq.password=passwd
```

### 4. 🔐 Setup ACM + HTTPS

* Create public certificate in **AWS ACM** for `vprofile.in`
* Validate via **CNAME in GoDaddy DNS**
* Attach certificate to Beanstalk Load Balancer HTTPS listener

### 5. 🧭 Domain Mapping

* In GoDaddy:

  * Create a CNAME or A Record pointing to **CloudFront** or **Beanstalk ELB** DNS

### 6. 🌐 Add CloudFront (Optional)

* Create a CloudFront distribution
* Origin: Beanstalk Load Balancer DNS
* Attach ACM certificate
* Update GoDaddy to point domain to CloudFront URL

---

## 📁 File Structure (This Folder)

```
vprofile_refactor/
├── screenshots/              # Setup evidence/screens
├── README.md                 # This file
```

----------------------------------
Database
Here,we used Mysql DB sql dump file:

/src/main/resources/db_backup.sql
db_backup.sql file is a mysql dump file.we have to import this dump to mysql db server
mysql -u <user_name> -p accounts < db_backup.sql

----------------------------------

## ✅ Outcome

* Elastic Beanstalk auto-scales and manages app lifecycle
* No EC2 provisioning or manual service setup
* HTTPS via ACM and CloudFront
* Integration with fully-managed Amazon MQ, ElastiCache, RDS
* Clean separation of layers and environment variables

---

> 💡 This re-architecture demonstrates a cloud-native, production-grade DevOps deployment on AWS using only managed services and platform tools.
