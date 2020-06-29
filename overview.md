# AWS Control Tower Bootcamp for partners

## Workshop Overview

AWS Control Tower is a managed service, that provides the easiest way to set up and govern a new, secure, multi-account AWS environment based on best practices established through AWS’ experience working with thousands of enterprises as they move to the cloud. 

We will cover, through out several labs, general characteristics of AWS Control Tower as well as some more complex scenarios that AWS Control Tower administrator may face on daily operations.

As requirement for the labs, AWS Control Tower has to be deployed in advance due to the fact that it takes around 60 minutes to complete, so we’ll describe in this overview a high-level review of what happens behind the scenes when deploy AWS Control Tower. However, we’ll also have the steps to deploy AWS Control Tower next, as guidance.

## Deploy the AWS Control Tower 

In this section, we will deploy the AWS Control Tower service. Following tasks are performed on the successful launch of the service:

* Creates 2 organizational units, one for your shared accounts and other for user accounts.
* Provisions 2 additional accounts on top of master account. These are used for log archive, and audit.
* 20 preventive guardrails to enforce policies and 6 detective guardrails to detect violations.
* A native cloud directory with preconfigured groups and single sign-on access.

**We recommend performing this ahead of the session as the initial setup could take 60-90 minutes.** If you are using an account which already got an AWS Control Tower installed by somebody else, still recommend going through the below steps to get familiar with installation:

1. Log in to the AWS Management Console of the account where you plan to deploy AWS Control Tower. This account will be referred to as Master account henceforth.
2. Select the service **Control Tower** under **Management & Governance**.
3. Make sure you are in one of the five supported regions (N. Virginia, Ohio, Oregon, Ireland and Sydney). Refer to Latest Region Support Documentation (https://docs.aws.amazon.com/controltower/latest/userguide/how-control-tower-works.html)
4. On AWS Control Tower home page, select **Set up your landing zone** button.
5. Under Set up your AWS Control Tower, provide the email IDs which you plan to use for shared accounts.

|Input	|Description  |
|---	|---	|
|Log archive account Email Address	|The log archive account is a repository of immutable logs of API activities and resource configurations from all accounts  |
|Audit Account Email Address	|The audit account is a restricted account for your security and compliance teams to gain read and write access to all accounts |

6. Expand **Learn more about permissions** under **Service permissions** to review the roles used to launch the AWS Control Tower service. Please note the two roles **AWSControlTowerAdmin** and **AWSControlTowerExecution** are created as part of initialization. We will use these roles in the rest of our labs.
7. Checkbox **I understand the permissions AWS Control Tower will use to administer AWS resources and enforce rules on my behalf.** and click on **Set up Landing Zone**.
8. You will be redirected to **AWS Control Tower Dashboard**. The launch progress is shown in the blue bar on top of the Dashboard.
9. Check your emails for account creation confirmation from AWS Single Sign-On (https://aws.amazon.com/single-sign-on/).
10. In a few minutes, you will receive an email with subject **Invitation to join AWS Single Sign-On** to the master account email address. Make sure to open the email and click on **Accept invitation**.
11. The email you received also contains **User portal URL**, recommend to bookmark this, we will use it to access the AWS environment throughout the labs.
12. On selecting **Accept Invitation**, you will be redirected to the **AWS Single Sign-On** page and from where you could set **New Password** to your master account. Repeat Password and Update User to proceed.
13. You will receive one more email with subject **AWS Organizations email verification request** to the master account email address. Click on **Verify your email address** to continue with inviting newly created accounts into AWS Organization.
14. Wait for the blue progress bar with *% complete* to disappear on top of the AWS Control Tower dashboard. ***Note:*** It is normal to see the progress staying at 99% for 15-20 minutes

***Note:*** The email address you provided for the audit account will receive **AWS Notification - Subscription Confirmation** emails from every AWS Region supported by AWS Control Tower. To receive compliance emails in your audit account, you must choose the **Confirm subscription** link within each email from each AWS Region supported by AWS Control Tower.

## Exploring the Solution 

AWS Control Tower uses AWS CloudFormation StackSets (https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/what-is-cfnstacksets.html) to establish the baselines and create the guardrails across multiple accounts and regions. In this section we will walk through the StackSets used under the hood.

1. Login in to *AWS SSO* Console using the bookmark and password that you setup in step 10 and 11.
2. Click on the Master account to expand. Select *Management console* next to *AWSAdministratorAccess* Role to login AWS Management console of the *master* account.
3. Select *CloudFormation* under *Management & Governance* and select *StackSets* under CloudFormation

### CloudFormation StackSets used as part of Control Tower Initialization 

|StackSet name	|Description  |
|---	|---	|
|AWSControlTowerBP-BASELINE-CLOUDTRAIL	|Configure AWS CloudTrail on all accounts |
|AWSControlTowerBP-BASELINE-CLOUDWATCH	|Configure Cloudwatch Rule, local SNS Topic, forwarding notifications from local SNS Topic to Security Topic |
|AWSControlTowerBP-BASELINE-CONFIG	|Configure AWS Config on all accounts/regions |
|AWSControlTowerBP-BASELINE-ROLES	|Creates all required baseline roles on all the accounts |
|AWSControlTowerBP-BASELINE-SERVICE-ROLES	|Creates all required service roles on all the accounts for services (like AWS Config, SNS) used by CT |
|AWSControlTowerBP-SECURITY-TOPICS	|Central monitoring and alerting using SNS and AWS CloudWatch |
|AWSControlTowerGuardrailAWS-GR-AUDIT-BUCKET-PUBLIC-READ-PROHIBITED	|Configure AWS Config rules on core accounts to check that your S3 buckets do not allow public access |
|AWSControlTowerGuardrailAWS-GR-AUDIT-BUCKET-PUBLIC-WRITE-PROHIBITED	|Configure AWS Config rules on core accounts to check that your S3 buckets do not allow public access |
|AWSControlTowerGuardrailAWS-GR-S3-BUCKET-PUBLIC-READ-PROHIBITED	|StackSet for applying guardrail |
|AWSControlTowerGuardrailAWS-GR-S3-BUCKET-PUBLIC-WRITE-PROHIBITED	|StackSet for applying guardrail |
|AWSControlTowerLoggingResources	|StackSet to setup required resources on Log archive Account |
|AWSControlTowerSecurityResources	|StackSet to setup required resources on Audit Account |
|AWSControlTowerBP-VPC-ACCOUNT-FACTORY-V1	|Once the Account is provisioned this stack will be run to setup account configuration based on the baselines |

***You may see additional stacksets depending on the number of guardrails you enable on Control Tower.***


## Labs:

1. [Lab 1: AWS Control Tower Account Factory](https://github.com/denegrij/controltower/blob/master/lab1-account-factory.md) 
2. [Lab 2: Tasks in AWS Control Tower](https://github.com/denegrij/controltower/blob/master/lab2-tasks-in-control-tower.md) 
3. [Lab 3: AWS Control Tower Lifecycle Events](https://github.com/denegrij/controltower/blob/master/lab3-control-tower-lifecycle-events.md) 
