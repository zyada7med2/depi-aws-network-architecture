# DEPI Capstone Project: Secure AWS Network Architecture & Hybrid Connectivity

> **Program:** Digital Egypt Pioneers Initiative (DEPI)  
> **Track:** AWS Security Cloud Computing  
> **Status:** 🟢 Sprint 1 (Week 1: Core Network Foundation)  

---

## 📌 Project Overview

This repository hosts the graduation capstone project for the **AWS Security Cloud Computing Track** under the **Digital Egypt Pioneers Initiative (DEPI)**.

Our team is designing and implementing a **Secure AWS Network Architecture with Hybrid Connectivity** modeled after enterprise-grade AWS Well-Architected Security principles. The goal is to build a reliable, segmented cloud network capable of securely interconnecting multiple environments, defending against web-layer threats, and detecting anomalous traffic.

---

## 🎯 Target Architecture Vision

The target architecture is an enterprise **Hub-and-Spoke** topology designed to scale securely:

```mermaid
flowchart TB
    subgraph Edge[" 🛡️ Edge Layer (Planned) "]
        CF["CloudFront & Route 53"] --> WAF["AWS WAF"]
    end

    subgraph Core_VPC[" ☁️ Core VPC Environment (Active Focus: Week 1) "]
        ALB["Application Load Balancer"]
        subgraph Subnets["Multi-AZ Subnets"]
            direction TB
            Pub["Public Subnets (ALB & NAT)"]
            App["Private App Subnets (EC2)"]
            DB["Isolated Database Subnets (RDS)"]
        end
        ALB --> App
        App --> DB
    end

    subgraph Hub[" 🔄 Central Hub (Phase 2) "]
        TGW["AWS Transit Gateway"]
    end

    subgraph OnPrem[" 🏢 Corporate Network (Phase 2) "]
        CGW["Customer Gateway (Site-to-Site VPN)"]
    end

    Edge -.-> ALB
    Core_VPC <===> Hub
    Hub <===> OnPrem
```

---

## 🚀 Current Milestone: Week 1 - Core VPC Foundation

We are currently in **Week 1**, focusing on laying down a solid, compliant network foundation before introducing routing hubs or edge defenses.

### 📋 Week 1 Objectives & Deliverables
- [ ] **IP Addressing & CIDR Design:** Finalize non-overlapping CIDR block planning (`10.0.0.0/16`) across 2 Availability Zones.
- [ ] **Multi-Tier Subnet Segmentation:**
  - Public Subnets for ingress load balancers and NAT gateways.
  - Private Application Subnets for compute workloads.
  - Isolated Database Subnets with no default route to the internet.
- [ ] **Ingress & Egress Controls:** Configure Internet Gateways (IGW) and managed NAT Gateways.
- [ ] **Least-Privilege Security Groups:** Establish strict chaining rules (ALB ➔ App Tier ➔ Database Tier).
- [ ] **Secure Management Plane:** Eliminate public bastion jump-boxes and direct SSH port 22 access by adopting **AWS Systems Manager (SSM) Session Manager**.
- [ ] **Initial IaC Implementation:** Structure and review initial Terraform modules for the baseline VPC.

---

## 🗺️ Project Roadmap (High-Level Phases)

Detailed deliverables for upcoming weeks will be added iteratively as we progress:

| Phase | Milestone | Focus Area | Status |
| :---: | :--- | :--- | :---: |
| **Week 1** | **Core VPC Foundation** | Multi-AZ Subnets, Routing, NAT, Security Groups & SSM | 🟡 **In Progress** |
| **Week 2** | **Hybrid Connectivity & Hub Routing** | AWS Transit Gateway, Site-to-Site VPN, VPC Endpoints | ⚪ Upcoming |
| **Week 3** | **Edge Security & Application Protection** | Amazon CloudFront, AWS WAF Rulesets, Route 53 | ⚪ Upcoming |
| **Week 4** | **Observability & Incident Response** | VPC Flow Logs, Athena Analytics & Host Isolation | ⚪ Upcoming |

---

## 📁 Repository Structure

The workspace is organized to support clean, modular development as the project expands:

```text
depi-aws-network-architecture/
├── docs/             # Sprint planning, design notes, and architecture specs
├── terraform/        # Infrastructure as Code (Terraform) workspace
├── .gitignore        # Standard ignore rules for Terraform and AWS credentials
├── CONTRIBUTING.md   # Team collaboration and branch conventions
└── README.md         # Project overview and active milestone tracker
```

---

## 🌿 Team Workflow

To ensure smooth collaboration across the team:
- The `main` branch tracks verified, review-approved milestones.
- Work for the current sprint is developed in feature branches (e.g., `feat/week-1-vpc-subnets`).
- Each feature must be tested and reviewed before merging into `main`.

---

## 📜 Acknowledgments

Developed as part of the **Digital Egypt Pioneers Initiative (DEPI)**, sponsored by the **Ministry of Communications and Information Technology (MCIT)**, Egypt.
