---
title: "AWS SAP - Networking & Hybrid Architecture - Phần 1"
author: thanhnv1808
date: 2026-02-09 10:00:00 +0700
categories: [AWS, Solutions Architect Professional]
tags: [aws, sap, networking, vpc, security-groups, nacl, bgp, dhcp, firewall]
description: Hướng dẫn chi tiết về AWS Networking Fundamentals, VPC Components, Security, và BGP cho kỳ thi AWS Solutions Architect Professional (SAP-C02). Bao gồm Private/Public Services, DHCP, VPC Router, Stateful/Stateless Firewalls, NACLs, Security Groups.
pin: false
comments: true
---

# AWS SAP-C02: Networking & Hybrid Architecture - Phần 1

## Giới Thiệu

Networking là một trong những chủ đề quan trọng nhất trong kỳ thi **AWS Solutions Architect Professional (SAP-C02)**. Phần 1 này sẽ cover các foundational concepts và building blocks của AWS networking mà bạn cần master để thiết kế enterprise-grade architectures.

## Mục Lục - Phần 1
- [1. Private and Public AWS Services](#1-private-and-public-aws-services)
- [2. DHCP In a VPC](#2-dhcp-in-a-vpc)
- [3. VPC Router Deep Dive](#3-vpc-router-deep-dive)
- [4. Stateful vs Stateless Firewalls](#4-stateful-vs-stateless-firewalls)
- [5. Network Access Control Lists (NACL)](#5-network-access-control-lists-nacl)
- [6. Security Groups](#6-security-groups)
- [7. AWS Local Zones](#7-aws-local-zones)
- [8. Border Gateway Protocol 101](#8-border-gateway-protocol-101)
- [9. AWS Global Accelerator](#9-aws-global-accelerator)
- [10. IPSec VPN Fundamentals](#10-ipsec-vpn-fundamentals)
- [11. Site-to-Site VPN](#11-site-to-site-vpn)

---

## 1. Private and Public AWS Services

### 1.1 Ba Vùng Mạng (Three Network Zones)

AWS services được phân loại theo 3 network zones:

```
┌─────────────────────────────────────────────────────────┐
│                  PUBLIC INTERNET ZONE                   │
│  - Your on-premises network                             │
│  - Home/office networks                                 │
│  - Mobile devices                                       │
└──────────────────────┬──────────────────────────────────┘
                       │
                       │ Internet Gateway
                       │
┌──────────────────────▼──────────────────────────────────┐
│              AWS PUBLIC ZONE                            │
│  - S3 (public endpoints)                                │
│  - DynamoDB (public endpoints)                          │
│  - CloudFront                                           │
│  - API Gateway (default)                                │
│  - Route 53                                             │
│  - AWS Console                                          │
│                                                         │
│  Characteristics:                                       │
│  - Publicly routable IP addresses                       │
│  - Accessible from internet (with proper auth)          │
│  - No VPC required                                      │
└──────────────────────┬──────────────────────────────────┘
                       │
                       │ VPC Interface/Gateway Endpoints
                       │ PrivateLink
                       │
┌──────────────────────▼──────────────────────────────────┐
│              AWS PRIVATE ZONE (VPC)                     │
│                                                         │
│  ┌──────────────────────────────────────────┐          │
│  │         Your VPC (10.0.0.0/16)           │          │
│  │                                          │          │
│  │  Private Subnets:                        │          │
│  │  - EC2 Instances                         │          │
│  │  - RDS Databases                         │          │
│  │  - Lambda (VPC mode)                     │          │
│  │  - ECS/EKS                               │          │
│  │  - ElastiCache                           │          │
│  │                                          │          │
│  │  Characteristics:                        │          │
│  │  - Private IP addresses (RFC 1918)       │          │
│  │  - Isolated by default                   │          │
│  │  - Controlled via Security Groups/NACLs  │          │
│  └──────────────────────────────────────────┘          │
└─────────────────────────────────────────────────────────┘
```

### 1.2 Public vs Private Services

| Service Type | Examples | Access Method | IP Type | VPC Required? |
|-------------|----------|---------------|---------|---------------|
| **Public Services** | S3, DynamoDB, SQS, SNS | Public endpoints | Public IPs | ❌ No |
| **VPC Services** | EC2, RDS, Lambda (VPC) | Private or public (via IGW) | Private IPs | ✅ Yes |
| **Hybrid** | API Gateway, ECS | Both modes | Both | Optional |

### 1.3 Access Patterns

#### Pattern 1: Public Access to Public Service

```
┌──────────────┐         ┌──────────────┐
│   Internet   │         │      S3      │
│    User      ├────────►│   (Public)   │
│              │  HTTPS  │              │
└──────────────┘         └──────────────┘

URL: https://my-bucket.s3.amazonaws.com/object
Authentication: IAM credentials, Presigned URL, or Public ACL
```

#### Pattern 2: VPC Instance → Public Service (Qua Internet)

```
┌─────────────────────────────────────┐
│           VPC                       │
│                                     │
│  ┌──────────────┐                   │
│  │ EC2 Instance │                   │
│  │ (Private)    │                   │
│  └──────┬───────┘                   │
│         │                           │
│    ┌────▼─────┐    ┌──────────┐    │
│    │ NAT GW   ├───►│   IGW    │    │
│    └──────────┘    └────┬─────┘    │
└─────────────────────────┼──────────┘
                          │
                          │ Internet
                          │
                    ┌─────▼──────┐
                    │     S3     │
                    │  (Public)  │
                    └────────────┘

Cost: NAT Gateway processing + Data transfer
```

#### Pattern 3: VPC Instance → Public Service (Qua VPC Endpoint)

```
┌─────────────────────────────────────────┐
│              VPC                        │
│                                         │
│  ┌──────────────┐                       │
│  │ EC2 Instance │                       │
│  │ (Private)    │                       │
│  └──────┬───────┘                       │
│         │                               │
│         │ Private connection            │
│         │                               │
│    ┌────▼────────────────┐              │
│    │  VPC Gateway        │              │
│    │  Endpoint (S3)      │              │
│    └────┬────────────────┘              │
│         │                               │
└─────────┼───────────────────────────────┘
          │
          │ AWS Private Network
          │ (không qua Internet)
          │
    ┌─────▼──────┐
    │     S3     │
    │  (Public)  │
    └────────────┘

Cost: FREE (no NAT Gateway, no data transfer charges)
Benefits: Better security, better performance
```

### 1.4 VPC Endpoints Deep Dive

#### Gateway Endpoints (S3 & DynamoDB)

```
Configuration:
1. Create Gateway Endpoint trong VPC
2. Select route tables để update
3. AWS tự động add prefix list route

Route Table sau khi có Gateway Endpoint:
┌───────────────────────────────────────┐
│  Destination          Target          │
├───────────────────────────────────────┤
│  10.0.0.0/16          local           │ ← VPC CIDR
│  0.0.0.0/0            nat-gateway     │ ← Default route
│  pl-xxxxx (S3)        vpce-gateway    │ ← Prefix list (AWS managed)
└───────────────────────────────────────┘

Traffic flow:
- S3 traffic → Gateway Endpoint (FREE)
- Other traffic → NAT Gateway (CHARGED)
```

#### Interface Endpoints (PrivateLink)

```
Configuration:
1. Create Interface Endpoint trong subnet
2. ENI (Elastic Network Interface) được tạo
3. Private DNS enabled (optional)

Architecture:
┌────────────────────────────────────────┐
│           VPC (10.0.0.0/16)            │
│                                        │
│  ┌──────────────┐                      │
│  │ EC2 Instance │                      │
│  │ 10.0.1.50    │                      │
│  └──────┬───────┘                      │
│         │                              │
│         │ Query: sqs.us-east-1.amazonaws.com
│         │                              │
│    ┌────▼────────────────────┐         │
│    │  Interface Endpoint     │         │
│    │  (ENI)                  │         │
│    │  10.0.1.100             │         │
│    │  10.0.2.100 (AZ-B)      │         │
│    └────┬────────────────────┘         │
└─────────┼───────────────────────────────┘
          │
          │ AWS PrivateLink
          │
    ┌─────▼──────┐
    │    SQS     │
    └────────────┘

Private DNS:
- Enabled: sqs.us-east-1.amazonaws.com → 10.0.1.100
- Disabled: vpce-xxx.sqs.us-east-1.vpce.amazonaws.com

Cost: $0.01/hour per AZ + $0.01/GB data processed
```

### 1.5 Use Cases cho SAP Workloads

```
SAP ON AWS ARCHITECTURE:

┌──────────────────────────────────────────────────────┐
│                  VPC (SAP Production)                │
│                                                      │
│  ┌─────────────┐           ┌─────────────┐          │
│  │ SAP HANA    │           │ SAP App     │          │
│  │ (Private    │◄─────────►│ Servers     │          │
│  │  Subnet)    │           │ (Private    │          │
│  └─────────────┘           │  Subnet)    │          │
│                            └──────┬──────┘          │
│                                   │                 │
│  VPC Endpoints:                   │                 │
│  ┌─────────────────────────────┐  │                 │
│  │ S3 Gateway Endpoint         │◄─┤                 │
│  │ (Backup storage)            │  │                 │
│  └─────────────────────────────┘  │                 │
│                                   │                 │
│  ┌─────────────────────────────┐  │                 │
│  │ CloudWatch Interface EP     │◄─┤                 │
│  │ (Monitoring, no internet)   │  │                 │
│  └─────────────────────────────┘  │                 │
│                                   │                 │
│  ┌─────────────────────────────┐  │                 │
│  │ Systems Manager Interface EP│◄─┘                 │
│  │ (Patching, no internet)     │                    │
│  └─────────────────────────────┘                    │
└──────────────────────────────────────────────────────┘

Benefits cho SAP:
✅ Backup SAP HANA to S3 (không cần NAT Gateway)
✅ CloudWatch monitoring (secure, private)
✅ Systems Manager patching (no internet exposure)
✅ Better security compliance
✅ Lower costs (no NAT Gateway fees)
```

### 1.6 Exam Tips

**🎯 Điểm Thi Quan Trọng:**

1. **Public vs Private Services:**
   - S3, DynamoDB = Public services (có public endpoints)
   - EC2, RDS = VPC services (private by default)
   - Không cần VPC để access S3, nhưng VPC Endpoint tốt hơn

2. **VPC Endpoints:**
   - Gateway Endpoints (S3, DynamoDB) = FREE
   - Interface Endpoints (other services) = $0.01/hour/AZ + data
   - Gateway Endpoint updates route tables automatically
   - Interface Endpoint creates ENI trong subnet

3. **Cost Optimization:**
   - VPC Endpoint (S3) vs NAT Gateway → VPC Endpoint MIỄN PHÍ
   - High S3 traffic từ VPC → Always use Gateway Endpoint
   - 1 TB qua NAT = ~$45, qua VPC Endpoint = $0

4. **Security:**
   - VPC Endpoint không expose traffic ra internet
   - Private DNS với Interface Endpoint
   - Endpoint policies để control access

---

## 2. DHCP In a VPC

### 2.1 DHCP Là Gì?

**Dynamic Host Configuration Protocol (DHCP)** tự động cấp phát:
- IP address
- Subnet mask
- Default gateway
- DNS servers
- NTP servers
- NetBIOS settings

```
DHCP PROCESS:

Client                    DHCP Server
  │                            │
  ├─ DISCOVER ────────────────►│  "Tôi cần IP address!"
  │                            │
  │◄──── OFFER ────────────────┤  "Dùng 10.0.1.50 nhé"
  │                            │
  ├─ REQUEST ─────────────────►│  "OK, tôi lấy 10.0.1.50"
  │                            │
  │◄──── ACK ──────────────────┤  "Confirmed!"
  │                            │
  └─ Using 10.0.1.50           │
```

### 2.2 DHCP trong AWS VPC

AWS VPC có **built-in DHCP service** tại địa chỉ `.2` của mỗi subnet:

```
VPC: 10.0.0.0/16
├─ Subnet A: 10.0.1.0/24
│  ├─ 10.0.1.1 → VPC Router
│  ├─ 10.0.1.2 → DHCP Server (reserved by AWS)
│  ├─ 10.0.1.3 → AWS reserved
│  ├─ 10.0.1.4 → Available (first usable)
│  ├─ 10.0.1.5 → Available
│  └─ 10.0.1.255 → Broadcast (reserved)
│
└─ Subnet B: 10.0.2.0/24
   ├─ 10.0.2.1 → VPC Router
   ├─ 10.0.2.2 → DHCP Server (reserved by AWS)
   └─ 10.0.2.4 → Available (first usable)
```

### 2.3 DHCP Options Set

**DHCP Options Set** định nghĩa thông tin DHCP cấp cho instances:

```
Default DHCP Options Set:
┌─────────────────────────────────────────┐
│  domain-name = ec2.internal             │ (us-east-1)
│  domain-name = region.compute.internal  │ (other regions)
│  domain-name-servers = AmazonProvidedDNS│
└─────────────────────────────────────────┘

Custom DHCP Options Set Example:
┌─────────────────────────────────────────┐
│  domain-name = company.local            │
│  domain-name-servers = 10.0.0.10        │ ← Custom DNS
│  ntp-servers = 10.0.0.20                │ ← Custom NTP
│  netbios-name-servers = 10.0.0.30       │ ← For Windows
│  netbios-node-type = 2                  │
└─────────────────────────────────────────┘
```

### 2.4 Reserved IP Addresses

**AWS reserves 5 IP addresses** trong mỗi subnet:

```
Example Subnet: 10.0.1.0/24 (256 addresses)

Reserved by AWS:
┌─────────────┬──────────────────────────────────┐
│ IP Address  │ Purpose                          │
├─────────────┼──────────────────────────────────┤
│ 10.0.1.0    │ Network address                  │
│ 10.0.1.1    │ VPC Router                       │
│ 10.0.1.2    │ DNS Server (AmazonProvidedDNS)   │
│ 10.0.1.3    │ AWS reserved (future use)        │
│ 10.0.1.255  │ Broadcast address                │
└─────────────┴──────────────────────────────────┘

Usable addresses: 10.0.1.4 - 10.0.1.254 = 251 addresses

Với /28 subnet (16 addresses):
- Reserved: 5 addresses
- Usable: 11 addresses only!
```

### 2.5 Custom DHCP Options Set Configuration

```bash
# Create custom DHCP options set
aws ec2 create-dhcp-options \
    --dhcp-configurations \
        "Key=domain-name,Values=company.local" \
        "Key=domain-name-servers,Values=10.0.0.10,10.0.0.11" \
        "Key=ntp-servers,Values=10.0.0.20" \
        "Key=netbios-name-servers,Values=10.0.0.30" \
        "Key=netbios-node-type,Values=2"

# Associate với VPC
aws ec2 associate-dhcp-options \
    --dhcp-options-id dopt-1234567890abcdef0 \
    --vpc-id vpc-0a1b2c3d4e5f6g7h8

# Verify
aws ec2 describe-vpcs \
    --vpc-ids vpc-0a1b2c3d4e5f6g7h8 \
    --query 'Vpcs[0].DhcpOptionsId'
```

### 2.6 Use Case: SAP Domain Integration

```
SAP ENVIRONMENT WITH CUSTOM DNS:

┌────────────────────────────────────────────┐
│          VPC (10.0.0.0/16)                 │
│                                            │
│  DHCP Options Set:                         │
│  - domain-name: sap.company.local          │
│  - DNS servers: 10.0.0.10, 10.0.0.11       │
│                                            │
│  ┌──────────────┐     ┌──────────────┐    │
│  │ AD/DNS Svr   │     │ AD/DNS Svr   │    │
│  │ 10.0.0.10    │     │ 10.0.0.11    │    │
│  │ (Primary)    │     │ (Secondary)  │    │
│  └──────────────┘     └──────────────┘    │
│                                            │
│  ┌──────────────────────────────┐          │
│  │  SAP Application Servers     │          │
│  │                              │          │
│  │  Hostname: sapapp01          │          │
│  │  FQDN: sapapp01.sap.company.local       │
│  │  DNS auto-registered         │          │
│  └──────────────────────────────┘          │
│                                            │
│  ┌──────────────────────────────┐          │
│  │  SAP HANA Database           │          │
│  │                              │          │
│  │  Hostname: saphana01         │          │
│  │  FQDN: saphana01.sap.company.local      │
│  └──────────────────────────────┘          │
└────────────────────────────────────────────┘

Benefits:
✅ SAP systems can resolve each other by hostname
✅ Integrated với corporate DNS/AD
✅ Supports SAP hostnames requirements
✅ Works with SAP HANA System Replication
```

### 2.7 Troubleshooting DHCP

```bash
# On Linux instance - check DHCP lease
cat /var/lib/dhcp/dhclient.leases

# Check current DNS configuration
cat /etc/resolv.conf

# Output:
# search ec2.internal
# nameserver 10.0.0.2

# Test DNS resolution
nslookup saphana01.sap.company.local

# Check routing
ip route show
# default via 10.0.1.1 dev eth0
```

### 2.8 DHCP Limitations

**⚠️ Important Limitations:**

1. **Cannot modify DHCP Options Set:**
   - Phải tạo mới và associate với VPC
   - Existing instances cần reboot để nhận options mới

2. **Cannot delete in-use DHCP Options Set:**
   - Phải disassociate khỏi all VPCs trước

3. **One DHCP Options Set per VPC:**
   - Không thể có different options cho different subnets

4. **Static IP assignments:**
   - AWS DHCP luôn assign cùng private IP cho instance
   - IP không thay đổi khi instance stop/start (unless released)

### 2.9 Exam Tips

**🎯 Điểm Thi Quan Trọng:**

1. **Reserved IPs:**
   - AWS reserves 5 IPs per subnet
   - `.0` = network, `.1` = router, `.2` = DNS, `.3` = reserved, `.255` = broadcast
   - Subnet /28 (16 IPs) → Only 11 usable
   - Subnet /24 (256 IPs) → Only 251 usable

2. **DHCP Options Set:**
   - Can customize DNS servers, domain name, NTP servers
   - Cannot modify existing set, must create new
   - One DHCP options set per VPC
   - Instances need reboot to get new options

3. **AmazonProvidedDNS:**
   - Located at `.2` of subnet
   - Also available at `169.254.169.253`
   - Provides Route 53 Resolver functionality
   - Free to use

4. **SAP Scenarios:**
   - Custom domain name for SAP landscapes
   - Custom DNS servers (AD integration)
   - Hostname resolution between SAP systems

---

## 3. VPC Router Deep Dive

### 3.1 VPC Router Là Gì?

**VPC Router** là thành phần core của VPC, tự động routing traffic giữa subnets và đến các destinations khác.

```
VPC ROUTER ARCHITECTURE:

┌─────────────────────────────────────────────┐
│              VPC (10.0.0.0/16)              │
│                                             │
│         ┌───────────────────┐               │
│         │    VPC Router     │               │
│         │   (10.x.x.1 in    │               │
│         │   each subnet)    │               │
│         └─────────┬─────────┘               │
│                   │                         │
│     ┌─────────────┼─────────────┐           │
│     │             │             │           │
│ ┌───▼────┐   ┌───▼────┐   ┌───▼────┐       │
│ │Subnet A│   │Subnet B│   │Subnet C│       │
│ │.1.0/24 │   │.2.0/24 │   │.3.0/24 │       │
│ │Router: │   │Router: │   │Router: │       │
│ │10.0.1.1│   │10.0.2.1│   │10.0.3.1│       │
│ └────────┘   └────────┘   └────────┘       │
└─────────────────────────────────────────────┘

Key Points:
- VPC Router chạy ở `.1` address của mỗi subnet
- Highly available (AWS managed, no single point of failure)
- Infinitely scalable
- No performance bottleneck
- No configuration required
```

### 3.2 Route Tables

Mỗi subnet có **một Route Table** quyết định traffic routing:

```
ROUTE TABLE STRUCTURE:

┌──────────────────────────────────────────────┐
│          Route Table (rtb-123456)            │
├──────────────────┬───────────────────────────┤
│  Destination     │  Target                   │
├──────────────────┼───────────────────────────┤
│  10.0.0.0/16     │  local                    │ ← Always present
│  0.0.0.0/0       │  igw-abcdef              │ ← Internet Gateway
│  192.168.0.0/16  │  vgw-123456              │ ← Virtual Private GW
│  172.16.0.0/12   │  pcx-789012              │ ← VPC Peering
│  10.1.0.0/16     │  tgw-xyz123              │ ← Transit Gateway
└──────────────────┴───────────────────────────┘

Route Priority (Longest Prefix Match):
1. Most specific route wins
2. Example: 10.0.1.0/24 > 10.0.0.0/16 > 0.0.0.0/0
```

### 3.3 Route Table Types

#### Main Route Table

```
MAIN ROUTE TABLE:

┌────────────────────────────────────────┐
│        VPC (10.0.0.0/16)               │
│                                        │
│  Main Route Table (default):          │
│  ┌────────────────────────────┐       │
│  │ 10.0.0.0/16 → local        │       │
│  └────────────────────────────┘       │
│                                        │
│  ┌──────────┐  ┌──────────┐           │
│  │ Subnet A │  │ Subnet B │           │
│  │ (using   │  │ (using   │           │
│  │  Main RT)│  │  Main RT)│           │
│  └──────────┘  └──────────┘           │
└────────────────────────────────────────┘

Characteristics:
- Created automatically với VPC
- Subnets use Main RT by default
- Best practice: Keep Main RT private (no IGW route)
- Cannot delete Main RT
```

#### Custom Route Tables

```
CUSTOM ROUTE TABLES:

┌─────────────────────────────────────────────┐
│           VPC (10.0.0.0/16)                 │
│                                             │
│  ┌─────────────────┐  ┌─────────────────┐  │
│  │ Public RT       │  │ Private RT      │  │
│  │                 │  │                 │  │
│  │ 10.0.0.0/16     │  │ 10.0.0.0/16     │  │
│  │ → local         │  │ → local         │  │
│  │ 0.0.0.0/0       │  │ 0.0.0.0/0       │  │
│  │ → igw-xxx       │  │ → nat-gw-xxx    │  │
│  └────────┬────────┘  └────────┬────────┘  │
│           │                    │            │
│      ┌────▼────┐          ┌───▼─────┐      │
│      │ Public  │          │ Private │      │
│      │ Subnet  │          │ Subnet  │      │
│      └─────────┘          └─────────┘      │
└─────────────────────────────────────────────┘
```

### 3.4 Route Targets

```
COMMON ROUTE TARGETS:

Target                  Description                 Example
─────────────────────────────────────────────────────────────
local                   Within VPC (implicit)       10.0.0.0/16 → local

Internet Gateway (IGW)  Internet access             0.0.0.0/0 → igw-xxx

NAT Gateway            Outbound internet           0.0.0.0/0 → nat-xxx
                       (for private subnets)

NAT Instance           Self-managed NAT            0.0.0.0/0 → eni-xxx

Virtual Private        VPN connection              192.168.0.0/16 → vgw-xxx
Gateway (VGW)

VPC Peering            Another VPC                 172.16.0.0/16 → pcx-xxx

Transit Gateway        Multi-VPC hub               10.0.0.0/8 → tgw-xxx

VPC Endpoint           AWS service (private)       pl-xxx (S3) → vpce-xxx

Egress-Only IGW        IPv6 outbound only          ::/0 → eigw-xxx

Network Interface      Specific instance           10.1.0.0/16 → eni-xxx
(ENI)                  (e.g., firewall)

Gateway Load           Traffic inspection          10.2.0.0/16 → vpce-gwlb-xxx
Balancer Endpoint
```

### 3.5 Routing Decision Process

```
ROUTE SELECTION ALGORITHM:

┌─────────────────────────────────────────┐
│  Packet arrives with destination IP    │
│  Example: 10.0.2.50                     │
└──────────────┬──────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────┐
│  Step 1: Check Route Table              │
│  ┌────────────────────────────────┐     │
│  │ 10.0.2.50/32  → eni-specific   │ ◄── Most specific
│  │ 10.0.2.0/24   → nat-gateway    │
│  │ 10.0.0.0/16   → local          │
│  │ 0.0.0.0/0     → igw            │ ◄── Least specific
│  └────────────────────────────────┘     │
└──────────────┬──────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────┐
│  Step 2: Longest Prefix Match           │
│  10.0.2.50 matches:                      │
│  - 10.0.2.50/32  ✅ (32 bits matched)   │
│  - 10.0.2.0/24   ✅ (24 bits matched)   │
│  - 10.0.0.0/16   ✅ (16 bits matched)   │
│  - 0.0.0.0/0     ✅ (0 bits matched)    │
│                                          │
│  Winner: 10.0.2.50/32 (most specific)   │
└──────────────┬──────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────┐
│  Step 3: Forward to Target               │
│  Send packet to eni-specific             │
└──────────────────────────────────────────┘
```

### 3.6 Route Table Configuration Examples

#### Public Subnet Route Table

```bash
# Create route table
aws ec2 create-route-table \
    --vpc-id vpc-12345678 \
    --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=Public-RT}]'

# Add internet gateway route
aws ec2 create-route \
    --route-table-id rtb-abcdef \
    --destination-cidr-block 0.0.0.0/0 \
    --gateway-id igw-12345678

# Associate với subnet
aws ec2 associate-route-table \
    --route-table-id rtb-abcdef \
    --subnet-id subnet-public1
```

#### Private Subnet Route Table

```bash
# Create route table
aws ec2 create-route-table \
    --vpc-id vpc-12345678 \
    --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=Private-RT}]'

# Add NAT gateway route
aws ec2 create-route \
    --route-table-id rtb-private \
    --destination-cidr-block 0.0.0.0/0 \
    --nat-gateway-id nat-0123456789

# Add VPC peering route
aws ec2 create-route \
    --route-table-id rtb-private \
    --destination-cidr-block 172.16.0.0/16 \
    --vpc-peering-connection-id pcx-abc123

# Associate với subnet
aws ec2 associate-route-table \
    --route-table-id rtb-private \
    --subnet-id subnet-private1
```

### 3.7 Advanced Routing Scenario: Multi-Tier SAP

```
SAP 3-TIER ARCHITECTURE ROUTING:

┌────────────────────────────────────────────────────┐
│              VPC (10.0.0.0/16)                     │
│                                                    │
│  ┌──────────────────────────────────────────────┐ │
│  │  Web Tier (Public Subnet 10.0.1.0/24)       │ │
│  │                                              │ │
│  │  Route Table:                                │ │
│  │  10.0.0.0/16 → local                         │ │
│  │  0.0.0.0/0   → IGW (internet access)         │ │
│  │                                              │ │
│  │  Resources: SAP Web Dispatcher, SAP Router   │ │
│  └────────────────────┬─────────────────────────┘ │
│                       │                           │
│  ┌────────────────────▼─────────────────────────┐ │
│  │  App Tier (Private Subnet 10.0.2.0/24)      │ │
│  │                                              │ │
│  │  Route Table:                                │ │
│  │  10.0.0.0/16     → local                     │ │
│  │  0.0.0.0/0       → NAT Gateway (updates)     │ │
│  │  192.168.0.0/16  → VGW (on-premises)         │ │
│  │                                              │ │
│  │  Resources: SAP Application Servers (ASCS)   │ │
│  └────────────────────┬─────────────────────────┘ │
│                       │                           │
│  ┌────────────────────▼─────────────────────────┐ │
│  │  DB Tier (Isolated Subnet 10.0.3.0/24)      │ │
│  │                                              │ │
│  │  Route Table:                                │ │
│  │  10.0.0.0/16 → local (VPC only)              │ │
│  │  NO default route (no internet)              │ │
│  │                                              │ │
│  │  Resources: SAP HANA Database                │ │
│  └──────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────┘

Security Model:
- Web Tier: Public (internet-facing)
- App Tier: Private (outbound internet for updates)
- DB Tier: Isolated (no internet at all)
```

### 3.8 Route Propagation

Route propagation tự động add routes từ VPN hoặc Direct Connect:

```
ROUTE PROPAGATION EXAMPLE:

┌────────────────────────────────────────┐
│         VPC (10.0.0.0/16)              │
│                                        │
│  Route Table:                          │
│  ┌──────────────────────────────┐     │
│  │ Destination    │ Target      │     │
│  ├──────────────────────────────┤     │
│  │ 10.0.0.0/16    │ local       │     │
│  │ 192.168.1.0/24 │ vgw-xxx     │ ◄── Propagated
│  │ 192.168.2.0/24 │ vgw-xxx     │ ◄── Propagated
│  │ 172.16.0.0/12  │ vgw-xxx     │ ◄── Propagated
│  └──────────────────────────────┘     │
│           ▲                            │
│           │ BGP advertisements         │
│           │                            │
│    ┌──────┴─────┐                      │
│    │    VGW     │                      │
│    └──────┬─────┘                      │
└───────────┼────────────────────────────┘
            │ VPN/Direct Connect
            │
    ┌───────▼────────┐
    │  On-Premises   │
    │  Network       │
    └────────────────┘

Enable propagation:
aws ec2 enable-vgw-route-propagation \
    --route-table-id rtb-xxx \
    --gateway-id vgw-xxx
```

### 3.9 Exam Tips

**🎯 Điểm Thi Quan Trọng:**

1. **VPC Router Basics:**
   - Highly available (AWS managed)
   - Lives at `.1` of each subnet
   - No performance limits
   - No configuration required

2. **Route Tables:**
   - Each subnet has ONE route table
   - Main RT: Default for all subnets
   - Custom RT: Explicit association
   - Local route always present (cannot delete)

3. **Route Selection:**
   - Longest prefix match (most specific wins)
   - 10.0.1.0/24 beats 10.0.0.0/16
   - /32 > /24 > /16 > /8 > /0

4. **Route Propagation:**
   - Automatic route learning từ VGW/TGW
   - Used với VPN và Direct Connect
   - Can be disabled per route table
   - Propagated routes < Static routes (if same prefix)

5. **Common Mistakes:**
   - Forgetting to add 0.0.0.0/0 route cho internet access
   - Using Main RT for public subnets (security risk)
   - Not considering route priority
   - Overlapping CIDRs causing routing issues

---

## 4. Stateful vs Stateless Firewalls

### 4.1 Firewall Fundamentals

```
FIREWALL TYPES:

┌────────────────────────────────────────────────────┐
│              STATEFUL FIREWALL                     │
│                                                    │
│  Request:  Client → Server (port 443)              │
│  ┌──────────────────────────────────────────┐     │
│  │ Check: Outbound rule allows 443? ✅      │     │
│  │ Action: Allow + Remember connection      │     │
│  └──────────────────────────────────────────┘     │
│                                                    │
│  Response: Server → Client (source port 443)       │
│  ┌──────────────────────────────────────────┐     │
│  │ Check: Is this response to allowed       │     │
│  │        outbound connection? ✅           │     │
│  │ Action: Automatically allow (stateful)   │     │
│  └──────────────────────────────────────────┘     │
│                                                    │
│  Connection Tracking:                              │
│  - Remembers connections                           │
│  - Auto-allows return traffic                      │
│  - No need to explicitly allow responses           │
└────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────┐
│             STATELESS FIREWALL                     │
│                                                    │
│  Request:  Client → Server (port 443)              │
│  ┌──────────────────────────────────────────┐     │
│  │ Check: Outbound rule allows 443? ✅      │     │
│  │ Action: Allow (no memory)                │     │
│  └──────────────────────────────────────────┘     │
│                                                    │
│  Response: Server → Client (source port 443)       │
│  ┌──────────────────────────────────────────┐     │
│  │ Check: Inbound rule allows 443? ❌       │     │
│  │ Action: DENY (no connection tracking)    │     │
│  └──────────────────────────────────────────┘     │
│                                                    │
│  No Connection Tracking:                           │
│  - Treats each packet independently                │
│  - Must explicitly allow BOTH directions           │
│  - Need rules for ephemeral ports                  │
└────────────────────────────────────────────────────┘
```

### 4.2 Ephemeral Ports

```
EPHEMERAL PORT PROBLEM (Stateless):

Client (10.0.1.50)              Server (10.0.2.100)
    │                                  │
    │ SRC: 10.0.1.50:45678            │
    │ DST: 10.0.2.100:443             │
    ├─────────────────────────────────►│
    │         Request                  │
    │                                  │
    │                                  │
    │ SRC: 10.0.2.100:443             │
    │ DST: 10.0.1.50:45678            │ ← Ephemeral port
    │◄─────────────────────────────────┤
    │         Response                 │

Ephemeral Port Ranges:
┌─────────────────────────────────────────┐
│ OS Type        │ Port Range             │
├─────────────────────────────────────────┤
│ Linux          │ 32768 - 60999          │
│ Windows Server │ 49152 - 65535 (Vista+) │
│ Windows older  │ 1025 - 5000            │
│ NAT Gateway    │ 1024 - 65535           │
└─────────────────────────────────────────┘

Stateless Firewall Requirements:
Outbound Rules:
- Allow 443 to 10.0.2.100

Inbound Rules:
- Allow 32768-60999 from 10.0.2.100  ← Must allow ephemeral!

Stateful Firewall (simpler):
Outbound Rules:
- Allow 443 to 10.0.2.100
Inbound Rules:
- (Auto-allowed as return traffic)
```

### 4.3 AWS Implementation

```
AWS FIREWALL COMPONENTS:

┌────────────────────────────────────────┐
│      Security Groups (STATEFUL)        │
│  - Instance-level firewall             │
│  - Connection tracking                 │
│  - Return traffic auto-allowed         │
│  - Only ALLOW rules                    │
│  - Evaluates ALL rules                 │
└────────────────────────────────────────┘
            │
            │ Applied to ENI
            ▼
      ┌──────────┐
      │    EC2   │
      │ Instance │
      └──────────┘
            ▲
            │ Applied to Subnet
            │
┌────────────────────────────────────────┐
│    Network ACLs (STATELESS)            │
│  - Subnet-level firewall               │
│  - No connection tracking              │
│  - Must allow return traffic           │
│  - ALLOW and DENY rules                │
│  - Processes rules in order            │
└────────────────────────────────────────┘
```

### 4.4 Comparison Table

| Feature | Stateful (Security Groups) | Stateless (NACLs) |
|---------|---------------------------|-------------------|
| **Level** | Instance (ENI) | Subnet |
| **Connection Tracking** | ✅ Yes | ❌ No |
| **Return Traffic** | Auto-allowed | Must explicitly allow |
| **Rule Types** | ALLOW only | ALLOW and DENY |
| **Rule Evaluation** | All rules | In order (by rule number) |
| **Default** | Deny all | Allow all (default NACL) |
| **Ephemeral Ports** | Not needed | Must configure |
| **State** | Remembers connections | Treats packets independently |

### 4.5 Traffic Flow Example

```
COMPLETE TRAFFIC FLOW:

Internet                                    VPC
   │                                   ┌─────────────┐
   │                                   │   Subnet    │
   │                                   │             │
   │  1. Request (port 443)            │             │
   ├──────────────────────────────────►│ NACL        │
   │                                   │ (Stateless) │
   │  ✅ Inbound rule 100: Allow 443   │             │
   │                                   └──────┬──────┘
   │                                          │
   │                                          │
   │                                   ┌──────▼──────┐
   │                                   │  Security   │
   │                                   │    Group    │
   │                                   │  (Stateful) │
   │  ✅ Inbound: Allow 443            │             │
   │                                   └──────┬──────┘
   │                                          │
   │                                   ┌──────▼──────┐
   │                                   │     EC2     │
   │                                   │  Instance   │
   │                                   └──────┬──────┘
   │                                          │
   │  2. Response (ephemeral port)            │
   │  ┌───────────────────────────────────────┘
   │  │                                ┌─────────────┐
   │  │                                │  Security   │
   │  │                                │    Group    │
   │  │  ✅ Auto-allowed (stateful)    │  (Stateful) │
   │  │                                └──────┬──────┘
   │  │                                       │
   │  │                                ┌──────▼──────┐
   │  │                                │    NACL     │
   │  │                                │ (Stateless) │
   │◄─┴────────────────────────────────┤             │
      ✅ Outbound rule 100:            │ Must allow  │
         Allow ephemeral ports         │ 1024-65535! │
                                       └─────────────┘
```

### 4.6 Exam Tips

**🎯 Điểm Thi Quan Trọng:**

1. **Stateful = Security Groups:**
   - Connection tracking enabled
   - Return traffic automatically allowed
   - No need to worry about ephemeral ports
   - Simpler to configure

2. **Stateless = NACLs:**
   - No connection tracking
   - Must explicitly allow both directions
   - Must configure ephemeral port ranges
   - More complex but more control

3. **Ephemeral Ports:**
   - Linux: 32768-60999
   - Windows: 49152-65535
   - NAT Gateway: 1024-65535
   - Always required for stateless firewalls

4. **When Asked in Exam:**
   - "Auto-allow return traffic" = Stateful
   - "Must configure ephemeral ports" = Stateless
   - "Connection tracking" = Stateful
   - "Evaluate each packet independently" = Stateless

---

## 5. Network Access Control Lists (NACL)

### 5.1 NACL Overview

**Network ACLs (NACLs)** là stateless firewalls hoạt động ở subnet level.

```
NACL ARCHITECTURE:

┌────────────────────────────────────────────┐
│              VPC (10.0.0.0/16)             │
│                                            │
│  ┌──────────────────────────────────────┐ │
│  │    Subnet A (10.0.1.0/24)            │ │
│  │                                      │ │
│  │  NACL (stateless)                    │ │
│  │  ┌────────────────────────────┐     │ │
│  │  │ Inbound Rules:             │     │ │
│  │  │ 100: Allow 0.0.0.0/0:80    │     │ │
│  │  │ 110: Allow 0.0.0.0/0:443   │     │ │
│  │  │ 120: Allow VPC:32768-60999 │     │ │
│  │  │ *: Deny all                │     │ │
│  │  │                            │     │ │
│  │  │ Outbound Rules:            │     │ │
│  │  │ 100: Allow 0.0.0.0/0:all   │     │ │
│  │  │ *: Deny all                │     │ │
│  │  └────────────────────────────┘     │ │
│  │                                      │ │
│  │  ┌──────────┐  ┌──────────┐         │ │
│  │  │   EC2    │  │   EC2    │         │ │
│  │  │ Instance │  │ Instance │         │ │
│  │  └──────────┘  └──────────┘         │ │
│  └──────────────────────────────────────┘ │
└────────────────────────────────────────────┘

Key Characteristics:
- Subnet-level protection
- Stateless (no connection tracking)
- Numbered rules (processed in order)
- ALLOW and DENY rules
- Applies to ALL instances in subnet
```

### 5.2 NACL Rules Structure

```
RULE PROCESSING ORDER:

Rule #  Type      Protocol  Port Range    Source/Dest     Allow/Deny
──────────────────────────────────────────────────────────────────────
100     Inbound   TCP       80            0.0.0.0/0       ALLOW
110     Inbound   TCP       443           0.0.0.0/0       ALLOW
120     Inbound   TCP       22            203.0.113.0/24  ALLOW
130     Inbound   TCP       3389          203.0.113.0/24  ALLOW
*       Inbound   ALL       ALL           0.0.0.0/0       DENY

100     Outbound  TCP       1024-65535    0.0.0.0/0       ALLOW
110     Outbound  TCP       80            0.0.0.0/0       ALLOW
120     Outbound  TCP       443           0.0.0.0/0       ALLOW
*       Outbound  ALL       ALL           0.0.0.0/0       DENY

Processing:
1. Rules evaluated in numerical order (lowest first)
2. First matching rule wins (stops processing)
3. * (asterisk) = default rule (always last)
4. Rule numbers: 1-32766 (recommended: increment by 10)
```

### 5.3 Default NACL vs Custom NACL

```
DEFAULT NACL (automatically created with VPC):
┌────────────────────────────────────────┐
│  Inbound Rules:                        │
│  Rule #100: Allow ALL traffic          │
│  Rule #*: Deny ALL traffic             │
│                                        │
│  Outbound Rules:                       │
│  Rule #100: Allow ALL traffic          │
│  Rule #*: Deny ALL traffic             │
└────────────────────────────────────────┘
Result: Effectively allows ALL traffic (rule 100 matches everything)

CUSTOM NACL (user-created):
┌────────────────────────────────────────┐
│  Inbound Rules:                        │
│  Rule #*: Deny ALL traffic             │
│                                        │
│  Outbound Rules:                       │
│  Rule #*: Deny ALL traffic             │
└────────────────────────────────────────┘
Result: Denies ALL traffic by default (must explicitly allow)
```

### 5.4 NACL Configuration Examples

#### Example 1: Public Web Server

```bash
# Create NACL
aws ec2 create-network-acl \
    --vpc-id vpc-12345678 \
    --tag-specifications 'ResourceType=network-acl,Tags=[{Key=Name,Value=WebServer-NACL}]'

# Inbound: Allow HTTP from anywhere
aws ec2 create-network-acl-entry \
    --network-acl-id acl-abcdef \
    --rule-number 100 \
    --protocol tcp \
    --port-range From=80,To=80 \
    --cidr-block 0.0.0.0/0 \
    --rule-action allow \
    --ingress

# Inbound: Allow HTTPS from anywhere
aws ec2 create-network-acl-entry \
    --network-acl-id acl-abcdef \
    --rule-number 110 \
    --protocol tcp \
    --port-range From=443,To=443 \
    --cidr-block 0.0.0.0/0 \
    --rule-action allow \
    --ingress

# Inbound: Allow ephemeral ports (for return traffic)
aws ec2 create-network-acl-entry \
    --network-acl-id acl-abcdef \
    --rule-number 120 \
    --protocol tcp \
    --port-range From=1024,To=65535 \
    --cidr-block 0.0.0.0/0 \
    --rule-action allow \
    --ingress

# Outbound: Allow all
aws ec2 create-network-acl-entry \
    --network-acl-id acl-abcdef \
    --rule-number 100 \
    --protocol -1 \
    --cidr-block 0.0.0.0/0 \
    --rule-action allow \
    --egress
```

#### Example 2: Deny Specific IP Address

```
NACL Rules (Block malicious IP):

Inbound Rules:
┌────────────────────────────────────────────────┐
│ 90:  DENY   TCP  ALL  203.0.113.50/32         │ ← Block first!
│ 100: ALLOW  TCP  80   0.0.0.0/0               │
│ 110: ALLOW  TCP  443  0.0.0.0/0               │
│ *:   DENY   ALL  ALL  0.0.0.0/0               │
└────────────────────────────────────────────────┘

Important: Lower number = higher priority
- Rule 90 (DENY) evaluated before Rule 100 (ALLOW)
- 203.0.113.50 traffic matches Rule 90 → DENIED
- All other traffic matches Rule 100/110 → ALLOWED
```

### 5.5 NACL vs Security Groups

| Feature | NACL | Security Group |
|---------|------|----------------|
| **Level** | Subnet | Instance (ENI) |
| **State** | Stateless | Stateful |
| **Rules** | ALLOW and DENY | ALLOW only |
| **Processing** | In order (by rule #) | All rules evaluated |
| **Applies to** | All instances in subnet | Tagged instances |
| **Default** | Allow all (default NACL) | Deny all |
| **Return traffic** | Must explicitly allow | Auto-allowed |
| **Use case** | Subnet-level protection | Instance-level protection |

### 5.6 Common NACL Patterns

#### Pattern 1: Layered Security

```
DEFENSE IN DEPTH:

Internet
   │
   ▼
┌────────────────────────────┐
│  NACL (Subnet Level)       │
│  - Block known bad IPs     │
│  - Allow required ports    │
│  - Deny all else           │
└────────────┬───────────────┘
             │
             ▼
      ┌────────────┐
      │  Security  │
      │   Group    │
      │  (Instance)│
      └──────┬─────┘
             │
             ▼
       ┌─────────┐
       │   EC2   │
       └─────────┘

Benefits:
✅ NACL blocks traffic before it reaches instances
✅ Security Group provides fine-grained control
✅ NACL can DENY (Security Groups cannot)
```

#### Pattern 2: Subnet Isolation

```
VPC (10.0.0.0/16)
├─ Public Subnet (10.0.1.0/24)
│  NACL: Allow 80, 443 from 0.0.0.0/0
│
├─ App Subnet (10.0.2.0/24)
│  NACL: Allow only from 10.0.1.0/24 (public subnet)
│
└─ DB Subnet (10.0.3.0/24)
   NACL: Allow only from 10.0.2.0/24 (app subnet)
```

### 5.7 NACL Troubleshooting

```bash
# Check NACL associations
aws ec2 describe-network-acls \
    --filters "Name=association.subnet-id,Values=subnet-12345678"

# View NACL rules
aws ec2 describe-network-acls \
    --network-acl-ids acl-abcdef \
    --query 'NetworkAcls[0].Entries'

# Common issues:
1. Forgot to allow ephemeral ports (1024-65535)
2. Rule order wrong (ALLOW after DENY)
3. Missing return traffic rules
4. Blocking VPC CIDR range
```

### 5.8 Exam Tips

**🎯 Điểm Thi Quan Trọng:**

1. **Stateless Nature:**
   - NACL does NOT track connections
   - Must allow BOTH inbound and outbound
   - Ephemeral ports (1024-65535) required

2. **Rule Processing:**
   - Lowest rule number processed first
   - First match wins (stops processing)
   - Use rule numbers with gaps (100, 110, 120) for flexibility

3. **DENY Rules:**
   - Only NACLs can DENY (Security Groups cannot)
   - Use DENY rules to block specific IPs
   - DENY rule must come before ALLOW rule

4. **Default NACL:**
   - Allows all traffic by default
   - All new subnets use default NACL
   - Best practice: Create custom NACLs

5. **Subnet Association:**
   - Each subnet has exactly ONE NACL
   - One NACL can be associated with multiple subnets
   - Subnet cannot exist without a NACL

---

## 6. Security Groups

### 6.1 Security Group Overview

**Security Groups** are stateful firewalls that control traffic at the instance (ENI) level.

```
SECURITY GROUP ARCHITECTURE:

┌──────────────────────────────────────────┐
│         VPC (10.0.0.0/16)                │
│                                          │
│  ┌────────────────────────────────────┐ │
│  │  Security Group: WebServers-SG     │ │
│  │                                    │ │
│  │  Inbound Rules:                    │ │
│  │  - 80/TCP from 0.0.0.0/0           │ │
│  │  - 443/TCP from 0.0.0.0/0          │ │
│  │  - 22/TCP from BastionSG           │ │
│  │                                    │ │
│  │  Outbound Rules:                   │ │
│  │  - All traffic to 0.0.0.0/0        │ │
│  └──────────┬─────────────────────────┘ │
│             │ Applied to ENI            │
│      ┌──────▼──────┐  ┌──────────┐     │
│      │  EC2 Web1   │  │ EC2 Web2 │     │
│      │  (WebSG)    │  │ (WebSG)  │     │
│      └─────────────┘  └──────────┘     │
│                                          │
│  ┌────────────────────────────────────┐ │
│  │  Security Group: Bastion-SG        │ │
│  │  Inbound: 22/TCP from MyIP         │ │
│  └──────────┬─────────────────────────┘ │
│      ┌──────▼──────┐                    │
│      │   Bastion   │                    │
│      └─────────────┘                    │
└──────────────────────────────────────────┘

Key Characteristics:
- Instance (ENI) level protection
- Stateful (tracks connections)
- ALLOW rules only (no DENY)
- Return traffic automatically allowed
- Can reference other Security Groups
```

### 6.2 Security Group Rules

```
RULE STRUCTURE:

Type        Protocol  Port Range  Source/Destination
─────────────────────────────────────────────────────
HTTP        TCP       80          0.0.0.0/0          (Allow from anywhere)
HTTPS       TCP       443         0.0.0.0/0          (Allow from anywhere)
SSH         TCP       22          203.0.113.0/24     (Allow from specific IP)
MySQL       TCP       3306        sg-app123          (Allow from SG)
Custom      TCP       8080        10.0.1.0/24        (Allow from CIDR)
All traffic ALL       ALL         sg-self            (Allow within same SG)

Source/Destination Options:
1. CIDR block: 0.0.0.0/0, 10.0.0.0/16
2. Security Group ID: sg-12345678 (most powerful!)
3. Prefix List: pl-id (for AWS services)
```

### 6.3 Security Group References

```
SECURITY GROUP CHAINING:

┌────────────────────────────────────────────┐
│  LoadBalancer-SG                           │
│  Inbound: 443 from 0.0.0.0/0               │
└──────────┬─────────────────────────────────┘
           │
           ▼
┌────────────────────────────────────────────┐
│  WebServer-SG                              │
│  Inbound: 443 from LoadBalancer-SG         │ ← SG reference!
└──────────┬─────────────────────────────────┘
           │
           ▼
┌────────────────────────────────────────────┐
│  AppServer-SG                              │
│  Inbound: 8080 from WebServer-SG           │ ← SG reference!
└──────────┬─────────────────────────────────┘
           │
           ▼
┌────────────────────────────────────────────┐
│  Database-SG                               │
│  Inbound: 3306 from AppServer-SG           │ ← SG reference!
└────────────────────────────────────────────┘

Benefits:
✅ No need to manage IPs
✅ Scales automatically with instances
✅ Secure communication between tiers
✅ Easy to maintain
```

### 6.4 Common Security Group Patterns

#### Pattern 1: Web Application

```bash
# Load Balancer Security Group
aws ec2 create-security-group \
    --group-name LoadBalancer-SG \
    --description "ALB Security Group" \
    --vpc-id vpc-12345678

aws ec2 authorize-security-group-ingress \
    --group-id sg-alb123 \
    --protocol tcp \
    --port 443 \
    --cidr 0.0.0.0/0

# Web Server Security Group
aws ec2 create-security-group \
    --group-name WebServer-SG \
    --description "Web Servers" \
    --vpc-id vpc-12345678

aws ec2 authorize-security-group-ingress \
    --group-id sg-web123 \
    --protocol tcp \
    --port 443 \
    --source-group sg-alb123  # Only from ALB!

# Database Security Group
aws ec2 create-security-group \
    --group-name Database-SG \
    --description "RDS MySQL" \
    --vpc-id vpc-12345678

aws ec2 authorize-security-group-ingress \
    --group-id sg-db123 \
    --protocol tcp \
    --port 3306 \
    --source-group sg-web123  # Only from Web Servers!
```

#### Pattern 2: Bastion Host

```
BASTION HOST PATTERN:

Internet
   │
   ▼
┌────────────────────────┐
│  Bastion-SG            │
│  In: 22 from MyIP      │
│  Out: 22 to PrivateSG  │
└──────┬─────────────────┘
       │
       ▼
┌────────────────────────┐
│  PrivateServer-SG      │
│  In: 22 from BastionSG │ ← Only from Bastion
│  Out: All              │
└────────────────────────┘

Configuration:
# Bastion SG
Inbound: 22/TCP from 203.0.113.50/32 (your IP)
Outbound: 22/TCP to sg-private

# Private Server SG
Inbound: 22/TCP from sg-bastion
Outbound: All traffic
```

#### Pattern 3: ECS/EKS Cluster

```
CONTAINER SECURITY GROUPS:

┌────────────────────────────────────┐
│  ALB-SG                            │
│  In: 443 from 0.0.0.0/0            │
│  Out: 8080 to ECS-Task-SG          │
└──────┬─────────────────────────────┘
       │
       ▼
┌────────────────────────────────────┐
│  ECS-Task-SG                       │
│  In: 8080 from ALB-SG              │
│  Out: 3306 to RDS-SG               │
│  Out: 443 to 0.0.0.0/0 (API calls)│
└──────┬─────────────────────────────┘
       │
       ▼
┌────────────────────────────────────┐
│  RDS-SG                            │
│  In: 3306 from ECS-Task-SG         │
└────────────────────────────────────┘
```

### 6.5 Security Group Limits

```
DEFAULT LIMITS (per region):

Resource                        Default Limit  Adjustable?
──────────────────────────────────────────────────────────
Security Groups per VPC         2,500          Yes (up to 10,000)
Rules per Security Group        60             Yes (up to 1,000)
Security Groups per ENI         5              Yes (up to 16)

Rule calculation:
- Inbound rules + Outbound rules = Total
- Example: 30 inbound + 30 outbound = 60 total ✅
- Example: 40 inbound + 40 outbound = 80 total ❌

Best practices:
✅ Use SG references (counts as 1 rule, covers many IPs)
✅ Consolidate rules (use CIDR ranges)
❌ Avoid creating too many rules per SG
```

### 6.6 Security Group vs NACL Decision Tree

```
WHEN TO USE WHAT?

Need to DENY specific IP?
├─ YES → Use NACL (only NACLs can DENY)
└─ NO → Continue

Need subnet-level protection?
├─ YES → Use NACL
└─ NO → Continue

Need to reference other Security Groups?
├─ YES → Use Security Groups
└─ NO → Continue

Want stateful (automatic return traffic)?
├─ YES → Use Security Groups
└─ NO → Use NACL

Default answer: Use Security Groups
(Easier to manage, stateful, sufficient for most cases)
```

### 6.7 Troubleshooting Security Groups

```bash
# List Security Groups
aws ec2 describe-security-groups \
    --filters "Name=vpc-id,Values=vpc-12345678"

# Check specific Security Group
aws ec2 describe-security-groups \
    --group-ids sg-abcdef

# Find which instances use a SG
aws ec2 describe-instances \
    --filters "Name=instance.group-id,Values=sg-abcdef" \
    --query 'Reservations[].Instances[].[InstanceId,Tags[?Key==`Name`].Value|[0]]'

# Check ENI's Security Groups
aws ec2 describe-network-interfaces \
    --filters "Name=network-interface-id,Values=eni-12345678" \
    --query 'NetworkInterfaces[0].Groups'

# Common issues:
1. Outbound rules blocking traffic (check egress)
2. Missing SG on ENI
3. SG in wrong VPC
4. Circular SG references (legitimate, but check carefully)
```

### 6.8 Advanced Security Group Scenarios

#### Scenario 1: Multi-Tier SAP Application

```
SAP ON AWS SECURITY GROUPS:

┌────────────────────────────────────────┐
│  SAP-Router-SG (DMZ)                   │
│  In: 3299 from 0.0.0.0/0               │
│  Out: 3200-3299 to SAP-App-SG          │
└──────┬─────────────────────────────────┘
       │
       ▼
┌────────────────────────────────────────┐
│  SAP-App-SG (Application Servers)      │
│  In: 3200-3299 from SAP-Router-SG      │
│  In: 3200-3299 from SAP-App-SG (self)  │ ← Cluster comm
│  Out: 3##13-3##15 to SAP-HANA-SG       │
└──────┬─────────────────────────────────┘
       │
       ▼
┌────────────────────────────────────────┐
│  SAP-HANA-SG (Database)                │
│  In: 3##13-3##15 from SAP-App-SG       │
│  In: 3##13-3##15 from SAP-HANA-SG      │ ← HSR replication
│  No outbound to internet               │
└────────────────────────────────────────┘

(### = instance number, e.g., 00, 01, 02)
```

#### Scenario 2: Managed Prefix Lists

```bash
# Create managed prefix list (for reusable IP sets)
aws ec2 create-managed-prefix-list \
    --prefix-list-name Office-IPs \
    --entries "Cidr=203.0.113.0/24,Description=HQ" \
              "Cidr=198.51.100.0/24,Description=Branch" \
    --max-entries 10 \
    --address-family IPv4

# Use in Security Group
aws ec2 authorize-security-group-ingress \
    --group-id sg-12345678 \
    --ip-permissions IpProtocol=tcp,FromPort=443,ToPort=443,PrefixListIds=[{PrefixListId=pl-123456}]

# Benefits:
✅ Update once, applies to all SGs using it
✅ Easier management for multiple locations
✅ Supports up to 1,000 entries
```

### 6.9 Exam Tips

**🎯 Điểm Thi Quan Trọng:**

1. **Stateful Nature:**
   - Return traffic automatically allowed
   - No need to configure ephemeral ports
   - Connection tracking enabled

2. **ALLOW Only:**
   - Cannot create DENY rules
   - Default: Deny all (unless explicitly allowed)
   - To block IP, use NACL

3. **Security Group References:**
   - Most powerful feature
   - Use sg-xxx instead of IP addresses
   - Scales automatically with instances
   - Works across VPC peering (same region)

4. **Multiple SGs per Instance:**
   - Up to 5 SGs per ENI (default)
   - Rules from all SGs are aggregated
   - More permissive wins (OR logic)

5. **Changes Take Effect Immediately:**
   - No reboot required
   - Applies to existing connections
   - Use for dynamic security adjustments

6. **Common Exam Scenarios:**
   - "Cannot connect" → Check both SG and NACL
   - "Block specific IP" → Use NACL (SG can't deny)
   - "Communication between tiers" → Use SG references
   - "Return traffic blocked" → Check if NACL allows ephemeral

---

## 7. AWS Local Zones

### 7.1 Local Zones Overview

**AWS Local Zones** extend AWS regions to place compute and storage closer to end-users for ultra-low latency applications.

```
AWS REGIONS AND LOCAL ZONES:

┌──────────────────────────────────────────────────┐
│        AWS REGION (us-east-1)                    │
│                                                  │
│  ┌─────────────────────────────────────┐        │
│  │  Availability Zone us-east-1a       │        │
│  │  - Full AWS services                │        │
│  │  - Multiple data centers            │        │
│  └─────────────────────────────────────┘        │
│                                                  │
│  ┌─────────────────────────────────────┐        │
│  │  Availability Zone us-east-1b       │        │
│  └─────────────────────────────────────┘        │
│                                                  │
│  ┌─────────────────────────────────────┐        │
│  │  Availability Zone us-east-1c       │        │
│  └─────────────────────────────────────┘        │
└──────────────────────────────────────────────────┘
                    │
                    │ Low-latency connection
                    │
┌──────────────────┬▼───────────────────────────────┐
│  LOCAL ZONE (us-east-1-bos-1a) - Boston          │
│  - Subset of AWS services                        │
│  - Single data center location                   │
│  - Sub-millisecond latency to Boston             │
│  - Extends VPC from parent region                │
└──────────────────────────────────────────────────┘

Latency Examples:
- us-east-1 (Virginia) to Boston: ~15-20ms
- Local Zone (Boston) to Boston: <5ms ✅
```

### 7.2 Local Zones Architecture

```
VPC EXTENSION TO LOCAL ZONE:

┌────────────────────────────────────────────────┐
│       VPC (10.0.0.0/16) in us-east-1           │
│                                                │
│  Parent Region Subnets:                        │
│  ┌──────────────────────────────────┐          │
│  │  Subnet A (us-east-1a)           │          │
│  │  10.0.1.0/24                     │          │
│  │  - RDS Master                    │          │
│  │  - ECS Control Plane             │          │
│  └──────────────────────────────────┘          │
│                                                │
│  ┌──────────────────────────────────┐          │
│  │  Subnet B (us-east-1b)           │          │
│  │  10.0.2.0/24                     │          │
│  └──────────────────────────────────┘          │
└──────────────────┬─────────────────────────────┘
                   │
                   │ VPC Extension
                   │
┌──────────────────▼─────────────────────────────┐
│  Local Zone Subnet                             │
│  (us-east-1-bos-1a)                            │
│  10.0.10.0/24                                  │
│                                                │
│  Resources:                                    │
│  - EC2 Instances (ultra-low latency apps)      │
│  - EBS Volumes (gp3, io2)                      │
│  - Application Load Balancer                   │
│  - VPC features (SG, NACL, etc.)               │
└────────────────────────────────────────────────┘

Key Points:
- Same VPC extends from Region to Local Zone
- Same Security Groups and NACLs apply
- Private connectivity (no internet hop)
```

### 7.3 Available Services in Local Zones

```
SERVICES AVAILABLE:

Compute:
✅ EC2 Instances (select instance types)
✅ ECS (Fargate not available)
✅ EKS worker nodes

Storage:
✅ EBS (gp3, io1, io2, st1, sc1)
❌ EFS (not available)

Networking:
✅ VPC
✅ Subnets
✅ Elastic IPs
✅ Application Load Balancer
✅ Network Load Balancer
❌ NAT Gateway (use NAT instance or parent region)

Databases:
✅ RDS (MySQL, PostgreSQL, SQL Server - not Aurora)
✅ ElastiCache
❌ DynamoDB (use parent region)

NOT Available:
❌ S3 (use parent region)
❌ Lambda
❌ CloudFront
❌ Route 53 (parent region)
```

### 7.4 Use Cases

```
USE CASE 1: REAL-TIME GAMING

┌────────────────────────────────────────────┐
│  Player in Boston                          │
└───────┬────────────────────────────────────┘
        │ <5ms latency
        ▼
┌────────────────────────────────────────────┐
│  Local Zone (Boston)                       │
│  - Game servers (EC2)                      │
│  - In-memory cache (ElastiCache)           │
│  - Local session state                     │
└───────┬────────────────────────────────────┘
        │
        │ Backend sync (can tolerate latency)
        ▼
┌────────────────────────────────────────────┐
│  Parent Region (us-east-1)                 │
│  - Master game database (RDS)              │
│  - Player profiles (DynamoDB)              │
│  - Game assets (S3)                        │
│  - Analytics (Redshift)                    │
└────────────────────────────────────────────┘

Benefits:
✅ Ultra-low latency for gameplay (<5ms)
✅ Central management of persistent data
✅ Cost-effective (only latency-sensitive in Local Zone)
```

```
USE CASE 2: MEDIA & ENTERTAINMENT

Local Zone (Los Angeles) for:
- Live video processing (EC2)
- Real-time rendering (GPU instances)
- Low-latency streaming
- On-set data processing

Parent Region (us-west-2) for:
- Media asset storage (S3)
- Transcoding jobs (MediaConvert)
- Content delivery (CloudFront)
- Archive (S3 Glacier)
```

```
USE CASE 3: SAP DISASTER RECOVERY

Primary Site (On-Premises):
- Production SAP system
- <1ms latency to users

Local Zone (Nearby city):
- DR SAP instances (stopped, ready to start)
- EBS snapshots for quick recovery
- 5-10ms latency acceptable for DR

Parent Region:
- Backup storage (S3)
- Long-term archives
- Monitoring (CloudWatch)
```

### 7.5 Configuration

```bash
# Enable Local Zone
aws ec2 modify-availability-zone-group \
    --group-name us-east-1-bos-1 \
    --opt-in-status opted-in

# Verify available zones
aws ec2 describe-availability-zones \
    --filters "Name=zone-type,Values=local-zone" \
    --region us-east-1

# Output:
# {
#   "ZoneName": "us-east-1-bos-1a",
#   "ZoneType": "local-zone",
#   "RegionName": "us-east-1",
#   "ParentZoneName": "us-east-1a"
# }

# Create subnet in Local Zone
aws ec2 create-subnet \
    --vpc-id vpc-12345678 \
    --cidr-block 10.0.10.0/24 \
    --availability-zone us-east-1-bos-1a \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=LocalZone-Boston}]'

# Launch instance in Local Zone
aws ec2 run-instances \
    --image-id ami-12345678 \
    --instance-type t3.medium \
    --subnet-id subnet-localzone \
    --security-group-ids sg-12345678
```

### 7.6 Local Zones Locations

```
AVAILABLE LOCAL ZONES (as of 2026):

United States:
- Boston (us-east-1-bos-1)
- Chicago (us-east-1-chi-1)
- Dallas (us-east-1-dfw-1)
- Houston (us-east-1-iah-1)
- Miami (us-east-1-mia-1)
- New York (us-east-1-nyc-1)
- Philadelphia (us-east-1-phl-1)
- Denver (us-west-2-den-1)
- Las Vegas (us-west-2-las-1)
- Los Angeles (us-west-2-lax-1)
- Phoenix (us-west-2-phx-1)
- Portland (us-west-2-pdx-1)

International:
- London (eu-west-2-lon-1)
- Tokyo (ap-northeast-1-tyo-1)
- And more...

Check latest:
https://aws.amazon.com/about-aws/global-infrastructure/localzones/locations/
```

### 7.7 Pricing Considerations

```
COST COMPARISON:

EC2 Instance (t3.medium):
- Standard Region: $0.0416/hour
- Local Zone: $0.0458/hour (+10% premium)

Data Transfer:
- Within Local Zone: FREE
- Local Zone → Parent Region: $0.01/GB
- Local Zone → Internet: Standard rates

EBS Volumes:
- Similar pricing to parent region
- Snapshots stored in parent region

Cost Optimization:
✅ Only run latency-sensitive workloads in Local Zone
✅ Store bulk data in parent region (S3)
✅ Use Direct Connect for high-volume transfers
```

### 7.8 Exam Tips

**🎯 Điểm Thi Quan Trọng:**

1. **What are Local Zones:**
   - Extension of AWS Region
   - Places compute/storage closer to end-users
   - Single AZ (not highly available across zones)

2. **Use Cases:**
   - Ultra-low latency (<10ms required)
   - Real-time gaming
   - Live video streaming
   - ML inference at edge
   - Applications near major cities

3. **Key Limitations:**
   - Subset of AWS services
   - Single location (not multi-AZ)
   - Must opt-in to use
   - Slight price premium

4. **Architecture:**
   - Extends VPC from parent region
   - Same Security Groups/NACLs
   - Private connectivity to region
   - Use parent region for non-latency-sensitive services

5. **Comparison:**
   - Local Zone vs AZ: Local Zone = city level, AZ = region level
   - Local Zone vs Edge Location: Local Zone has compute, Edge only caching
   - Local Zone vs Wavelength: Local Zone = general purpose, Wavelength = 5G networks

---

## 8. Border Gateway Protocol 101

### 8.1 BGP Fundamentals

**Border Gateway Protocol (BGP)** is the routing protocol of the Internet and critical for AWS hybrid connectivity.

```
BGP OVERVIEW:

┌─────────────────────────────────────────────┐
│  What is BGP?                               │
│  - Path vector protocol                     │
│  - Routes traffic between Autonomous        │
│    Systems (AS)                             │
│  - Policy-based routing                     │
│  - Used by Direct Connect, VPN, Transit GW  │
└─────────────────────────────────────────────┘

Autonomous System (AS):
- Collection of IP networks under single admin
- Identified by AS Number (ASN)
- Example: AWS = AS16509
- Your company = Private ASN (64512-65534)

┌──────────────┐            ┌──────────────┐
│ Your Network │            │     AWS      │
│  AS 65000    │ ◄────────► │  AS 16509    │
│              │    BGP     │              │
└──────────────┘            └──────────────┘

BGP exchanges routes between ASes
```

### 8.2 BGP Concepts

```
KEY CONCEPTS:

1. AS Path:
   Route advertisement includes path of ASes
   Example: AS 65000 → AS 65001 → AS 16509

   Prevents loops: If ASN already in path, reject route

2. Path Selection:
   BGP selects best path based on attributes:

   Priority (highest to lowest):
   1. Highest Weight (Cisco-specific)
   2. Highest Local Preference
   3. Locally originated routes
   4. Shortest AS Path ← Most common
   5. Lowest Origin type
   6. Lowest MED (Multi-Exit Discriminator)
   7. eBGP over iBGP
   8. Lowest IGP metric

3. BGP Attributes:
   ┌─────────────────────────────────────────┐
   │ AS_PATH:    [65000, 65001, 16509]       │ ← Loop prevention
   │ NEXT_HOP:   192.168.1.1                 │ ← Next router
   │ LOCAL_PREF: 100                         │ ← Prefer path (higher=better)
   │ MED:        50                          │ ← Suggest path (lower=better)
   │ COMMUNITY:  no-export                   │ ← Policy tags
   └─────────────────────────────────────────┘
```

### 8.3 BGP in AWS

```
AWS BGP USAGE:

┌────────────────────────────────────────────┐
│  Scenario 1: AWS Direct Connect           │
│                                            │
│  ┌──────────────┐     ┌──────────────┐    │
│  │  On-Premises │     │     AWS      │    │
│  │  Router      │     │  VGW/DGW     │    │
│  │  AS 65000    │◄───►│  AS 64512*   │    │
│  │              │ BGP │              │    │
│  └──────────────┘     └──────────────┘    │
│                                            │
│  BGP advertises:                           │
│  - On-prem: 192.168.0.0/16 → AWS          │
│  - AWS: 10.0.0.0/16 → On-prem             │
└────────────────────────────────────────────┘

* AWS VGW uses private ASN (you can specify)
  Default: 7224 (legacy) or 64512
```

```
┌────────────────────────────────────────────┐
│  Scenario 2: Site-to-Site VPN             │
│                                            │
│  Static routing:                           │
│  - Manually define routes (no BGP)         │
│  - Simple, less flexible                   │
│                                            │
│  Dynamic routing (BGP):                    │
│  - Automatic route propagation             │
│  - Failover capability                     │
│  - Preferred for production                │
└────────────────────────────────────────────┘
```

### 8.4 BGP Path Selection Example

```
SCENARIO: Multiple Paths to AWS

                    ┌──────────────┐
                    │     AWS      │
                    │  10.0.0.0/16 │
                    └───┬──────┬───┘
                        │      │
              Path A    │      │    Path B
              AS_PATH:  │      │    AS_PATH:
              [65001]   │      │    [65001,65002]
              (1 hop)   │      │    (2 hops)
                        │      │
                    ┌───▼──────▼───┐
                    │  Your Router │
                    │   AS 65000   │
                    └──────────────┘

Decision:
Path A: AS_PATH length = 1 ✅ (shorter, wins!)
Path B: AS_PATH length = 2 ❌

Traffic uses Path A (shortest AS path)
Path B kept as backup
```

### 8.5 BGP Configuration Example (AWS VPN)

```bash
# Create Customer Gateway (your router)
aws ec2 create-customer-gateway \
    --type ipsec.1 \
    --bgp-asn 65000 \
    --public-ip 203.0.113.50 \
    --tag-specifications 'ResourceType=customer-gateway,Tags=[{Key=Name,Value=OnPrem-CGW}]'

# Create Virtual Private Gateway
aws ec2 create-vpn-gateway \
    --type ipsec.1 \
    --amazon-side-asn 64512 \
    --tag-specifications 'ResourceType=vpn-gateway,Tags=[{Key=Name,Value=AWS-VGW}]'

# Attach to VPC
aws ec2 attach-vpn-gateway \
    --vpn-gateway-id vgw-12345678 \
    --vpc-id vpc-abcdef

# Create VPN Connection (Dynamic routing)
aws ec2 create-vpn-connection \
    --type ipsec.1 \
    --customer-gateway-id cgw-12345678 \
    --vpn-gateway-id vgw-12345678 \
    --options '{"StaticRoutesOnly":false}' \  ← Dynamic BGP routing
    --tag-specifications 'ResourceType=vpn-connection,Tags=[{Key=Name,Value=VPN-Dynamic}]'
```

### 8.6 BGP and AWS Transit Gateway

```
TRANSIT GATEWAY BGP:

┌────────────────────────────────────────────────┐
│        AWS Transit Gateway                     │
│           AS 64512                             │
│                                                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│  │  VPC A   │  │  VPC B   │  │  VPC C   │     │
│  │10.1.0.0  │  │10.2.0.0  │  │10.3.0.0  │     │
│  └──────────┘  └──────────┘  └──────────┘     │
└───────────────────┬────────────────────────────┘
                    │
                    │ BGP peering
                    │
┌───────────────────▼────────────────────────────┐
│     On-Premises Data Center                    │
│           AS 65000                             │
│                                                │
│  Receives BGP advertisements:                  │
│  - 10.1.0.0/16 via TGW                         │
│  - 10.2.0.0/16 via TGW                         │
│  - 10.3.0.0/16 via TGW                         │
│                                                │
│  Advertises to AWS:                            │
│  - 192.168.0.0/16 (corporate network)          │
└────────────────────────────────────────────────┘

Benefits:
✅ Automatic route updates when VPCs added/removed
✅ Dynamic failover
✅ Simplified route management
```

### 8.7 BGP Best Practices for AWS

```
BEST PRACTICES:

1. AS Path Prepending:
   Manipulate path selection by repeating ASN

   Example: Make Path B less preferred
   Normal: AS_PATH = [65000, 16509]
   Prepended: AS_PATH = [65000, 65000, 65000, 16509]
   Result: Path looks longer → deprioritized

2. BGP Communities:
   Tag routes for policy application

   AWS supports:
   - 7224:9100 = Low preference
   - 7224:9200 = Medium preference
   - 7224:9300 = High preference (default)

3. Route Filtering:
   Only advertise/accept specific prefixes

   Advertise to AWS:
   - 192.168.0.0/16 ✅ (corporate network)
   - 10.0.0.0/8 ❌ (conflicts with AWS VPC)

4. BFD (Bidirectional Forwarding Detection):
   Fast failure detection (faster than BGP keepalive)

   Enable for Direct Connect and VPN
   Detects failures in seconds vs minutes
```

### 8.8 Exam Tips

**🎯 Điểm Thi Quan Trọng:**

1. **BGP Purpose:**
   - Routes traffic between Autonomous Systems
   - Used for Direct Connect and dynamic VPN
   - Automatic route propagation
   - Enables redundancy and failover

2. **AS Numbers:**
   - AWS: AS 16509 (public)
   - VGW: Private ASN (64512 by default, customizable)
   - Your network: Must use private ASN (64512-65534)

3. **Path Selection:**
   - Shortest AS_PATH wins (most common)
   - Can manipulate with AS path prepending
   - Local preference for inbound traffic
   - MED for outbound traffic

4. **Dynamic vs Static:**
   - Dynamic (BGP): Automatic, recommended for production
   - Static: Manual routes, simpler but less flexible
   - Dynamic enables automatic failover

5. **Route Propagation:**
   - Enable in route tables to auto-populate routes
   - Works with VGW and TGW
   - Simplifies route management

---

## 9. AWS Global Accelerator

### 9.1 Global Accelerator Overview

**AWS Global Accelerator** improves availability and performance of applications using AWS global network.

```
WITHOUT GLOBAL ACCELERATOR:

User in Tokyo
   │
   │ Public Internet (multiple hops, variable latency)
   │ Route: Tokyo → ISP → Multiple ISPs → US East Coast
   │ Latency: 150-200ms, jitter, packet loss
   │
   ▼
┌─────────────────────────┐
│  Application in         │
│  us-east-1 (Virginia)   │
└─────────────────────────┘

Problems:
❌ High latency (variable)
❌ Packet loss on congested routes
❌ No intelligent routing
❌ Failover depends on DNS (slow, cache issues)
```

```
WITH GLOBAL ACCELERATOR:

User in Tokyo
   │
   │ 10ms to nearest AWS edge
   ▼
┌──────────────────────┐
│  AWS Edge Location  │ ← Anycast IP (closest edge)
│      (Tokyo)        │
└──────┬───────────────┘
       │
       │ AWS Global Network (dedicated fiber)
       │ Low latency, no congestion
       │ Latency: 80-100ms ✅ (50% improvement!)
       │
       ▼
┌─────────────────────────┐
│  Application in         │
│  us-east-1 (Virginia)   │
└─────────────────────────┘

Benefits:
✅ Traffic enters AWS network immediately
✅ Travels on AWS backbone (faster, reliable)
✅ Instant failover (no DNS caching)
✅ Static anycast IPs (no DNS changes needed)
```

### 9.2 How Global Accelerator Works

```
GLOBAL ACCELERATOR ARCHITECTURE:

┌────────────────────────────────────────────────┐
│  AWS Global Accelerator                        │
│                                                │
│  Static Anycast IPs (provided by AWS):        │
│  - 198.51.100.1 (IP Set 1)                     │
│  - 198.51.100.2 (IP Set 2)                     │
│                                                │
│  These IPs announced from ALL edge locations   │
└───────────┬────────────────────────────────────┘
            │
            │ Anycast routing
            │
    ┌───────┼───────┐
    │       │       │
┌───▼───┐ ┌─▼─────┐ ┌─▼─────┐
│ Tokyo │ │London │ │ NYC   │ AWS Edge Locations
│ Edge  │ │ Edge  │ │ Edge  │
└───┬───┘ └─┬─────┘ └─┬─────┘
    │       │         │
    │       │         │ AWS Global Network
    │       │         │
    └───────┼─────────┘
            │
    ┌───────▼───────────────────────────────────┐
    │  Endpoints (load balancers, EC2, EIPs)    │
    │                                           │
    │  ┌──────────┐        ┌──────────┐        │
    │  │  us-east-1        │ us-west-2         │
    │  │  ALB/NLB  │        │ ALB/NLB  │        │
    │  └──────────┘        └──────────┘        │
    └───────────────────────────────────────────┘

Traffic Flow:
1. User connects to anycast IP
2. Routed to nearest edge location (via BGP)
3. Traffic enters AWS global network
4. Intelligent routing to healthy endpoint
5. Continuous health monitoring
```

### 9.3 Key Features

```
FEATURE 1: STATIC ANYCAST IPS

Traditional (without Global Accelerator):
- ALB DNS: myapp-123456.us-east-1.elb.amazonaws.com
- Changes if you recreate ALB
- Must update DNS records
- DNS caching issues (TTL)

With Global Accelerator:
- Static IPs: 198.51.100.1, 198.51.100.2
- Never change
- No DNS required (or optional)
- Instant failover (no cache)

Use case: Whitelisting IPs in firewalls
✅ Whitelist 2 IPs vs dozens of ALB IPs
```

```
FEATURE 2: INSTANT FAILOVER

Health Checks:
┌────────────────────────────────────────┐
│  Global Accelerator                    │
│                                        │
│  Endpoint Group: us-east-1             │
│  ├─ ALB-1: Healthy ✅                  │
│  └─ ALB-2: Unhealthy ❌                │
│                                        │
│  Endpoint Group: us-west-2             │
│  ├─ ALB-1: Healthy ✅                  │
│  └─ ALB-2: Healthy ✅                  │
└────────────────────────────────────────┘

Failover process:
1. us-east-1 ALB fails
2. Global Accelerator detects (30 seconds)
3. Traffic instantly shifted to us-west-2
4. No DNS propagation needed

Compare to DNS failover:
- DNS TTL: 60 seconds minimum
- Cache: Can take minutes to hours
- Global Accelerator: Instant (network layer)
```

```
FEATURE 3: TRAFFIC DIALS

Control traffic distribution:

┌────────────────────────────────────────┐
│  us-east-1: Traffic Dial = 100%        │
│  (Receives 100% of assigned traffic)   │
└────────────────────────────────────────┘
┌────────────────────────────────────────┐
│  us-west-2: Traffic Dial = 0%          │
│  (Receives 0%, on standby)             │
└────────────────────────────────────────┘

Use cases:
- Blue/Green deployments
- Gradual traffic shift
- Testing new regions
- Maintenance windows

Example: Deploy new version
1. us-east-1 = 100%, us-west-2 = 0%
2. us-east-1 = 75%, us-west-2 = 25% (test)
3. us-east-1 = 50%, us-west-2 = 50%
4. us-east-1 = 0%, us-west-2 = 100% (cutover)
```

### 9.4 Endpoint Types

```
SUPPORTED ENDPOINTS:

1. Application Load Balancer (ALB)
   ✅ HTTP/HTTPS applications
   ✅ Health checks at ALB level
   Example: Web applications

2. Network Load Balancer (NLB)
   ✅ TCP/UDP applications
   ✅ Ultra-high performance
   Example: Gaming, IoT

3. EC2 Instances
   ✅ Direct to instance
   ⚠️  Must have public IP
   Example: Legacy apps

4. Elastic IP
   ✅ Static IP endpoint
   ✅ Can be behind NLB/instance
   Example: Fixed IP requirements

NOT Supported:
❌ Lambda
❌ S3
❌ API Gateway
```

### 9.5 Configuration Example

```bash
# Create Global Accelerator
aws globalaccelerator create-accelerator \
    --name MyApp-Accelerator \
    --ip-address-type IPV4 \
    --enabled

# Output includes static IPs:
# {
#   "Accelerator": {
#     "AcceleratorArn": "arn:aws:globalaccelerator::123456789012:accelerator/abc123",
#     "IpSets": [
#       {
#         "IpFamily": "IPv4",
#         "IpAddresses": ["198.51.100.1", "198.51.100.2"]
#       }
#     ]
#   }
# }

# Create Listener (port 80)
aws globalaccelerator create-listener \
    --accelerator-arn arn:aws:globalaccelerator::123456789012:accelerator/abc123 \
    --port-ranges FromPort=80,ToPort=80 \
    --protocol TCP

# Create Endpoint Group (us-east-1)
aws globalaccelerator create-endpoint-group \
    --listener-arn arn:aws:globalaccelerator::123456789012:listener/def456 \
    --endpoint-group-region us-east-1 \
    --traffic-dial-percentage 100 \
    --health-check-interval-seconds 30 \
    --endpoint-configurations \
        EndpointId=arn:aws:elasticloadbalancing:us-east-1:123456789012:loadbalancer/app/myapp/abc123,Weight=100

# Create Endpoint Group (us-west-2) for redundancy
aws globalaccelerator create-endpoint-group \
    --listener-arn arn:aws:globalaccelerator::123456789012:listener/def456 \
    --endpoint-group-region us-west-2 \
    --traffic-dial-percentage 0 \
    --endpoint-configurations \
        EndpointId=arn:aws:elasticloadbalancing:us-west-2:123456789012:loadbalancer/app/myapp/xyz789,Weight=100
```

### 9.6 Global Accelerator vs CloudFront

| Feature | Global Accelerator | CloudFront |
|---------|-------------------|------------|
| **Use Case** | TCP/UDP applications | Static/dynamic content (HTTP/HTTPS) |
| **Caching** | ❌ No caching | ✅ Caching at edge |
| **Protocols** | TCP, UDP | HTTP, HTTPS, WebSocket |
| **Static IPs** | ✅ Yes (2 anycast IPs) | ❌ No (uses DNS) |
| **Failover** | Instant (network layer) | DNS-based (slower) |
| **Endpoints** | ALB, NLB, EC2, EIP | S3, ALB, EC2, custom origins |
| **Client IP** | Preserved | In X-Forwarded-For header |
| **Best for** | Gaming, IoT, VoIP, non-HTTP | Web apps, API, video streaming |

```
DECISION TREE:

Need caching?
├─ YES → CloudFront
└─ NO → Continue

HTTP/HTTPS only?
├─ YES → CloudFront (probably)
└─ NO (TCP/UDP) → Global Accelerator

Need static IPs for whitelisting?
├─ YES → Global Accelerator
└─ NO → CloudFront

Need instant failover (not DNS)?
├─ YES → Global Accelerator
└─ NO → CloudFront
```

### 9.7 Use Cases

```
USE CASE 1: ONLINE GAMING

Requirements:
- Low latency (<100ms globally)
- UDP protocol
- No caching needed
- Instant failover

Solution: Global Accelerator ✅
- Static IPs for game clients
- UDP support
- Anycast routing to nearest edge
- Instant region failover
```

```
USE CASE 2: VOIP APPLICATION

Requirements:
- Real-time communication
- TCP/UDP
- Consistent performance
- Global user base

Solution: Global Accelerator ✅
- Reduces jitter and packet loss
- Predictable performance (AWS network)
- Automatic failover
```

```
USE CASE 3: MULTI-REGION WEB APP

Requirements:
- HTTP/HTTPS
- Active-active across regions
- Static IPs for firewall rules
- Disaster recovery

Solution: Global Accelerator ✅
- Static anycast IPs (firewall whitelist)
- Intelligent routing to healthy endpoints
- Traffic dials for gradual migration
- Instant failover
```

### 9.8 Pricing

```
PRICING MODEL:

Fixed Fee:
- $0.025 per hour per accelerator
- Includes 2 static anycast IPs

Data Transfer:
- $0.015 per GB (DT-Premium-Out)
- Charged for data leaving AWS via accelerator

Example calculation (1 TB/month):
- Fixed: $0.025/hour × 730 hours = $18.25
- Data: $0.015 × 1,000 GB = $15.00
- Total: $33.25/month (plus endpoint costs)

Compare to standard data transfer:
- Standard internet out: $0.09/GB
- Via Global Accelerator: $0.015/GB
- Savings on data transfer! ✅
```

### 9.9 Exam Tips

**🎯 Điểm Thi Quan Trọng:**

1. **What is Global Accelerator:**
   - Improves performance using AWS global network
   - Provides 2 static anycast IPs
   - Works with TCP/UDP (any protocol)
   - Instant failover (no DNS caching issues)

2. **vs CloudFront:**
   - CloudFront: HTTP/HTTPS, caching, dynamic IPs
   - Global Accelerator: Any protocol, no caching, static IPs
   - CloudFront = content delivery
   - Global Accelerator = network performance

3. **Use Cases:**
   - Online gaming (UDP, low latency)
   - VoIP applications
   - IoT applications
   - Need static IPs for whitelisting
   - Multi-region active-active architectures

4. **Key Features:**
   - Static anycast IPs (2 IPs per accelerator)
   - Automatic health checks
   - Traffic dials (0-100% per endpoint group)
   - Instant failover (network layer, not DNS)
   - Client IP preservation

5. **Endpoints:**
   - ALB, NLB, EC2 instances, Elastic IPs
   - Can mix endpoint types
   - Regional endpoint groups
   - Health checks at endpoint level

---

## 10. IPSec VPN Fundamentals

### 10.1 IPSec Overview

**IPSec (Internet Protocol Security)** is a protocol suite for securing IP communications by authenticating and encrypting packets.

```
IPSEC BASICS:

Purpose: Secure communication over untrusted networks (internet)

┌────────────────┐         ┌────────────────┐
│  On-Premises   │         │      AWS       │
│    Network     │◄───────►│      VPC       │
│ 192.168.0.0/16 │  IPSec  │  10.0.0.0/16   │
└────────────────┘  Tunnel └────────────────┘
                    (Encrypted)

What IPSec provides:
1. Confidentiality: Data encryption (AES)
2. Integrity: Tamper detection (SHA)
3. Authentication: Verify sender (PSK, certificates)
4. Anti-replay: Sequence numbers prevent replays
```

### 10.2 IPSec Components

```
IPSEC ARCHITECTURE:

┌─────────────────────────────────────────────┐
│  IKE (Internet Key Exchange)                │
│  Phase 1: Establish secure channel          │
│  Phase 2: Establish IPSec tunnel            │
└─────────────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────────┐
│  IPSec Protocols                            │
│  - AH (Authentication Header)               │
│  - ESP (Encapsulating Security Payload)     │
└─────────────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────────┐
│  Security Associations (SA)                 │
│  - Encryption algorithms                    │
│  - Authentication methods                   │
│  - Lifetime settings                        │
└─────────────────────────────────────────────┘
```

### 10.3 IKE Phases

```
IKE PHASE 1 (ISAKMP SA):

Purpose: Establish secure management channel

┌────────────────────────────────────────────┐
│  Initiator                  Responder      │
│  (On-Premises)              (AWS VGW)      │
│                                            │
│  1. Send proposals ────────────────►       │
│     - Encryption: AES-256                  │
│     - Hash: SHA-256                        │
│     - Auth: Pre-shared key                 │
│     - DH Group: Group 14                   │
│                                            │
│  2.     ◄──────────────── Accept proposal  │
│                                            │
│  3. Exchange keys ◄────────────────►       │
│     (Diffie-Hellman)                       │
│                                            │
│  4. ✅ Phase 1 tunnel established          │
│     (Management channel secured)           │
└────────────────────────────────────────────┘

Phase 1 Modes:
- Main Mode: 6 messages, more secure
- Aggressive Mode: 3 messages, faster (AWS uses Main Mode)

Result: ISAKMP Security Association created
```

```
IKE PHASE 2 (IPSEC SA):

Purpose: Establish data tunnel(s)

┌────────────────────────────────────────────┐
│  Uses Phase 1 tunnel to negotiate          │
│                                            │
│  1. Send IPSec proposals ─────────────►    │
│     - Protocol: ESP                        │
│     - Encryption: AES-256                  │
│     - Hash: SHA-256                        │
│     - Perfect Forward Secrecy: Group 14    │
│     - Lifetime: 3600 seconds               │
│                                            │
│  2.     ◄──────────────── Accept           │
│                                            │
│  3. ✅ Phase 2 tunnel established          │
│     (Data tunnel secured)                  │
└────────────────────────────────────────────┘

Result: IPSec Security Association created
- Can have multiple Phase 2 SAs
- Each SA is unidirectional (need 2 for bidirectional)
```

### 10.4 IPSec Protocols

```
ESP vs AH:

┌────────────────────────────────────────────┐
│  ESP (Encapsulating Security Payload)      │
│                                            │
│  Provides:                                 │
│  ✅ Encryption (confidentiality)           │
│  ✅ Authentication (integrity)             │
│  ✅ Anti-replay protection                 │
│                                            │
│  Most commonly used (AWS uses ESP)         │
└────────────────────────────────────────────┘

┌────────────────────────────────────────────┐
│  AH (Authentication Header)                │
│                                            │
│  Provides:                                 │
│  ✅ Authentication (integrity)             │
│  ✅ Anti-replay protection                 │
│  ❌ No encryption                          │
│                                            │
│  Rarely used (incompatible with NAT)       │
└────────────────────────────────────────────┘

ESP Packet Structure:
┌─────────────────────────────────────────┐
│ IP Header (new)                         │
├─────────────────────────────────────────┤
│ ESP Header                              │
├─────────────────────────────────────────┤
│ Original IP Header    │ ← Encrypted     │
│ TCP/UDP Header        │                 │
│ Payload (data)        │                 │
├─────────────────────────────────────────┤
│ ESP Trailer (padding)                   │
├─────────────────────────────────────────┤
│ ESP Auth (MAC)                          │
└─────────────────────────────────────────┘
```

### 10.5 IPSec Modes

```
TUNNEL MODE (AWS uses this):

Original Packet:
┌────────────┬────────────┬─────────┐
│ IP Header  │ TCP Header │ Payload │
│ 192.168.1.1│            │         │
│ → 10.0.1.1 │            │         │
└────────────┴────────────┴─────────┘

After IPSec Tunnel Mode:
┌──────────┬────────┬────────────┬────────────┬─────────┬────────┐
│ New IP   │  ESP   │ Original   │ TCP Header │ Payload │  ESP   │
│  Header  │ Header │ IP Header  │            │         │ Trailer│
│ (VPN GW  │        │ (encrypted)│ (encrypted)│(encrypt)│        │
│  IPs)    │        │            │            │         │        │
└──────────┴────────┴────────────┴────────────┴─────────┴────────┘

- Entire original packet encrypted
- New IP header for tunnel endpoints
- Used for site-to-site VPNs


TRANSPORT MODE (not used by AWS Site-to-Site VPN):

After IPSec Transport Mode:
┌────────────┬────────┬────────────┬─────────┬────────┐
│ IP Header  │  ESP   │ TCP Header │ Payload │  ESP   │
│ (original) │ Header │ (encrypted)│(encrypt)│ Trailer│
└────────────┴────────┴────────────┴─────────┴────────┘

- Only payload encrypted, original IP header intact
- Used for host-to-host (e.g., client VPN)
```

### 10.6 Encryption & Authentication Algorithms

```
AWS SITE-TO-SITE VPN SUPPORTED:

Phase 1 (IKE):
┌──────────────────────────────────────────┐
│ Encryption:                              │
│ - AES-128, AES-256 (recommended)         │
│ - AES-128-GCM-16, AES-256-GCM-16         │
│                                          │
│ Integrity:                               │
│ - SHA-256 (recommended)                  │
│ - SHA-384, SHA-512                       │
│ - SHA-1 (legacy, not recommended)        │
│                                          │
│ DH (Diffie-Hellman) Groups:              │
│ - Group 14 (2048-bit) - recommended      │
│ - Group 15, 16, 17, 18 (stronger)        │
│ - Group 19-24 (Elliptic Curve)           │
│ - Group 2 (1024-bit) - legacy            │
└──────────────────────────────────────────┘

Phase 2 (IPSec):
┌──────────────────────────────────────────┐
│ Encryption:                              │
│ - AES-128, AES-256 (recommended)         │
│ - AES-128-GCM-16, AES-256-GCM-16         │
│                                          │
│ Integrity:                               │
│ - SHA-256 (recommended)                  │
│ - SHA-384, SHA-512                       │
│                                          │
│ Perfect Forward Secrecy (PFS):           │
│ - Enabled: DH Group 14+ recommended      │
│ - Disabled: Not recommended              │
└──────────────────────────────────────────┘

Recommended Configuration:
Phase 1: AES-256, SHA-256, DH Group 14
Phase 2: AES-256, SHA-256, DH Group 14
```

### 10.7 Dead Peer Detection (DPD)

```
DEAD PEER DETECTION:

Purpose: Detect when VPN peer is unreachable

┌──────────────┐               ┌──────────────┐
│  On-Premises │               │     AWS      │
│     VPN      │               │     VGW      │
└──────┬───────┘               └──────┬───────┘
       │                              │
       │ ◄───── DPD Request ──────────┤
       │                              │
       ├────── DPD Reply ────────────►│
       │                              │
       │  (Every 10 seconds)          │
       │                              │
       │ ◄───── DPD Request ──────────┤
       │  (No reply - timeout)        │
       │                              │
       │                              │
       │ ❌ Tunnel marked as DOWN     │
       │ Attempt to re-establish      │

AWS VGW DPD Settings:
- DPD Timeout: 30 seconds (default)
- DPD Action: Restart (re-establish tunnel)
- Cannot be disabled

Important:
- Ensure on-premises device responds to DPD
- Configure DPD timeout consistently
- Monitor tunnel status in CloudWatch
```

### 10.8 NAT Traversal (NAT-T)

```
NAT-T (UDP Encapsulation):

Problem: NAT devices modify IP headers
         IPSec authentication fails (integrity check)

Solution: Encapsulate ESP in UDP port 4500

Without NAT-T:
┌───────────┬──────────────┐
│ IP Header │ ESP Packet   │
└───────────┴──────────────┘
NAT changes IP → Auth fails ❌

With NAT-T:
┌───────────┬────────────┬──────────────┐
│ IP Header │ UDP Header │ ESP Packet   │
│           │ Port 4500  │              │
└───────────┴────────────┴──────────────┘
NAT changes IP, but ESP inside UDP ✅

AWS automatically enables NAT-T:
- Detected during IKE negotiation
- UDP port 4500 used automatically
- No configuration needed (on AWS side)

Firewall rules needed:
- UDP 500 (IKE)
- UDP 4500 (NAT-T / ESP)
- IP Protocol 50 (ESP, if no NAT)
```

### 10.9 Exam Tips

**🎯 Điểm Thi Quan Trọng:**

1. **IKE Phases:**
   - Phase 1: Establish secure management channel (ISAKMP SA)
   - Phase 2: Establish data tunnel (IPSec SA)
   - Phase 1 must succeed before Phase 2

2. **Protocols:**
   - ESP: Encryption + Authentication (AWS uses this)
   - AH: Authentication only (rarely used)
   - AWS Site-to-Site VPN uses ESP in Tunnel Mode

3. **Encryption:**
   - Recommended: AES-256
   - Integrity: SHA-256
   - DH Group: 14 or higher
   - Perfect Forward Secrecy: Enabled

4. **Ports Required:**
   - UDP 500: IKE (Phase 1)
   - UDP 4500: NAT-T (ESP encapsulated)
   - IP Protocol 50: ESP (if no NAT)

5. **Dead Peer Detection:**
   - AWS VGW: 30 second timeout (default)
   - Action: Restart tunnel
   - Required for automatic failover

6. **Tunnel vs Transport:**
   - Tunnel Mode: Encrypts entire packet (site-to-site)
   - Transport Mode: Encrypts payload only (host-to-host)
   - AWS uses Tunnel Mode

---

## 11. Site-to-Site VPN

### 11.1 Site-to-Site VPN Overview

**AWS Site-to-Site VPN** connects your on-premises network to AWS VPC over the internet using IPSec.

```
SITE-TO-SITE VPN ARCHITECTURE:

┌───────────────────────────────────────────┐
│  On-Premises Data Center                  │
│  192.168.0.0/16                            │
│                                            │
│  ┌──────────────────┐                      │
│  │  Customer        │                      │
│  │  Gateway Device  │                      │
│  │  (CGW)           │                      │
│  │  203.0.113.50    │                      │
│  └────────┬─────────┘                      │
└───────────┼────────────────────────────────┘
            │
            │ Internet (IPSec encrypted)
            │ Tunnel 1 & Tunnel 2
            │
┌───────────▼────────────────────────────────┐
│  AWS Region                                │
│                                            │
│  ┌─────────────────┐                       │
│  │  Virtual Private│                       │
│  │  Gateway (VGW)  │                       │
│  │  Attached to VPC│                       │
│  └────────┬────────┘                       │
│           │                                │
│  ┌────────▼─────────────────────┐          │
│  │  VPC (10.0.0.0/16)           │          │
│  │                              │          │
│  │  Private Subnets             │          │
│  │  - EC2 Instances             │          │
│  │  - RDS Databases             │          │
│  │  - Private resources         │          │
│  └──────────────────────────────┘          │
└────────────────────────────────────────────┘

Key Components:
1. Customer Gateway (CGW): Your on-prem router
2. Virtual Private Gateway (VGW): AWS side VPN endpoint
3. VPN Connection: 2 IPSec tunnels for HA
```

### 11.2 High Availability

```
REDUNDANT TUNNEL ARCHITECTURE:

AWS automatically provisions 2 tunnels per VPN connection:

┌──────────────────────────────────────────────┐
│  On-Premises                                 │
│                                              │
│  ┌────────────┐                              │
│  │ CGW Device │                              │
│  │            │                              │
│  └──┬──────┬──┘                              │
└─────┼──────┼────────────────────────────────┘
      │      │
      │      │  Both tunnels active
      │      │  (Active/Active with BGP)
      │      │
      │      │  Internet
      │      │
┌─────┼──────┼────────────────────────────────┐
│  ┌──▼──┐ ┌▼───┐                             │
│  │VGW  │ │VGW │  ← Two tunnel endpoints     │
│  │AZ-A │ │AZ-B│    in different AZs         │
│  └──┬──┘ └┬───┘                             │
│     └─────┘                                 │
│  VGW (highly available)                     │
│                                             │
│  VPC (10.0.0.0/16)                          │
└─────────────────────────────────────────────┘

Benefits:
✅ Automatic failover
✅ No single point of failure
✅ Both tunnels in different AZs
✅ Active/Active (with BGP) or Active/Standby

Important:
- Tunnel 1 and Tunnel 2 terminate in different AZs
- Monitor both tunnels (CloudWatch metrics)
- Configure on-prem device to use both tunnels
- BGP recommended for automatic failover
```

### 11.3 Configuration Components

```
1. CUSTOMER GATEWAY (CGW):
   - Represents your on-premises VPN device
   - Requires public IP address
   - Specify BGP ASN (if using dynamic routing)

aws ec2 create-customer-gateway \
    --type ipsec.1 \
    --public-ip 203.0.113.50 \
    --bgp-asn 65000 \
    --tag-specifications 'ResourceType=customer-gateway,Tags=[{Key=Name,Value=OnPrem-CGW}]'

2. VIRTUAL PRIVATE GATEWAY (VGW):
   - AWS-side VPN endpoint
   - Attached to VPC
   - Specify Amazon-side BGP ASN (default: 64512)

aws ec2 create-vpn-gateway \
    --type ipsec.1 \
    --amazon-side-asn 64512 \
    --tag-specifications 'ResourceType=vpn-gateway,Tags=[{Key=Name,Value=Prod-VGW}]'

aws ec2 attach-vpn-gateway \
    --vpn-gateway-id vgw-12345678 \
    --vpc-id vpc-abcdef

3. VPN CONNECTION:
   - Links CGW to VGW
   - Creates 2 IPSec tunnels
   - Static or Dynamic (BGP) routing

aws ec2 create-vpn-connection \
    --type ipsec.1 \
    --customer-gateway-id cgw-12345678 \
    --vpn-gateway-id vgw-12345678 \
    --options '{"StaticRoutesOnly":false,"TunnelOptions":[{"TunnelInsideCidr":"169.254.10.0/30","PreSharedKey":"MySecureKey123"},{"TunnelInsideCidr":"169.254.11.0/30","PreSharedKey":"MySecureKey456"}]}' \
    --tag-specifications 'ResourceType=vpn-connection,Tags=[{Key=Name,Value=Prod-VPN}]'
```

### 11.4 Routing Options

```
STATIC ROUTING:

Configuration:
- Manually specify on-premises CIDR blocks
- VGW only routes to specified CIDRs
- Simple, but no automatic failover

aws ec2 create-vpn-connection-route \
    --vpn-connection-id vpn-12345678 \
    --destination-cidr-block 192.168.0.0/16

Use case:
- Simple network topology
- Single site
- No need for automatic failover
- Legacy devices without BGP support

┌─────────────────────────────────────┐
│  VGW Route Table                    │
│                                     │
│  192.168.0.0/16 → vpn-12345678      │ ← Manually added
│  172.16.0.0/12  → vpn-12345678      │ ← Manually added
└─────────────────────────────────────┘
```

```
DYNAMIC ROUTING (BGP):

Configuration:
- BGP automatically exchanges routes
- Automatic failover between tunnels
- Route propagation to VPC route tables

Enable route propagation:
aws ec2 enable-vgw-route-propagation \
    --route-table-id rtb-12345678 \
    --gateway-id vgw-12345678

Use case:
- Multiple sites
- Automatic failover required
- Complex network topology
- Production environments (recommended)

┌─────────────────────────────────────┐
│  VGW Route Table (auto-populated)   │
│                                     │
│  192.168.0.0/16 → vpn-tunnel1       │ ← Learned via BGP
│  192.168.0.0/16 → vpn-tunnel2       │ ← Learned via BGP
│  172.16.0.0/12  → vpn-tunnel1       │ ← Learned via BGP
└─────────────────────────────────────┘

Benefits:
✅ Automatic failover (tunnel failure)
✅ No manual route updates
✅ Supports multiple VPN connections
✅ Load balancing with ECMP (Equal Cost Multi-Path)
```

### 11.5 Tunnel Inside CIDR

```
TUNNEL INSIDE ADDRESSING:

Each tunnel needs a /30 subnet for BGP peering:

Tunnel 1:
┌──────────────────────────────────────┐
│  Inside CIDR: 169.254.10.0/30        │
│                                      │
│  169.254.10.0    Network address     │
│  169.254.10.1    AWS VGW endpoint    │
│  169.254.10.2    CGW endpoint        │
│  169.254.10.3    Broadcast           │
└──────────────────────────────────────┘

Tunnel 2:
┌──────────────────────────────────────┐
│  Inside CIDR: 169.254.11.0/30        │
│                                      │
│  169.254.11.0    Network address     │
│  169.254.11.1    AWS VGW endpoint    │
│  169.254.11.2    CGW endpoint        │
│  169.254.11.3    Broadcast           │
└──────────────────────────────────────┘

Requirements:
- Must be in 169.254.0.0/16 range
- Must be /30 subnet (4 addresses)
- Must not overlap with other tunnels
- AWS assigns automatically (or specify manually)
```

### 11.6 VPN CloudHub

```
VPN CLOUDHUB ARCHITECTURE:

Multiple sites connected to same VGW:

┌──────────────┐          ┌──────────────┐
│   Branch     │          │   Branch     │
│   Office A   │          │   Office B   │
│ 192.168.1.0  │          │ 192.168.2.0  │
└──────┬───────┘          └──────┬───────┘
       │                         │
       │ VPN Connection 1        │ VPN Connection 2
       │                         │
       └───────────┬─────────────┘
                   │
           ┌───────▼────────┐
           │  AWS VGW       │ ← Hub
           │  (BGP enabled) │
           └───────┬────────┘
                   │
          ┌────────▼───────┐
          │  VPC (optional)│
          └────────────────┘

Features:
✅ Site-to-site communication via AWS
✅ No on-premises hub required
✅ Uses BGP for route advertisement
✅ Traffic routes through AWS

Example:
- Branch A (192.168.1.0/24)
- Branch B (192.168.2.0/24)
- Branch A → VGW → Branch B

Configuration:
1. Create CGW for each branch
2. Create VPN connection for each CGW to same VGW
3. Enable BGP on all connections
4. Branches advertise routes via BGP
5. All branches receive routes from VGW

Cost:
- Standard VPN connection pricing
- Data transfer charges apply
- No additional CloudHub fees
```

### 11.7 VPN Monitoring

```bash
# Check VPN connection status
aws ec2 describe-vpn-connections \
    --vpn-connection-ids vpn-12345678 \
    --query 'VpnConnections[0].[State,VgwTelemetry]'

# Output:
# {
#   "State": "available",
#   "VgwTelemetry": [
#     {
#       "OutsideIpAddress": "203.0.113.1",
#       "Status": "UP",
#       "LastStatusChange": "2026-02-10T12:00:00.000Z",
#       "StatusMessage": "IPSEC IS UP",
#       "AcceptedRouteCount": 5
#     },
#     {
#       "OutsideIpAddress": "203.0.113.2",
#       "Status": "DOWN",
#       "LastStatusChange": "2026-02-10T11:00:00.000Z",
#       "StatusMessage": "IPSEC IS DOWN",
#       "AcceptedRouteCount": 0
#     }
#   ]
# }

# CloudWatch Metrics
- TunnelState (0=DOWN, 1=UP)
- TunnelDataIn (bytes)
- TunnelDataOut (bytes)

# Create CloudWatch Alarm for tunnel down
aws cloudwatch put-metric-alarm \
    --alarm-name VPN-Tunnel-Down \
    --alarm-description "Alert when VPN tunnel is down" \
    --metric-name TunnelState \
    --namespace AWS/VPN \
    --statistic Average \
    --period 300 \
    --threshold 0.5 \
    --comparison-operator LessThanThreshold \
    --evaluation-periods 2 \
    --dimensions Name=VpnId,Value=vpn-12345678 Name=TunnelIpAddress,Value=203.0.113.1

# VPN Logs (if enabled)
- Published to CloudWatch Logs
- Troubleshooting IKE, IPSec issues
```

### 11.8 Limitations and Considerations

```
VPN LIMITATIONS:

Bandwidth:
- Maximum: 1.25 Gbps per tunnel
- Total: 2.5 Gbps (2 tunnels, ECMP)
- For higher bandwidth, use Direct Connect

Latency:
- Dependent on internet
- Variable, unpredictable
- Typically 20-100ms (regional)
- Use Direct Connect for consistent latency

Availability:
- 99.95% SLA (per tunnel)
- 2 tunnels = higher availability
- Monitor both tunnels

MTU (Maximum Transmission Unit):
- Internet path MTU: 1500 bytes (standard)
- IPSec overhead: ~100 bytes
- Effective MTU: ~1400 bytes
- Configure TCP MSS clamping: 1360 bytes

Route Limits:
- Static routing: 100 routes per VPN
- Dynamic (BGP): 100 prefixes advertised to AWS
- AWS advertises: VPC CIDR + propagated routes
```

### 11.9 Accelerated Site-to-Site VPN

```
ACCELERATED VPN (with Global Accelerator):

Standard VPN:
┌──────────┐         ┌──────────┐
│ On-Prem  │         │   AWS    │
│   CGW    ├────────►│   VGW    │
│          │ Internet│          │
└──────────┘ (slow)  └──────────┘

Latency: Variable, can be high


Accelerated VPN:
┌──────────┐         ┌──────────┐         ┌──────────┐
│ On-Prem  │         │   AWS    │         │   AWS    │
│   CGW    ├────────►│   Edge   ├────────►│   VGW    │
│          │ Internet│ Location │  AWS    │          │
└──────────┘         └──────────┘ Network └──────────┘
                     (nearby)    (fast)

Latency: Improved by using AWS global network

Enable accelerated VPN:
aws ec2 create-vpn-connection \
    --type ipsec.1 \
    --customer-gateway-id cgw-12345678 \
    --vpn-gateway-id vgw-12345678 \
    --options '{"EnableAcceleration":true}'

Benefits:
✅ Better performance (AWS network)
✅ More consistent latency
✅ Better reliability

Considerations:
- Additional cost ($0.05/hour per VPN)
- Not available in all regions
- Requires compatible CGW device
```

### 11.10 VPN vs Direct Connect

| Feature | Site-to-Site VPN | Direct Connect |
|---------|------------------|----------------|
| **Connection** | Over internet (IPSec) | Dedicated fiber |
| **Setup Time** | Minutes | Weeks/months |
| **Bandwidth** | Up to 1.25 Gbps per tunnel | 1, 10, 100 Gbps |
| **Cost** | $0.05/hour + data | $0.30/hour+ (port) + data |
| **Latency** | Variable (internet) | Consistent, low |
| **Security** | Encrypted (IPSec) | Private (no internet) |
| **Availability** | 99.95% SLA | 99.9% SLA (add redundancy) |
| **Use Case** | Quick setup, low cost | High bandwidth, consistent |

```
HYBRID APPROACH:

Best Practice: VPN + Direct Connect

┌────────────────┐
│  On-Premises   │
└───┬────────┬───┘
    │        │
    │ DX     │ VPN (backup)
    │        │
┌───▼────────▼───┐
│      AWS       │
└────────────────┘

Direct Connect: Primary (high bandwidth, low latency)
Site-to-Site VPN: Backup (immediate failover)

Benefits:
✅ High performance (DX primary)
✅ Instant failover (VPN backup)
✅ Cost-effective (VPN only pays during failover)
```

### 11.11 Exam Tips

**🎯 Điểm Thi Quan Trọng:**

1. **Components:**
   - Customer Gateway (CGW): Your router, needs public IP
   - Virtual Private Gateway (VGW): AWS endpoint, attached to VPC
   - VPN Connection: Creates 2 tunnels (HA)

2. **High Availability:**
   - Always 2 tunnels per VPN connection
   - Tunnels in different AZs
   - Monitor both tunnels (CloudWatch)
   - Use BGP for automatic failover

3. **Routing:**
   - Static: Manual routes, simpler
   - Dynamic (BGP): Automatic, recommended for production
   - Route propagation: Enable in route tables (dynamic only)

4. **Bandwidth & Latency:**
   - Max 1.25 Gbps per tunnel
   - Total 2.5 Gbps (with ECMP)
   - Latency variable (depends on internet)
   - Use Direct Connect for >2.5 Gbps

5. **VPN CloudHub:**
   - Connect multiple sites via single VGW
   - Site-to-site communication via AWS
   - Requires BGP
   - No additional fees (standard VPN pricing)

6. **Common Scenarios:**
   - "Need immediate connectivity" → Site-to-Site VPN
   - "Need consistent latency" → Direct Connect
   - "Need both redundancy and performance" → VPN + Direct Connect
   - "Multiple branches need to communicate" → VPN CloudHub

7. **Troubleshooting:**
   - Tunnel down: Check firewall (UDP 500, 4500, IP 50)
   - Routes not appearing: Enable route propagation
   - No traffic: Check Security Groups and NACLs
   - Slow performance: Check MTU/MSS settings

---

## Tổng Kết Phần 1

Phần 1 đã cover các foundational concepts của AWS networking:

✅ **Private vs Public Services** - Hiểu cách AWS services được phân loại
✅ **DHCP** - Auto-configuration trong VPC
✅ **VPC Router** - Route tables và routing logic
✅ **Stateful vs Stateless Firewalls** - NACLs vs Security Groups
✅ **NACLs** - Subnet-level stateless protection
✅ **Security Groups** - Instance-level stateful protection
✅ **Local Zones** - Ultra-low latency deployments
✅ **BGP** - Routing protocol cho hybrid connectivity
✅ **Global Accelerator** - Performance optimization via AWS network
✅ **IPSec** - VPN encryption fundamentals
✅ **Site-to-Site VPN** - Hybrid connectivity qua internet

**Phần 2 sẽ cover:**
- Direct Connect (DX)
- Transit Gateway (TGW)
- VPC Peering
- PrivateLink & VPC Endpoints
- Route 53 DNS
- CloudFront
- Advanced networking patterns

Stay tuned! 🚀