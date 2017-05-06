# Deep Dive into Creating Reporting Framework on AWS

The goal of this project is to create a custom reporting solution/framework using AWS services for fulfilling the requirement of creating Reporting Dashboard displaying multiple analytics reports representing data from different data sources. 

The reporting solution/framework architecture is based on [ReportingData](https://martinfowler.com/bliki/ReportingDatabase.html) design pattern recommended by [Martin Fowler](https://martinfowler.com/). As per the recommended design pattern, given the fact that transactional/operational database and reporting database fulfill different data requirements/needs, it would be good to have different databases for managing transactions and reporting data. 

Taking above into consideration, following represents key building blocks of technology architecture of reporting solution/framework:

* ETL jobs to extract-transform-load data from different data sources to reporting database
* Reporting application that exposes reporting data through REST APIs

Following could be different AWS services which can be used for storing reporting data:

* AWS RDS with databases such as MYSQL
* AWS ElasticSearch (ES)

We shall take into consideration both of the above AWS storage service for our reporting database.

Following could be some of the different solution approaches which can be used to create reporting solution/framework using different AWS services: 

* AWS Batch 
* AWS ECS
* AWS EC2
