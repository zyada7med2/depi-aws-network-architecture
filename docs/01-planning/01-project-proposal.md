# Project Proposal

**Project:** Secure AWS Network Architecture & Hybrid Connectivity
**Program:** DEPI, AWS Cloud Security & Infrastructure (Project 2)
**Author:** Hisham Mohamed
**Status:** Draft v1. This is the target design. No component counts as deployed until there is evidence in `docs/07-implementation` and `screenshots/`.

## 1. Overview

This project designs, builds, tests and documents a secure multi-tier AWS network. It covers a segmented VPC, hub-and-spoke connectivity, hybrid connectivity to a simulated on-premises site, edge protection, and network monitoring with an incident response (IR) exercise.

## 2. Problem Statement

Cloud networks are often built flat and public by default. Typical consequences are workloads reachable directly from the internet, no segmentation between environments, edge protections bypassed, and no network visibility when an instance is compromised. This project shows how to avoid these problems with layered controls (defence in depth) and how to prove, with tests, that the controls work.

## 3. Objectives

1. Build a multi-AZ VPC with public, private-app and private-data tiers and least-privilege Security Groups and NACLs.
2. Connect multiple VPCs using VPC Peering and a Transit Gateway hub-and-spoke model, and prove segmentation with routing tests.
3. Provide hybrid access through a Site-to-Site VPN (simulated on-premises) and a Client VPN lab.
4. Protect the public edge with CloudFront, S3 Origin Access Control and AWS WAF so that the origin cannot be bypassed.
5. Collect and analyse network telemetry (VPC Flow Logs, CloudWatch, Athena) and define detection use cases.
6. Execute and document a compromised-instance IR exercise: isolate, snapshot, collect evidence, recover.
7. Keep the lab cost-controlled and reproducible with Infrastructure as Code.

## 4. Scope

Levels: **Build** = deployed and tested. **Lab** = deployed briefly as a controlled demonstration. **Concept** = documented and compared only.

| Area | Component | Level | Justification |
|---|---|---|---|
| Core network | VPC, 6 subnets across 2 AZs, IGW, route tables | Build | Foundation of the project |
| | NAT Gateway | Build | 1 NAT for labs; a second NAT only for an AZ-failover test (hourly cost). TODO(Hisham): confirm |
| | Security Groups, NACLs | Build | Core of the least-privilege design |
| | EC2 (Auto Scaling Group), ALB, RDS | Build | Sample workload needed to test the network. RDS single-AZ by default, Multi-AZ for one failover test |
| | SSM Session Manager | Build | Replaces a bastion host (no inbound SSH). Bastion is documented as an alternative |
| Connectivity | VPC Peering (Main ↔ Extra VPC) | Build | Cheap, shows non-transitive behaviour |
| | VPC Endpoints (S3 gateway, SSM interface) | Build | Private access to AWS services |
| | PrivateLink | Lab | SSM interface endpoints use PrivateLink. Provider-side endpoint service is documented |
| | Transit Gateway (Main, Spoke A, Spoke B, VPN) | Build | Hub-and-spoke and segmentation tests |
| | Site-to-Site VPN | Lab | On-premises simulated with strongSwan on EC2 in a separate VPC |
| | Client VPN | Lab | One short session (hourly billing) |
| | Direct Connect | Concept | Needs physical circuits or a partner |
| Edge | CloudFront + S3 (OAC) | Build | Static content delivery |
| | AWS WAF | Build | Managed rules plus a rate-based rule, with block/allow tests |
| | Route 53 | Lab | DNS design and health checks. TODO(Hisham): decide if a real domain is used |
| | Shield, Firewall Manager, Global Accelerator | Concept | Shield Standard is automatic. Shield Advanced has a high fixed fee. Firewall Manager needs AWS Organizations |
| Monitoring | VPC Flow Logs → S3 → Athena | Build | Core detection capability |
| | CloudWatch alarms + SNS | Build | Alerting |
| | Traffic Mirroring | Lab or Concept | TODO(Hisham): decide after Phase 4 cost check |
| | IR simulation (isolate, snapshot, evidence) | Build | Core deliverable |

## 5. Out of Scope

Production-grade multi-region DR, real on-premises hardware, Direct Connect circuits, third-party firewalls, and attacks against any system I do not own.

## 6. Assumptions and Constraints

- Single AWS account, one Region. TODO(Hisham): state the Region.
- Budget cap: TODO(Hisham): set a number and create an AWS Budgets alert.
- Resources are torn down after each lab session and rebuilt from IaC (Terraform).
- Deadlines (Industry DEPI): design 6 Nov, implementation 30 Nov, final 4 Dec 2026.

## 7. Deliverables

Architecture diagrams, CIDR/subnet plan, SG/NACL matrix, route table documentation, hybrid connectivity test report, WAF rule set, DDoS mitigation plan, Flow Log queries, IR runbook, evidence notes, final secure network report.
