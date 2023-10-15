# Devops Assignment

## Overview

This repository contains four sections: section-1, section-2, section-3, section-4. 


### Section-1 AWS Deployment

This section represents section-1 diretory and its AWS Deployment for a web application . 

The directory, section-1/source/, includes files or sample files that are used to prove that task is completed. for example: source/frond-end/index.html to upload into the S3 bucket.

The directory, section-1/templates/, includes cloudformation templates for both frond-end and back-end requirements: S3 with Cloudfront and EC2 with VPC & Security Group resepctively. Details are as follows:

| S/N | File Name | Function(s) |
| :-: | :------- | :--------- |
| 1 | templates/front-end/web-app-frontend-s3.yaml | Create a S3 bucket, a bucketpolicy and Cloudfront distirbution. |
| 2 | templates/backend/web-app-backend-vpc.yaml | Create vpc, route tables, public subnet (No Instance / Empty), private subnet (backend server instance). |
| 3 | tempaltes/backend/web-app-backend-sg.yaml | Create security group, EC2 instance. For design to be used togehter with frond-end, I consider using Route 53 to point to Elastic IP so that frond-end codes can fetch data from backend server through Route 53 (via browser). For simplicity, I dont include Route 53 implementaiton.  |
| 4 |back-end/policy.json | A policy to create a IAM role so that DevOps person can assume to create both frond-end and backend stacks. Stacks can be created using via AWS Cloudformation UI or aws cli command as below.|


```
$ aws cloudformation deploy --stack-name <stack-name> --template-file <template-file>
```

#### Steps to run Section-1

##### 1. Create Role with policy.json

Create a role using policy.json so that any IAM user (for example: DevOps Personnel) can assume role to execute the stacks. It will help to limit the the permissions (Only necessary permissions should be given). If we are automating through a tool, for example, AWS CodeDeploy, we could use this role to assign to the service user. 

##### 2. Create Front-end Stack

Create a front-end Stack via AWS CloudFormation UI using templates/front-end/web-app-frontend-s3.yaml . Once completed, upload source/front-end/index.html to the bucket. You may use aws s3 cp command to upload (for example: With CodeDeploy or other tool). 

From Browser where you have logged into AWS using your IAM account, request "https://dxxxxxxxx.cloudfront.net" to see whether index.html content is loaded. Exact Distribution name can be found in cloudfront page after stack is successfully executed.

Please note that there is a single template for front-end for simplicity. We may think of separating to multiple templates such as one for security gateway, one for EC2 instance. By separating, we could re-use the templates for creating common stacks. 

##### 3. Create Backend Stacks
There will be two stacks for backend: VPC and EC2 instance Stacks. 

1. Create VPC Stack using templates/back-end/web-app-backend-vpc.yaml 
2. Create EC2 and Security Group Stack using templates/backend/web-app-backend-sg.yaml 

Optional: Route 53 stack (not included here)

Verify All resources which are created through AWS UI. 
You may consider ssh into EC2 using keypair pem file. Keypair can be retrieve back via terminal as follows:

Use the describe-key-pairs command as follows to get the ID of the key pair.
```
aws ec2 describe-key-pairs --filters Name=key-name,Values=new-key-pair --query KeyPairs[*].KeyPairId --output text
```
The following is example output.
```
key-05abb699beEXAMPLE
```
Use the get-parameter command as follows to get the parameter for your key and save the key material in a .pem file.
```
aws ssm get-parameter --name /ec2/keypair/key-05abb699beEXAMPLE --with-decryption --query Parameter.Value --output text > new-key-pair.pem
```

### Section 2: On-Prem Server Deployment

#### Prerequisites
- Configure a user
- Install/Upgrade AWS CLI
- Create a service role
- What type of web-application would be deployed? Microsoft Based (for example: ASP.net ) or Opensource (For example: Java/React/Angular) ? Then, we could decide which server (Operation System) we will require. If ASP or similar, use Microsoft Windows Server. Otherwise, use RedHat Enterprise or Ubuntu.
- As said it is a simple web applocation, we shall choose relatively small capacity VM.


#### Steps
##### 1. Configure on-premises instance
If we are using AWS CodeDeploy, we can register an on-premises instance by using an IAM role ARN. 
For example:
```
register-on-premises-instance --instance-name <value> --other-necessary-arguments <value>
```

Or we could be using the existing on-premise tools such as Jenkins to execute CI/CD pipelines.

##### 2. Create, Bundle and Deploy
Deploy applications using IIS or Apache or Tomcat or other opensource relevant server. 
- Check if it is a clean state VM or server.
- Create directories as necessary
- Check user's permission on VM or Server
- Follow deployment procedure as given from development team(s)

##### 3. Verify All deployments
- Verify the deployment
- Test the availabiltiy of critical end-points
- Dry Run with users

##### 4. Clean up
- clean up resources which are used or stored in server after sucessful deployment 

#### 5. Security & Updates
- SSL/TLS are properly deployed , encryption is verified 
- Loggings and Audits are verified to be integrated with other channels such as ELK, Cloud Monitoring Tools
- All softwares are updated, patches and its controls are done regualarly and scheduled
- Check firewall and ports to confirm only necessary ports are open. 

### Section-3: CI/CD Techniques
This section is for deploying 3 microservices using Jenkins at Cloud based services (e.g., ECS Fargate or EC2) or on-premises server (e.g., Kubernetes or similar) or VMs.

#### Microservices to Deploy
Code base for each microservice below should be avaiable on Github as App-1, App-2, App-3 repositories. Please note that these repo names will be used respectively.

| S/N | App Name | Repository | Assumption |
| :---: | :--------: | :----------: | :-----------: |
| 1   | Microservice A | Own Repo | API Client or UI Client |
| 2   | Microservice B | Own Repo | API Service |
| 3   | Microservice C | Own Repo | A Database (RDBMS or NoSQL) |

#### Prerequisites
1. Install AWS CLI on the Jenkins Machine.
2. Jenkins to have Git integrated and authenticated properly.
3. Assumption: Code repositories of all 3 microservices are housed in GitHub. Webhooks for each repo shall be created so that Jenkins can know when repository is updated. 

#### Steps to build, test and deploy microservices
##### 1. Connect to Jenkins
###### Create an access key
Go to Amazon Console, then IAM, then Users, [your user], then Security credentials, and then Create Access Key. 

###### Use Access key in Jenkins to authenticate to AWS
Go to Manage Jenkins, then Manage Credentials, then Jenkins Store, then Global Credentials (unrestricted), and Add Credentials.
Fill in the respective fields such as kind, id, access key id, secret access key. Click OK to save.


##### 2. Create repos on AWS ECR using the Jenkins pipeline via GitHub
Go to the Jenkins Dashboard, then New Item.

Give your pipeline a name and select the Pipeline item, then OK.

Webhooks are created for each repo in Github. for example: http://[jenkins-url]/github-webhook-app-1 

Add jenkins pipelines files from section-3/pipelines/build-and-publish to the root level of repositories respectively. 

Remember to replace [ECR-Repository-id] for each microservices. 

##### 3. Clone, Build, Test and Push microservices

Upload both .Jenkinfile and deployment yaml files from deploy-cluster directory into the root level of each repository.

Setup Jenkins pipelines accordingly for each microservice.


##### 3. Deploy to EKS Cluster for Each microservice

If running manually, do the following: Otherwise, Jenkinsfile will take care of those.

Run apply k8s command;
kubectl apply -f deployment-cluster-app-1.yml
kubectl apply -f deployment-cluster-app-2.yml
kubectl apply -f deployment-cluster-app-3.yml

Check commands:

kubectl get all

kubectl get pod — watch

kubectl get service

### Section-4: Monitoring Techniques


#### Section 1
Default metrics such as requests can be used to display number of viewer requests coming to front-end website. If required, additional metrics such as Origin latency, error rates can be enabled to monitor total time spent when cloudfront receiving a request(s), number of error (for example: 401, 403, 502, etc).

#### Section 2
We can have cloudwatch agent installed on on-premises server as mentioned in https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/install-CloudWatch-Agent-on-premise.html. 

Once installed and configured, metrics will be sent to cloudwatch by agents. 




