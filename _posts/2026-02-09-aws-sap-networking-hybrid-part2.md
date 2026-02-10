---
title: "AWS SAP - Networking & Hybrid Architecture - Phần 2"
author: thanhnv1808
date: 2026-02-09 11:00:00 +0700
categories: [AWS, Solutions Architect Professional]
tags: [aws, sap, networking, transit-gateway, vpn, client-vpn, advanced-routing, site-to-site-vpn]
description: Hướng dẫn chi tiết về Transit Gateway, Advanced VPC Routing, Accelerated VPN, và AWS Client VPN cho kỳ thi AWS Solutions Architect Professional (SAP-C02). Bao gồm network segmentation, routing patterns, và các kịch bản thực tế cho SAP workloads.
pin: false
comments: true
---

# AWS SAP-C02: Networking & Hybrid Architecture - Phần 2

## Mục Lục - Phần 2
- [12. AWS Transit Gateway](#12-aws-transit-gateway)
- [13. Advanced VPC Routing](#13-advanced-vpc-routing)
- [14. Accelerated Site-to-Site VPN](#14-accelerated-site-to-site-vpn)
- [15. AWS Client VPN](#15-aws-client-vpn)
- [Câu Hỏi Ôn Tập](#câu-hỏi-ôn-tập)
- [Bảng Tổng Kết](#bảng-tổng-kết)

---

## 12. AWS Transit Gateway

### 12.1 Transit Gateway Là Gì?

**Transit Gateway (TGW)** là dịch vụ network transit hub cho phép bạn kết nối:
- Hàng nghìn VPC
- On-premises networks qua VPN
- Direct Connect Gateways
- Peering với Transit Gateways khác

```
TRƯỚC KHI CÓ TRANSIT GATEWAY:
===================================
        VPC-A
       /  |  \
      /   |   \
     /    |    \
  VPC-B VPC-C VPC-D
     \    |    /
      \   |   /
       \  |  /
    On-Premises

Mô hình MESH phức tạp: n(n-1)/2 connections
Với 10 VPC = 45 connections!

VỚI TRANSIT GATEWAY:
===================================
    VPC-A   VPC-B   VPC-C   VPC-D
      |       |       |       |
      +-------+-------+-------+
              |
        Transit Gateway
              |
      +-------+-------+
      |               |
  On-Premises    Direct Connect

Mô hình HUB-SPOKE: n+1 connections
Với 10 VPC = 11 connections!
```

### 12.2 Kiến Trúc Transit Gateway

```
┌─────────────────────────────────────────────────┐
│         AWS Transit Gateway (Regional)          │
│                                                 │
│  ┌──────────────────────────────────────────┐  │
│  │    Default Route Table                   │  │
│  │  - Associations                          │  │
│  │  - Propagations                          │  │
│  │  - Static Routes                         │  │
│  └──────────────────────────────────────────┘  │
│                                                 │
│  ┌──────────────────────────────────────────┐  │
│  │    Custom Route Table (Isolated)         │  │
│  │  - Separate routing domains              │  │
│  └──────────────────────────────────────────┘  │
│                                                 │
└─────────────────────────────────────────────────┘
         │        │        │         │
         │        │        │         │
    ┌────┴──┐ ┌──┴────┐ ┌─┴─────┐ ┌┴──────────┐
    │ VPC-A │ │ VPC-B │ │ VPN   │ │ DX Gateway│
    └───────┘ └───────┘ └───────┘ └───────────┘
```

### 12.3 Các Khái Niệm Quan Trọng

#### Attachments (Điểm Gắn Kết)
- **VPC Attachment**: Kết nối VPC với TGW (chọn subnets trong mỗi AZ)
- **VPN Attachment**: Kết nối Site-to-Site VPN
- **Direct Connect Gateway Attachment**: Kết nối DX
- **Peering Attachment**: Kết nối với TGW khác (cross-region, cross-account)

#### Route Tables
- **Association**: VPC được gắn với Route Table nào
- **Propagation**: Routes tự động được học từ attachment
- **Static Routes**: Routes được cấu hình thủ công

### 12.4 Transit Gateway Deep Dive

#### Cấu Hình Network Segmentation

```
SCENARIO: Tách biệt môi trường Production và Development

┌─────────────────────────────────────────┐
│       Transit Gateway                   │
│                                         │
│  ┌────────────────┐  ┌──────────────┐  │
│  │ Prod RT        │  │ Dev RT       │  │
│  │                │  │              │  │
│  │ Associations:  │  │ Associations:│  │
│  │ - Prod VPCs    │  │ - Dev VPCs   │  │
│  │ - VPN (Prod)   │  │ - VPN (Dev)  │  │
│  │                │  │              │  │
│  │ Propagations:  │  │ Propagations:│  │
│  │ - Prod VPCs    │  │ - Dev VPCs   │  │
│  │ - Prod VPN     │  │ - Dev VPN    │  │
│  └────────────────┘  └──────────────┘  │
└─────────────────────────────────────────┘
        │                    │
        │                    │
   ┌────┴─────┐         ┌───┴──────┐
   │ Prod VPCs│         │ Dev VPCs │
   │ Prod VPN │         │ Dev VPN  │
   └──────────┘         └──────────┘

Kết quả: Prod và Dev hoàn toàn isolated!
```

#### Shared Services Architecture

```
SCENARIO: Shared Services (DNS, Monitoring, Security)

┌──────────────────────────────────────────────┐
│         Transit Gateway                      │
│                                              │
│  ┌─────────────────────────────────────┐    │
│  │  Shared Services Route Table        │    │
│  │                                     │    │
│  │  Associations:                      │    │
│  │  - Shared Services VPC              │    │
│  │                                     │    │
│  │  Propagations:                      │    │
│  │  - ALL VPCs (Prod + Dev)           │    │
│  └─────────────────────────────────────┘    │
│                                              │
│  ┌─────────────┐       ┌─────────────┐      │
│  │ Prod RT     │       │ Dev RT      │      │
│  │             │       │             │      │
│  │ Static Route│       │ Static Route│      │
│  │ to Shared   │       │ to Shared   │      │
│  │ Services    │       │ Services    │      │
│  └─────────────┘       └─────────────┘      │
└──────────────────────────────────────────────┘
          │                     │
          │                     │
    ┌─────┴──────┐       ┌─────┴──────┐
    │ Prod VPCs  │       │ Dev VPCs   │
    └────────────┘       └────────────┘
              \             /
               \           /
                \         /
            ┌────┴────────┴────┐
            │ Shared Services  │
            │ - DNS Resolver   │
            │ - Monitoring     │
            │ - Security Tools │
            └──────────────────┘
```

### 12.5 Transit Gateway Peering

```
CROSS-REGION PEERING:

Region: us-east-1              Region: eu-west-1
┌─────────────────┐            ┌─────────────────┐
│   TGW-US        │            │   TGW-EU        │
│                 │  Peering   │                 │
│  VPC-US-1       │◄──────────►│  VPC-EU-1       │
│  VPC-US-2       │            │  VPC-EU-2       │
│                 │            │                 │
└─────────────────┘            └─────────────────┘

Static Routes Required:
- TGW-US: 10.100.0.0/16 → Peering Attachment (EU)
- TGW-EU: 10.0.0.0/16 → Peering Attachment (US)
```

### 12.6 Transit Gateway Configuration Example

#### Tạo Transit Gateway

```bash
aws ec2 create-transit-gateway \
    --description "Hub Transit Gateway" \
    --options \
        AmazonSideAsn=64512,\
        DefaultRouteTableAssociation=enable,\
        DefaultRouteTablePropagation=enable,\
        VpnEcmpSupport=enable,\
        DnsSupport=enable
```

#### Attach VPC

```bash
aws ec2 create-transit-gateway-vpc-attachment \
    --transit-gateway-id tgw-0123456789abcdef0 \
    --vpc-id vpc-0a1b2c3d4e5f6g7h8 \
    --subnet-ids subnet-111 subnet-222 subnet-333 \
    --tag-specifications 'ResourceType=transit-gateway-attachment,Tags=[{Key=Name,Value=Prod-VPC-Attachment}]'
```

#### Tạo Custom Route Table

```bash
aws ec2 create-transit-gateway-route-table \
    --transit-gateway-id tgw-0123456789abcdef0 \
    --tag-specifications 'ResourceType=transit-gateway-route-table,Tags=[{Key=Name,Value=Prod-RT}]'
```

#### Associate Attachment với Route Table

```bash
aws ec2 associate-transit-gateway-route-table \
    --transit-gateway-route-table-id tgw-rtb-0123456789 \
    --transit-gateway-attachment-id tgw-attach-0123456789
```

#### Enable Route Propagation

```bash
aws ec2 enable-transit-gateway-route-table-propagation \
    --transit-gateway-route-table-id tgw-rtb-0123456789 \
    --transit-gateway-attachment-id tgw-attach-0123456789
```

### 12.7 Transit Gateway cho SAP Workloads

```
SAP LANDSCAPE VỚI TRANSIT GATEWAY:

┌─────────────────────────────────────────────┐
│         Transit Gateway                     │
│                                             │
│  ┌────────────┐  ┌────────────┐            │
│  │ SAP Prod RT│  │ SAP Dev RT │            │
│  └────────────┘  └────────────┘            │
└─────────────────────────────────────────────┘
         │                │
    ┌────┴──────┐    ┌───┴──────┐
    │           │    │          │
┌───▼────┐ ┌───▼────┐ ┌────▼────┐
│SAP Prod│ │Shared  │ │SAP Dev  │
│        │ │Services│ │         │
│HANA DB │ │        │ │HANA DB  │
│App Svr │ │Backup  │ │App Svr  │
│        │ │Monitoring│        │
└────────┘ └────────┘ └─────────┘
                │
                │
         ┌──────▼──────┐
         │ On-Premises │
         │ SAP Router  │
         │ SAP GUI     │
         └─────────────┘

Lợi ích:
- Centralized connectivity management
- Network segmentation (Prod/Dev isolated)
- Shared backup và monitoring services
- Single VPN connection từ on-premises
- Simplified routing
```

### 12.8 So Sánh: VPC Peering vs Transit Gateway

| Tiêu Chí | VPC Peering | Transit Gateway |
|----------|-------------|-----------------|
| **Số lượng VPC** | Phù hợp < 10 VPCs | Phù hợp 10+ VPCs |
| **Topology** | Mesh (phức tạp) | Hub-and-spoke (đơn giản) |
| **Transitive Routing** | ❌ Không | ✅ Có |
| **On-premises** | ❌ Không trực tiếp | ✅ Qua VPN/DX |
| **Route Tables** | Trong VPC | Centralized trong TGW |
| **Bandwidth** | Không giới hạn | 50 Gbps per AZ |
| **Cost** | Chỉ data transfer | $0.05/GB + attachment fee |
| **Complexity** | Tăng theo số VPC | Constant |
| **Cross-region** | ✅ Có | ✅ Có (via peering) |

### 12.9 Transit Gateway - Exam Tips

**🎯 Điểm Thi Quan Trọng:**

1. **Khi nào dùng TGW?**
   - Nhiều hơn 5-10 VPCs cần kết nối
   - Cần kết nối on-premises với nhiều VPCs
   - Cần network segmentation phức tạp
   - Cần centralized routing management

2. **Route Table Logic:**
   - **Association** = VPC thuộc Route Table nào
   - **Propagation** = Routes được học tự động
   - **No association** = VPC không thể giao tiếp qua TGW
   - **No propagation** = Phải dùng static routes

3. **Network Segmentation:**
   - Dùng multiple route tables để tách biệt môi trường
   - Prod và Dev cần isolated → 2 route tables riêng
   - Shared Services → Allow từ tất cả môi trường

4. **Limitations:**
   - Regional service (cần peering cho cross-region)
   - Maximum 5000 attachments per TGW
   - Maximum 50 peering connections per TGW
   - Bandwidth: 50 Gbps per AZ, 100 Gbps với ECMP

5. **Cost Optimization:**
   - $0.05 per GB data processing
   - $5 per attachment per month
   - Consolidate VPN connections qua TGW thay vì per-VPC
   - Dùng TGW peering thay vì VPN cho cross-region

---

## 13. Advanced VPC Routing

### 13.1 VPC Routing Fundamentals Review

```
VPC ROUTING ARCHITECTURE:

┌─────────────────────────────────────────┐
│              VPC (10.0.0.0/16)          │
│                                         │
│  ┌──────────────┐   ┌──────────────┐   │
│  │ Public Subnet│   │Private Subnet│   │
│  │ 10.0.1.0/24  │   │ 10.0.2.0/24  │   │
│  │              │   │              │   │
│  │ Route Table: │   │ Route Table: │   │
│  │ 10.0.0.0/16  │   │ 10.0.0.0/16  │   │
│  │ → local      │   │ → local      │   │
│  │ 0.0.0.0/0    │   │ 0.0.0.0/0    │   │
│  │ → IGW        │   │ → NAT-GW     │   │
│  └──────────────┘   └──────────────┘   │
│         │                   │           │
└─────────┼───────────────────┼───────────┘
          │                   │
    ┌─────▼──────┐     ┌─────▼──────┐
    │   IGW      │     │  NAT-GW    │
    └────────────┘     └────────────┘
```

### 13.2 Advanced Routing Scenarios

#### Scenario 1: Traffic Inspection (Firewall Appliance)

```
ARCHITECTURE: Centralized Traffic Inspection

                Internet
                   │
                   │
              ┌────▼────┐
              │   IGW   │
              └────┬────┘
                   │
    ┌──────────────┼──────────────┐
    │         Security VPC         │
    │                              │
    │  ┌────────────────────────┐  │
    │  │  Ingress Route Table   │  │
    │  │  10.0.0.0/8 → Firewall │  │
    │  └────────────────────────┘  │
    │              │                │
    │      ┌───────▼────────┐      │
    │      │    Firewall    │      │
    │      │   (EC2/GWLB)   │      │
    │      └───────┬────────┘      │
    │              │                │
    │  ┌────────────────────────┐  │
    │  │  Egress Route Table    │  │
    │  │  0.0.0.0/0 → IGW       │  │
    │  └────────────────────────┘  │
    └──────────────┬───────────────┘
                   │
         ┌─────────▼─────────┐
         │  Transit Gateway  │
         └─────────┬─────────┘
                   │
        ┌──────────┴──────────┐
        │                     │
   ┌────▼────┐          ┌────▼────┐
   │ App VPC │          │ DB VPC  │
   └─────────┘          └─────────┘

Traffic Flow:
1. Internet → IGW
2. IGW → Firewall (via Ingress RT)
3. Firewall inspect → Pass to TGW
4. TGW → App VPC
5. Response: App VPC → TGW → Firewall → IGW → Internet
```

#### Scenario 2: Egress VPC Pattern

```
EGRESS VPC: Centralized Internet Access

┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│  Prod VPC   │  │  Dev VPC    │  │  Test VPC   │
│             │  │             │  │             │
│ Private     │  │ Private     │  │ Private     │
│ Subnets     │  │ Subnets     │  │ Subnets     │
└──────┬──────┘  └──────┬──────┘  └──────┬──────┘
       │                │                │
       │   Route: 0.0.0.0/0 → TGW       │
       │                │                │
       └────────────────┼────────────────┘
                        │
                 ┌──────▼──────┐
                 │    TGW      │
                 └──────┬──────┘
                        │
                        │ TGW Route: 0.0.0.0/0 → Egress VPC
                        │
                 ┌──────▼──────────────────┐
                 │     Egress VPC          │
                 │                         │
                 │  ┌──────┐   ┌──────┐   │
                 │  │NAT-GW│   │NAT-GW│   │
                 │  │ AZ-A │   │ AZ-B │   │
                 │  └───┬──┘   └───┬──┘   │
                 │      │          │      │
                 │      └────┬─────┘      │
                 │           │            │
                 │      ┌────▼────┐       │
                 │      │   IGW   │       │
                 └──────┴─────────┴───────┘
                          │
                          │
                      Internet

Benefits:
- Centralized NAT management
- Single egress point for monitoring
- Shared NAT Gateway costs
- Centralized allow/deny lists
```

#### Scenario 3: Multi-Region Active-Active SAP

```
ACTIVE-ACTIVE SAP ACROSS REGIONS:

Region: us-east-1                Region: eu-west-1
┌──────────────────┐            ┌──────────────────┐
│  SAP VPC (US)    │            │  SAP VPC (EU)    │
│                  │            │                  │
│  HANA Primary    │  ◄────────►│  HANA Secondary  │
│  (Read/Write)    │   HSR Sync │  (Read Replica)  │
│                  │            │                  │
│  App Servers     │            │  App Servers     │
│  (Active)        │            │  (Active)        │
└────────┬─────────┘            └─────────┬────────┘
         │                                │
    ┌────▼────┐                      ┌────▼────┐
    │ TGW-US  │                      │ TGW-EU  │
    └────┬────┘                      └────┬────┘
         │                                │
         │       TGW Peering              │
         └────────────────────────────────┘
                        │
                 ┌──────▼──────┐
                 │   Route 53  │
                 │  Geo-routing│
                 └─────────────┘
                        │
              ┌─────────┴─────────┐
              │                   │
         US Users              EU Users
       (→ us-east-1)         (→ eu-west-1)

Routing Configuration:
- Each region serves local users
- HANA System Replication across regions
- TGW Peering for backend communication
- Route 53 geo-routing for user traffic
```

### 13.3 Gateway Route Tables (Advanced)

```
GATEWAY ROUTE TABLE: Control traffic AT the IGW

Normal Flow (Without Gateway RT):
Internet → IGW → VPC Route Table → Instance

With Gateway RT:
Internet → IGW → Gateway RT → Firewall → VPC RT → Instance

┌─────────────────────────────────────────┐
│              VPC                        │
│                                         │
│  ┌────────────────────────────────┐    │
│  │  Gateway Route Table           │    │
│  │  (attached to IGW)              │    │
│  │                                 │    │
│  │  10.0.1.0/24 → Firewall ENI    │    │
│  │  10.0.2.0/24 → Firewall ENI    │    │
│  └────────────────────────────────┘    │
│                                         │
│  ┌──────────────┐   ┌──────────────┐   │
│  │ Public Sub   │   │ Firewall Sub │   │
│  │ 10.0.1.0/24  │   │ 10.0.0.0/28  │   │
│  │              │   │              │   │
│  │ EC2 Instance │◄──┤  Firewall    │   │
│  │              │   │  Appliance   │   │
│  └──────────────┘   └──────────────┘   │
└─────────────────────────────────────────┘

Use Case:
- Force ALL traffic qua firewall appliance
- Traffic inspection for inbound connections
- IDS/IPS integration
```

### 13.4 Route Priority

**Route Selection Logic:**

```
Route Priority (Most specific first):

1. Most specific prefix match (longest prefix)
   /32 > /24 > /16 > /8 > /0

Example Routes:
- 10.0.1.5/32     → ENI (most specific)
- 10.0.1.0/24     → NAT Gateway
- 10.0.0.0/16     → VPC local
- 0.0.0.0/0       → Internet Gateway (least specific)

Query: Where does 10.0.1.5 go?
Answer: ENI (10.0.1.5/32 is most specific)

Query: Where does 10.0.1.100 go?
Answer: NAT Gateway (10.0.1.0/24 is most specific match)

Query: Where does 10.0.2.50 go?
Answer: VPC local (10.0.0.0/16 is most specific match)

Query: Where does 8.8.8.8 go?
Answer: Internet Gateway (0.0.0.0/0 matches everything else)
```

### 13.5 Routing với VPC Endpoints

```
ROUTING TO VPC ENDPOINTS:

Scenario: S3 Access từ Private Subnet

WITHOUT VPC Endpoint:
Private Subnet → NAT Gateway → IGW → Internet → S3
Cost: NAT Gateway processing + Data Transfer

WITH Gateway VPC Endpoint:
Private Subnet → S3 Gateway Endpoint → S3
Cost: FREE (no data transfer charges)

Route Table Configuration:
┌─────────────────────────────────┐
│  Private Subnet Route Table     │
│                                 │
│  Destination      Target        │
│  10.0.0.0/16      local         │
│  0.0.0.0/0        nat-gateway   │
│  pl-xxxxx (S3)    vpce-gateway  │ ← Prefix List
└─────────────────────────────────┘

Prefix List = Managed list of S3 IP ranges

Route Priority:
- S3 prefix list (/16 or /17) more specific than 0.0.0.0/0
- S3 traffic → VPC Endpoint (FREE)
- Other traffic → NAT Gateway (CHARGED)
```

### 13.6 Middlebox Routing Architecture

```
MIDDLEBOX ROUTING PATTERN:

┌──────────────────────────────────────────┐
│             Inspection VPC               │
│                                          │
│  ┌────────┐         ┌────────┐          │
│  │ Public │         │ Private│          │
│  │ Subnet │         │ Subnet │          │
│  │        │         │        │          │
│  │  GWLB  │────────►│Firewall│          │
│  │Endpoint│         │Instance│          │
│  │        │         │  (ASG) │          │
│  └────────┘         └────────┘          │
└──────────────────────────────────────────┘
      ▲                       │
      │                       │
      │   GENEVE Protocol     │
      │   (encapsulation)     │
      │                       │
┌─────┴───────────────────────▼──────┐
│          Gateway Load Balancer      │
│  - Transparent (Layer 3)            │
│  - Stateful                         │
│  - Auto-scaling firewall instances  │
└─────────────────────────────────────┘
              │
    ┌─────────┼─────────┐
    │                   │
┌───▼────┐        ┌────▼───┐
│App VPC │        │DB VPC  │
└────────┘        └────────┘

Traffic Flow:
1. Traffic enters GWLB
2. GWLB sends to firewall via GWLB Endpoint
3. Firewall inspects and returns decision
4. GWLB forwards to destination
5. Return traffic follows same path
```

### 13.7 Advanced Routing - Configuration Examples

#### Tạo Gateway Route Table

```bash
# Tạo Gateway Route Table
aws ec2 create-route-table \
    --vpc-id vpc-12345678 \
    --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=Gateway-RT}]'

# Associate với Internet Gateway
aws ec2 associate-route-table \
    --route-table-id rtb-abcd1234 \
    --gateway-id igw-12345678

# Thêm route đến firewall
aws ec2 create-route \
    --route-table-id rtb-abcd1234 \
    --destination-cidr-block 10.0.1.0/24 \
    --network-interface-id eni-firewall123
```

#### Route Propagation từ Transit Gateway

```bash
# Enable route propagation
aws ec2 enable-transit-gateway-route-table-propagation \
    --transit-gateway-route-table-id tgw-rtb-123 \
    --transit-gateway-attachment-id tgw-attach-456

# Add static route
aws ec2 create-transit-gateway-route \
    --destination-cidr-block 0.0.0.0/0 \
    --transit-gateway-route-table-id tgw-rtb-123 \
    --transit-gateway-attachment-id tgw-attach-egress-vpc
```

### 13.8 Advanced Routing - Exam Tips

**🎯 Điểm Thi Quan Trọng:**

1. **Route Priority:**
   - Most specific prefix wins (longest prefix match)
   - Local routes always have priority
   - Static routes > Propagated routes (trong TGW)

2. **Gateway Route Tables:**
   - Áp dụng routing TRƯỚC KHI traffic vào VPC
   - Use case: Force traffic qua inspection appliance
   - Chỉ có thể attach vào IGW hoặc Virtual Private Gateway

3. **Middlebox Patterns:**
   - Gateway Load Balancer: Modern approach (bump-in-the-wire)
   - Transit Gateway + Inspection VPC: Traditional approach
   - Dùng GWLB khi cần transparent inspection + auto-scaling

4. **Egress VPC:**
   - Centralized NAT for multiple VPCs
   - Cost optimization: Shared NAT Gateways
   - Simplified security: Single egress point

5. **VPC Endpoint Routing:**
   - Gateway Endpoints (S3, DynamoDB): Sử dụng prefix lists
   - Interface Endpoints: Sử dụng ENI private IPs
   - Priority: Prefix list routes > 0.0.0.0/0

6. **Asymmetric Routing:**
   - Problem: Request và response đi different paths
   - Solution: Ensure stateful firewalls see both directions
   - Use GWLB hoặc TGW appliance mode

---

## 14. Accelerated Site-to-Site VPN

### 14.1 Standard Site-to-Site VPN Review

```
STANDARD SITE-TO-SITE VPN:

On-Premises                            AWS
┌──────────────┐                  ┌──────────────┐
│              │                  │     VPC      │
│  Customer    │    Internet      │              │
│  Gateway     │◄────────────────►│   Virtual    │
│  (CGW)       │  IPSec Tunnels   │   Private    │
│              │                  │   Gateway    │
│              │                  │   (VGW)      │
└──────────────┘                  └──────────────┘

Characteristics:
- Travels over public internet
- Variable latency and jitter
- Up to 1.25 Gbps per tunnel
- 2 tunnels per VPN connection (HA)
- Affected by internet routing
```

### 14.2 Accelerated VPN Architecture

```
ACCELERATED SITE-TO-SITE VPN:

On-Premises                AWS Global Network              AWS Region
┌──────────┐          ┌────────────────────┐          ┌──────────────┐
│          │          │                    │          │     VPC      │
│ Customer │ Internet │  AWS Edge Location │ AWS      │              │
│ Gateway  ├─────────►│                    │ Backbone │   Virtual    │
│  (CGW)   │          │  Global Accelerator│◄────────►│   Private    │
│          │          │  Endpoint (AnyIP)  │          │   Gateway    │
│          │          │                    │          │   (VGW)      │
└──────────┘          └────────────────────┘          └──────────────┘
     ▲                                                        │
     │                                                        │
     └────────────────────────────────────────────────────────┘
                    IPSec Tunnels (over AWS Network)

Benefits:
✅ Lower latency (AWS global network)
✅ More consistent performance
✅ Better throughput
✅ Automatic failover to healthy endpoints
✅ DDoS protection (Shield Standard)
```

### 14.3 Accelerated VPN vs Standard VPN

| Tiêu Chí | Standard VPN | Accelerated VPN |
|----------|--------------|-----------------|
| **Network Path** | Public Internet | AWS Global Network |
| **Latency** | Variable (ISP dependent) | Consistent (AWS optimized) |
| **Jitter** | High (internet routing) | Low (AWS backbone) |
| **Performance** | Affected by internet | Optimized by AWS |
| **Availability** | 2 tunnels (standard) | 2 tunnels + AWS GA failover |
| **Cost** | Standard VPN charges | VPN + $0.05/GB accelerated |
| **Setup** | Simple | Requires Global Accelerator |
| **Use Case** | Standard connectivity | Mission-critical workloads |
| **DDoS Protection** | ❌ | ✅ Shield Standard |

### 14.4 Accelerated VPN Deep Dive

#### Traffic Flow

```
TRAFFIC FLOW ANALYSIS:

1. CONNECTION ESTABLISHMENT:
   ┌──────────────┐
   │ On-Premises  │
   └──────┬───────┘
          │ BGP Announcement
          │ Finds nearest AWS Edge
          ▼
   ┌──────────────┐
   │  Edge POP    │ ← Anycast IP (closest to customer)
   │  (Global     │
   │  Accelerator)│
   └──────┬───────┘
          │ AWS Private Backbone
          │ (Optimized routing)
          ▼
   ┌──────────────┐
   │  AWS Region  │
   │  VGW         │
   └──────────────┘

2. PACKET JOURNEY:
   On-Prem → Local ISP → Nearest AWS Edge (via Anycast)
           → AWS Backbone (private fiber)
           → Target AWS Region
           → VGW → VPC

3. FAILOVER SCENARIO:
   If Edge-1 fails:
   - Global Accelerator detects failure (< 30s)
   - Traffic automatically routed to Edge-2
   - No customer action required
```

#### Khi Nào Nên Dùng Accelerated VPN?

```
DECISION TREE:

Có cần VPN không?
├─ KHÔNG → Dùng Direct Connect hoặc Internet
└─ CÓ
   │
   Workload có yêu cầu cao về performance?
   ├─ KHÔNG → Standard VPN (cheaper)
   └─ CÓ
      │
      Budget cho $0.05/GB?
      ├─ KHÔNG → Standard VPN + optimize routing
      └─ CÓ
         │
         Có multi-region hoặc global traffic?
         ├─ KHÔNG → Standard VPN có thể đủ
         └─ CÓ → ✅ ACCELERATED VPN

Use Cases for Accelerated VPN:
✅ SAP production workloads (HANA replication)
✅ Real-time trading applications
✅ VoIP/Video conferencing
✅ Large file transfers (backups)
✅ Multi-region architectures
✅ Customers with poor internet routing
```

### 14.5 Configuration

#### Create Accelerated VPN Connection

```bash
# Create Customer Gateway
aws ec2 create-customer-gateway \
    --type ipsec.1 \
    --public-ip 203.0.113.5 \
    --bgp-asn 65000 \
    --tag-specifications 'ResourceType=customer-gateway,Tags=[{Key=Name,Value=OnPrem-CGW}]'

# Create VPN Connection with Acceleration
aws ec2 create-vpn-connection \
    --type ipsec.1 \
    --customer-gateway-id cgw-123456 \
    --vpn-gateway-id vgw-789012 \
    --options '{
        "StaticRoutesOnly": false,
        "EnableAcceleration": true,
        "TunnelOptions": [
            {
                "PreSharedKey": "your-preshared-key-1",
                "TunnelInsideCidr": "169.254.10.0/30"
            },
            {
                "PreSharedKey": "your-preshared-key-2",
                "TunnelInsideCidr": "169.254.10.4/30"
            }
        ]
    }' \
    --tag-specifications 'ResourceType=vpn-connection,Tags=[{Key=Name,Value=Accelerated-VPN}]'
```

#### Verify Accelerated Status

```bash
# Check VPN connection details
aws ec2 describe-vpn-connections \
    --vpn-connection-ids vpn-abc123 \
    --query 'VpnConnections[0].{
        AccelerationEnabled: Options.EnableAcceleration,
        State: State,
        Tunnel1: VgwTelemetry[0].Status,
        Tunnel2: VgwTelemetry[1].Status,
        AcceleratorIP1: VgwTelemetry[0].OutsideIpAddress,
        AcceleratorIP2: VgwTelemetry[1].OutsideIpAddress
    }'
```

### 14.6 Monitoring and Troubleshooting

#### CloudWatch Metrics

```bash
# Monitor VPN tunnel status
aws cloudwatch get-metric-statistics \
    --namespace AWS/VPN \
    --metric-name TunnelState \
    --dimensions Name=VpnId,Value=vpn-abc123 \
    --start-time 2026-02-08T00:00:00Z \
    --end-time 2026-02-09T00:00:00Z \
    --period 300 \
    --statistics Average

# Monitor throughput
aws cloudwatch get-metric-statistics \
    --namespace AWS/VPN \
    --metric-name TunnelDataIn \
    --dimensions Name=VpnId,Value=vpn-abc123 \
    --start-time 2026-02-08T00:00:00Z \
    --end-time 2026-02-09T00:00:00Z \
    --period 300 \
    --statistics Sum
```

#### Common Issues

```
TROUBLESHOOTING GUIDE:

1. TUNNEL DOWN:
   Symptoms: Both tunnels showing DOWN
   Check:
   - CGW configuration (IPSec parameters)
   - Pre-shared keys match
   - Security Group allows UDP 500, 4500
   - NACL allows traffic
   - BGP ASN configuration

2. POOR PERFORMANCE:
   Symptoms: High latency despite acceleration
   Check:
   - VPN is actually accelerated (EnableAcceleration=true)
   - Edge location being used (traceroute)
   - MTU size (set to 1400 for VPN)
   - TCP window scaling
   - Application-layer issues

3. ASYMMETRIC ROUTING:
   Symptoms: Traffic works one way only
   Check:
   - Route propagation enabled
   - Both tunnels UP
   - BGP routes advertised correctly
   - No more specific routes conflicting

4. HIGH COST:
   Symptoms: Unexpected charges
   Check:
   - Accelerated VPN charges ($0.05/GB)
   - Data transfer OUT charges
   - Consider Direct Connect for high volume
```

### 14.7 Accelerated VPN cho SAP

```
SAP USE CASE: HANA System Replication Over VPN

On-Premises DC              AWS Region (DR)
┌────────────────┐         ┌────────────────┐
│                │         │                │
│  SAP HANA      │ HSR     │  SAP HANA      │
│  Primary       │◄───────►│  Secondary     │
│                │ Sync    │                │
│  (Production)  │         │  (Standby)     │
└───────┬────────┘         └───────┬────────┘
        │                          │
   ┌────▼──────┐              ┌────▼──────┐
   │    CGW    │              │    VGW    │
   └────┬──────┘              └────┬──────┘
        │                          │
        │    Accelerated VPN       │
        └──────────────────────────┘

Requirements:
- Low latency: < 1ms for SYNC replication
- High bandwidth: 100+ Mbps sustained
- Consistency: No packet loss
- Security: Encrypted connection

Why Accelerated VPN?
✅ Consistent low latency (AWS backbone)
✅ Better for HANA replication
✅ DDoS protection
✅ Automatic failover
✅ More predictable than internet

Alternative: Direct Connect
- Lower latency (< 10ms)
- Higher bandwidth (1-100 Gbps)
- More expensive
- Longer setup time

Decision:
- SYNC mode → Direct Connect preferred
- ASYNC mode → Accelerated VPN acceptable
- Hybrid → DX + Accelerated VPN (backup)
```

### 14.8 Cost Analysis

```
COST COMPARISON:

Scenario: 1 TB/month data transfer over VPN

STANDARD VPN:
- VPN Connection: $0.05/hour = $36/month
- Data OUT: $0.09/GB × 1000 = $90/month
- Total: $126/month

ACCELERATED VPN:
- VPN Connection: $0.05/hour = $36/month
- Acceleration: $0.05/GB × 1000 = $50/month
- Data OUT: $0.09/GB × 1000 = $90/month
- Total: $176/month

Additional Cost: $50/month (40% increase)

DIRECT CONNECT (for comparison):
- Port Fee: $0.30/hour (1 Gbps) = $216/month
- Data OUT: $0.02/GB × 1000 = $20/month
- Total: $236/month

BREAK-EVEN ANALYSIS:
- < 500 GB/month → Standard VPN
- 500 GB - 5 TB/month → Accelerated VPN
- > 5 TB/month → Direct Connect

Additional factors:
- Performance requirements
- Criticality of workload
- SLA requirements
```

### 14.9 Accelerated VPN - Exam Tips

**🎯 Điểm Thi Quan Trọng:**

1. **Key Differentiators:**
   - Uses AWS Global Accelerator
   - Traffic goes over AWS backbone (not public internet)
   - Anycast IP addresses (automatic edge selection)
   - Lower, more consistent latency

2. **When to Recommend:**
   - Mission-critical workloads
   - Real-time applications (SAP, trading)
   - Poor internet performance
   - Multi-region architectures
   - Budget allows $0.05/GB

3. **Limitations:**
   - Chỉ available với VGW (not Transit Gateway)
   - Additional $0.05/GB cost
   - Still limited to VPN bandwidth (1.25 Gbps/tunnel)
   - Not a replacement for Direct Connect

4. **Architecture Patterns:**
   - Primary: Direct Connect
   - Backup: Accelerated VPN (better than standard VPN)
   - Use with Transit Gateway: Create VPN to VGW, then VGW to TGW

5. **Monitoring:**
   - Same CloudWatch metrics as standard VPN
   - Additional Global Accelerator metrics
   - Monitor: TunnelState, TunnelDataIn/Out, AcceleratorActiveFlows

---

## 15. AWS Client VPN

### 15.1 Client VPN Overview

**AWS Client VPN** là managed client-based VPN service cho phép users kết nối an toàn vào AWS resources và on-premises networks.

```
CLIENT VPN ARCHITECTURE:

Remote Users                  AWS Region
┌──────────────┐         ┌────────────────────┐
│              │         │       VPC          │
│  Laptop with │         │                    │
│  VPN Client  │ OpenVPN │ ┌────────────────┐ │
│  (OpenVPN)   ├────────►│ │ Client VPN     │ │
│              │         │ │ Endpoint       │ │
└──────────────┘         │ │ (ENI in subnet)│ │
                         │ └────────┬───────┘ │
┌──────────────┐         │          │         │
│              │         │    ┌─────▼──────┐  │
│  Mobile with │ OpenVPN │    │  Private   │  │
│  VPN Client  ├────────►│    │  Resources │  │
│              │         │    │  (EC2, RDS)│  │
└──────────────┘         │    └────────────┘  │
                         │                    │
                         └────────────────────┘

Protocol: OpenVPN (industry standard)
Authentication: Active Directory, Certificate-based, SAML 2.0
Encryption: TLS 1.2+
```

### 15.2 Components

#### Client VPN Endpoint
```
CLIENT VPN ENDPOINT:

┌─────────────────────────────────────────┐
│      Client VPN Endpoint                │
│                                         │
│  Components:                            │
│  - Target Network (VPC + Subnets)      │
│  - Client IPv4 CIDR (pool)             │
│  - Server Certificate (TLS)            │
│  - Authentication (AD/Cert/SAML)       │
│  - Authorization Rules                  │
│  - Route Table                          │
│  - Security Groups                      │
│  - Connection Logging (CloudWatch)     │
│                                         │
│  Configuration:                         │
│  - Split-tunnel: ✅/❌                  │
│  - DNS Servers                          │
│  - Self-service portal: ✅/❌           │
│  - Session timeout                      │
└─────────────────────────────────────────┘
```

#### Target Networks
```
TARGET NETWORK ASSOCIATION:

┌────────────────────────────────────┐
│          VPC (10.0.0.0/16)         │
│                                    │
│  ┌──────────────┐ ┌─────────────┐ │
│  │ Subnet A     │ │ Subnet B    │ │
│  │ us-east-1a   │ │ us-east-1b  │ │
│  │              │ │             │ │
│  │ ENI (VPN EP) │ │ ENI (VPN EP)│ │ ← HA across AZs
│  └──────────────┘ └─────────────┘ │
│                                    │
└────────────────────────────────────┘

Best Practice:
- Associate với >= 2 subnets (multi-AZ)
- Use dedicated subnets for Client VPN
- Apply Security Groups to ENIs
```

### 15.3 Authentication Methods

#### 1. Mutual Certificate Authentication

```
CERTIFICATE-BASED AUTHENTICATION:

Setup:
1. Create CA (Certificate Authority)
2. Generate server certificate
3. Generate client certificates (per user)
4. Upload server cert to AWS ACM
5. Distribute client certs to users

┌──────────────┐                    ┌──────────────┐
│    Client    │                    │  Client VPN  │
│              │                    │  Endpoint    │
│ Client Cert  │  TLS Handshake     │ Server Cert  │
│ (unique)     ├───────────────────►│ (validates   │
│              │  Validates certs   │  client)     │
└──────────────┘                    └──────────────┘

Pros:
✅ No additional infrastructure
✅ Simple for small teams
✅ Per-user certificates

Cons:
❌ Certificate management overhead
❌ Revocation complexity
❌ No centralized user management
```

#### 2. Active Directory Authentication

```
ACTIVE DIRECTORY AUTHENTICATION:

┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│    Client    │     │  Client VPN  │     │  Directory   │
│              │     │  Endpoint    │     │  Service     │
│ Username:    │  1  │              │  2  │  (Managed    │
│ Password:    ├────►│  Forwards    ├────►│   Microsoft  │
│              │     │  credentials │     │   AD)        │
│              │  3  │              │     │              │
│              │◄────┤  Allow/Deny  │     │  Validates   │
└──────────────┘     └──────────────┘     └──────────────┘

Setup:
- Create AWS Directory Service (Managed Microsoft AD)
- Or connect to on-premises AD
- Configure Client VPN to use directory

Pros:
✅ Centralized user management
✅ Existing AD infrastructure
✅ MFA support (with RADIUS)
✅ Group-based authorization

Cons:
❌ Additional cost (Directory Service)
❌ More complex setup
❌ Requires AD infrastructure
```

#### 3. SAML 2.0 Federation

```
SAML 2.0 AUTHENTICATION (SSO):

┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│  Client  │   │ Client   │   │   IdP    │   │   AWS    │
│          │   │   VPN    │   │ (Okta/   │   │  Client  │
│ 1.Connect│──►│ 2.Redirect──►│  Azure)  │   │   VPN    │
│          │   │   to IdP │   │          │   │          │
│          │   │          │   │ 3.User   │   │          │
│          │   │          │   │  Login   │   │          │
│          │   │ 4.SAML   │◄──│ 5.SAML   │   │          │
│          │   │  Response│   │ Assertion│   │          │
│ 6.Connect│──►│ 7.Verify ├──────────────────►│ 8.Validate│
│   VPN    │   │   SAML   │   │          │   │   Token  │
└──────────┘   └──────────┘   └──────────┘   └──────────┘

Supported IdPs:
- Okta
- Azure AD
- Google Workspace
- Auth0
- Any SAML 2.0 compliant IdP

Pros:
✅ Single Sign-On (SSO)
✅ MFA support
✅ No password exposure
✅ Enterprise-ready

Cons:
❌ Complex setup
❌ Requires IdP infrastructure/subscription
❌ Additional cost
```

### 15.4 Authorization Rules

```
AUTHORIZATION RULES:

┌─────────────────────────────────────────┐
│       Authorization Rules               │
│                                         │
│  Rule 1:                                │
│  - Group: "Developers"                  │
│  - Network: 10.0.1.0/24 (App subnet)    │
│  - Access: ALLOW                        │
│                                         │
│  Rule 2:                                │
│  - Group: "DBAs"                        │
│  - Network: 10.0.2.0/24 (DB subnet)     │
│  - Access: ALLOW                        │
│                                         │
│  Rule 3:                                │
│  - Group: "All Users"                   │
│  - Network: 10.0.0.0/16 (entire VPC)    │
│  - Access: DENY                         │
│                                         │
│  Evaluation: First match wins           │
└─────────────────────────────────────────┘

Example CLI:
aws ec2 authorize-client-vpn-ingress \
    --client-vpn-endpoint-id cvpn-endpoint-123 \
    --target-network-cidr 10.0.1.0/24 \
    --authorize-all-groups \
    --description "Allow all users to app subnet"

With AD Groups:
aws ec2 authorize-client-vpn-ingress \
    --client-vpn-endpoint-id cvpn-endpoint-123 \
    --target-network-cidr 10.0.2.0/24 \
    --access-group-id "S-1-5-21-..." \
    --description "Allow DBA group to DB subnet"
```

### 15.5 Routing Configuration

#### Split Tunnel vs Full Tunnel

```
SPLIT TUNNEL (recommended):
┌──────────────────────────────────────┐
│         Remote User                  │
│                                      │
│  Corporate Traffic (10.0.0.0/16)    │
│         │                            │
│         ├─► VPN Tunnel ─► AWS       │
│         │                            │
│  Internet Traffic (0.0.0.0/0)       │
│         │                            │
│         └─► Local ISP ─► Internet   │
└──────────────────────────────────────┘

Benefits:
✅ Lower VPN bandwidth usage
✅ Better user experience (direct internet)
✅ Lower AWS costs (less data transfer)

Configuration:
- Enable split-tunnel = true
- Only corporate CIDRs go through VPN


FULL TUNNEL:
┌──────────────────────────────────────┐
│         Remote User                  │
│                                      │
│  ALL Traffic (0.0.0.0/0)             │
│         │                            │
│         └─► VPN Tunnel ─► AWS       │
│                     │                │
│                     └─► Internet     │
└──────────────────────────────────────┘

Benefits:
✅ Complete traffic visibility
✅ All traffic inspected
✅ Better security control

Drawbacks:
❌ Higher VPN bandwidth
❌ Higher latency for internet
❌ Higher AWS costs
```

#### Route Table Example

```bash
# Add route to VPN endpoint
aws ec2 create-client-vpn-route \
    --client-vpn-endpoint-id cvpn-endpoint-123 \
    --destination-cidr-block 10.0.0.0/16 \
    --target-vpc-subnet-id subnet-abc123 \
    --description "Route to VPC"

# Add route to on-premises (via TGW)
aws ec2 create-client-vpn-route \
    --client-vpn-endpoint-id cvpn-endpoint-123 \
    --destination-cidr-block 192.168.0.0/16 \
    --target-vpc-subnet-id subnet-abc123 \
    --description "Route to on-premises"

# List routes
aws ec2 describe-client-vpn-routes \
    --client-vpn-endpoint-id cvpn-endpoint-123
```

### 15.6 Client VPN với Transit Gateway

```
CLIENT VPN + TRANSIT GATEWAY:
Access to Multiple VPCs and On-Premises

        Remote Users
             │
             │ OpenVPN
             ▼
    ┌─────────────────┐
    │  Client VPN EP  │
    │  (in VPC-1)     │
    └────────┬────────┘
             │
             │ VPC Routing
             ▼
    ┌─────────────────┐
    │ Transit Gateway │
    └────────┬────────┘
             │
      ┌──────┼──────┬──────────┐
      │      │      │          │
   ┌──▼──┐ ┌─▼──┐ ┌▼───┐ ┌────▼──────┐
   │VPC-2│ │VPC3│ │VPC4│ │On-Premises│
   └─────┘ └────┘ └────┘ └───────────┘

Configuration Steps:
1. Create Client VPN Endpoint in VPC-1
2. Attach VPC-1 to Transit Gateway
3. Add route in VPC-1 subnet: 0.0.0.0/0 → TGW
4. Add route in Client VPN: 10.0.0.0/8 → subnet (TGW subnet)
5. Configure TGW route tables for VPC-2, VPC-3, VPC-4
6. Users can now access all connected networks
```

### 15.7 Complete Setup Example

```bash
# STEP 1: Create certificates (if using certificate auth)
# Generate CA
openssl genrsa -out ca-key.pem 2048
openssl req -new -x509 -days 3650 -key ca-key.pem -out ca-cert.pem

# Generate server certificate
openssl genrsa -out server-key.pem 2048
openssl req -new -key server-key.pem -out server.csr
openssl x509 -req -days 3650 -in server.csr -CA ca-cert.pem \
    -CAkey ca-key.pem -CAcreateserial -out server-cert.pem

# Import to ACM
aws acm import-certificate \
    --certificate fileb://server-cert.pem \
    --private-key fileb://server-key.pem \
    --certificate-chain fileb://ca-cert.pem

# STEP 2: Create Client VPN Endpoint
aws ec2 create-client-vpn-endpoint \
    --client-cidr-block 172.16.0.0/22 \
    --server-certificate-arn arn:aws:acm:us-east-1:123456:certificate/abc123 \
    --authentication-options Type=certificate-authentication,MutualAuthentication={ClientRootCertificateChainArn=arn:aws:acm:us-east-1:123456:certificate/def456} \
    --connection-log-options Enabled=true,CloudwatchLogGroup=client-vpn-logs \
    --split-tunnel \
    --vpc-id vpc-12345678 \
    --security-group-ids sg-abcdef \
    --tag-specifications 'ResourceType=client-vpn-endpoint,Tags=[{Key=Name,Value=CompanyVPN}]'

# STEP 3: Associate with subnets (multi-AZ)
aws ec2 associate-client-vpn-target-network \
    --client-vpn-endpoint-id cvpn-endpoint-123 \
    --subnet-id subnet-111 # AZ-A

aws ec2 associate-client-vpn-target-network \
    --client-vpn-endpoint-id cvpn-endpoint-123 \
    --subnet-id subnet-222 # AZ-B

# STEP 4: Add authorization rule
aws ec2 authorize-client-vpn-ingress \
    --client-vpn-endpoint-id cvpn-endpoint-123 \
    --target-network-cidr 10.0.0.0/16 \
    --authorize-all-groups \
    --description "Allow access to VPC"

# STEP 5: Add routes
aws ec2 create-client-vpn-route \
    --client-vpn-endpoint-id cvpn-endpoint-123 \
    --destination-cidr-block 10.0.0.0/16 \
    --target-vpc-subnet-id subnet-111

# STEP 6: Download client configuration
aws ec2 export-client-vpn-client-configuration \
    --client-vpn-endpoint-id cvpn-endpoint-123 \
    --output text > client-config.ovpn
```

### 15.8 Client Configuration

```
CLIENT CONFIGURATION FILE (.ovpn):

client
dev tun
proto udp
remote cvpn-endpoint-abc123.prod.clientvpn.us-east-1.amazonaws.com 443
remote-random-hostname
resolv-retry infinite
nobind
persist-key
persist-tun
remote-cert-tls server
cipher AES-256-GCM
verb 3

<ca>
-----BEGIN CERTIFICATE-----
[CA certificate content]
-----END CERTIFICATE-----
</ca>

<cert>
-----BEGIN CERTIFICATE-----
[Client certificate content]
-----END CERTIFICATE-----
</cert>

<key>
-----BEGIN PRIVATE KEY-----
[Client private key content]
-----END PRIVATE KEY-----
</key>

# Optional: Custom DNS
dhcp-option DNS 10.0.0.2

# Optional: Custom routes
route 10.0.0.0 255.255.0.0
```

### 15.9 Monitoring and Logging

```bash
# Enable connection logging
aws ec2 modify-client-vpn-endpoint \
    --client-vpn-endpoint-id cvpn-endpoint-123 \
    --connection-log-options Enabled=true,CloudwatchLogGroup=/aws/clientvpn,CloudwatchLogStream=connections

# CloudWatch Logs format:
{
  "connection-id": "cvpn-connection-abc123",
  "client-ip": "172.16.0.5",
  "username": "user@company.com",
  "connection-start-time": "2026-02-09T10:00:00Z",
  "connection-end-time": "2026-02-09T18:00:00Z",
  "ingress-bytes": "1048576000",
  "egress-bytes": "524288000",
  "connection-duration-seconds": "28800"
}

# CloudWatch Metrics:
- ActiveConnectionsCount
- AuthenticationFailures
- IngressBytes
- EgressBytes
- IngressPackets
- EgressPackets
```

### 15.10 Best Practices

```
SECURITY BEST PRACTICES:

1. Authentication:
   ✅ Use AD or SAML for large deployments
   ✅ Enable MFA when possible
   ✅ Rotate certificates regularly
   ✅ Use strong password policies

2. Network Security:
   ✅ Use dedicated subnets for VPN ENIs
   ✅ Apply restrictive Security Groups
   ✅ Use authorization rules (least privilege)
   ✅ Enable split-tunnel (reduce attack surface)

3. Monitoring:
   ✅ Enable connection logging
   ✅ Set CloudWatch alarms for:
      - High connection failures
      - Unusual bandwidth usage
      - Connection spikes
   ✅ Regular audit of access logs

4. High Availability:
   ✅ Associate with >= 2 subnets (multi-AZ)
   ✅ Use Auto Scaling for capacity
   ✅ Test failover scenarios

5. Cost Optimization:
   ✅ Use split-tunnel (reduce data transfer)
   ✅ Right-size endpoint capacity
   ✅ Monitor per-user bandwidth usage
   ✅ Implement session timeouts
```

### 15.11 Client VPN vs Other Solutions

| Tiêu Chí | Client VPN | Site-to-Site VPN | Direct Connect |
|----------|------------|------------------|----------------|
| **Use Case** | Remote users | Office networks | Dedicated connection |
| **Protocol** | OpenVPN (TLS) | IPSec | Private fiber |
| **Scale** | Thousands of users | Limited by bandwidth | High throughput |
| **Setup Time** | Minutes | Hours | Weeks/Months |
| **Cost** | $0.10/endpoint-hour + $0.05/connection-hour | $0.05/hour + data | Port fee + data |
| **Authentication** | AD/Cert/SAML | Pre-shared key/cert | Physical connection |
| **Bandwidth** | Varies (internet) | Up to 1.25 Gbps/tunnel | 1-100 Gbps |
| **Mobility** | ✅ High (anywhere) | ❌ Fixed location | ❌ Fixed location |

### 15.12 Client VPN cho SAP Access

```
SAP ACCESS SCENARIO:

┌────────────────────────────────────────┐
│        Remote SAP Users                │
│  - SAP GUI users                       │
│  - SAP Fiori users                     │
│  - Administrators                      │
└───────────┬────────────────────────────┘
            │
            │ Client VPN (OpenVPN)
            │
    ┌───────▼────────────────────────┐
    │    AWS Client VPN Endpoint     │
    │    (in Management VPC)         │
    └───────┬────────────────────────┘
            │
            │ Transit Gateway
            │
    ┌───────▼────────────────────────┐
    │       SAP Production VPC       │
    │                                │
    │  ┌─────────┐    ┌──────────┐  │
    │  │ SAP App │    │SAP HANA  │  │
    │  │ Servers │    │ Database │  │
    │  │ (port   │    │ (port    │  │
    │  │ 3200)   │    │ 30013)   │  │
    │  └─────────┘    └──────────┘  │
    └────────────────────────────────┘

Authorization Rules:
1. SAP-Admins → Full access (all ports)
2. SAP-Users → App ports only (3200, 443)
3. SAP-Basis → DB admin ports
4. Deny all by default

Security Groups:
- Allow 3200-3299 from Client VPN CIDR
- Allow 443 from Client VPN CIDR
- Deny direct DB access (use bastion)
```

### 15.13 Troubleshooting

```
COMMON ISSUES:

1. Connection Failures:
   ❌ Issue: Client cannot connect
   ✅ Check:
      - Client VPN endpoint state (available)
      - Security Group allows UDP 443
      - Certificate validity (not expired)
      - Authorization rules exist
      - Target network associated

2. Cannot Access Resources:
   ❌ Issue: Connected but cannot reach instances
   ✅ Check:
      - Authorization rules (network + group)
      - Routes in Client VPN route table
      - Security Groups on target instances
      - NACLs on subnets
      - Instance routing (response route)

3. Authentication Failures:
   ❌ Issue: Invalid username/password
   ✅ Check:
      - Directory Service status
      - User exists in AD
      - Password correct
      - MFA configured correctly
      - Certificate matches (if cert auth)

4. Slow Performance:
   ❌ Issue: High latency, slow speeds
   ✅ Check:
      - Split-tunnel enabled (if appropriate)
      - Client internet connection
      - VPN endpoint capacity (scale up)
      - Target instance network performance
      - MTU size (1400 recommended)

5. DNS Resolution Issues:
   ❌ Issue: Cannot resolve hostnames
   ✅ Check:
      - DNS servers configured in VPN
      - Route 53 Resolver endpoints
      - VPC DNS settings
      - Custom DNS in .ovpn file
```

### 15.14 Client VPN - Exam Tips

**🎯 Điểm Thi Quan Trọng:**

1. **Use Cases:**
   - Remote workers accessing AWS resources
   - Temporary contractor access
   - Administrative access to private resources
   - Replace traditional VPN appliances

2. **Authentication Priority:**
   - Small teams: Certificate-based (simple)
   - Enterprise: AD or SAML (centralized management)
   - MFA required: AD with RADIUS or SAML

3. **Split Tunnel:**
   - Enabled: Only AWS traffic goes through VPN (recommended)
   - Disabled: All traffic goes through VPN (full inspection)
   - Cost impact: Split tunnel reduces data transfer costs

4. **Authorization vs Authentication:**
   - **Authentication**: Who are you? (AD/Cert/SAML)
   - **Authorization**: What can you access? (Network rules)
   - Both required for access

5. **HA Architecture:**
   - Associate với multiple subnets (multi-AZ)
   - Automatic failover between AZs
   - No single point of failure

6. **Integration:**
   - Works with Transit Gateway (multi-VPC access)
   - CloudWatch Logs for auditing
   - AWS SSO for federated access
   - Security Hub for compliance

7. **Limitations:**
   - Regional service (one endpoint per region)
   - Client CIDR cannot overlap with VPC or target networks
   - Maximum connections: Based on endpoint capacity

---

## Câu Hỏi Ôn Tập

### Câu Hỏi Lý Thuyết

**1. Transit Gateway:**
Công ty bạn có 15 VPCs và 3 on-premises data centers cần kết nối. Hiện tại đang dùng VPC Peering và nhiều Site-to-Site VPN connections. Giải pháp nào tối ưu nhất?

A) Tiếp tục dùng VPC Peering và thêm VPN connections
B) Migrate sang AWS Transit Gateway
C) Dùng VPC Peering cho VPCs và Direct Connect cho on-premises
D) Tạo một "hub VPC" với VPN software

<details>
<summary>Đáp án và Giải thích</summary>

**Đáp án: B - Migrate sang AWS Transit Gateway**

**Giải thích:**
- 15 VPCs với VPC Peering = 105 connections (n(n-1)/2)
- Transit Gateway = 15 + 3 = 18 connections (n+1)
- Simplified management
- Centralized routing
- Transitive routing support
- Easy to scale thêm VPCs
- Cost-effective cho số lượng VPC này

**Tại sao không phải các đáp án khác:**
- A: Không scalable, quá phức tạp
- C: Không giải quyết VPC mesh complexity
- D: Operational overhead, không managed
</details>

---

**2. Advanced VPC Routing:**
Bạn cần inspect tất cả traffic giữa Internet và private instances trong VPC. Solution architecture nào đúng?

A) Use Security Groups to filter traffic
B) Use NACLs at subnet boundaries
C) Use Gateway Route Table to route traffic qua firewall appliance
D) Use AWS WAF at ALB

<details>
<summary>Đáp án và Giải thích</summary>

**Đáp án: C - Use Gateway Route Table to route traffic qua firewall appliance**

**Giải thích:**
- Gateway Route Table áp dụng routing tại IGW level
- Route traffic VÀO VPC qua firewall TRƯỚC KHI đến instances
- Centralized inspection point
- Works with both inbound and outbound traffic

```
Internet → IGW → Gateway RT → Firewall → Instance
Instance → Firewall → IGW RT → IGW → Internet
```

**Tại sao không phải các đáp án khác:**
- A: Security Groups filter but don't inspect content
- B: NACLs filter but don't inspect at application layer
- D: WAF only for HTTP/HTTPS, không phải all traffic
</details>

---

**3. Accelerated VPN:**
SAP HANA System Replication từ on-premises sang AWS yêu cầu low latency và consistent performance. On-premises có poor internet routing. Solution nào tốt nhất?

A) Standard Site-to-Site VPN
B) Accelerated Site-to-Site VPN
C) Direct Connect (1 Gbps)
D) Multiple VPN tunnels with ECMP

<details>
<summary>Đáp án và Giải thích</summary>

**Đáp án: C - Direct Connect (1 Gbps)**

**Giải thích:**
- HANA System Replication cần **very low latency** (<1ms cho SYNC)
- **Consistent performance** (no jitter)
- Direct Connect provides:
  - Dedicated connection (< 10ms latency typical)
  - Consistent bandwidth
  - No internet routing issues
  - Best cho mission-critical SAP workloads

**Khi nào dùng Accelerated VPN:**
- Quick setup cần ngay (DX takes weeks)
- Backup for Direct Connect
- ASYNC replication mode (can tolerate higher latency)
- Budget constraints

**So sánh:**
- Standard VPN: 20-100ms (variable)
- Accelerated VPN: 10-50ms (more consistent)
- Direct Connect: <10ms (most consistent)
</details>

---

**4. Client VPN:**
1000 remote employees cần access AWS resources với existing Active Directory for authentication. Requirements: MFA, audit logging, cost-effective. Solution?

A) Site-to-Site VPN từ mỗi employee's home
B) Client VPN với certificate authentication
C) Client VPN với AD authentication + MFA
D) Third-party VPN solution on EC2

<details>
<summary>Đáp án và Giải thích</summary>

**Đáp án: C - Client VPN với AD authentication + MFA**

**Giải thích:**
- ✅ Scalable: 1000+ concurrent connections
- ✅ AD integration: Existing user database
- ✅ MFA: Via RADIUS with AD
- ✅ Audit: CloudWatch Logs
- ✅ Managed: No operational overhead
- ✅ Cost: Pay per use ($0.05/connection-hour)

**Configuration:**
```
Client VPN Endpoint
├─ Authentication: AWS Directory Service (Managed Microsoft AD)
├─ MFA: Enabled via RADIUS
├─ Authorization: AD group-based rules
└─ Logging: CloudWatch Logs
```

**Tại sao không phải các đáp án khác:**
- A: Not practical (1000 VPN connections?)
- B: Certificate management overhead cho 1000 users
- D: Operational burden, không managed, higher cost
</details>

---

**5. Network Segmentation với Transit Gateway:**
Multi-account environment: Production, Development, Shared Services. Requirements: Prod và Dev isolated, both can access Shared Services. Transit Gateway architecture?

A) Single route table với conditional routing
B) Separate TGW cho Prod và Dev
C) Three route tables: Prod RT, Dev RT, Shared RT với custom associations/propagations
D) VPC Peering giữa tất cả VPCs

<details>
<summary>Đáp án và Giải thích</summary>

**Đáp án: C - Three route tables: Prod RT, Dev RT, Shared RT với custom associations/propagations**

**Giải thích:**

```
Architecture:
┌──────────────────────────────────┐
│      Transit Gateway             │
│                                  │
│  ┌──────────┐  ┌──────────┐     │
│  │ Prod RT  │  │  Dev RT  │     │
│  │          │  │          │     │
│  │ Assoc:   │  │ Assoc:   │     │
│  │ Prod VPCs│  │ Dev VPCs │     │
│  │          │  │          │     │
│  │ Prop:    │  │ Prop:    │     │
│  │ Prod VPCs│  │ Dev VPCs │     │
│  │ +Shared  │  │ +Shared  │     │
│  └──────────┘  └──────────┘     │
│                                  │
│  ┌──────────────────────┐        │
│  │ Shared Services RT   │        │
│  │                      │        │
│  │ Assoc: Shared VPC    │        │
│  │ Prop: Prod + Dev VPCs│        │
│  └──────────────────────┘        │
└──────────────────────────────────┘

Result:
✅ Prod cannot reach Dev (different RTs, no propagation)
✅ Dev cannot reach Prod (different RTs, no propagation)
✅ Prod CAN reach Shared (propagated in Prod RT)
✅ Dev CAN reach Shared (propagated in Dev RT)
✅ Shared CAN reach both (both propagated in Shared RT)
```

**Key Concepts:**
- **Association** = Which RT controls this VPC's traffic
- **Propagation** = Which VPCs' routes appear in this RT
- Isolation = No propagation between Prod and Dev RTs
</details>

---

### Câu Hỏi Kịch Bản

**6. Multi-Region SAP Deployment:**

```
Scenario:
- SAP Production: us-east-1 (Primary)
- SAP DR: eu-west-1 (Standby)
- Corporate HQ: New York
- Remote offices: London, Singapore

Requirements:
- HANA System Replication: us-east-1 ↔ eu-west-1
- Corporate access to both regions
- Remote office access with best latency
- Centralized management

Thiết kế architecture phù hợp?
```

<details>
<summary>Solution Architecture</summary>

**Recommended Architecture:**

```
                    ┌─────────────────┐
                    │   Route 53      │
                    │  Geo-Routing    │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │                             │
    ┌─────────▼──────────┐      ┌──────────▼─────────┐
    │  Region: us-east-1 │      │  Region: eu-west-1 │
    │                    │      │                    │
    │  ┌──────────────┐  │      │  ┌──────────────┐  │
    │  │ SAP Prod VPC │  │      │  │  SAP DR VPC  │  │
    │  │              │  │      │  │              │  │
    │  │ HANA Primary │◄─┼──────┼─►│HANA Secondary│  │
    │  │ (Active)     │  │ HSR  │  │ (Standby)    │  │
    │  └──────┬───────┘  │      │  └──────┬───────┘  │
    │         │          │      │         │          │
    │  ┌──────▼───────┐  │      │  ┌──────▼───────┐  │
    │  │   TGW-US     │  │      │  │   TGW-EU     │  │
    │  └──────┬───────┘  │      │  └──────┬───────┘  │
    └─────────┼──────────┘      └─────────┼──────────┘
              │                           │
              │       TGW Peering         │
              └───────────────────────────┘
                      │
         ┌────────────┼────────────┐
         │            │            │
    ┌────▼────┐  ┌───▼────┐  ┌────▼─────┐
    │   HQ    │  │ London │  │Singapore │
    │ New York│  │        │  │          │
    │         │  │        │  │          │
    │ Direct  │  │Acc. VPN│  │Acc. VPN  │
    │ Connect │  │        │  │          │
    └─────────┘  └────────┘  └──────────┘
```

**Components:**

1. **Inter-Region Connectivity:**
   - Transit Gateway Peering (us-east-1 ↔ eu-west-1)
   - HANA System Replication over TGW peering
   - Static routes for cross-region traffic

2. **Corporate HQ (New York):**
   - Direct Connect to us-east-1 TGW
   - Low latency (<5ms)
   - High bandwidth (10 Gbps)
   - Primary access point

3. **Remote Offices:**
   - London: Accelerated VPN to TGW-EU
   - Singapore: Accelerated VPN to TGW-US
   - Automatic routing to nearest region via AWS Global Network

4. **User Access:**
   - Route 53 Geolocation routing
   - US users → us-east-1
   - EU users → eu-west-1
   - APAC users → closest region based on latency

**Configuration Steps:**

```bash
# 1. Create TGW in both regions
aws ec2 create-transit-gateway \
    --region us-east-1 \
    --options AmazonSideAsn=64512

aws ec2 create-transit-gateway \
    --region eu-west-1 \
    --options AmazonSideAsn=64513

# 2. Create TGW Peering
aws ec2 create-transit-gateway-peering-attachment \
    --transit-gateway-id tgw-us \
    --peer-transit-gateway-id tgw-eu \
    --peer-region eu-west-1

# 3. Add routes for cross-region
aws ec2 create-transit-gateway-route \
    --destination-cidr-block 10.100.0.0/16 \
    --transit-gateway-route-table-id tgw-rtb-us \
    --transit-gateway-attachment-id tgw-attach-peering

# 4. Create Accelerated VPN for London
aws ec2 create-vpn-connection \
    --type ipsec.1 \
    --transit-gateway-id tgw-eu \
    --customer-gateway-id cgw-london \
    --options EnableAcceleration=true

# 5. Route 53 Geo-routing
aws route53 change-resource-record-sets \
    --hosted-zone-id Z123456 \
    --change-batch file://geo-routing.json
```

**Cost Analysis:**
- TGW: $0.05/hour × 2 = $72/month
- TGW Peering: $0.02/GB cross-region
- Direct Connect: $1620/month (10 Gbps port) + $0.02/GB
- Accelerated VPN: $36/month + $0.05/GB × 2 offices
- Estimated total: ~$2500-3500/month (depends on data transfer)

**Benefits:**
✅ Low latency HANA replication
✅ Geographic redundancy
✅ Optimized user access (geo-routing)
✅ Centralized management via TGW
✅ Cost-effective for remote offices (VPN vs DX)
</details>

---

**7. Security Inspection Architecture:**

```
Scenario:
Company policy: ALL traffic in/out của AWS MUST go qua security inspection appliance (Palo Alto Networks).

Current:
- 5 VPCs (Prod, Dev, Test, DMZ, Shared)
- Internet-facing applications
- On-premises connectivity via VPN

Requirements:
- Centralized security inspection
- HA và auto-scaling
- Minimal latency impact
- Easy to manage

Thiết kế solution?
```

<details>
<summary>Solution Architecture</summary>

**Recommended: Gateway Load Balancer Architecture**

```
                    Internet
                        │
                        ▼
                   ┌────────┐
                   │  IGW   │
                   └────┬───┘
                        │
        ┌───────────────┼───────────────┐
        │     Security/Inspection VPC   │
        │                               │
        │  ┌─────────────────────────┐  │
        │  │  Ingress Route Table    │  │
        │  │  0.0.0.0/0 → GWLB EP    │  │
        │  └────────────┬────────────┘  │
        │               │               │
        │      ┌────────▼─────────┐     │
        │      │ Gateway Load     │     │
        │      │ Balancer (GWLB)  │     │
        │      └────────┬─────────┘     │
        │               │               │
        │      ┌────────▼──────────┐    │
        │      │ Firewall Instances│    │
        │      │ (Auto Scaling)    │    │
        │      │ - Palo Alto       │    │
        │      │ - 2+ instances    │    │
        │      │ - Multi-AZ        │    │
        │      └───────────────────┘    │
        │                               │
        └───────────────┬───────────────┘
                        │
                ┌───────▼────────┐
                │ Transit Gateway│
                └───────┬────────┘
                        │
        ┌───────────────┼────────────────┐
        │               │                │
    ┌───▼────┐    ┌────▼────┐     ┌────▼────┐
    │Prod VPC│    │ Dev VPC │     │Test VPC │
    └────────┘    └─────────┘     └─────────┘

Traffic Flows:
1. Inbound (Internet → App):
   Internet → IGW → GWLB EP → Firewall → GWLB → TGW → App VPC

2. Outbound (App → Internet):
   App VPC → TGW → GWLB → Firewall → GWLB EP → IGW → Internet

3. VPC-to-VPC:
   VPC-A → TGW → Inspection VPC → Firewall → TGW → VPC-B
```

**Implementation Details:**

**1. Create Gateway Load Balancer:**
```bash
aws elbv2 create-load-balancer \
    --name security-inspection-gwlb \
    --type gateway \
    --subnets subnet-111 subnet-222 \
    --tags Key=Name,Value=SecurityGWLB
```

**2. Create Target Group (Firewall Instances):**
```bash
aws elbv2 create-target-group \
    --name firewall-targets \
    --protocol GENEVE \
    --port 6081 \
    --vpc-id vpc-security \
    --health-check-protocol TCP \
    --health-check-port 443
```

**3. Create GWLB Endpoint:**
```bash
aws ec2 create-vpc-endpoint \
    --vpc-id vpc-security \
    --service-name com.amazonaws.vpce.us-east-1.vpce-svc-123456 \
    --subnet-ids subnet-333 subnet-444 \
    --vpc-endpoint-type GatewayLoadBalancer
```

**4. Gateway Route Table (at IGW):**
```bash
aws ec2 create-route-table \
    --vpc-id vpc-security

aws ec2 associate-route-table \
    --route-table-id rtb-gateway \
    --gateway-id igw-123456

aws ec2 create-route \
    --route-table-id rtb-gateway \
    --destination-cidr-block 0.0.0.0/0 \
    --vpc-endpoint-id vpce-gwlb-123
```

**5. Auto Scaling Group for Firewalls:**
```bash
aws autoscaling create-auto-scaling-group \
    --auto-scaling-group-name firewall-asg \
    --launch-template LaunchTemplateId=lt-firewall \
    --min-size 2 \
    --max-size 10 \
    --desired-capacity 4 \
    --target-group-arns arn:aws:elasticloadbalancing:...:targetgroup/firewall-targets \
    --health-check-type ELB \
    --health-check-grace-period 300 \
    --vpc-zone-identifier "subnet-555,subnet-666"

# Scaling policies based on connections
aws autoscaling put-scaling-policy \
    --auto-scaling-group-name firewall-asg \
    --policy-name scale-up-on-connections \
    --policy-type TargetTrackingScaling \
    --target-tracking-configuration file://scaling-config.json
```

**Scaling Config:**
```json
{
  "PredefinedMetricSpecification": {
    "PredefinedMetricType": "ALBRequestCountPerTarget",
    "ResourceLabel": "app/security-inspection-gwlb/..."
  },
  "TargetValue": 1000.0
}
```

**Alternative: Transit Gateway Appliance Mode (Traditional)**

```
                    Internet
                        │
                        ▼
                   ┌────────┐
                   │  IGW   │
                   └────┬───┘
                        │
        ┌───────────────┼───────────────┐
        │     Inspection VPC            │
        │  ┌─────────────────────────┐  │
        │  │  Firewall Instances     │  │
        │  │  (Manual HA)            │  │
        │  └────────────┬────────────┘  │
        └───────────────┼───────────────┘
                        │
                ┌───────▼────────┐
                │ Transit Gateway│
                │ (Appliance Mode)│
                └───────┬────────┘
                        │
        ┌───────────────┼────────────────┐
        │               │                │
    ┌───▼────┐    ┌────▼────┐     ┌────▼────┐
    │Prod VPC│    │ Dev VPC │     │Test VPC │
    └────────┘    └─────────┘     └─────────┘

Configuration:
aws ec2 modify-transit-gateway-vpc-attachment \
    --transit-gateway-attachment-id tgw-attach-inspection \
    --options ApplianceModeSupport=enable
```

**Comparison:**

| Feature | GWLB Solution | TGW Appliance Mode |
|---------|---------------|-------------------|
| **Auto-scaling** | ✅ Native support | ❌ Manual |
| **HA** | ✅ Automatic | ⚠️ Manual failover |
| **Complexity** | ⚠️ More components | ✅ Simpler |
| **Performance** | ✅ Better (distributed) | ⚠️ Bottleneck |
| **Cost** | ⚠️ GWLB + TGW | ✅ Only TGW |
| **Management** | ✅ Easier | ⚠️ More operational |
| **Modern** | ✅ AWS recommended | ⚠️ Legacy pattern |

**Recommendation:** Use GWLB for new deployments (modern, scalable, managed)

**Benefits:**
✅ Centralized security policy enforcement
✅ Auto-scaling based on traffic
✅ Transparent to applications (bump-in-the-wire)
✅ Multi-AZ high availability
✅ Simplified management

**Cost Estimation:**
- GWLB: $0.0125/hour + $0.004/GB = ~$110/month + data processing
- Firewall Instances: r5.2xlarge × 4 = $1,152/month (On-Demand)
  - Savings Plans: ~$700/month (40% savings)
- TGW: $0.05/hour = $36/month
- Data processing: $0.02/GB
- **Total: ~$850-1300/month** (depends on usage)
</details>

---

## Bảng Tổng Kết

### So Sánh Connectivity Options

| Option | Use Case | Bandwidth | Latency | Cost | Setup Time | HA |
|--------|----------|-----------|---------|------|------------|-----|
| **Site-to-Site VPN** | Standard connectivity | 1.25 Gbps/tunnel | Variable (internet) | $ | Hours | 2 tunnels |
| **Accelerated VPN** | Mission-critical VPN | 1.25 Gbps/tunnel | Consistent (AWS backbone) | $$ | Hours | 2 tunnels + GA |
| **Direct Connect** | Dedicated connection | 1-100 Gbps | < 10ms | $$$ | Weeks | Requires 2nd connection |
| **Direct Connect + VPN** | Hybrid (encrypted DX) | DX bandwidth | DX latency | $$$+ | Weeks | DX + VPN backup |
| **Client VPN** | Remote user access | Internet dependent | Variable | $$ | Minutes | Multi-AZ |
| **Transit Gateway** | Multi-VPC/hybrid hub | 50 Gbps/AZ | Minimal | $$ | Minutes | Regional (multi-AZ) |
| **VPC Peering** | VPC-to-VPC | No limit | Minimal | $ (data only) | Minutes | Automatic |

### Transit Gateway Features

| Feature | Description | Use Case | Limitation |
|---------|-------------|----------|------------|
| **Attachments** | VPC, VPN, DX Gateway, Peering | Connect resources | 5000 max per TGW |
| **Route Tables** | Custom routing domains | Network segmentation | - |
| **Associations** | Which RT controls VPC traffic | Assign VPC to segment | 1 per attachment |
| **Propagations** | Auto-learn routes | Dynamic routing | Multiple allowed |
| **Peering** | Cross-region/account connectivity | Multi-region | 50 max, static routes |
| **ECMP** | Equal-cost multi-path | Bandwidth scaling | VPN only |
| **Appliance Mode** | Stateful appliance support | Firewall integration | Per-attachment |

### Advanced Routing Patterns

| Pattern | Purpose | Components | Complexity |
|---------|---------|------------|------------|
| **Gateway Route Table** | Ingress traffic control | IGW, VGW, Routes | Medium |
| **Middlebox Routing** | Traffic inspection | GWLB, Firewalls, TGW | High |
| **Egress VPC** | Centralized internet access | NAT GW, TGW, Multiple VPCs | Medium |
| **VPC Endpoint Routing** | AWS service access | Gateway/Interface Endpoints | Low |
| **Multi-Region Active-Active** | Global applications | TGW Peering, Route 53 | High |

### Client VPN Comparison

| Authentication | Best For | Pros | Cons | MFA Support |
|----------------|----------|------|------|-------------|
| **Certificate** | Small teams | Simple setup | Certificate management | ❌ |
| **Active Directory** | Enterprise | Centralized management | Requires AD | ✅ (RADIUS) |
| **SAML 2.0** | SSO environments | Best UX, SSO | Complex setup | ✅ (IdP) |

### VPN Cost Comparison (1 TB/month)

| Solution | Components | Monthly Cost | Best For |
|----------|------------|--------------|----------|
| **Standard VPN** | VPN connection + data | $126 | Standard workloads |
| **Accelerated VPN** | VPN + acceleration + data | $176 | Mission-critical |
| **Client VPN** (100 users) | Endpoint + connections + data | $450-550 | Remote users |
| **Direct Connect** | 1 Gbps port + data | $236 | High volume |
| **DX + VPN** | DX + VPN backup | $270 | Production with backup |

---

## Tổng Kết Phần 2

Trong phần 2 này, chúng ta đã deep-dive vào các chủ đề nâng cao:

### Transit Gateway
- **Hub-and-spoke architecture** thay thế VPC Peering mesh
- **Route Tables** cho network segmentation
- **Cross-region peering** cho multi-region architectures
- Ideal cho **SAP landscapes** với multiple VPCs

### Advanced VPC Routing
- **Gateway Route Tables** cho traffic inspection
- **Middlebox patterns** với GWLB
- **Egress VPC** architecture
- Route priority và selection logic

### Accelerated VPN
- Uses **AWS Global Accelerator** và backbone network
- **Consistent performance** vs standard VPN
- Use cases: **SAP HANA replication**, real-time applications
- Cost: Additional **$0.05/GB** for acceleration

### Client VPN
- **Remote user access** to AWS resources
- **Multiple authentication**: Certificate, AD, SAML
- **Split-tunnel** vs full-tunnel
- Integration với **Transit Gateway** cho multi-VPC access

---

**🎯 Exam Strategy:**

1. **Transit Gateway:**
   - Số lượng VPCs > 5-10 → Consider TGW
   - Cần network segmentation → Multiple route tables
   - On-premises connectivity → TGW with VPN/DX

2. **Advanced Routing:**
   - Traffic inspection required → GWLB (modern) or Gateway RT
   - Centralized egress → Egress VPC pattern
   - Always check route priority (most specific wins)

3. **VPN Selection:**
   - Standard workloads → Standard VPN
   - Mission-critical → Accelerated VPN or DX
   - Remote users → Client VPN
   - High volume (> 5 TB/month) → Direct Connect

4. **Cost Optimization:**
   - VPC Peering: Free routing, pay data transfer
   - TGW: $0.05/GB + attachment fees
   - Accelerated VPN: Extra $0.05/GB
   - Consider volume for DX break-even

---

**📚 Next Steps:**

1. Practice trong AWS Console:
   - Create Transit Gateway
   - Configure route tables
   - Set up Client VPN
   - Test routing scenarios

2. Hands-on Labs:
   - Multi-VPC connectivity with TGW
   - Security inspection với GWLB
   - Client VPN với AD authentication

3. Review:
   - AWS whitepapers on networking
   - SAP on AWS best practices
   - Transit Gateway documentation

Chúc các bạn học tốt và thi đạt kết quả cao! 🚀
