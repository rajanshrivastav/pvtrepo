## AWS SES-SNS-SQS Decoupled Architecture

We are going to build a system using a SQS queue, a SNS topic, and a SES verified email id where we are going to send the email using AWS SES service. Upon every email sent/receive/failed, SES generates an event and will publish the message to the SNS topic and a SQS queue will be subscribed to that SNS topic. Application which is running on AWS EC2 machine will listen to that SQS queue and process the data in SQS queue.

![diagram](ses.PNG?raw=true)

## Overview / Objective

AWS CloudFormation provides a common language for you to model and provision AWS and third party application resources in your cloud environment. AWS CloudFormation allows you to use programming languages or a simple text file to model and provision, in an automated and secure manner, all the resources needed for your applications across all regions and accounts. This gives you a single source of truth for your AWS and third party resources.

We're using 3 services of AWS for this setup--

## 1.Simple Email Service (SES)
Amazon Simple Email Service (SES) is a cost-effective, flexible, and scalable email service that enables developers to send mail from within any application. Amazon SES supports all industry-standard authentication mechanisms, including Domain Keys Identified Mail (DKIM), Sender Policy Framework (SPF), and Domain-based Message Authentication, Reporting and Conformance (DMARC).
When you use Amazon SES to receive incoming emails, you have complete control over which emails you accept, and what to do with them after you receive them. You can accept or reject mail based on the email address, IP address, or domain of the sender. Once Amazon SES has accepted the email, you can store it in an Amazon S3 bucket, execute custom code using an AWS Lambda function, or publish notifications to Amazon SNS.

## 2.Simple Queue Service (SQS)

Amazon SQS is a distributed queue system that enables web service applications to quickly and reliably queue messages that one component in the application generates to be consumed by another component. A queue is a temporary repository for messages that are awaiting processing.
SQS eliminates the complexity and overhead associated with managing and operating message oriented middleware, using SQS, you can send, store, and receive messages between software components at any volume, without losing messages or requiring other services to be available.

## 3.Amazon Simple Notification Service (SNS)
Amazon Simple Notification Service (SNS) is a highly available, durable, secure, fully managed pub/sub messaging service that enables you to decouple microservices, distributed systems, and serverless applications. Amazon SNS provides topics for high-throughput, push-based, many-to-many messaging.
Using Amazon SNS topics, your publisher systems can fan out messages to a large number of subscriber endpoints for parallel processing, including Amazon SQS queues, AWS Lambda functions, and HTTP/S webhooks.

## Overview of Notification Flow
    ## 1. First, application sends an email using Amazon SES and saves the email ID sent back from Amazon SES. The application will use the Amazon SDK to send emails and receive the email ID.
    2. Next, Amazon SES dispatches emails to the users. When the email status changes from sent to delivered, read, rejected, or any other status, SES will automatically notify Amazon SNS about the updated status via a SNS topic. 
    3. Finally, the SNS topic will send a new email status to the endpoints that were registered with the topic. In our case, it will send to a SQS queue.
    4. In application, we'll need to compare the IDs — the stored email ID received from SES with the ID sent by SNS. If the IDs match,then application will update the email status  in the database.

    That's how we’ll be able track the email status.


## Installation

We've created a terraform template which will further call to a Cloudformation template to deploy this architecture. 

## Configuration

  ## 1.Code Clone
 Open the [Optum Git](https://github.optum.com/hcentive-techops) on your favourite browser and clone the code.

First, we'll need to edit `variable.tf` file, so we'll pick `variable.tf` template and we'll follow the below option:

* Add `Region` according to your use case, e.g.- us-east-1
* Add `EmailAddress` variable and enter the email id from which you want to send the email to the customers. This email id will be used as Sender Email id.
* Add `NameOfTheQueue` variable and enter the desired name of the queue.
* Add `SNSTopicName` variable and enter the desired SNS topic name.
* Add `accountid` variable and enter the account id of aws account where you're going to deploy this template.


## 2. Terraform run
 Run the terraform plan command `terraform plan`, if everything looks good, then run apply command `terraform apply`.

 When you run `terraform apply` command, it will do following things -
 1. It will create a CloudFormation template in your defined region.
 2. Cloudformation will create a SNS topic, a primary SQS queue, a DLQ queue and a SES Email id.
 3. If any message is not processed for 5 times, then that message will be sent to DLQ (dead letter queue).
 4. AWS SES service sends a verification email on the given email id which you need to click and confirm it. You'll be able to send email only after you verify the email id.
 5. Terraform will then enable the header option in SES email id for `BOUNCE` and `DELIVERY` event type. This option will contain all header info and it will send all the detail of the email to SNS topic which will eventually sent to SQS queue. 

## License Summary

This code is made available for Optum. Prior consent is required to re-distribute this code.
