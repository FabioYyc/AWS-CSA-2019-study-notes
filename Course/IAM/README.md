# CHAPTER 3 | IAM Identity Access Management

## [What's IAM](https://aws.amazon.com/iam/)

Allows you to manage users and their level of access to the AWS console.

## IAM Consist of the following

* Users: People, end users.
* Groups: Collection of users, users in the groud will inherited the policy from the group.
* Policies: Are basically the Permissions. Are made up of Documents, formatted in JSON.
* Roles: Is a way to allow one part of AWS to another part.

## Features

* Centralised control of your AWS account.
* Shared Access to your AWS account.
* Gives you granular Permission (differentiates authority levels).
* Does Identity Federation (Active directory, Facebook, etc), your users can use account for other entity for the AWS
* MultiFactor Authentication, when you logging in, you need the authentication code not only the username and password.
* Provides temporary access for users/devices etc.
* Allows you to set up your own password rotation policy, for example you can let the user set new password in a period.
* Supports PCI DSS compliance, compliant with credit card policy etc.
* integrated with a lot of AWS services.


## [From Console](https://aws.amazon.com/console/)

IAM users sign-in link:
You can customize the URL used by users to sign-in

## New Users and account set up

* IAM is universal, the region is on a global level, no region is applied.
* Root account: Is the user you used to register on AWS, no one should use it to log-in.
* New user has no permission when created, we have to give them new premission.
* Always enable MFA on your Root account.
* New Users have No permissions once created.
* Access keys ID & Secret are not the same as password, can be used to interact with AWS console.
* Autogenated password should be changed after user first signed in (reset password setting).
* You can create and customize your password rotation policies.
* When creating a group, there are job function options for policies. 
* To inspect policy, it provides a json format of details, AdminAccess is the most powerful policy you can have.

## [Billing Alarm](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/monitor_estimated_charges_with_cloudwatch.html)

You can set a billing alarm to get notified when the amount of spendings for the current month reaches your threshold.
Before doing it you probably have to go on:
```https://console.aws.amazon.com/billing/home?#/preferences```
and enable: Receive Billing Alerts

After that, you can go and use cloudwatch services to set a new Billing Alarm

* Potential exam question: How to send notification when bill exceeds 1000 USD? -> set billing alarm in CloudWatch which uses an sns topic

## IAM Lab notes
* To sign into console, you should use your IAM username and password. Or you can login into the root account to manage the console with AWS account.
* To access user credential, you can rest the password and download the new csv.
* Access ID and screte is used for: Use access keys to make secure REST or HTTP Query protocol requests to AWS service APIs. Like sendgrid or other APIs, they cannot be used to log into the console.


