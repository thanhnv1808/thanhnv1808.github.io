---
title: "AWS Solutions Architect - Lesson 02: IAM Service & Organization Service"
author: thanhnv1808
date: 2025-12-24 11:00:00 +0700
categories: [AWS, Solutions Architect]
tags: [aws, solutions-architect, iam, organizations, security, access-management]
description: Deep dive into AWS IAM (Identity and Access Management) and AWS Organizations. Learn about users, groups, roles, policies, and multi-account management.
pin: false
comments: true
---

This lesson covers **IAM (Identity and Access Management)** and **AWS Organizations** - two critical services for managing access control and multi-account governance in AWS.

## 📚 Part 1: Knowledge

### 1. IAM (Identity and Access Management)

**IAM** is a service that helps control access to AWS resources.

#### Key Characteristics

- **Billing**: All billing is charged to the root user
- **Service Type**: Global Service (not region-specific)
- **Pricing**: **Free**

#### IAM Flow

The IAM flow consists of several components:

1. **Principals**
   - A person or application that makes a request to AWS resources
   - Must be **Authenticated** (signed in) and **Authorized** (has permissions) through policies
   - AWS checks if the principal has permission to access the resource through policies

2. **Request**
   - After authentication, AWS executes the actions and operations in the request
   - Different services have different actions and operations

3. **Resources**
   - The AWS resources from various services

> **Reference**: [IAM Structure](https://docs.aws.amazon.com/IAM/latest/UserGuide/intro-structure.html)
{: .prompt-info }

#### IAM Components

##### Identity

Identity includes:
- **Users** (Root User, IAM User, Federating existing user)
- **Groups**
- **Roles** / Temporary security credentials
- **AWS resources** (S3, EC2, applications, ...)

---

### 2. Root User

The **Root User** is the account owner created when you first set up your AWS account.

#### Login Methods

- **UI Console**: Login with email & password
- **CLI, SDK**: Login through access keys

#### Characteristics

- Has **full admin permissions**
- Cannot be restricted or limited
- All billing is charged to this account

#### Best Practices

> ⚠️ **Critical Security Practices**:
{: .prompt-warning }

1. **Don't use frequently** - Only log in when you need to:
   - Grant permissions to Users
   - View and pay billing

2. **Enable MFA** and **remove access keys**
   - Prevents hackers from scanning if code is uploaded to Git
   - Access keys should never be committed to version control

3. **Create IAM Group & IAM User with admin permissions**
   - Use IAM users for daily operations instead of root user

4. **Set password policy for IAM Users**
   - Ensures security for all IAM users

#### Root User Attributes

- **Access Key** (access key ID + secret access key)
- **Password policy** for IAM Users
- **MFA (Multi-Factor Authentication)**
  - Requires additional token from another device to log in
- **Security Token Service (STS)**

#### Root User Settings

**Login**: Configure login credentials

**Basic information**: Account details

**Security**:
- Password policy
- MFA configuration

---

### 3. IAM User

**IAM Users** are identities within your AWS account that represent people or applications.

#### Login Methods

- **UI Console**: Login with username & password
- **CLI, SDK**: Login through access keys

#### Characteristics

- **Limited permissions** as granted by Root User
- Can have multiple IAM users within one Root user
- Each IAM user is independent within the Root user

#### Best Practices

- Each IAM user can represent:
  - A person
  - An application
  - A service
  - ... or any entity that needs AWS access

- **Long-term credentials** (unlike temporary credentials)

#### IAM User Attributes

1. **Account ID (Alias Account)**, username, password, console link
   - Information for user to log in through console

2. **Access Key** (access key ID + secret access key)
   - For login through CLI, SDK, API
   - Can be used for services connecting to each other (can use Role instead)
   - **Maximum 2 access keys per user**

3. **Password rotation policy**
   - Enforce periodic password changes

4. **MFA**
   - Multi-factor authentication for additional security

5. **Access Type**
   - **Programmatic access**: Access key and secret key for tools like API, CLI, SDK, ...
   - **AWS Management Console access**: Password for console login (generated or custom)

6. **Permissions**
   - Use policies to define permissions
   - Options:
     - **Group for user** (recommended)
     - **Copy permission from another user**
     - **Attach permission directly to user**

7. **Permissions Boundary**
   - Limits maximum permissions
   - Cannot add permissions beyond what's defined

8. **X.509 Certificates**
   - For SSL/TLS server certificates for HTTPS connections for websites & applications

9. **SSH keys for AWS CodeCommit**
   - Similar to private Git

10. **Access Advisor**
    - Shows information about the last access to services the user has used

#### Bulk User Creation

- Can create multiple users at once
- Download user information (CSV format)
- Send notifications to users

---

### 4. Federating Existing User

**Federated Users** allow login through Single Sign-On (SSO) without creating IAM Users.

#### Use Cases

- Login with Amazon, Facebook, Google, any OpenID Connect (OIDC), ...
- Allow users already logged in through company accounts
- Use **Session Role** to define permissions

#### Benefits

- No need to create and manage IAM users
- Centralized identity management
- Users can use existing credentials

---

### 5. IAM Group

**IAM Groups** are collections of IAM users that share the same policies.

#### Characteristics

- Can have multiple Groups within one Root user
- One group includes a list of users with the same policies
- **One user can belong to many groups**

#### Best Practices

- Each User in a group typically has the same role
- Examples: project, department, position, ...

#### Group Attributes

- **Name**
- **Permissions** (maximum 10 policies)
- **IAM users**
- **Access Advisor**
  - Shows information about the last access to services users have used

> **Note**: Groups cannot be nested. A user can belong to multiple groups, and permissions are additive.
{: .prompt-info }

---

### 6. IAM Role

**IAM Roles** are similar to users but don't have username, password, or access keys.

#### Characteristics

- Contains a set of policies
- **Reusable** across different entities
- **Temporary credentials** (unlike IAM users)

> Some AWS services need to perform actions on your behalf
{: .prompt-tip }

#### Use Cases

Roles can be assigned to:
1. **AWS Service**
   - Allows AWS services (EC2, Lambda, ...) to perform actions on behalf of the root user
   - After creating Role, assign it to EC2
   - **More secure** than storing access keys on EC2 applications

2. **Other AWS Account**
   - Allows another account or 3rd party to access resources
   - Example: Create 2 different accounts for dev and production
   - Allow developers to update resources on production account

3. **Web Identity**
   - Used for Cognito or OpenID

4. **SAML 2.0 Federation**
   - Used for Single Sign-On (SSO), corporate directory (company network)
   - Assign to Federating existing user to use as an external user (instead of IAM user)

#### Role Attributes

- **ARN** (Amazon Resource Name)
- **Maximum session duration**
- **Type of trusted entity**:
  - AWS Service (EC2, Lambda, ... other AWS services)
  - Other AWS Account (for other accounts or 3rd party)
  - Web Identity (for Cognito or OpenID)
  - SAML 2.0 federation (for corporate directory)
- **Permissions**
- **Access Advisor**

#### Role vs IAM User

| Feature | IAM User | IAM Role |
|---------|----------|----------|
| Credentials | Permanent | Temporary |
| Username/Password | Yes | No |
| Access Keys | Yes | No |
| Reusable | No | Yes |
| Use Case | People/Applications | Services/Cross-account |

---

### 7. Temporary Security Credentials

**Temporary security credentials** are temporary login information that automatically expire after a period of time.

#### Characteristics

- Similar to Role but **not reusable**
- Session is generated from **AWS Security Token Service (AWS STS)**
- **Short-term** credentials

#### Use Cases

- Used in combination with Federating existing user for single sign-on login
- Cross-account access
- Service-to-service communication

> **Reference**: [AWS STS](https://sts.amazonaws.com)
{: .prompt-info }

---

### 8. Policy

**Policies** define which actions an identity can perform on which resources under what conditions.

#### Characteristics

- Defined in **JSON format**
- Can attach policy to multiple Identities and AWS resources
- When attaching policy to identity, it's called **permission**
- AWS calculates and combines policies to determine if request is allowed or denied
- **Deny takes precedence over Allow**
- Can create multiple policies and combine them
- **Principle of least privilege**: Only grant necessary permissions

#### Policy Components

1. **Version**: Policy language version
2. **Statement**: A single permission
   - **Sid** (Optional): Statement ID
   - **Effect**: Allow or deny request
   - **Principal**: Specifies identity
   - **Action/NotAction**: Action to perform with AWS Service
   - **Resource**: Specifies resource through ARN (resource ID)
   - **Condition** (Optional): Conditions for resource to apply

#### Policy Example

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3Read",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:user/John"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "StringEquals": {
          "aws:MultiFactorAuthPresent": "true"
        }
      }
    }
  ]
}
```

#### Policy Types

1. **Identity-based Policy**
   - Policy for identity (User, Group, Role)

2. **Resource-based Policy**
   - Inline policy for resource (S3 bucket, EC2, ..., IAM User, IAM Group, IAM Role)
   - Can be on the same account or grant cross-account access (similar to Other AWS Account in IAM Role)

3. **Session Policy**
   - Temporary Session for Role or Federated User in API, CLI

4. **Permissions Boundary**
   - Limits permissions that can be performed for User, Role (does not support Group)
   - Action can only be performed when both identity-based policy & Permission boundary allow it
   - Use cases:
     - Create Role for Lambda function
     - Create Role for EC2 instances
     - Admin limits permissions for Groups & Users

5. **Organizations SCP**
   - Used for AWS Organizations service control policy (SCP)
   - SCP is a service to group and centrally manage AWS accounts

6. **ACL (Access Control List)**
   - Does not use JSON
   - Allows other accounts to access (similar to Resource-based policy)

> **Reference**: 
> - [Identity vs Resource-based Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_identity-vs-resource.html)
> - [Policy Evaluation Logic](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html)
> - [Policy Examples](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_examples.html)
{: .prompt-info }

#### Policy Evaluation Logic

AWS evaluates policies in this order:

1. **Explicit Deny** - Overrides everything
2. **Explicit Allow** - Grants permission
3. **Implicit Deny** - Default if no explicit allow

#### Policy Management Types

1. **Customer Managed**
   - Policies you create yourself
   - Full control over content

2. **AWS Managed**
   - Policies created by AWS
   - Updated by AWS automatically
   - Cannot be modified

---

### 9. IAM Tools

#### Security Tools

1. **Access Advisor (User-level)**
   - View which services IAM User has used most recently
   - Review policies granted to User
   - Identify unused permissions

2. **Credentials Report (Account-level)**
   - Similar to Access Advisor but lists all IAM Users in Account
   - Shows password age, access key age, last login, etc.

3. **IAM Access Analyzer**
   - Tool to analyze policies and permissions
   - Identifies resources shared with external entities
   - Helps identify security risks

4. **Visual Editor**
   - Tool to visualize policy
   - Easier to understand policy structure

#### Access Methods

1. **Console**
   - Login through Email/Username, password, MFA
   - Allows operations with AWS through UI

2. **CLI (Command Line Interface)**
   - Login through access keys
   - Tools to operate with AWS through command-line shell
   - Can use CloudShell directly on AWS Console

3. **SDK (Software Developer Kit)**
   - Login through access keys
   - Used for programming, operating with AWS service through API
   - Supports:
     - **Web**: JavaScript, Python, PHP, .NET, Java, Go, Node.js, C++
     - **Mobile**: Android, iOS
     - **IoT Devices**: Embedded C, Arduino, ...

---

### 10. AWS Organizations

**AWS Organizations** is a service to manage multiple accounts (not IAM Users).

#### Key Characteristics

- **Consolidated billing** in one organization
- **Global Service**
- **Pricing**: **Free** (with discounts available)

#### Benefits

- **Consolidated Billing**: All accounts billed to one account
- **Volume Discounts**: Better pricing with combined usage
- **Centralized Management**: Manage multiple accounts from one place
- **Service Control Policies**: Apply policies across accounts

---

### 11. Organizations Components

#### Organization Root

The **Organization Root** is a container for OUs and policies.

**Components:**
- **Management Account**
- **Organizational Units (OUs)**
- **Policies** (applied to all OUs and Member Accounts)

#### Organizational Unit (OU)

**OUs** are like containers or branches for accounts below them.

**Common Use Cases:**
- Departments
- Development Environments
- Projects
- Business Units

**OU Attributes:**
- **Nested OUs**: OUs can contain other OUs
- **Member Accounts**: Accounts within the OU
  - Can invite existing accounts or create new ones
  - Can move accounts to different OUs
- **Policies**: Applied to all Member Accounts

#### Account Types

1. **Management Account**
   - All billing of member accounts is charged to management account
   - Should only be used for management tasks (Add, edit, remove, ...), billing
   - **Cannot be moved** to another organization

2. **Member Account**
   - Each account has its own billing and resources
   - Can only participate in one organization at a time
   - Use SCP to limit permissions of each member account

#### Service Control Policy (SCP)

**SCPs** manage permissions for OUs and Member Accounts (does not apply to Management Account).

**Hierarchy:**
- When assigning policy to root → OUs inherit
- When assigning policy to OU → Member Accounts & nested OUs inherit
- When assigning policy to Member Account → IAM Users inside also inherit

**Syntax**: Similar to IAM Policy

**Policy Types:**

1. **Service Control Policy (SCP)**
   - Manage permissions for Root, OU, Account
   - Examples: [SCP Examples](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps_examples.html)

2. **AI Services Opt-out Policies**
   - Opt out of AI services

3. **Backup Policies**
   - Help backup resources for multiple accounts

4. **Tag Policies**
   - Enforce tagging standards across accounts

#### Organization Structure Example

```
Organization Root
├── Management Account
├── Production OU
│   ├── Production Account 1
│   └── Production Account 2
├── Development OU
│   ├── Dev Account 1
│   └── Dev Account 2
└── Testing OU
    └── Test Account 1
```

---

## 📝 Part 2: Assignment

### Exercise 1: IAM Service

#### Question 1: Create IAM Group & User

Using IAM Service, create groups and users as shown in the diagram following these steps:

**B1: Create User**

**B2: Create Group**

**B3: Create Password Policy for IAM User**
- At least 10 characters
- At least 1 uppercase letter, 1 number, and 1 special character
- Must change password every 1 month, new password cannot be the same as old password
- Change password on first login

**B4: Create MFA for Root User & IAM User**
- Use Google Authenticator or Authy to create MFA

**B5: Login to any IAM user on Console & CLI & CloudShell**

#### Question 2: Create Identity-based Policies & Grant to User

a) Grant Group Admins full permissions with all AWS Services

b) Grant Group Developer full permissions with services: S3, EC2
   - Jim is the leader, so he has additional read permissions for all information of Group Developers members

c) Grant Group Test read permissions with services: S3, EC2

d) Create Role & Policies to allow one of your classmate's accounts to access and view information of Group Developers

**Tutorial**: [YouTube Tutorial](https://www.youtube.com/watch?v=20tr9gUY4i0)

**Note**: After completing, test through Console or CLI

#### Question 3: Create Permissions Boundary

Limit permissions of Group Developer to only use Service: S3

**Note**: After completing, test through Console or CLI

**Reference**: [Policy Examples](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_examples.html)

#### Question 4: CLI (Personal Account)

Research a few CLI commands for services: IAM, S3, EC2, ...

#### Question 5: Tools (Personal Account)

Check user activities through 2 tools:

a) **Access Advisor** (User-level)

b) **Credentials Report** (Account-level)

---

### Exercise 2: Organizations Service (Personal Account)

#### Question 1: Create Organization

Using Organization Service, create OUs & Member Accounts as shown in the diagram following these steps:

**B1: Create Organization**

**B2: Create OU**

**B3: Create Member Account & add to each OU**

**B4: Add policy for OUs & Member Accounts**

#### Question 2: Check Billing of Accounts in Organization

---

## ✅ Part 3: Solutions

### Exercise 1: IAM Service

#### Question 1: Create IAM Group & User

##### Step 1: Create IAM Users

1. Navigate to **IAM** → **Users** → **Add users**
2. Enter username (e.g., `john`, `jane`, `jim`)
3. Select access type:
   - **AWS Management Console access** (for console login)
   - **Programmatic access** (for CLI/SDK)
4. Set password:
   - **Autogenerated password** or **Custom password**
   - **Require password reset** (recommended for first login)
5. Click **Next: Permissions**

##### Step 2: Create IAM Groups

1. Navigate to **IAM** → **Groups** → **Create New Group**
2. Group names to create:
   - `Admins`
   - `Developers`
   - `Test`
3. Click **Next Step** (we'll add policies later)

##### Step 3: Create Password Policy

1. Navigate to **IAM** → **Account settings** → **Password policy**
2. Click **Change password policy**
3. Configure:
   - ✅ **Minimum password length**: 10
   - ✅ **Require at least one uppercase letter**
   - ✅ **Require at least one lowercase letter**
   - ✅ **Require at least one number**
   - ✅ **Require at least one non-alphanumeric character**
   - ✅ **Enable password expiration**: 30 days
   - ✅ **Prevent password reuse**: 1 (cannot reuse last password)
   - ✅ **Require users to change their password on next sign-in**
4. Click **Save changes**

> **Best Practice**: Always enforce strong password policies to enhance security.
{: .prompt-tip }

##### Step 4: Enable MFA for Root User

1. Click on your **account name** (top right) → **Security credentials**
2. Scroll to **Multi-factor authentication (MFA)**
3. Click **Assign MFA device**
4. Choose **Virtual MFA device** (for Google Authenticator or Authy)
5. Open your authenticator app and scan the QR code
6. Enter two consecutive MFA codes
7. Click **Assign MFA device**

##### Step 5: Enable MFA for IAM Users

1. Navigate to **IAM** → **Users** → Select a user
2. Go to **Security credentials** tab
3. Click **Assign MFA device**
4. Follow the same steps as Root User MFA

##### Step 6: Login to IAM User

**Console Login:**
1. Sign out from current session
2. Use IAM user sign-in link: `https://YOUR-ACCOUNT-ID.signin.aws.amazon.com/console`
3. Enter username and password
4. Enter MFA code if enabled

**CLI Login:**
1. Configure AWS CLI:
```bash
aws configure
```
2. Enter:
   - AWS Access Key ID
   - AWS Secret Access Key
   - Default region
   - Default output format

**CloudShell:**
1. Click **CloudShell** icon in AWS Console
2. Already authenticated with current user
3. Start using AWS CLI commands

---

#### Question 2: Create Identity-based Policies

##### a) Grant Admins Full Permissions

1. Navigate to **IAM** → **Groups** → **Admins**
2. Click **Permissions** tab → **Add permissions** → **Attach policies**
3. Search and select: **AdministratorAccess**
4. Click **Add permissions**

##### b) Grant Developers Permissions

**For Group Developers:**
1. Navigate to **IAM** → **Groups** → **Developers**
2. Click **Permissions** tab → **Add permissions** → **Create inline policy**
3. Use JSON editor:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:*",
        "ec2:*"
      ],
      "Resource": "*"
    }
  ]
}
```
4. Name: `DevelopersFullAccess`
5. Click **Create policy**

**For Jim (Additional Read Permissions):**
1. Navigate to **IAM** → **Users** → **jim**
2. Click **Add permissions** → **Create inline policy**
3. JSON:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "iam:GetUser",
        "iam:ListUsers",
        "iam:GetGroup",
        "iam:ListGroups",
        "iam:ListGroupsForUser"
      ],
      "Resource": "*"
    }
  ]
}
```
4. Name: `ReadDevelopersInfo`
5. Click **Create policy**

##### c) Grant Test Group Read Permissions

1. Navigate to **IAM** → **Groups** → **Test**
2. Click **Permissions** tab → **Add permissions** → **Create inline policy**
3. JSON:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:Get*",
        "s3:List*",
        "ec2:Describe*"
      ],
      "Resource": "*"
    }
  ]
}
```
4. Name: `TestReadOnly`
5. Click **Create policy**

##### d) Create Cross-Account Role

1. Navigate to **IAM** → **Roles** → **Create role**
2. Select **Another AWS account**
3. Enter your classmate's **Account ID**
4. Optionally require MFA
5. Click **Next: Permissions**
6. Attach policy with read permissions for Developers group:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "iam:GetUser",
        "iam:ListUsers",
        "iam:GetGroup",
        "iam:ListGroups"
      ],
      "Resource": "*"
    }
  ]
}
```
7. Name: `CrossAccountDevelopersAccess`
8. Click **Create role**
9. Share the **Role ARN** with your classmate

**On Classmate's Account:**
1. Create a role that trusts your account
2. Use the role ARN you provided

---

#### Question 3: Create Permissions Boundary

1. Navigate to **IAM** → **Policies** → **Create policy**
2. Use JSON editor:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:*"
      ],
      "Resource": "*"
    }
  ]
}
```
3. Name: `S3OnlyBoundary`
4. Click **Create policy**

5. Navigate to **IAM** → **Groups** → **Developers**
6. Click **Permissions boundary** → **Set boundary**
7. Select `S3OnlyBoundary`
8. Click **Set boundary**

**Test:**
- Try to access EC2 → Should be denied
- Try to access S3 → Should be allowed

---

#### Question 4: CLI Commands

**IAM Commands:**
```bash
# List users
aws iam list-users

# Get user details
aws iam get-user --user-name john

# List groups
aws iam list-groups

# List policies
aws iam list-policies

# Attach policy to user
aws iam attach-user-policy --user-name john --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

**S3 Commands:**
```bash
# List buckets
aws s3 ls

# Create bucket
aws s3 mb s3://my-bucket

# Copy file
aws s3 cp file.txt s3://my-bucket/

# Sync directory
aws s3 sync ./local-folder s3://my-bucket/
```

**EC2 Commands:**
```bash
# List instances
aws ec2 describe-instances

# Create instance
aws ec2 run-instances --image-id ami-12345 --instance-type t2.micro

# Terminate instance
aws ec2 terminate-instances --instance-ids i-12345
```

---

#### Question 5: IAM Tools

##### a) Access Advisor (User-level)

1. Navigate to **IAM** → **Users** → Select a user
2. Click **Access Advisor** tab
3. Review:
   - **Last accessed** services
   - **Last accessed** date
   - **Service permissions**
4. Identify unused permissions
5. Remove unnecessary permissions

##### b) Credentials Report (Account-level)

1. Navigate to **IAM** → **Credential report**
2. Click **Download report**
3. Review CSV file for:
   - Password age
   - Access key age
   - Last login
   - MFA status
   - Password last changed
4. Identify security issues:
   - Old passwords
   - Unused access keys
   - Users without MFA

---

### Exercise 2: Organizations Service

#### Question 1: Create Organization

##### Step 1: Create Organization

1. Navigate to **AWS Organizations**
2. Click **Create organization**
3. Choose organization type:
   - **All features** (recommended)
   - **Consolidated billing only**
4. Click **Create organization**

> **Note**: Your current account becomes the Management Account.
{: .prompt-warning }

##### Step 2: Create Organizational Units (OUs)

1. In **AWS Organizations**, click **Organize accounts**
2. Select **Root**
3. Click **Actions** → **Create new**
4. Select **Organizational unit**
5. Name: `Production`
6. Click **Create organizational unit**
7. Repeat for:
   - `Development`
   - `Testing`
   - `Sandbox`

##### Step 3: Create Member Accounts

**Option 1: Create New Account**
1. Select the OU where you want to add account
2. Click **Actions** → **Create new** → **Account**
3. Enter:
   - **Account name**: e.g., `Production-Account-1`
   - **Email address**: Unique email (not used in AWS)
   - **IAM role name**: (optional)
4. Click **Create account**

**Option 2: Invite Existing Account**
1. Click **Actions** → **Invite account**
2. Enter:
   - **Email address** of account owner
   - **Account ID** (optional)
3. Click **Send invitation**
4. Account owner accepts invitation

**Move Account to OU:**
1. Select account
2. Click **Actions** → **Move**
3. Select target OU
4. Click **Move**

##### Step 4: Add Policies to OUs

1. Navigate to **Policies** → **Service control policies**
2. Click **Create policy**
3. Example: Deny all except S3 and EC2 for Development OU
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": ["us-east-1"]
        }
      }
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:*",
        "ec2:*"
      ],
      "Resource": "*"
    }
  ]
}
```
4. Name: `DevelopmentOUPolicy`
5. Click **Create policy**
6. Attach to Development OU:
   - Select **Development OU**
   - Click **Policies** tab
   - Click **Attach**
   - Select `DevelopmentOUPolicy`

---

#### Question 2: Check Billing

1. Navigate to **AWS Organizations** → **Billing**
2. View:
   - **Consolidated billing** summary
   - **Cost by account**
   - **Cost by service**
   - **Cost by linked account**
3. Navigate to **AWS Cost Explorer** for detailed analysis
4. Set up **Billing alerts** in CloudWatch

**Benefits:**
- Single bill for all accounts
- Volume discounts applied automatically
- Easier cost tracking and management

---

## 📌 Summary

In this lesson, we covered:

1. ✅ **IAM Service** - Identity and Access Management
   - Root User, IAM Users, Groups, Roles
   - Policies and Permissions
   - MFA and Security Best Practices
   - Access methods: Console, CLI, SDK

2. ✅ **AWS Organizations** - Multi-account Management
   - Organizational Units (OUs)
   - Service Control Policies (SCPs)
   - Consolidated Billing
   - Account Management

### Key Takeaways

- **Never use Root User** for daily operations
- **Enable MFA** for all accounts
- **Follow principle of least privilege**
- **Use IAM Roles** instead of access keys when possible
- **Organize accounts** using AWS Organizations
- **Monitor access** using Access Advisor and Credentials Report

---

**Next Lesson**: [Lesson 03-04: Billing Service & EC2 Service]

---

*Questions about IAM or Organizations? Leave a comment below!*

