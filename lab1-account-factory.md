[Back to main](https://github.com/denegrij/controltower/blob/master/overview.md) 

***

# Lab 1: AWS Control Tower Account Factory

## Overview

### Scenario

**John** is a member of the Cloud Center of Excellence (CCoE) team at **organization-A**, responsible for architecting the cloud infrastructure in the most secured way and at the same time without slowing down the business users. 

CCoE team is also accountable to provision AWS accounts to multiple Line of Businesses (LOBs) across the organization. **Martha** is one of the web developers in a LOB team who is responsible for maintaining the web-apps across the organization.
John decided to take advantage of *Account Factory* feature in *AWS Control Tower* to provision new accounts for LOBs. 

Provisioning accounts through *Account Factory* allows **John** to create new AWS accounts with AWS best practices in place along with proper governance from the beginning.

**Martha** develops and uses Infrastructure as Code templates to deploy web services on the new AWS accounts. **John** and his team will use *AWS Control Tower* to watch out for any violations that the LOB teams may cause unintentionally and take corrective actions.


### Lab description

This lab will walk you through the steps involved in configuring an AWS account which conforms to your company-wide policies on creation. We will also deploy a sample web application based on the LAMP stack using an *AWS CloudFormation* template, and see how the* AWS Control Tower* guardrails will watch out and report any policy violations

We will perform the following activities in this lab:

* Create an *Organizational Unit (OU)* and enable a guardrail from the *AWS Control Tower* dashboard.
* Modify the network baseline settings of Account Factory.
* Launch a new AWS account using Account Factory as an *AWS Control Tower* Admin user.
* *[Optional]* Designate a user/group with non-admin permissions to use Account Factory to launch AWS accounts.
* Launch a Web application based on the LAMP stack, which by default opens SSH ports for inbound connections without restrictions.
* Investigate the violation captured by the AWS Control Tower and take corrective action.

## 1. AWS Control Tower environment setup

In this section, we will walk through various AWS Control Tower operations that you could do before provisioning an account. 
In this lab, we are going to configure the AWS Organization structure that fits the web-apps use case we described earlier.

### 1.1 Create an Organizational Unit

1.1.1 With AWS SSO, log into the AWS Control Tower management console in the **Master** account.

1. When you launch AWS Control Tower, you will receive an email notification with User portal URL and Username (referred to as admin user).
2. The email notification will have instructions to log in to AWS SSO and then to AWS Console on the AWS Control Tower master account.
3. Click on the **Master** account to expand. Select Management console next to *AWSAdministratorAccess* Role to login AWS Management console of the master account (as shown below)

![alt text](https://github.com/denegrij/controltower/blob/master/sso-control-tower-login.png?raw=true)

1. Select the service Control Tower under Management & Governance.

1.1.2 Create a new Organization Unit from the *AWS Control Tower* dashboard:

1. From the AWS Control Tower Dashboard, click on **Organizational units** on the left Sidebar.
2. This opens up Organizational units page. Click on **Add an OU** button.
3. Provide a new OU Name (for this lab we will call it as DEVENV) and click on **Add** button. Wait for green Success notification on top of the page.

### 1.2 Enable a Strongly recommended Guardrail on the OU we just created

1.2.1 Enable a Strongly recommended Guardrail on new OU:

1. On AWS Control Tower Dashboard, click on **Guardrails** on the left Sidebar.
2. Search for **Disallow internet connection through SSH** and click on it.
3. Scroll down to Organizational units enabled section and click on **Enable Guardrail on OU** button.
4. Select the name of the OU created on step 2.1.1. (DEVENV for this lab) and click on **Enable guardrail on OU** button.
5. Wait for green Success notification on top of the page.

### 1.3 Modify Network baseline settings of the Account Factory

1.3.1 Modify network configurations for new accounts:

1. While you are still on AWS Control Tower Dashboard, Click on **Account factory** on the left Sidebar and click on **Edit** button.
2. Under Edit account factory configuration, enable **Internet-accessible subnet** (required for the lab) and click on **Save**.

![alt text](https://github.com/denegrij/controltower/blob/master/control-tower-af-network-settings.png?raw=true)
1. Wait for green Success notification on top of the page.

Please ***DO NOT SKIP*** this step. Creating a internet-accessible subnet is required for this lab.

So far we were able to create an OU, enable a Strongly recommended guardrail on that OU and modify the network baseline settings.

## 2. Launch a new AWS account using Account Factory

In this section we will walk-through the steps involved in provisioning a new AWS account as an AWS Control Tower admin user. We will use the Service Catalog product called AWS Control Tower Account Factory, created when AWS Control Tower was deployed.
In the next section we will see how to enable a user/group with no admin rights to use Account Factory.

Few thing to keep in mind before proceeding further with the lab:

* While admin user can access the Account Factory directly with AWSAdministratorAccess role, the new non-admin user with permissions to Account factory, should login using `AWSServiceCatalogEndUserAccess` role to create new accounts.
* Ensure you are in the same region as AWS Control Tower, this is needed as AWS Service Catalog is a regional service.
* If you login using `AWSServiceCatalogEndUserAccess` role, you won’t be able to access AWS Control Tower dashboard but you can directly access AWS Service Catalog.
* The email IDs used for the accounts in AWS Control Tower should be in the same domain. As an example, having master account in *@example.com* and new AWS account in *@noexample.com* will NOT work.

### 2.1 Launch a new AWS Account using Account Factory as an AWS Control Tower Admin user

2.1.1 Provision new account using `Enroll account` option:

1. On AWS Control Tower service console, choose **Account factory** from left side panel.
2. Click on the **Enroll account** button.
3. `Fill in the form`, pick a *ManagedOrganizationalUnit* from the drop down. Click on **Enroll account**

![alt text](https://github.com/denegrij/controltower/blob/master/quick-account-provisioning-launch.png?raw=true)
1. As indicated in the `blue banner` on top of your screen, you could trace the status of account provisioning from *AWS Service Catalog service console* under individual Provisioned Product Name.
2. The status of the launch can be monitored from the AWS Service Catalog dashboard, under the Provisioned products list by selecting the individual Provisioned Product Name

### 2.2 [optional] Create a new user and allow access to Account Factory

Use this procedure to delegate new AWS account creation activity to a user/group with no admin rights. We will use a pre-configured AWS SSO group to perform this task. 

2.2.1 Log into the AWS Control Tower management console in the **Master** account:

1. On *AWS Control Tower* management console, choose **Users and Access** from the left side navigation panel.
2. Under User identity management, choose **View in AWS Single Sign-On**

![alt text](https://github.com/denegrij/controltower/blob/master/control-tower-view-sso.png?raw=true)
1. An AWS SSO page will open. Then choose **Users** from the left side navigation panel.
2. Choose **Add user**
3. `Fill in the form` and choose **Next: Groups**
4. Select **AWSAccountFactory**, and then choose **Add user**.
5. The user will receive an email with a link to Accept Invitation, User portal URL, and Username.
6. When the user accepts the invitation in its mailbox, it’ll need to generate a new password.
7. The new user can log in to User portal URL with those credentials. The new user will now have the necessary AWSServiceCatalogEndUserAccess permissions to use Account Factory to create new accounts

![alt text](https://github.com/denegrij/controltower/blob/master/control-tower-new-user-login.png?raw=true)
## 3. Launching a PHP portal using LAMP Stack on newly provisioned account

**John** provisioned a new AWS account using Account Factory in AWS Control Tower. The new AWS account is ready to be used by the LOB team. 
**Martha** from The LOB team owns an AWS CloudFormation template which she will use to deploy a web-app across the organization. 
In this section we will see how **Martha** deploys their standard web-apps with an AWS CloudFormation template.

### 3.1 Launch a PHP portal on the newly provisioned account

3.1.1 Login to the new AWS account as account owner (or you can assign AWSAdministratorAccess permissions to the AWS Control Tower admin user for that AWS account):

1. When a new AWS account was launched by **John **from CCoE team, an email with SSO Portal URL and login information will be sent automatically to the email address provided during account creation.
2. Using the information provided in the auto-generated email, **Martha** can setup a password and login to the new AWS account.
3. **Martha** will see an AWS SSO screen identical to **John**, but with just the newly created AWS account listed out with role AWSAdministratorAccess.

3.1.2 Deploy the AWS CloudFormation stack to install a web application:

1. Login to AWS CloudFormation console screen [AWS Console](https://console.aws.amazon.com/cloudformation)
2. Click on **Create Stack**, under **Choose a template** select **Specify an Amazon S3 template URL**, copy-paste the below link and click Next:

`https://marketplace-sa-resources.s3.amazonaws.com/LAMP_Simple_single_instance_opt.json`

1. Provide a Stack name as *PHPSampleWebApp*, select *VPC* which is labelled as *aws-controltower-VPC*, select *WebSubnetId* whose label starts with *aws-controltower-PublicSubnet* and leave other options as default for this lab.
2. Click **Next**, review the options you select and click **Next** again.
3. Wait for the Stack Status becomes CREATE_COMPLETE.
4. Select Outputs tab and click on the value of WebsiteURL to visit the newly launched PHP website.

## 4. Investigate violation reported on the AWS Control Tower Dashboard

When **Martha** deployed the AWS CloudFormation stack they usually run across multiple accounts, she accidentally left the SSH ports opened to the entire world unintentionally. 
In the AWS Control Tower Dashboard, **John’s team** can easily trace this and take appropriate corrective actions on it.

In this part of this lab, we will see how the monitor your AWS Control Tower environment using the dashboard.

4.1.1 Check the AWS Control Tower Dashboard:

1. Login to AWS Control Tower Dashboard with *AWSAdministratorAccess *using the steps mentioned in 1.1.1
2. Scroll down to **Noncompliant resources**. You would see the Resource causing the violation.
3. Click on the Link under Account Name to open complete Account details of the account with violations. Note down the Account ID and Resource ID that is causing the violation (like sg-xxxxxxxxxxxx)
4. You could use the account owner information available on this page to notify about this violation.

4.1.2 Switch to LOB account to fix the violation

1. Click on Username on top right corner next to the region and select **Switch Role**.
2. Enter the **Account ID** noted earlier under Account. Type ***AWSControlTowerExecution* **under Role and click on Switch Role
3. Type http://console.aws.amazon.com/vpc on the browser to access VPCs on the LOB account
4. Click on **SecurityGroups** and select the Security Group that starts with *PHPHelloWorldSample-xxx*
5. Select Inbound Rules tab in the bottom panel and click on Edit rules. For SSH type the appropriate internal IP address range. For this lab we will select My IP and Save rules.

4.1.3 Verify that the violations are fixed

1. Switch back to the AWS Control Tower Dashboard in the master account by clicking on the username on top right and Back to AWSReservedSSO_AWSAdministratorAccess_*
2. Type http://console.aws.amazon.com/controltower on the browser to access Control Tower Dashboard on Master Account
3. Note the violation noted earlier is cleared now and all the resources are Compliant. It could take few minutes to get the dashboard updated.

* * *

## Deleting AWS resources deployed in this lab

1. You need to terminate any stacks deployed in addition for this lab to avoid extra costs.
2. Follow below steps to remove an account provisioned through Account Factory. 
    1. [Unmanage a member account](https://docs.aws.amazon.com/controltower/latest/userguide/account-factory.html#unmanage-account) 
    2. [Closing an Account Created in Account Factory](https://docs.aws.amazon.com/controltower/latest/userguide/account-factory.html#delete-account)
3. Follow the steps [here](https://docs.aws.amazon.com/controltower/latest/userguide/walkthrough-delete.html) to clean up AWS Control Tower managed resources. 
***

[Next> Lab 2: Tasks in AWS Control Tower](https://github.com/denegrij/controltower/blob/master/lab2-tasks-in-control-tower.md) 
