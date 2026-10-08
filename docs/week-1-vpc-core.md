# Week 1: VPC Core Architecture & Security Foundation

## 🎯 Objectives
- Build a production-grade VPC with high availability across 2 Availability Zones.
- Segment subnets into Public (ALB, NAT), Private (App Servers), and Isolated (RDS Database).
- Eliminate bastion jump boxes and direct SSH/RDP ports using **AWS Systems Manager (SSM) Session Manager**.
- Implement defense-in-depth using stateful Security Groups and stateless NACLs.

---

## 📐 Network Architecture Overview
- **VPC CIDR:** `10.0.0.0/16`
- **Subnets:**
  - `Public Subnet AZ-A`: `10.0.1.0/24` (IGW Attached)
  - `Public Subnet AZ-B`: `10.0.2.0/24` (IGW Attached)
  - `Private App Subnet AZ-A`: `10.0.10.0/24` (Route to NAT-A)
  - `Private App Subnet AZ-B`: `10.0.20.0/24` (Route to NAT-B)
  - `Isolated DB Subnet AZ-A`: `10.0.30.0/24` (No default route)
  - `Isolated DB Subnet AZ-B`: `10.0.40.0/24` (No default route)

---

## 🔒 Security Group Chaining Matrix

| Security Group | Ingress Allowed From | Ingress Ports | Egress Destination |
| :--- | :--- | :--- | :--- |
| **`sg-alb`** | `0.0.0.0/0` (or CloudFront Prefix List) | TCP 80, TCP 443 | `sg-app` (TCP 8080/443) |
| **`sg-app`** | `sg-alb` ONLY | TCP 8080 | HTTPS out via NAT Gateway + VPC Endpoints |
| **`sg-database`** | `sg-app` ONLY | TCP 5432 (Postgres) / 3306 (MySQL) | None |

---

## 🧪 Verification & Acceptance Criteria
- [ ] Internet traffic reaches ALB on port 443 and terminates SSL.
- [ ] Direct internet inbound to App servers is completely blocked.
- [ ] Database instances cannot initiate direct outbound traffic to the public internet.
- [ ] Engineers connect to private EC2 instances strictly via `aws ssm start-session` without SSH keys.
