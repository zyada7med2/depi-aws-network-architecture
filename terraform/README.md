# Infrastructure as Code (Terraform)

> **Status: empty scaffold.** No infrastructure is defined yet — Week 1 implementation is
> planned, not written. Every directory below currently holds a `.gitkeep` placeholder.

This directory hosts the modular Terraform configuration for the project.

## Directory Layout

- **`environments/`** — environment-specific root modules:
  - `dev/`
  - `prod/`
- **`modules/`** — reusable infrastructure modules. Naming convention: `NN-<name>`.
  - `01-vpc-core/` — VPC, subnets, routing, NAT, security groups _(Week 1)_
  - `02-hybrid-transit-gateway/` — Transit Gateway, Site-to-Site VPN _(Week 2)_
  - `03-edge-security-waf/` — CloudFront, WAF, Route 53, ACM _(Week 3)_
  - `04-monitoring-ir/` — VPC Flow Logs, Athena, incident response _(Week 4)_
