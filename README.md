## AWS SES-SNS-SQS Decoupled Architecture

We are going to build a system using a SQS queue, a SNS topic, and a SES verified email id where we are going to send the email using AWS SES service. Upon every email sent/receive/failed, SES generates an event and will publish the message to the SNS topic and a SQS queue will be subscribed to that SNS topic. Application which is running on AWS EC2 machine will listen to that SQS queue and process the data in SQS queue.

## Overview / Objective

AWS CloudFormation provides a common language for you to model and provision AWS and third party application resources in your cloud environment. AWS CloudFormation allows you to use programming languages or a simple text file to model and provision, in an automated and secure manner, all the resources needed for your applications across all regions and accounts. This gives you a single source of truth for your AWS and third party resources.

## Simple Email Service (SES)
Amazon Simple Email Service (SES) is a cost-effective, flexible, and scalable email service that enables developers to send mail from within any application. Amazon SES supports all industry-standard authentication mechanisms, including Domain Keys Identified Mail (DKIM), Sender Policy Framework (SPF), and Domain-based Message Authentication, Reporting and Conformance (DMARC).
When you use Amazon SES to receive incoming emails, you have complete control over which emails you accept, and what to do with them after you receive them. You can accept or reject mail based on the email address, IP address, or domain of the sender. Once Amazon SES has accepted the email, you can store it in an Amazon S3 bucket, execute custom code using an AWS Lambda function, or publish notifications to Amazon SNS.

## Simple Queue Service (SQS)

Amazon SQS is a distributed queue system that enables web service applications to quickly and reliably queue messages that one component in the application generates to be consumed by another component. A queue is a temporary repository for messages that are awaiting processing.
SQS eliminates the complexity and overhead associated with managing and operating message oriented middleware, using SQS, you can send, store, and receive messages between software components at any volume, without losing messages or requiring other services to be available.

## Amazon Simple Notification Service (SNS)
Amazon Simple Notification Service (SNS) is a highly available, durable, secure, fully managed pub/sub messaging service that enables you to decouple microservices, distributed systems, and serverless applications. Amazon SNS provides topics for high-throughput, push-based, many-to-many messaging.
Using Amazon SNS topics, your publisher systems can fan out messages to a large number of subscriber endpoints for parallel processing, including Amazon SQS queues, AWS Lambda functions, and HTTP/S webhooks.

## Overview of Notification Flow
    ## 1. First, application sends an email using Amazon SES and saves the email ID sent back from Amazon SES. The application will use the Amazon SDK to send emails and receive the email ID.
    2. Next, Amazon SES dispatches emails to the users. When the email status changes from sent to delivered, read, rejected, or any other status, SES will automatically notify Amazon SNS about the updated status via a SNS topic. 
    3. Finally, the SNS topic will send a new email status to the endpoints that were registered with the topic. In our case, it will send to a SQS queue.
    4. In application, we'll need to compare the IDs — the stored email ID received from SES with the ID sent by SNS. If the IDs match,then application will update the email status  in the database.

    That's how we’ll be able track the email status.








AWS CloudFormation Publisher creates S3 buckets for each region in your account. The bucket names have the following format:

`${bucket prefix}-{region name}`

For example: `cfn-0123456789012-us-east-1`

Your projects' CloudFormation templates will be stored as `{project name}/{version}/{template}`.


## Installation

To deploy, launch the AWS Cloudformation Publisher into your account:

Click below link or login to your aws account and select the appropriate region from the top right corner. 
We're using Mumbai(ap-south-1) region and we'll be deploying all our resources onto this region only. 

|Region|Launch Template|
|------|---------------|
|**Asia Pacific (Mumbai)** (ap-south-1) | [![Launch the aws-cloudformation-publisher Stack with CloudFormation](docs/deploy-to-aws.png)](https://console.aws.amazon.com/cloudformation/home?region=ap-south-1#/stacks/new?stackName=aws-cloudformation-publisher&templateURL=https://solution-builders-ap-south-1.s3.ap-south-1.amazonaws.com/aws-cloudformation-publisher/latest/main.template)|

## Configuration

  ## 1.VPC Setup
Log in to your AWS account and change the region to Mumbai(ap-south-1) and open the [Cloudformation console](https://console.aws.amazon.com/cloudformation/) or go to CloudFormation service by typing CloudFormation in search bar.


First, we'll be lauching VPC in this region, so we'll pick `VPC-Setup.cform` template and we'll follow the below option:

* Click `Create stack (with new resources)` option on the top right side.
* Here, you'll be presented three options: `Template is ready` , `Use a sample template` and `Create template in Designer`.
* Choose `Template is ready` and just below you'll be having option to either upload Cloudformation template from S3 or upload it from your local machine. So choose `Upload a template file` and select `VPC-Setup.cform` template and click `next` button.
* Here, you'll have to write your `stack name`, name this template in such a way that people can easily identify for what this template belongs to. Name the stack and click `next` button.
*  Here you'll have some additional info which you can pass, but we're not passing any info in our case, so we'll click `next` button.
* On this page, you'll get a review of your stack, go to bottom of this page and click `Create stack` button.
* Now stack creation will be under progress, once it creates all the resources, it will show the status as `CREATE_COMPLETE`

  ## 2. SNS Setup
* Click `Create stack (with new resources)` option on the top right side.
* Here, you'll be presented three options: `Template is ready` , `Use a sample template` and `Create template in Designer`.
* Choose `Template is ready` and just below you'll be having option to either upload Cloudformation template from S3 or upload it from your local machine. So choose `Upload a template file` and select `sns.cform` template and click `next` button.
* Here, you'll have to write your `stack name`, name this template in such a way that people can easily identify for what this template belongs to. Name the stack. After this you'll be getting a section called `Parameters`.
It should give you option something like this:
```
Endpoint="Enter a valid Email address where notification can come"
Protocol=email
Region=ap-south-1
```
Please note that, you'll be already seeing values under parameter section, you can change the value if you want or you can leave as it is, if you're statisfied with the default value.
* After entering the parameter values, click `next` button.
*  Here you'll have some additional info which you can pass, but we're not passing any info in our case, so we'll click `next` button.
* On this page, you'll get a review of your stack, go to bottom of this page and click `Create stack` button.
* Now stack creation will be under progress, once it creates all the resources, it will show the status as `CREATE_COMPLETE`

Please note that once this stack gets created, AWS will send a verification email on the email id which you've passed in the parameter section, you need to click that verification email in order to successfully start with SNS service.
You won't get any email from AWS until, you confirm that verification email link sent by AWS.

  ## 3. S3 Bucket Setup
* Click `Create stack (with new resources)` option on the top right side.
* Here, you'll be presented three options: `Template is ready` , `Use a sample template` and `Create template in Designer`.
* Choose `Template is ready` and just below you'll be having option to either upload Cloudformation template from S3 or upload it from your local machine. So choose `Upload a template file` and select `S3Bucket.cform` template and click `next` button.
* Here, you'll have to write your `stack name`, name this template in such a way that people can easily identify for what this template belongs to. Name the stack. After this you'll be getting a section called `Parameters`.
* Here you've to write the S3 bucket name, please note that this name should be unique globally. It means there should not be any bucket in AWS. Click next. 
* On this page, you'll get a review of your stack, go to bottom of this page and click `Create stack` button.
* Now stack creation will be under progress, once it creates all the resources, it will show the status as `CREATE_COMPLETE`


Note - if there is any bucket under this name, then this stack creation would fail. If it fails, then again launch this template with some different name.

  ## 4. Elastic IP Creation
* Click `Create stack (with new resources)` option on the top right side.
* Here, you'll be presented three options: `Template is ready` , `Use a sample template` and `Create template in Designer`.
* Choose `Template is ready` and just below you'll be having option to either upload Cloudformation template from S3 or upload it from your local machine. So choose `Upload a template file` and select `eip.cform` template and click `next` button.
* Here, you'll have to write your `stack name`, name this template in such a way that people can easily identify for what this template belongs to. Name the stack. After this you'll be getting a section called `Parameters`.
* Here you don't need to provide any option, leave it blank and click next. 
* Now stack creation will be under progress, once it creates all the resources, it will show the status as `CREATE_COMPLETE`

  ## 5. IAM Instance Role Setup
* Click `Create stack (with new resources)` option on the top right side.
* Here, you'll be presented three options: `Template is ready` , `Use a sample template` and `Create template in Designer`.
* Choose `Template is ready` and just below you'll be having option to either upload Cloudformation template from S3 or upload it from your local machine. So choose `Upload a template file` and select `IAMInstanceRole.yaml` template and click `next` button.
* Here, you'll have to write your `stack name`, name this template in such a way that people can easily identify for what this template belongs to. Name the stack. After this you'll be getting a section called `Parameters`.
* In Account ID section, you've to write the account id, it can be seen from top right corner where your name is showing up. Click that and you'll be getting `My Account` tab, click on that tab to open in a new window and from that page, accound id can be taken.
* Here you don't need to provide any option, leave it blank and click next. 
* On this page, you'll get a review of your stack, go to bottom of this page, there will be a check mark button , you need to click OK (this is just an validation that a role is going to create) and click `Create stack` button.
* Now stack creation will be under progress, once it creates all the resources, it will show the status as `CREATE_COMPLETE`

Once this stack gets created, go to the [IAM service console](https://console.aws.amazon.com/iam/) and check for the role which has just been created.
Click on the Role name, here you'll get 2 things:
1. Role ARN - arn:aws:iam::account_id:role/role_name
2. Instance Profile ARNs - arn:aws:iam::account_id:instance-profile/role_name

Note that `role_name` only, from `Instance Profile ARN`, you need to pass this info while creating the EC2. 

  ## 6. Security Group for EC2 server

* Click `Create stack (with new resources)` option on the top right side.
* Here, you'll be presented three options: `Template is ready` , `Use a sample template` and `Create template in Designer`.
* Choose `Template is ready` and just below you'll be having option to either upload Cloudformation template from S3 or upload it from your local machine. So choose `Upload a template file` and select `Security_Group_DEV_Server.cform` template and click `next` button.
* Here, you'll have to write your `stack name`, name this template in such a way that people can easily identify for what this template belongs to. Name the stack. After this you'll be getting a section called `Parameters`.
* Here you have to pass the VPC ID , it will be something like this - vpc-0202b70e50591a593, Go to [VPC Console](https://console.aws.amazon.com/vpc/) and check for the VPC which has been just created, take the VPC ID and pass here under parameter section.
* Here you don't need to provide any option, leave it blank and click next. 
* On this page, you'll get a review of your stack, go to bottom of this page, and click `Create stack` button.

Once this stack gets created, note the security group id from the resource tab or go to [VPC Console](https://console.aws.amazon.com/vpc/) and then check for Security group section, there you'll get all of your security group id, check for yours which has been just created.

  ## 7. EC2 server setup
  
* Click `Create stack (with new resources)` option on the top right side.
* Here, you'll be presented three options: `Template is ready` , `Use a sample template` and `Create template in Designer`.
* Choose `Template is ready` and just below you'll be having option to either upload Cloudformation template from S3 or upload it from your local machine. So choose `Upload a template file` and select `Dev_EC2.cform` template and click `next` button.
* Here, you'll have to write your `stack name`, name this template in such a way that people can easily identify for what this template belongs to. Name the stack. After this you'll be getting a section called `Parameters`.
It should give you option something like this:

```
EBSVolumeRequired=true                    (choose `yes` or `no` if additonal ebs volume is required, this will create new EBS volume)
EBSVolumeSize=10                          (Specify size of additonal volume, if you've selected `yes` above, if you've selected `false` then leave it blank)
EIP=eip-54435XXXXXXXX                     (Specify allocation id of elastic IP, this elastic IP will be attached to this instance)
EIPCondition=false                        (specify whether you want to attach fresh elastic IP for this instance, specifying this option as yes will launch new 
                                          elastic IP for this instance, if you're specifying 'yes' then specify `false` in `EIPExists` paramter, We'll choose `false` in our case)

EIPExists=true                            (specify if you want to attach elastic IP from already launched elasticIP, if you're specifying yes, then you must
                                           have the elastic IP, we'll choose `true` in our case)

IamInstanceProfile=IAM Instance role name (IAM role name which will be attached to this EC2 instance)
InstanceType=t2.micro                     (Type of instance that need to be lauched, t2.micro for our case)
KeyName=                                  (Name of an existing keypair, this needs to be manually created from EC2 console and give the name of key here)
LatestAmiId=ami-XXXXXXXX                  ( AMI ID , this can be picked from [EC2 service section](https://console.aws.amazon.com/ec2/v2/home?region=ap-south-1#Images:sort=desc:creationDate), )
SecurityGroup=sg-4535XXXXX                (specify security group id for this instance)
SubnetID=subnet-01XXXX                    (Choose from the drop down menu, where you want to launch the instance)
VPCID=vpc-XXXX                            (Specify the VPC id)

```
* Here you don't need to provide any option, leave it blank and click next. 
* On this page, you'll get a review of your stack, go to bottom of this page, and click `Create stack` button.

Once this stack gets created, it takes 5-10 mins to get EC2 instance fully UP & running.

Go to [EC2 Console](https://console.aws.amazon.com/ec2/v2/home?region=ap-south-1#Instances:sort=instanceType) and check for the EC2 which has been just created, and check for the private IP, it should be something 10.20.X.X. Note this IP and you need to pass this info in the next template.


  ## 8. Security Group RDS (Database security group)
* Click `Create stack (with new resources)` option on the top right side.
* Here, you'll be presented three options: `Template is ready` , `Use a sample template` and `Create template in Designer`.
* Choose `Template is ready` and just below you'll be having option to either upload Cloudformation template from S3 or upload it from your local machine. So choose `Upload a template file` and select `Security_Group_RDS.cform` template and click `next` button.
* Here, you'll have to write your `stack name`, name this template in such a way that people can easily identify for what this template belongs to. Name the stack. After this you'll be getting a section called `Parameters`.
* Here, you'll have to write your `stack name`, name this template in such a way that people can easily identify for what this template belongs to. Name the stack. After this you'll be getting a section called `Parameters`.
It should give you option something like this:
IPAddress=10.20.X.X          (Put the private IP of the EC2 instance)
VPCID=vppc-4654XXXX          (Specify VPC ID)

* Here you don't need to provide any option, leave it blank and click next. 
* On this page, you'll get a review of your stack, go to bottom of this page, and click `Create stack` button.

Once this stack gets created, note security group id from the resource section or from [Security group console](https://console.aws.amazon.com/ec2/v2/home?region=ap-south-1#SecurityGroups:)

  ## 9. RDS Setup
* Click `Create stack (with new resources)` option on the top right side.
* Here, you'll be presented three options: `Template is ready` , `Use a sample template` and `Create template in Designer`.
* Choose `Template is ready` and just below you'll be having option to either upload Cloudformation template from S3 or upload it from your local machine. So choose `Upload a template file` and select `rds.cform` template and click `next` button.
* Here, you'll have to write your `stack name`, name this template in such a way that people can easily identify for what this template belongs to. Name the stack. After this you'll be getting a section called `Parameters`.
It should give you option something like this:

```
AllocatedStorage=10                                      (Specify storage size for the RDS)
AvailabilityZone=ap-south-1b                             (Specify availability zone, Leave it to default value)
DBInstanceClass=db.t2.micro                              (Specify instance size for this RDS, Leave it to Default value)
DBInstanceIdentifier=Name of the RDS                     (Specify name for this RDS)
DBName= Name of the DB                                   (Specify DB Name)  
DBSecurityGroups=Security group id                       (Specify database security group you want to attach with this RDS)
Engine=MySQL                                             (Choose which type of RDS engine is required, Leave it to Default value)
EngineVersion=8.0.11
MasterUserPassword=Set the password
MasterUsername=Specify user name
PreferredBackupWindow=                                   (Specify backup window so that RDS can take snapshot this time, Leave it to Default value)
PubliclyAccessible=false                                 (Specify if you want this RDS to be PubliclyAccessible?, Leave it to Default value)
SubnetID=subnet id 1                                     (specify subnet id1 in which this rds instance will be launched)
SubnetID2=subnet id 2                                    (specify subnet id2 in which this rds instance will be launched)
Timezone=UTC                                             (Specify in which Timezone, this RDS should belong to, Leave it to Default value)  

```
* Here you don't need to provide any option, leave it blank and click next. 
* On this page, you'll get a review of your stack, go to bottom of this page and click `Create stack` button.
* Now stack creation will be under progress, once it creates all the resources, it will show the status as `CREATE_COMPLETE`.
* Please note that this resource creation takes 15-20 minutes of time, so until then it will be in `progress` mode, if it takes longer than 20 minutes, it will throw an error.



  ## 10. Cloudwatch Monitoring Alarm setup
* Click `Create stack (with new resources)` option on the top right side.
* Here, you'll be presented three options: `Template is ready` , `Use a sample template` and `Create template in Designer`.
* Choose `Template is ready` and just below you'll be having option to either upload Cloudformation template from S3 or upload it from your local machine. So choose `Upload a template file` and select `CloudWatch_V1.cform` template and click `next` button.
* Here, you'll have to write your `stack name`, name this template in such a way that people can easily identify for what this template belongs to. Name the stack. After this you'll be getting a section called `Parameters`.
It should give you option something like this:

```
Email=Email ARN                       (Specify email arn here, it can be taken from SNS console)
InstanceId=i-12344667                 (Specify instance id which you want to monitor)
RDSID=RDS Name                        (Specify RDS name)
RedisName=Redis name                  (Specify Redis name)

```
* Here you don't need to provide any option, leave it blank and click next. 
* On this page, you'll get a review of your stack, go to bottom of this page and click `Create stack` button.
* Now stack creation will be under progress, once it creates all the resources, it will show the status as `CREATE_COMPLETE`.

## 11. Redis Security Group
* Click `Create stack (with new resources)` option on the top right side.
* Here, you'll be presented three options: `Template is ready` , `Use a sample template` and `Create template in Designer`.
* Choose `Template is ready` and just below you'll be having option to either upload Cloudformation template from S3 or upload it from your local machine. So choose `Upload a template file` and select `Security_Group_Redis.cform` template and click `next` button.
* Here, you'll have to write your `stack name`, name this template in such a way that people can easily identify for what this template belongs to. Name the stack. After this you'll be getting a section called `Parameters`.
It should give you option something like this:

```
IPAddress=10.20.X.X     (Put IP of EC2 instance from which you want to allow access to the Redis)
VPCID=VPC-iD            (Put your vpc-id)
```

* Here you don't need to provide any option, leave it blank and click next. 
* On this page, you'll get a review of your stack, go to bottom of this page and click `Create stack` button.
* Now stack creation will be under progress, once it creates all the resources, it will show the status as `CREATE_COMPLETE`.


## 12. Redis Setup
* Click `Create stack (with new resources)` option on the top right side.
* Here, you'll be presented three options: `Template is ready` , `Use a sample template` and `Create template in Designer`.
* Choose `Template is ready` and just below you'll be having option to either upload Cloudformation template from S3 or upload it from your local machine. So choose `Upload a template file` and select `elasticcache.cform` template and click `next` button.
* Here, you'll have to write your `stack name`, name this template in such a way that people can easily identify for what this template belongs to. Name the stack. After this you'll be getting a section called `Parameters`.
It should give you option something like this:

```
AutoMinorVersionUpgrade=False      (Default value set to False)
CacheEngine=redis                  (Default value set to False)
CacheNodeType=cache.t2.micro       (Default value is cache.t2.micro)
ClusterName=Name of the cluster    (Specify name of this redis cluster)
EngineVersion=5.0.0                (Leave it to default value)
Port=6379                          (Leave it to default value)
PreferredAvailabilityZone=ap-south-1b 
SecurityGroup=sg-05efbc93ddd5aeb3a  (Specify securtiy groupd id, which is just created above)
SnapshotArns=arn:aws:s3:::some_name  (Specify ARN that uniquely identifies a Redis RDB snapshot file stored in Amazon S3.)
SnapshotRetentionLimit= 7           (Leave it to default value)
SnapshotWindow=sun:23:15-mon:03:15  (Leave it to default value)
SubnetID=subnet-02d97d80XXXX        (specify subnet id in which this redis instance will be launched)

```
* Here you don't need to provide any option, leave it blank and click next. 
* On this page, you'll get a review of your stack, go to bottom of this page and click `Create stack` button.
* Now stack creation will be under progress, once it creates all the resources, it will show the status as `CREATE_COMPLETE`.

## Usage

## 13. Notify Script setup
* Notify script sends email alert for every successful SSH on our EC2 server.
* To enable this, SSH into your EC2 instance as ec2-user, then switch to root user by command `sudo su - `
* Please update the `Email ARN` in the notify.sh script according to your ARN.
* Edit a file by command `vim /etc/profile.d/notify.sh` and enter the content of script `notify.sh`
* Run a command `chmod a+x /etc/profile.d/notify.sh`
* It's done

## License Summary

This code is made available for . Prior consent is required to re-distribute this code.
