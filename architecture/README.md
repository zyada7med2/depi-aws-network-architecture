# Architecture Specification & Network Design

This directory contains the detailed network diagrams, CIDR address allocation plans, and routing policies for the **Secure AWS Network Architecture & Hybrid Connectivity** capstone project.

---

## 🌐 CIDR IP Allocation Table

| Network Zone | AWS / Physical Entity | CIDR Block | Purpose / Workload |
| :--- | :--- | :--- | :--- |
| **Spoke VPC 1 (Prod)** | AWS VPC (`us-east-1`) | `10.0.0.0/16` | Main enterprise workload hosting ALB, App servers, and Database |
| ├── Public Subnet AZ-A | Subnet | `10.0.1.0/24` | Internet-facing ALB & NAT Gateway AZ-1 |
| ├── Public Subnet AZ-B | Subnet | `10.0.2.0/24` | Internet-facing ALB & NAT Gateway AZ-2 |
| ├── Private App AZ-A | Subnet | `10.0.10.0/24` | Application compute tier (EC2 instances managed by SSM) |
| ├── Private App AZ-B | Subnet | `10.0.20.0/24` | Application compute tier (Multi-AZ redundancy) |
| ├── Private DB AZ-A | Subnet | `10.0.30.0/24` | Primary Amazon RDS database tier (No Internet Egress) |
| └── Private DB AZ-B | Subnet | `10.0.40.0/24` | Secondary/Replica Amazon RDS database tier |
| **Spoke VPC 2 (Sec/Shared)** | AWS VPC (`us-east-1`) | `10.1.0.0/16` | Centralized security, inspection, and VPC Endpoints |
| ├── Endpoints Subnet | Subnet | `10.1.10.0/24` | AWS PrivateLink Interface Endpoints (SSM, Secrets, S3) |
| └── Inspection Subnet | Subnet | `10.1.20.0/24` | Traffic mirroring target & packet capture analyzers |
| **Simulated On-Premises** | Corporate Network | `192.168.0.0/16` | Customer Gateway (CGW) connected via IPsec Site-to-Site VPN |

---

## 🔀 Transit Gateway Routing & Segmentation

The AWS Transit Gateway acts as the central hub interconnecting all networks while maintaining strict segmentation:

### Route Table Associations & Propagations
1. **Production TGW Route Table (`tgw-rt-prod`):**
   - Associated with: Production VPC Attachment.
   - Routes:
     - `10.1.0.0/16` ➔ Spoke VPC 2 (Shared Services & Endpoints).
     - `192.168.0.0/16` ➔ Site-to-Site VPN Attachment (Corporate).
     - `0.0.0.0/0` ➔ Blackholed or routed to Central Egress inspection.

2. **Security & Shared Services Route Table (`tgw-rt-shared`):**
   - Associated with: Shared Services VPC Attachment.
   - Routes propagated from Production VPC and VPN Attachment.

3. **VPN / On-Premises Route Table (`tgw-rt-vpn`):**
   - Associated with: IPsec Site-to-Site VPN Attachment.
   - Only propagates access to permitted private subnets (`10.0.10.0/24`, `10.0.20.0/24`).

---

## 🛡️ Defense-in-Depth Layering

```
[ Layer 7 - Edge ]      CloudFront + AWS WAF (OWASP Top 10, Rate Limits)
       ▼
[ Layer 4 - Ingress ]   Internet-Facing Application Load Balancer (ALB)
       ▼
[ Layer 3/4 - Compute ] Private App Subnet (Security Group allows ingress only from ALB)
       ▼
[ Layer 3/4 - Storage ] Private RDS Subnet (Security Group allows ingress only from App SG)
       ▼
[ Control Plane ]       Zero SSH ports open (0.0.0.0/0:22 blocked). Access via AWS SSM.
```
