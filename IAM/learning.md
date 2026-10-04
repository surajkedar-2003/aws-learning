# AWS IAM

IAM stands for Identity and Access Management. It is used to control access to AWS resources.

## Root User

The root user is created when the AWS account is created.

It has full access to the AWS account, so I should not use it for normal day-to-day work.

## IAM User

An IAM user is an identity that can be given permissions to access AWS resources.

For example, I can create a user and give it permission to work with EC2 or S3.

## IAM Group

A group is used when multiple users need the same permissions.

Instead of attaching the same policy to every user, I can attach it to a group and add the users to that group.

## IAM Policy

A policy defines what actions are allowed or denied.

A policy can specify:

* Action
* Resource
* Effect (Allow or Deny)
* Conditions

Example:

```text
User
  |
  ↓
Policy
  |
  ↓
Allow → S3 read access
```

## IAM Role

An IAM role is used to give temporary permissions to a user, AWS service, or application.

One important use is with EC2.

For example, if an application running on EC2 needs to access S3, I can give the EC2 instance an IAM role with the required S3 permissions instead of putting AWS access keys in the application.

## Authentication vs Authorization

**Authentication** → Who are you?

**Authorization** → What are you allowed to do?

For example, logging in proves who the user is. Permissions decide what that user can access.

## Least Privilege

Only give the permissions that are actually required.

For example, if an application only needs to read files from S3, I don't need to give it permission to delete files.

## What I Practiced

### IAM User

I created an IAM user and worked with its permissions.

I checked what the user could access and what happened when a required permission was not available.

### IAM Group

I created a group and attached a policy to it.

Then I added a user to the group and checked the permissions received by the user.

### IAM Policy

I created and changed policies to see how the permissions affected access.

This helped me understand that creating an IAM user does not automatically give that user access to everything in AWS. The required permissions have to be provided.

## My Understanding

The basic way I understand IAM is:

```text
User / Application
        |
        ↓
       IAM
        |
        ↓
   Permissions
        |
   +----+----+
   |    |    |
  EC2  S3   RDS
```

IAM controls which actions an identity can perform on AWS resources.
