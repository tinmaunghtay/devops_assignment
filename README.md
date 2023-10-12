## devops assignment

Welcome to devops assignment Github! This project will contain four sections: section-1, section-2, section-3, section-4. 


### Section-1

This section represents section-1 diretory. 

The folder section-1/source/ will include files or sample files that are used to prove that task is completed. for example: source/frond-end/index.html to upload into the S3 bucket.

The folder section-1/templates/ includes templates for both frond-end and back-end requirements: S3 with Cloudfront and EC2 with VPC & Security Group resepctively. Details are as follows:
1. templates/front-end/web-app-frontend-s3.yaml will create S3 bucket, bucketpolicy and Cloudfront distirbution.
2. templates/backend/web-app-backend-vpc.yaml will create vpc, route tables, public subnet (No Instance / Empty), private subnet (backend server instance). 
3. tempaltes/backend/web-app-backend-sg.yaml will create security group, EC2 instance. For design to be used togehter with frond-end, I consider using Route 53 to point to Elastic IP so that frond-end codes can fetch data from backend server through Route 53 (via browser). For simplicity, I dont include Route 53 implementaiton. 
4. A policy (back-end/policy.json) to create role so that DevOps person can switch role to create both frond-end and backend stacks. Stacks can be created using via AWS Cloudformation UI or command as below.

```
$ aws cloudformation deploy --stack-name <stack-name> --template-file <template-file>
```

#### Steps to run Section-1

##### Create Role with policy.json

Create a role using policy.json so that any IAM user (for example: DevOps Personnel) can assume role to execute the stacks. It will help to limit the the permissions (only necessary permission shall be given).

##### Front-end

Create Stack via AWS CloudFormation UI using templates/front-end/web-app-frontend-s3.yaml . Once completed, upload source/front-end/index.html to the bucket. You may use aws s3 cp command to upload (for example: With CodeDeploy or other tool). 

From Browser where you have logged into AWS using your IAM account, request "https://dxxxxxxxx.cloudfront.net" to see whether index.html content is loaded. Distribution name can be found in cloudfront page.

##### Backend
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

### Section-2

#### Prerequisites
- Configure a user
- Install/Upgrade AWS CLI
- Create a service role
- What type of web-application ? Microsoft Based (ASP) or Opensource (Java/React/Angular) ? Then, we can decide which server (Operation System) we will require. If ASP or similar, use Microsoft Windows Server. Otherwise, use RedHat Enterprise or Ubuntu.
- As said it is a simple web applocation, we shall choose relatively small capacity VM.


#### Steps
##### 1. Configure on-premises instance
If we are using AWS CodeDeploy, we can register an on-premises instance by using an IAM role ARN. 
For example:
```
register-on-premises-instance --instance-name <value> --other-necessary-arguments <value>
```

Or we may be using the existing on-premise tools such as Jenkins to execute CI/CD pipelines.

##### 2. Create, Bundle and Deploy
Deploy applications using IIS or Apache or Tomcat or other opensource relevant server. 
- Check if it is a clean state VM or server.
- Create directories as necessary
- Check permission on VM or Server
- Follow deployment procedure as given from development team(s)

##### 3. Verify All deployments
- Verify the deployment
- Test the availabiltiy of critical end-points
- Dry Run with users

##### 4. Clean up
- clean up resources which are used or stored in server after sucessful deployment 

#### 4. Security & Updates
- SSL/TLS are properly deployed , encryption is verified 
- Loggings and Audits are verified to be integrated with other channels such as ELK, Cloud Monitoring Tools
- All softwares are updated, patches and its controls are done regualarly and scheduled
- Check firewall and ports to confirm only necessary ports are open. 

### Section-3

### Section-4


