---
title: "AWS Solutions Architect - Lesson 01: Overview"
author: thanhnv1808
date: 2025-12-24 10:00:00 +0700
categories: [AWS, Solutions Architect]
tags: [aws, solutions-architect, overview, cloud-computing, infrastructure]
description: Overview of AWS, Cloud Computing, and AWS Global Infrastructure. The first lesson in the AWS Solutions Architect series.
pin: false
comments: true
---

This is the first lesson in the **AWS Solutions Architect** series. This lesson provides an overview of AWS, Cloud Computing, and AWS Global Infrastructure.

## 📚 Part 1: Knowledge

### 1. Introduction to AWS

**AWS (Amazon Web Services)** is a cloud service provider that offers servers and related services such as:
- Hardware
- Software
- Domain
- Storage
- Network
- ... and many other services

AWS has strong **scaling** capabilities. For example: if you're using a machine with 4GB RAM, you can scale up to 8GB with just one action.

**Famous websites using AWS:**
- Amazon.com
- Netflix
- Facebook
- Twitch
- ... and many other websites

#### Advantages of AWS

- ✅ **Strong scaling capabilities** - suitable for startups
- ✅ **No upfront setup costs**
- ✅ **Fast and easy setup**

#### Disadvantages of AWS

- ⚠️ **Requires understanding of pricing** - can be very expensive if not managed properly

> **Important Note**: It's crucial to understand AWS pricing to avoid unexpected costs.
{: .prompt-warning }

### 2. Cloud Computing vs On-Premise Comparison

#### On-Premise (Self-Managed)

**Characteristics:**
- You set up your own infrastructure including: server rooms, electricity, network, devices, software, ...
- Complex, requires significant initial effort and investment
- Requires many people to manage
- When upgrading, you need to purchase and install new equipment
- When expanding to other countries, you need to set up everything from scratch
- High setup costs
- High maintenance costs

#### Cloud Computing

**Characteristics:**
- AWS provides infrastructure that users can connect to and use
- Examples: Google Drive, Dropbox

**Advantages:**
- ✅ No upfront setup costs
- ✅ Easy setup

**Disadvantages:**
- ⚠️ Budget can increase significantly if not well understood

### 3. Deployment Models

#### Public Cloud
- Available for everyone on the internet
- Examples: AWS, Google Cloud, Azure

#### Private Cloud
- Used within a company's internal network
- Infrastructure is managed privately

#### Hybrid Cloud
- Combination of Public Cloud and Private Cloud
- Provides flexibility in resource management

### 4. AWS Use Cases

AWS is commonly used for:

- 🏗️ **Building complex systems** with high scalability
- 💾 **Backup & Storage data**, big data analytics
- 🌐 **Hosting websites**, mobile & social applications
- 🎮 **Gaming**
- 🤖 **Machine learning, AI Applications**
- ... and many other use cases

### 5. Cloud Service Models

#### SaaS (Software as a Service)
- Applications that can be used on browsers or mobile devices
- Examples: Gmail, Salesforce, Office 365

#### PaaS (Platform as a Service)
- Provides platforms for developing applications
- Examples: AWS Elastic Beanstalk, Google App Engine

#### IaaS (Infrastructure as a Service)
- Provides resources and infrastructure to run applications
- Examples: AWS EC2, Google Compute Engine

> Compared to traditional software, the infrastructure is managed by the service provider.
{: .prompt-info }

### 6. AWS Services

AWS offers services across various domains:
- Storage
- Network
- Machine Learning
- Data Analytics
- ... and many other domains

**View all services at:** [AWS Documentation](https://docs.aws.amazon.com)

**Main domains:**
- Compute
- Storage
- Database
- ... and many other domains

### 7. AWS Certification

AWS offers various certifications to validate knowledge and skills:
- AWS Certified Cloud Practitioner
- AWS Certified Solutions Architect
- AWS Certified Developer
- AWS Certified SysOps Administrator
- ... and many other certifications

### 8. Reference Materials

- **Home page**: [https://aws.amazon.com](https://aws.amazon.com)
- **Documentation**: [https://docs.aws.amazon.com/index.html](https://docs.aws.amazon.com/index.html)
- **Workshops**: [https://workshops.aws/](https://workshops.aws/)

---

### 9. Setting Up AWS Account

#### Academy-Provided Account
- Use the account provided by the academy (if available)

#### Creating Personal Account

**Signup:**
1. Go to [http://console.aws.amazon.com](http://console.aws.amazon.com)
2. Create an account following the instructions

**Sign in:**
- Log in at [http://console.aws.amazon.com](http://console.aws.amazon.com)

**Setup MFA (Multi-Factor Authentication):**
- Enable MFA to enhance account security

> **Security Note**: Always enable MFA for your AWS account to protect it from attacks.
{: .prompt-tip }

---

### 10. AWS Global Infrastructure

> **Note**: AWS currently does not have infrastructure in Vietnam.

#### Main Components

1. **Region**
2. **Availability Zones (AZ)**
3. **Data Centers**
4. **Edge Locations**

View details at: [AWS Infrastructure](https://infrastructure.aws)

#### Region

**Definition:**
- A physical location that is geographically separated from others (to prevent natural disasters from affecting each other)
- Commonly used for multi-country scenarios

**Characteristics:**
- Contains multiple Availability Zones (typically 2 to 6 zones, usually around 6 zones)
- Zones are also physically independent
- Each AZ contains many data centers
- Example: `ap-southeast-2a`, `ap-southeast-2b`, `ap-southeast-2c`
- AWS has Regions worldwide
- Most services operate independently within each region

**View list of Regions:** [AWS Global Infrastructure](https://aws.amazon.com/about-aws/global-infrastructure)

#### Availability Zones (AZ)

**Characteristics:**
- Independent in terms of network, power, connectivity, ...
- Can consist of one or multiple data centers
- Zones are connected via **high bandwidth, ultra-low latency networking**

**Types of Availability Zones:**

1. **Availability Zones** (Commonly used)
   - Standard AZ for most use cases

2. **Local Zones**
   - Has its own internet connection
   - Supports AWS Direct Connect
   - Located near major metropolitan areas

3. **Wavelength Zones**
   - Integrated with telecommunications service providers
   - Reduces latency for applications requiring low latency

#### Edge Locations

**Definition:**
- An endpoint for caching content → reduces latency
- Commonly used for CloudFront, CDN

**Statistics:**
- AWS has **206 Edge Locations** across **64 cities** and **42 countries**

**View details:** [AWS CloudFront Features](https://aws.amazon.com/cloudfront/features)

> **Note**: Edge Locations are different from Availability Zones. Edge Locations are used for caching, while AZs are used for hosting resources.
{: .prompt-info }

#### Reference Materials

- [Using Regions and Availability Zones](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html)
- [RDS Regions and Availability Zones](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html)

---

## 📝 Part 2: Assignment

### Exercise 1: Create AWS Account (Personal Account)

**Question 1:** Create an AWS account

- Visit: [http://console.aws.amazon.com/](http://console.aws.amazon.com/)

### Exercise 2: Login to AWS Console

**Question 1:** Use the account created above or use the account provided by the academy and log in to AWS console

- Visit: [http://console.aws.amazon.com/](http://console.aws.amazon.com/)
- Log in with the created account or the provided account

---

## ✅ Part 3: Solutions

### Exercise 1: Create AWS Account

#### Step 1: Access Registration Page

1. Open your browser and visit: [http://console.aws.amazon.com/](http://console.aws.amazon.com/)
2. Click the **"Create an AWS Account"** or **"Sign up"** button

#### Step 2: Enter Account Information

1. **Email address**: Enter your email (this will be your username for login)
2. **Password**: Create a strong password (at least 8 characters, with uppercase, lowercase, numbers, and special characters)
3. **Confirm password**: Re-enter the password
4. **AWS account name**: Name your AWS account

#### Step 3: Contact Information

1. **Full name**: Your full name
2. **Phone number**: Phone number (required for verification)
3. **Country/Region**: Select your country
4. **Address**: Your address
5. **City**: City
6. **State/Province**: State/Province
7. **Postal code**: Postal code

#### Step 4: Payment Verification

1. **Payment method**: Enter credit card or debit card information
   - AWS will hold $1 to verify the card (will be refunded)
   - AWS offers Free Tier allowing free use of some services for the first 12 months

#### Step 5: Identity Verification

1. AWS will call or send SMS to verify your phone number
2. Enter the verification code received

#### Step 6: Choose Support Plan

1. **Basic Plan** (Free) - Sufficient for most use cases
2. **Developer Plan** ($29/month)
3. **Business Plan** ($100/month)
4. **Enterprise Plan** ($15,000/month)

> **Recommendation**: Choose **Basic Plan** when starting out.
{: .prompt-tip }

#### Step 7: Complete Registration

1. Confirm terms and conditions
2. Click **"Create Account and Continue"**
3. Wait for confirmation email from AWS

### Exercise 2: Login to AWS Console

#### Step 1: Access AWS Console

1. Open your browser and visit: [http://console.aws.amazon.com/](http://console.aws.amazon.com/)

#### Step 2: Login

1. **Root account**: Use the email and password you registered
2. **IAM user**: If you have an IAM user, use account ID, username, and password

#### Step 3: Authentication (if MFA is enabled)

- If MFA is enabled, enter the code from your MFA device

#### Step 4: Select Region

- After logging in, select an appropriate Region (e.g., `ap-southeast-1` - Singapore)

> **Note**: 
> - Always choose the nearest Region to reduce latency
> - Some services are only available in certain Regions
> - Costs may vary between Regions
{: .prompt-info }

#### Step 5: Explore AWS Console

After logging in, you will see:
- **Services menu**: List of all AWS services
- **Resource groups**: Manage resources
- **CloudWatch**: Monitoring and logging
- **Billing**: View usage costs

### Best Practices After Creating Account

1. **Enable MFA (Multi-Factor Authentication)**
   - Enhance security for root account
   - Settings → Security credentials → MFA

2. **Create IAM User**
   - Don't use root account for daily tasks
   - Create IAM user with appropriate permissions

3. **Set Up Billing Alerts**
   - Get alerts when costs exceed thresholds
   - CloudWatch → Billing → Alarms

4. **Use AWS Free Tier**
   - Take advantage of free services for the first 12 months
   - Monitor usage to avoid exceeding limits

5. **Choose Appropriate Region**
   - Choose the nearest Region to reduce latency
   - Some services are only available in certain Regions

> **Important**: Always monitor AWS usage costs to avoid unexpected charges!
{: .prompt-warning }

---

## 📌 Summary

In this lesson, we learned about:

1. ✅ **Introduction to AWS** - Leading cloud service provider
2. ✅ **Cloud Computing vs On-Premise Comparison** - Advantages and disadvantages of each model
3. ✅ **Deployment Models** - Public, Private, Hybrid Cloud
4. ✅ **Cloud Service Models** - SaaS, PaaS, IaaS
5. ✅ **AWS Global Infrastructure** - Regions, Availability Zones, Edge Locations
6. ✅ **Setting Up AWS Account** - How to create and log in to AWS Console

The next lesson will dive deeper into **IAM Service and Organization Service**.

---

**Lessons in this series:**
- ✅ [Lesson 01: Overview](/posts/aws-sa-overview/) (This lesson)
- 📌 Lesson 02: IAM Service & Organization Service
- 📌 Lesson 03-04: Billing Service & EC2 Service
- 📌 Lesson 05: EFS Service & S3 Service
- ... and many more lessons

---

*Do you have any questions about AWS Overview? Leave a comment below!*
