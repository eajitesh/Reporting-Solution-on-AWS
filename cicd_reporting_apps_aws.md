# Continuous Delivery of Reporting Apps to AWS 

Following is done to continuously deliver reporting app and containerized ETL jobs to AWS:

* Configure push-based trigger from BitBucket to Jenkins
* Configure Jenkins to integrate with BitBucket
* CI/CD of reporting APIs on AWS Elastic Beanstalk (EB)
* CI/CD of containerized ETL jobs on EC2


## Configure Push-based Trigger from BitBucket to Jenkins

* **Add the inbound rule on CI box on AWS** for following IP: 104.192.143.192/28. This is BitBucket cloud IP outbound address (for hooks like POST) when inbound IP address for BitBucket is 104.192.143.1. I figured this IP (104.192.143.1) by pinging bitbucket.org. The outbound IP address was found from this page: https://confluence.atlassian.com/bitbucket/what-are-the-bitbucket-cloud-ip-addresses-i-should-use-to-configure-my-corporate-firewall-343343385.html
* **Add following URL while creating WebHook in BitBucket**:  http://ci.fieldrepo.com:8080/bitbucket-hook/ . URL takes the format of *http://<repository_url>/bitbucket-hook/*
* **Configure Git as Source code management tool within Jenkins**: Following is a sample of input parameter:
** Repository_URL: https://ajitraksan@bitbucket.org/ajitraksan/smartetl_mongo_elasticsearch.git
** Credentials: Create a credential by providing bitbucket username and password
* **Setup Build Triggers**:  Check the option "Build when a change is pushed to BitBucket"

## Configure Jenkins to integrate with BitBucket

Following needs to be setup/configured within Jenkins in order to integrate Jenkins with BitBucket:
 

## CI/CD of Reporting APIs on AWS Elastic Beanstalk (EB)

Following are different aspects of achieving continuous integration/continuous deployment (Jenkins as CI/CD server) of Reporting APIs to AWS Elastic Beanstalk:

* **Configure push-based trigger from Bitbucket to Jenkins**: Follow the steps mentioned in above section. Note that Jenkins is setup on AWS EC2 instance. 
* **Setup AWS EB CLI on Jenkins Server**: Setup AWS EB CLI on Jenkins server
* **Configure Jenkins user to execute docker command**: As part of Post Steps - Execute Shell, it would be required to execute docker command. This is where one would required to do following:
** Add  NOPASSWD option to allow for promptless execution of sudo command: "%jenkins ALL=(ALL) NOPASSWD: ALL" 
* **Configure Post Steps** to execute docker commands to build the image, push the image to image repository, initialize elasticbeanstalk, deploy the container.

### Configure Post Steps to Execute Docker Commands & Shell Scripts
 
Following is achieved as part of configuring post-steps:

* Build the docker image
* Push the image to docker repository
* Initialize elastic beanstalk (EB)
* Deploy the container

Following is the code that would go as a shell script in "Post Steps - Execute Shell". 

```
# Remove existing image
sudo docker rmi ajitesh/srapis:latest

# Build the fresh docker images
sudo docker build -t ajitesh/srapis:latest /var/lib/jenkins/workspace/reporting_apis

# Login into dockerhub
sudo docker login -u="dockerhub_username" -p="dockerhub_password"

# Push the image to dockerhub
sudo docker push ajitesh/srapis:latest

# Go to directory consisting of Dockerrun.aws.json
cd /var/lib/jenkins/workspace/reporting_apis/aws-eb

# Execute the ebinit.sh script
./ebinit.sh

# Deploy the container
eb deploy
```

### Initialize EB - Promptless "EB INIT" execution

Following is the code for ebinit.sh which facilitates the promptless execution of "eb init" command. Do note the numeric value used for specifying default region and application to use. This value comes from executing the "eb init" command on command prompt and noting down the appropriate number prior to setting in the script. There can be better ways to achieve this. 

```
#!/usr/bin/expect -f
set timeout -1
spawn $env(SHELL)
match_max 100000
send -- "eb init\r"
expect  "Select a default region"
send -- "6\r"
expect "Select an application to use"
send -- "2\r"
send -- "exit\r"
expect eof
```

#### Install Expect tool
Note that one may have to install expect on the Jenkins server for above to work. Following code can be used to install Expect:
```
apt install expect
```

## CI/CD of Containerized ETL Job (MongoDB to ElasticSearch) on AWS EC2

One may recall that due to lack of support of AWS ECS or AWS Batch in Mumbai region, the solution architecture was updated to use EC2 instance for running the containerized ETL job. Doing continuous delivery on EC2 for these containerized jobs would mean the removal of existing docker image and setting up of fresh image from docker image repository. 
 
Following are different aspects of achieving continuous integration/continuous deployment of containerized ETL job to AWS EC2:

* **Configure push-based trigger from Bitbucket to Jenkins**: Follow the steps mentioned in the first section. Note that Jenkins is setup on AWS EC2 instance. 
* **Configure Jenkins user to execute docker command**: As part of Post Steps - Execute Shell, it would be required to execute docker command. This is where one would required to do following:
** Add  NOPASSWD option to allow for promptless execution of sudo command: "%jenkins ALL=(ALL) NOPASSWD: ALL" 
* **Configure Post Steps** to execute docker commands to build the image, push the image to image repository, SSH to EC2, and execute docker commands to  setup docker image.

### Configure Post Steps to Execute Docker commands
 
Following is achieved as part of configuring post-steps:

* Build the docker image
* Push the image to docker repository
* SSH login to EC2 instance; One can use AWS SSM command to remotely execute the commands on one or more EC2 instance. However, at this moment, SSM is not supported for Mumbai region. So, this method is used.
* Execute commands (on remote EC2 instance) such as remove the existing docker image and setup new docker image.

Following is the code that would go as a shell script in "Post Steps - Execute Shell". 
```
# Remove existing image
sudo docker rmi ajitesh/etlmongo2es:latest

# Build the fresh docker images
sudo docker build -t ajitesh/etlmongo2es:latest /var/lib/jenkins/workspace/ETL_Mongo_ElasticSearch

# Login into dockerhub
sudo docker login -u="dockerhub_username" -p="dockerhub_password"

# Push the image to dockerhub
sudo docker push ajitesh/etlmongo2es:latest

# Re-install the docker image on EC2 instance
cat /var/lib/jenkins/scripts/setup_etlmongo2es.sh | ssh  -i /pems/pem_35.154.143.91.pem -tt ubuntu@ec2-35-154-143-91.ap-south-1.compute.amazonaws.com
```

#### Code - Setup_etlmongo2es.sh
```
docker rmi -f ajitesh/etlmongo2es
docker pull ajitesh/etlmongo2es:latest
exit
exit
```
