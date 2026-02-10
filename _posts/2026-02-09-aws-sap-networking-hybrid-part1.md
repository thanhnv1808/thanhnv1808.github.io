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

(Continued in next response due to length...)