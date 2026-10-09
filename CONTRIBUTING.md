# Contributing Guidelines

Welcome to the **Secure AWS Network Architecture & Hybrid Connectivity** project team! To ensure a seamless collaboration and keep our codebase clean, all team members are expected to follow these guidelines.

> **Note on Phase 0:** This repository was scaffolded during **Phase 0** (proposal, architecture
> specification, and workspace setup), and those initial setup commits were made directly to
> `main`. The branching and pull-request rules below govern **active development, starting with
> the Week 1 implementation sprint.**

---

## 🌿 Branching Strategy

We follow a structured branch naming convention based on our 4-week milestones:

- `main`: Production-ready code, reviewed and approved by the team.
- `develop`: Integration branch where weekly deliverables are assembled.
- Feature branches format:
  - `feat/week-1-vpc-core`
  - `feat/week-2-transit-gateway`
  - `feat/week-2-site-to-site-vpn`
  - `feat/week-3-cloudfront-waf`
  - `feat/week-4-flow-logs-monitoring`
  - `fix/issue-description`
  - `docs/update-architecture`

---

## 🔄 Commit Message Convention

Please use clear and semantic commit messages:

- `feat:` Adds a new Terraform resource, module, or architecture component.
- `fix:` Fixes a syntax error, routing issue, or security group rule.
- `docs:` Updates architecture diagrams, runbooks, or markdown documentation.
- `refactor:` Restructures Terraform modules without changing infrastructure behavior.
- `test:` Adds or updates validation scripts and incident simulation scenarios.

Example:
```bash
git commit -m "feat(vpc): add multi-az public and private subnets with nat gateway"
```

---

## 📋 Pull Request (PR) Process

1. Create a feature branch from `develop`.
2. Implement your module or documentation.
3. Validate your code locally:
   ```bash
   terraform fmt -check
   terraform validate
   ```
4. Push your branch and open a PR against `develop`.
5. Fill out the **Pull Request Template**.
6. At least **one peer review approval** is required before merging.

---

## 🔒 Security Best Practices

- **NEVER** commit AWS access keys, secret keys, SSH private keys, or `.tfvars` containing secrets.
- Always use `terraform.tfvars.example` to document expected variables.
- Ensure all security groups follow the principle of least privilege.
