[Back to main](https://github.com/denegrij/controltower/blob/master/overview.md) 

[Back to Lab 1: AWS Control Tower Account Factory](https://github.com/denegrij/controltower/blob/master/lab1-account-factory.md) 

***

# Lab 2: Tasks in AWS Control Tower

## Overview

In Lab 1: AWS Control Tower Account Factory, we saw how **John** from CCoE team of organization-A deployed AWS Control Tower successfully and used Account Factory to provision brand new AWS accounts with company policies and governance in place. In this lab, we will walk through some of the day-to-day tasks that CCoE team would perform and see how to do those.

### Things to know

AWS Control Tower management operations like create/delete OUs, enable/disable guardrails, and create AWS account are single threaded. You will not be able to start second operation, until the current operation completes.

### What to expect in this lab:

* Access multiple accounts in the AWS Control Tower environment.
* Enable an auditor using pre-populated User Groups.
* [Optional] Enable and disable a guardrail on OU.
* Handling the drift.
* Access billing information across all accounts [Need 24-hours of data].
* AWS Control Tower Dashboard walk-through.

## 1. How to access multiple accounts in AWS Control Tower environment

By default, the *AWS Control Tower Administrator* will have *AWSAdministratorAccess *permissions to the *Core* accounts (Master, Log Archive and Audit). However, for AWS accounts provisioned using Account Factory, only AWSOrganizationsFullAccess permissions are granted.
In this section, we will see how to access the shared accounts using AWS SSO. Then we will switch to other accounts with the AWS Control Tower environment using switch-role functionality. We will review the steps involved in performing these operations both from the AWS Console and AWS CLI.

### 1.1 AWS Console

1.1.1 Log into the AWS Control Tower management console in the Master account:

1. When you launch AWS Control Tower, you will receive an email notification with User, portal URL and Username (referred to as admin user) to login to the AWS SSO authentication portal.
2. The email notification will have instructions to log in to AWS SSO and then to AWS Console on the AWS Control Tower master account.
3. Click on the Master account to expand. Select Management console next to AWSAdministratorAccess Role to login AWS Management console of the master account (as shown below).

![alt text](https://github.com/denegrij/controltower/blob/master/sso-control-tower-login.png?raw=true)
1. Select the service Control Tower under Management & Governance.

1.1.2 Switch roles to access the child account as Administrator:

The AWS Control Tower admin provides an email address when he/she creates a new AWS account using the Account Factory. 
The owner of that email address will be granted with *AWSAdministratorAccess* to the new account. 
By default, AWS Control Tower Administrator will have only *AWSOrganizationsFullAccess* to user accounts. 
If AWS Control Tower Administrator need to access the user account as administrator for operation reasons, the switch role functionality can be used (or granting permissions through AWS SSO). 
We will walk through how to perform the switch role.

We did this task already on the Lab 1, please feel free to skip this section if you are comfortable to do so.

1. Once you login into AWS Control Tower console in the **master** account, Click on *Username* on the top right corner next to the region and select Switch Role.
2. Enter the User Account ID that you like to connect under Account. Type *AWSControlTowerExecution* under Role and click on **Switch Role**.
3. This opens up the management console of the user Account with Administrator privileges.

### 1.2 Programmatic access

1.2.1 Instructions on how to use the CLI on AWS Control Tower environment:

1. Log into the AWS SSO screen, as described in the console access process. Choose **Command line or programmatic access**.

![alt text](https://controltower.aws-management.tools/core/cttasks/images/control-tower-access-keys.png?raw=true)
1. Copy the credential information from Option 1 and paste them on your terminal or putty session.

Run some sample awscli commands:


* You should see AWS Control Tower master account number by executing below command.

```
aws sts get-caller-identity --query 'Account' --output text
```

* Run below command to list out all the CloudFormation StackSets present on the master account.

```
aws cloudformation list-stacks --query 'StackSummaries[?StackStatus==`CREATE_COMPLETE`].StackName'
```

* * *

## 2. Enable an auditor using AWS SSO User Groups

As part of initial AWS Control Tower setup, an AWS SSO is configured with some pre-defined user groups to simplify the delegation of certain common operational tasks to other user/teams. In this section, as an example we will see how to enable an auditor to access the AWS environment with read-only access to all accounts under Control Tower through the predefined user group on SSO.

2.1.1 Create a user in AWS SSO:

1. Log into AWS Control Tower as the admin user, select Users and access from the left panel, and choose View in AWS Single Sign-on to open the AWS SSO console.
2. In the AWS SSO console, select Users from the left panel and choose Add User.
3. Fill out the form and select Generate a one-time password that you can share with the user to skip verification through email.
4. Choose Next: Groups.
5. Select AWSSecurityAuditors group and click on Add User

2.1.2 Share the login information:

1. Choose Copy details and share the information with the person who will be your auditor.
2. Using User Portal URL, Email and One-time password from the shared information, the auditor can log in to AWS environment.
3. On first login, the auditor will be prompted to reset the password (take note of the new password).
4. After that, the auditor will be granted read-only access to all the accounts in Control Tower.

Here is the list of pre-defined user groups in AWS SSO:

|User group name	|Description	|
|---	|---	|
|AWSAccountFactory	|Read-only access to account factory in AWS Service Catalog for end users	|
|AWSAuditAccountAdmins	|Admin rights to cross-account audit account	|
|AWSControlTowerAdmins	|Admin rights to AWS Control Tower core and provisioned accounts	|
|AWSLogArchiveAdmins	|Admin rights to log archive account	|
|AWSLogArchiveViewers	|Read-only access to log archive account	|
|AWSSecurityAuditors	|Read-only access to all accounts for security audits	|
|AWSSecurityAuditPowerUsers	|Power user access to all accounts for security audits	|
|AWSServiceCatalogAdmins	|Admin rights to account factory in AWS Service Catalog	|

* * *

## 3. AWS Control Tower dashboard operations

The AWS Control Tower initialization process creates two new core accounts for log archive, and security audit. 
It also creates two Organizational Units (OU) named as **Core** and **Custom**. The core accounts log archive, and audit account are placed in Core OU. The Custom OU is an empty OU created for accounts that will be provisioned for your users using Account Factory (you might choose to not use that one and create your own instead). 
The AWS Control Tower Administrator can create more OUs from the dashboard directly as per organizations requirement. 

At the moment, 20 preventive guardrails and 6 detective guardrails are available. Expect this number to grow in the near future. The strongly recommended guardrails are not enabled on any OUs by default. The administrator can enable them at OU level as needed.

In this section of the lab, we will see how-to:

* Create an Organizational Unit
* Enable a Guardrail on the Organizational Unit
* Disable a Guardrail on the Organizational Unit
* Delete an Organizational Unit

**Note**: Most of these operations are covered as part of the Lab 1. Feel free to skip this session as needed.

### 3.1 Create / Delete Organizational units

In this section, we will see how to create and delete an OU from AWS Control Tower dashboard. 

3.1.1 Create a new Organization Unit:

1. Login to AWS Control Tower Dashboard and click on Organizational units on the left Sidebar.
2. This opens up an Organizational units window. Click on Add an OU button.
3. Provide a new OU Name (for this lab we will call it as LABOU) and click on Add button.
4. Wait for green Success notification on top of the page.

3.1.2 Delete an Organizational Unit.

1. While you are still on AWS Control Tower Dashboard, Click on Organizational units on the left Sidebar.
2. Select the radio button next to the OU Name (LABOU for this lab) and click on Delete button.
3. This opens a warning message for confirmation as this operation cannot be undone. Click on Delete. 

**Note**: The dashboard doesn’t let you delete an OU if it contains any child accounts. Additionally it might take a few seconds until you are able to delete a recently created OU (as the mandatory guardrails will be enabled after creation).

### 3.2 Enable / Disable Guardrails

As part of Lab 1, we have already enabled the **Disallow internet connection through SSH** guardrail on the DEVENV organizational unit, we will now disable it:

1. Click on Guardrails on the left Sidebar and search for **Disallow internet connection through SSH** and click on it.
2. Scroll down to Organizational units enabled section and select the OU Name you want to disable guardrail on.
3. Click on Disable guardrail and it opens a window. Then click on Disable.
4. Wait for green Success notification on top of the page.

Please note that every time you enable or disable a guardrail, the CloudFormation StackSet will update the stacks belongs to that OU. Depending on number of AWS accounts you have in each OU, it may take few minutes for the process to complete. You could check the status of the operation by looking in to the StackSet instances of the individual guardrails. Please refer to [AWS CloudFormation Stacks Updates](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks.html) if you need additional help.

### 3.3 Drift detection and corrective steps

When you create your landing zone, all the OUs, accounts, and resources are compliant with all the governance rules enforced by your chosen guardrails. As you and your users use the landing zone, changes in this compliance status may occur. Some changes may be accidental, and some may be made intentionally to respond to time-sensitive operational events.

Regardless, changes can complicate your compliance story. You can use drift detection to identify resources that need changes or configuration updates to resolve the drift. Resolving drift helps to ensure your compliance with governance regulations, and is a regular operations task for your master account administrators.

Most common drift causes are related to changes in the OUs (like moving accounts between them) by doing it through the AWS Organizations console. Also, modifying SCPs or AWS Config rules are usual suspects on drifts.

Since the corrective operations are manual and could be time consuming, we are not going to have any scenarios in this lab at this point. Please refer to https://docs.aws.amazon.com/controltower/latest/userguide/drift.html?icmpid=docs_ctower_console for updated information on type of events that could cause drift and how to take corrective operations.
* * *

## 4. AWS Control Tower Dashboard walk-through

The AWS Control Tower dashboard provides a single pane of glass view for your AWS environment. It gives you all the information about your accounts and their compliance status at OU, Account and resources level. You could choose the name of the OU, to view the details of specific OU. Choose the name of the account, guardrails to view specific details. All the Non-compliant resources are listed on the dashboard and resources causing the violation can be browsed through directly from the dashboard.

In this section, we will walk through the components of the dashboard.

1. Login to AWS Control Tower Dashboard with AWSAdministratorAccess using the steps mentioned in 1.1.1
2. The Recommended actions pane provides the overview of actions you could do on the dashboard.
3. Scroll down and you could see OUs and Accounts summary under Environmental summary pane.
4. Under Guardrail summary, you could see summary of active guardrails in the environment
5. The Organizational units pane provides the overview of OUs in your AWS environment with their current Compliance status. You could click on OU name to look in to the details of the OU, Accounts, and Enabled guardrails.
6. Similar to Organizations units, you can browse through the Accounts, Enabled guardrails and Noncompliant resources directly from the dashboard.


***

[Next> Lab 3: AWS Control Tower Lifecycle Events](https://github.com/denegrij/controltower/blob/master/lab3-control-tower-lifecycle-events.md) 
