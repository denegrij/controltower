[Back to main](https://github.com/denegrij/controltower/blob/master/README.md) 

# Lab 3: AWS Control Tower Lifecycle Events

## Overview

AWS Control Tower allows you to create new AWS accounts in your [AWS Organization](https://aws.amazon.com/organizations/) with AWS recommended best practices and guardrails in place. Our customers and partners often ask for ways to automatically execute some customizations specific to their organizations triggered by the creation of a new AWS accounts. The customizations include creating IAM roles to auto-integrate with the partner products, automate enabling services like VPC Flow Logs, Amazon GuardDuty, AWS Security Hub, and much more.

In this lab, you will use AWS Control Tower life cycle events to achieve the below-mentioned tasks automatically when you create a new AWS account using Account Factory.

* Create IAM roles and automatically register with [Check Point Dome9](https://dome9.com/)
* Create VPC flow logs for all VPCs across the AWS Organization

## Architecture Overview

In this lab, you are going to use CloudFormation template provided to create an Amazon CloudWatch Event to watch for an *AWS Service Event via CloudTrail* and trigger an AWS Lambda function (or any other [supported trigger](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/WhatIsCloudWatchEvents.html)) when one of the eight Control Tower Life Cycle events are received. For this lab, you are going to use AWS Lambda as a trigger.

[Image: image.png]The CloudFormation template provided as part of this lab, will create following resources:

* An Amazon CloudWatch Event on the `master` account.
* An AWS Lambda function on the `master` account, which acts as a trigger for the event.
* You will type-in the partner product (Dome9) credentials as `Parameters`. These parameters are stored as secrets in AWS Secrets Manager.
* Launch multiple baseline AWS CloudFormation StackSet on master account.

When you create a new AWS Account, the stack-instances are added to these by the Lambda function.

### Prerequisites

1.1 This lab requires an account with Administrator privileges and Control Tower.
1.2 Use `CloudFormation New Console`. All instructions of this lab are based off new console.
1.3 If you are still on old console, expand CloudFormation on top left and choose New Console.
1.4 Launch the CloudFormation stack in the region where AWS Control Tower is deployed.

## Register and collect a Dome9 account credentials

Registering with a partner product and getting the credentials will take around 10-15 minutes. We will skip this step by using *dummy credentials* given below. This will allow you to create the IAM roles on new account, skip registration with Dome9, and continue with remaining part of the lab.

|Parameter Name	|Dummy Input	|
|---	|---	|
|Dome9ApiId	|controlt-ower-make-myli-fesimple2020	|
|---	|---	|
|Dome9ApiSecret	|noadditionalchargesforct	|
|Dome9ExternalId	|conTrolTowerisAweSomeSrv	|
|Dome9AccountId	|`634729597623` [Use this AS-IS]	|

Please Read: Using above values will CREATE required IAM roles. However it will NOT register your new account with Dome9.
* * *

## Launch the lab resources

In this part of the Lab, you will launch an AWS CloudFormation Stack to setup the lab environment in your account.

### Create a new Organizational unit

In this section you will create an Organizational Unit (OU) from AWS Control Tower. We will use this OU to enable a guardrail in the next step:

4.1. Login to AWS Control Tower Dashboard and click on Organizational units on the left Sidebar.
4.2. This opens up an Organizational units window. Click on Add an OU button.
4.3. Provide a new OU Name (say `Sandboxes`) and click on Add button.
4.4. Wait for green Success notification on top of the page.

### Enable Guardrails

In this section you will enable a guardrail on newly created OU. Later in this lab when you create a new AWS account, it will have this guardrail enabled by default:

5.1. While you are still on AWS Control Tower Dashboard, Click on Guardrails on the left Sidebar.
5.2. This lists all the Guardrails. Search for **Disallow internet connection through SSH** and click on it.
5.3. Scroll down to Organizational units enabled section and click on Enable guardrail on OU button.
5.4. Select the OU named as `Sandboxes` and click on Enable guardrail on OU button.
5.5. Wait for green Success notification on top of the page.

### Deploy stack to create lab environment in your account

6.1. Log in to your AWS Control Tower `master` account.
6.2. Choose the appropriate region.
6.3. Click on the following link to start deploying the stack:

https://console.aws.amazon.com/cloudformation#/stacks/new?stackName=CtLifeCycleEventsLabSetup&templateURL=https://marketplace-sa-resources-ct-us-west-2.s3.amazonaws.com/ct_lifecycle_labenv_setup.yaml

6.4. While on Create Stack page, choose NEXT.
6.5. On Specify stack details page, enter the information you noted in `steps 5.4` and `5.8` and choose NEXT
[Image: image.png]
6.6. Under the Configure stack options page, choose NEXT.
6.7. Under the Review page, `scroll down`, and select, I acknowledge that AWS CloudFormation might create IAM resources with custom names. Under Capabilities. Then choose Create stack.
[Image: image.png]6.8. Wait for the stack state change to CREATE_COMPLETE
[Image: image.png]

### Create a new AWS account from Account Factory

7.1. Log in to Control Tower `Master` account using the `User portal URL` and `credentials`.
7.2. [Click HERE](http://console.aws.amazon.com/controltower/home/accountfactory) to jump on to the Account Factory home page.
7.3. In the Account Factory page, under Network configuration, choose Edit.

[Image: image.png]7.4. Uncheck all five regions under Regions for VPC Creation and choose Save. By unselecting the regions, you are skipping the creation of VPCs on the new AWS account, reducing the overall time it takes for creating new AWS Account.
[Image: image.png]
7.5. In the Account Factory page, choose Enroll account.
7.6. This opens up the Create account page. Key in below information

|Parameter Name	|Input value	|
|---	|---	|
|Account email	|`your-email-alias`-stud-1@amazon.com	|
|---	|---	|
|Display Name	|`Your-Account-Name`	|
|AWS SSO email	|`SSO-email-address`	|
|AWS SSO user name `First name`	|Your First Name	|
|AWS SSO user name `Last name`	|Your Last Name	|
|Organizational unit	|`Custom` or OU you created	|

7.7. Follow the instructions on the screen, leave all `default` values and Enroll account.

Congratulations, you successfully launched creation of a new AWS Account. 
**Please Note!!** The account creation process includes creating a new AWS account, applying baselines, and applying the appropriate guardrails on the newly created account. This account creation operation could take up to 30 minutes.

While the new AWS account is being created, proceed to the next part of the lab.

## Test results verification

### Verify the components that are setup as part of the lab setup

While you wait for the creation of a new AWS account, look into the details of the resources deployed in steps 6.x.
8.1. While you are still on Control Tower `master` account, go to [CloudWatch Console](https://console.aws.amazon.com/cloudwatch).
8.2. Under CloudWatch and Events, choose Rules to open the Rules page.
8.3. On the Rules page, click on CaptureControlTowerLifeCycleEvents to open Summary page.
8.4. With in Event pattern content look for eventName list. These are the list of Events we watch for.
[Image: image.png]
8.5. Scroll down the page and look at the Targets section.
8.6. Click on the Lambda function to inspect the code used to process the event received.
8.7. From the Lambda code you could see, when a new AWS Account is successfully provisioned, a new stack instance is added to the existing StackSets.

### Verify the CloudFormation StackSets in master account

9.1. Go to the [CloudFormation Console](http://console.aws.amazon.com/cloudformation) and choose StackSets.
9.2. Notice the StackSets `DOME9-ROLES-CREATION` and `VPC-FLOWLOG-CREATION` StackSets are created.
[Image: image.png]
9.3. Click on each StackSet and look for Stack instances. There are `no stacks` created for DOME9 stack, but you will a `Stack Instances(1)` for VPC-FLOWLOG stack.
[Image: image.png]
9.4. These are the stack instances deployed as part of the lab initialization template you ran in 4.x steps
9.5. _Bonus_: find out what is the trigger frequency for Lambda in `VPC-FLOWLOG-CREATION`, this can help you to do verification later.

Wait for the provisioning of new account to complete. This could take 20-30 minutes if you have disabled all the five regions in step 7.4.

### Verify the status of Account Creation

10.1. Log in to [Service Catalog Console](https://controltower.aws-management.tools/core/lifecycle/console.aws.amazon.com/servicecatalog/) and make sure you are in the `correct region`.
10.2. Choose Provisioned Products on the left side panel.
10.3. In Provisioned products list page, wait for the status to become `AVAILABLE`.
[Image: image.png]

### Verify VPC Flow Log configuration in new AWS account

Run the following steps on the new AWS account you just created on the previous step.
12.1. Sign in into your newly created AWS account with AWSAdministratorAccess role.
12.2. Navigate to VPC console. You can do this in any AWS regions.
12.3. Click Launch VPC Wizard.
[Image: image.png]
- Select VPC with a Single Public Subnet - Enter the VPC name, i.e. Test-VPC - Click Create VPC

12.4. Select the VPC from the list and go to Tags tab.
12.5. Click Add/Edit Tags.
12.6. Click Create Tag to add new key-value
12.7. Enter the following key-value tag - Key = `flowlog` - Value = choose either `ALL`, `ACCEPT` or `REJECT`
12.8. Click Save
12.9. Lambda function will trigger periodically to scan for VPC with the tag key `flowlog`, this could take up to 5 minutes (based on the `CheckFrequency` value from `VPC-FLOWLOG-CREATION` StackSet)
12.10. Click on Flow Logs tab in the VPC console to verify if the flow log created successfully.

[Image: image.png]**Note**: The S3 bucket is owned by `log archive` account, there’s no cross-account access from your new AWS account.

12.11 To generate Flow Log traffic, feel free to explore by launching EC2 instances in this VPC.
12.12 Flow Log is stored in the `log archive` account. Login to your `log archive` account with AWSAdministratorAccess role to locate the S3 bucket.

## Lab cleanup procedure

To clean up the lab:
C.1 Sign in into your newly created AWS account, locate the new VPC from step 12.x.
C.2. Click on Flow Logs tab, select the Flow log from the list. Click Action > Delete.
C.3 On Control Tower `master` account, go to [CloudFormation Console](https://console.aws.amazon.com/cloudformation).
C.4 Locate the stack you created previously on steps 6.x.
C.5. Click Delete stack, wait until stack deleted successfully.
C.6. Navigate to StackSet, confirm that StackSet `DOME9-ROLES-CREATION` and `VPC-FLOWLOG-CREATION` also deleted.
C.7 On the `log archive` account, go to [S3 Console](https://console.aws.amazon.com/s3).
C.8. Locate S3 bucket with name format `[$account_id]-vpcflowlog`, empty and delete the S3 bucket

