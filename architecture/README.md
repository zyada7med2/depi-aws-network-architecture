# Architecture Specification & Network Design

> **Status: Target State (multi-phase).** This document specifies the *complete* intended
> architecture across all four sprints. Only the **Week 1** subset is implemented so far —
> see [`docs/week-1-vpc-core.md`](../docs/week-1-vpc-core.md) for what is actually built.
> Anything marked _(Week N)_ below is design intent, not a deployed resource.

This directory contains the network design, CIDR address allocation plan, and routing policies
for the **Secure AWS Network Architecture & Hybrid Connectivity** capstone project.

---

## 🌐 CIDR Address Allocation

### Global address plan

| Network | AWS / Physical entity | CIDR | Phase |
| :--- | :--- | :--- | :--- |
| **Primary Workload VPC** | AWS VPC (`us-east-1`) | `10.0.0.0/16` | Week 1 |
| **Shared Services / Security VPC** | AWS VPC (`us-east-1`) | `10.1.0.0/16` | Week 2 |
| **Simulated On-Premises** | Corporate network | `192.168.0.0/16` | Week 2 |

These three ranges do not overlap, which is a precondition for Transit Gateway routing.

### Primary VPC subnet rule

The third octet encodes the tier; the fourth octet encodes the Availability Zone. New subnets
are derived from the rule rather than chosen ad hoc.

| Third octet | Tier |
| :--- | :--- |
| `0–9`   | Public / ingress |
| `10–19` | Private application |
| `20–29` | Isolated data |
| `250–255` | Transit (Transit Gateway attachment) |

Fourth octet: `0` = AZ-A, `1` = AZ-B, `2` = AZ-C (reserved).

### Primary VPC subnets

| Network zone | AZ-A | AZ-B | AZ-C (reserved) | Phase |
| :--- | :--- | :--- | :--- | :--- |
| Public (ALB + NAT) | `10.0.0.0/24` | `10.0.1.0/24` | `10.0.2.0/24` | Week 1 |
| Private application | `10.0.10.0/24` | `10.0.11.0/24` | `10.0.12.0/24` | Week 1 |
| Isolated data (RDS) | `10.0.20.0/24` | `10.0.21.0/24` | `10.0.22.0/24` | Week 1 |
| Transit (TGW attachment) | `10.0.250.0/24` | `10.0.251.0/24` | `10.0.252.0/24` | Week 2 |

### Shared Services VPC subnets _(Week 2)_

| Subnet | CIDR | Purpose |
| :--- | :--- | :--- |
| Endpoints | `10.1.10.0/24` | AWS PrivateLink interface endpoints (SSM, Secrets Manager, S3) |
| Inspection | `10.1.20.0/24` | Traffic mirroring target & packet capture analyzers |

> Future spoke VPCs are **independent VPCs** with their own CIDRs (e.g. `10.2.0.0/16`), not
> carved out of the primary VPC's `10.0.0.0/16`. The `10.0.250.0/24`–`10.0.252.0/24` blocks are
> Transit Gateway *attachment* subnets **inside** the primary VPC.

---

## 🔀 Transit Gateway Routing & Segmentation _(Week 2 — design intent)_

The AWS Transit Gateway acts as the central hub interconnecting all networks while maintaining
strict segmentation:

### Route table associations & propagations

1. **Production TGW route table (`tgw-rt-prod`):**
   - Associated with: Production VPC attachment.
   - Routes:
     - `10.1.0.0/16` ➔ Shared Services VPC attachment.
     - `192.168.0.0/16` ➔ Site-to-Site VPN attachment (corporate).
     - `0.0.0.0/0` ➔ **Blackholed by default.** Once the central egress inspection path exists
       in the Shared Services VPC, this route is repointed there. Until then, the production
       VPC has no Transit Gateway internet egress.

2. **Security & Shared Services route table (`tgw-rt-shared`):**
   - Associated with: Shared Services VPC attachment.
   - Routes propagated from the Production VPC and the VPN attachment.

3. **VPN / On-Premises route table (`tgw-rt-vpn`):**
   - Associated with: IPsec Site-to-Site VPN attachment.
   - Propagates access only to the permitted private subnets
     (`10.0.10.0/24`, `10.0.11.0/24`).

---

## 🛡️ Defense-in-Depth Layering

```
[ Layer 7 - Edge ]      CloudFront + AWS WAF (OWASP Top 10, rate limits)   (Week 3)
       ▼
[ Layer 4 - Ingress ]   Internet-Facing Application Load Balancer (ALB)    (Week 1, HTTP)
       ▼
[ Layer 3/4 - Compute ] Private App Subnet (ingress only from sg-alb)      (Week 1)
       ▼
[ Layer 3/4 - Storage ] Isolated Data Subnet (ingress only from sg-app)    (Week 1)
       ▼
[ Control Plane ]       Zero SSH ports open. Access via AWS SSM Session Manager.
```
