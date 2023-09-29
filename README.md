# Autoscaling using AWS CloudFront, CloudWatch & Elastic Beanstalk

## AWS Services used *CloudFront, Auto Scaling, Cloudwatch, Elastic Beanstalk*

In this project, we are going to make an application that needs to support the high demand of a large number of users accessing it simultaneously. This application will be used in a conference for participants from all over the world to fill out their emails to be shortlisted in a raffle to win a $50 VISA gift card. 
To provide the requirement, we are going to use Amazon Elastic Beanstalk to deploy a web application and Amazon DynamoDB to the email list. We shall also use Amazon CloudFront to cache static and dynamic files to different Edge Locations to decrease latency when this application is accessed by users at various locations around the world. 

 # Solution

## Part 1: Deploying DynamoDB + Elastic Beanstalk (EC2, SG, ELB, TG, AutoScaling...)

## Step 1:  DynamoDB (Table)

• Name: users
• Partition key | Primary key: email

- Reviewing resources created: EC2, SG, ELB, TG, AutoScalling

- Creating a new key pair (optional):

Network & Security | Key Pairs | Create key pair
Name: Provide a security key pair
Private key file format: .pem or .ppk


## Step 2: Elastic Beanstalk | Validating the roles ‘elastic' created…


Creating an application

1) Configure environment

Environment tier
(*) Web server environment

Application information
Application Name: Provide app name

Platform
Platform: Python
Platform version: (Recommended)

Application code
Upload your code
Version label: version-01
(*) Public S3 URL:
https://tcb-bootcamps.s3.amazonaws.com/bootcamp-aws/en/tcb-conf-app-EN.zip (for example)

You can hit a "TAB" key to access the URL field.

Presets
Configuration presets
(*) High availability

Next


2) Configure service access

Service access
(*) Create and use new service role | 02 needed roles “service” and “ec2”

‘Service’ role name: aws-elasticbeanstalk-service-role

[ View service role permissions ]

► aws-elasticbeanstalk-service-role

Permissions:

- AWSElasticBeanstalkEnhancedHealth
- AWSElasticBeanstalkManagedUpdatesCustomerRolePolicy

Trust relationships

>{
>"Version": "2012-10-17",
>"Statement": [
>{
>"Effect": "Allow",
>"Principal": {
>"Service": "elasticbeanstalk.amazonaws.com"
>},
>"Action": "sts:AssumeRole",
>"Condition": {
>"StringEquals": {
>"sts:ExternalId": "elasticbeanstalk"
>}
>}
>}
>]
>}


EC2 key pair: Provide Key Pair

Now, let’s create an ‘EC2 instance profile’:

IAM | Roles | Create Role | Trusted entity type:  AWS service

Common use cases: EC2

Next

Add permissions

- AWSElasticBeanstalkWebTier
- AWSElasticBeanstalkWorkerTier
- AWSElasticBeanstalkMulticontainerDocker

Next

Role name: aws-elasticbeanstalk-ec2-role
Select trusted entities

Trust relationships

>{
>"Version": "2012-10-17",
>"Statement": [
>{
>"Effect": "Allow",
>"Action": [
>"sts:AssumeRole"
>],
>"Principal": {
>"Service": [
>"ec2.amazonaws.com"
>]
>}
>}
>]
}


Create role
Refresh…

Next


3) Set up networking, database, and tags

Virtual Private Cloud (VPC): N. Virginia - Default VPC

Instance settings
Public IP address
[ ✔ ]  Activated
Instance subnets
Availability Zone: us-east-1a

Select only one zone, we will change the instance size in the next step. Then, we will come back to this step to select all zones.
If you select all zones, you’ll get an error because shapes are not available.

Next


4) Configure instance traffic and scaling

Instances
Root volume type: General Purpose (SSD)
Size: 10 GB

Capacity
Auto scaling group
Load balanced
Min: 2
Max: 4
Fleet composition: (*) On-Demand instances
Instance types: t2.micro

Scaling triggers
Metric: CPUUtilization
Unit: Percent
Min: 1
Min: 1
Upper: 50
Scale up: 1
Lower: 40
Scale down: -1

Load balancer network settings
Visibility: Public
Load balancer subnets
[ ✔ ] check all

Previous - Let’s get back to the previous steps and select all zones

Next

Next


5) Configure updates, monitoring, and logging

…scroll down…
Environment properties
Add environment property
Name: AWS_REGION
Value: us-east-1

Next


6) Review

Submit
Elastic Beanstalk is launching your environment. This will take a few minutes.


## Part 2: Validating the Resources Created, Add an Email and AWS CloudFront

- Validate the Resources Created

- Testing the application

Tcb-conference-env | Domain:
Example: http://tcb-conference-env.eba-cwmmuapk.us-east-1.elasticbeanstalk.com/

Before adding any e-mail…
Let’s validate ‘DynamoDB’ and ‘Users’ table items
Tables | Explore items

Try to add an e-mail:
abc@abc.com


Internal Server Error
The server encountered an internal error and was unable to complete your request.
Either the server is overloaded or there is an error in the application.


Validating the Logs
Logs | Request logs | Last 100 lines

Add the required permission into “aws-elasticbeanstalk-ec2-role” role:

IAM | Roles
aws-elasticbeanstalk-ec2-role
Add permissions | Attach policies
AmazonDynamoDBFullAccess

Add permissions

Try again to add an e-mail:
abc@abc.com

- CloudFront | CDN - Content Delivery Network
Create a CloudFront distribution

Origin domain: select the ‘Elastic Load Balancer’ created by Elastic Beanstalk
Protocol: HTTP only

Allowed HTTP methods: GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE

Cache key and origin requests
(*) Cache policy and origin request policy (recommended)
Cache policy: CachingOptimized

Web Application Firewall (WAF)
(*) Enable security protections

Create Distribution | Last modified: Deploying…  ~ 5 minutes [ pause the video… ]

- Testing CloudFront
Copy 'Distribution Domain Name' and try it
Have you realized that this time we have HTTPS connection? CloudFront

Let’s add a new record and validate it at DynamoDB table
tcb-admin@tcb.com


##Part 3: Stress/Overloading test

Checking EC2, LB, TG, AutoScaling, Elastic Beanstalk health/status

► Accessing and installing 'Stress' tool in the EC2:
>ssh -i <key.pem> ec2-user@ip<instance>

Installing and running “Stress” tool
>sudo yum install stress -y
>stress -c 4

Checking Elastic Beanstalk status | 'Warning'

Let’s open a new SSH connection and check the process running in the EC2 instance:
>ps aux
>ps aux --sort=-pcpu
>top

Some cases that justify an increase in CPU consumption:

Black Friday | DDoS Attack | Hacker Mining Bitcoins

Let’s monitor the resources: EC2, ELB, Auto Scaling Group


## Removing Resources:

- Elastic Beanstalk 'Application' | Action Delete [ wait a few minutes ~ 5 minutes ]
- The previous action will delete the ‘Elastic Beanstalk Environment’ (terminated)
- Disable and Delete the CloudFront distribution [ waiting a few minutes ~ 5 minutes ]
- Delete the DynamoDB 'users' table

“Terminated” means ‘deleted’ and it can take a while to “disappear” from the console.

