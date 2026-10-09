# Week 1: VPC Core Architecture & Security Foundation

> **Scope:** This document covers **Week 1 deliverables only**.
> The complete multi-phase target design (Transit Gateway, hybrid connectivity, edge
> security, observability) is specified in
> [`architecture/README.md`](../architecture/README.md) and is **not** implemented in Week 1.

## 🎯 Objectives
- Build a production-grade VPC with high availability across 2 Availability Zones.
- Segment subnets into Public (ALB, NAT), Private App (compute), and Isolated Data (RDS) tiers.
- Eliminate bastion jump boxes and direct SSH/RDP ports using **AWS Systems Manager (SSM) Session Manager**.
- Implement defense-in-depth using stateful Security Groups and stateless NACLs.

---

## 📐 Network Architecture Overview

- **VPC CIDR:** `10.0.0.0/16` (`us-east-1`)

### CIDR Allocation Rule

The third octet encodes the **tier**; the fourth octet encodes the **Availability Zone**:

| Third octet | Tier |
| :--- | :--- |
| `0–9`   | Public / ingress |
| `10–19` | Private application |
| `20–29` | Isolated data |
| `250–255` | Transit (Transit Gateway attachment) |

Fourth octet: `0` = AZ-A, `1` = AZ-B, `2` = AZ-C (reserved).

### Week 1 Subnets

| Tier | AZ-A | AZ-B | AZ-C (reserved) | Default route |
| :--- | :--- | :--- | :--- | :--- |
| **Public** | `10.0.0.0/24` | `10.0.1.0/24` | `10.0.2.0/24` | `0.0.0.0/0` → IGW |
| **Private App** | `10.0.10.0/24` | `10.0.11.0/24` | `10.0.12.0/24` | `0.0.0.0/0` → NAT (same AZ) |
| **Isolated Data** | `10.0.20.0/24` | `10.0.21.0/24` | `10.0.22.0/24` | none |

> The AZ-C blocks are reserved up front so a third Availability Zone can be added later
> without renumbering anything.

---

## 🔀 Week 1 Management Access (SSM)

Week 1 deploys **no VPC Endpoints**. Engineers reach private instances with
`aws ssm start-session`, and the SSM agent reaches the public SSM service endpoints
(`ssm`, `ssmmessages`, `ec2messages`) **through the NAT Gateway** on TCP 443.

- No SSH/RDP ports are opened anywhere. No SSH keys. No bastion host.
- Interface VPC Endpoints — which would let SSM work with no NAT internet egress at all —
  are a **Week 2** deliverable in the Shared Services VPC.
- Instances still require an IAM instance profile carrying the
  `AmazonSSMManagedInstanceCore` policy; correct networking alone is not sufficient.

---

## 🔒 Security Group Chaining Matrix

| Security Group | Ingress From | Ingress Ports | Egress |
| :--- | :--- | :--- | :--- |
| **`sg-alb`** | `0.0.0.0/0` | TCP 80 | → `sg-app` TCP 8080 |
| **`sg-app`** | `sg-alb` ONLY | TCP 8080 | TCP 443 → AWS SSM prefix lists (via NAT) |
| **`sg-data`** | `sg-app` ONLY | TCP 5432 (Postgres) | None |

**Notes**
- HTTPS (TCP 443) ingress, ACM certificates, Route 53, and CloudFront are **Week 3**
  deliverables and are deliberately absent here.
- `sg-app` egress is restricted to TCP 443 toward the AWS-managed prefix lists for
  `ssm`, `ssmmessages`, and `ec2messages` — not `0.0.0.0/0`.
- The Isolated Data tier has no egress and no route to the internet.

---

## 🧪 Verification & Acceptance Criteria
- [ ] HTTP traffic reaches the ALB on port 80.
- [ ] Direct internet inbound to the App servers is completely blocked.
- [ ] Database instances cannot initiate any outbound traffic to the public internet.
- [ ] Engineers connect to private EC2 instances via `aws ssm start-session` with no SSH keys.
- [ ] `sg-app` accepts inbound traffic only from `sg-alb`, on TCP 8080 only.
