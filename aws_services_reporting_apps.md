# AWS Services for Reporting Application

One can simply use AWS Elastic Beanstalk (EB) to quickly deploy a web application. Following approach was used to create and deploy reporting ap using AWS EB:

* Create Reporting application using Springboot; Expose the REST APIs using @RestController 
* Dockerize the reporting app
* Create an application on AWS EB which is based on [Single Container Docker platform configuration](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker.html).
* Create a Dockerrun.aws.json specifying the container details. Following is the sample code:
```
{
  "AWSEBDockerrunVersion": "1",
  "Image": {
    "Name": "ajitesh/srapis",
    "Update": "true"
  },
  "Ports": [
    {
      "ContainerPort": "8080"
    }
  ],
  "Logging": "/var/log"
}
```
* Deploy the app manually using AWS EB application console or using Jenkins using instructions mentioned on the following page: [Continuous Deplivery of Reporting Apps to AWS](https://github.com/eajitesh/Reporting-Solution-on-AWS/blob/master/cicd_reporting_apps_aws.md)

One could as well use multi-container docker platform configuration of AWS EB or [AWS ECS](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html) to deploy the reporting app/APIs.
