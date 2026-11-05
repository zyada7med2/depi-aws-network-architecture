# Requirements

**Status:** Draft v1

## 1. Stakeholder Analysis

The scenario is a hypothetical company moving a web application to AWS.

| Stakeholder | Interest / need |
|---|---|
| Instructor / evaluators | Meets DEPI deliverables, correct design, evidence of work |
| Project owner (student) | Learn, build, defend the design in interviews |
| Security team | Segmentation, least privilege, visibility, fast incident containment |
| Application owners | Reliable, reachable app, private database |
| Operations | Simple access (no open SSH), monitoring, repeatable deployment |
| Finance | Predictable and limited cost |
| Remote staff | Secure access to internal resources |
| Branch/on-premises office | Private connectivity to AWS |
| End users | Fast, safe access to the website |

## 2. Use Cases

| ID | Use case | Actor |
|---|---|---|
| UC-01 | Browse the website via CloudFront | End user |
| UC-02 | Submit a request to the dynamic application through ALB | End user |
| UC-03 | Access an instance through Session Manager | Administrator |
| UC-04 | Reach internal resources over Client VPN | Remote user |
| UC-05 | Reach AWS resources from on-premises over Site-to-Site VPN | On-premises user |
| UC-06 | Query Flow Logs to investigate traffic | Security analyst |
| UC-07 | Isolate a compromised instance and collect evidence | Security analyst |
| UC-08 | Deploy or tear down the environment from IaC | Administrator |

## 3. User Stories

1. As an end user, I want the site to load quickly and safely, so that I can use it without exposure to common web attacks.
2. As a security engineer, I want the database to have no internet route, so that it cannot be reached or exfiltrated from directly.
3. As an administrator, I want to open a shell without exposing SSH to the internet, so that the attack surface stays small.
4. As a security engineer, I want spoke environments isolated from each other, so that a breach in one does not reach the other.
5. As a remote employee, I want a VPN into the internal network, so that I can reach internal services securely.
6. As a security analyst, I want searchable network logs, so that I can detect and investigate suspicious traffic.
7. As a security analyst, I want a tested runbook for a compromised instance, so that I can contain it quickly and preserve evidence.
8. As a finance owner, I want the lab to be torn down when not in use, so that cost stays within budget.

## 4. Functional Requirements

| ID | Requirement |
|---|---|
| FR-01 | The system shall provide a VPC with public, private-app and private-data subnets in two AZs |
| FR-02 | Public subnets shall route to an Internet Gateway; private-app subnets shall reach the internet only through NAT; private-data subnets shall have no internet route |
| FR-03 | The ALB shall be the only public entry point to the application tier and shall accept traffic only from CloudFront |
| FR-04 | EC2 instances shall run in an Auto Scaling Group in private-app subnets with no public IPs |
| FR-05 | RDS shall run in private-data subnets and accept connections only from the application tier |
| FR-06 | Administrative access shall use SSM Session Manager |
| FR-07 | The system shall use a gateway endpoint for S3 and interface endpoints for SSM |
| FR-08 | The system shall connect Main VPC, Spoke A and Spoke B through a Transit Gateway with route tables enforcing segmentation |
| FR-09 | The system shall include a VPC Peering connection between Main VPC and an extra VPC |
| FR-10 | The system shall provide a Site-to-Site VPN to a simulated on-premises network |
| FR-11 | The system shall provide a Client VPN endpoint for remote access (lab) |
| FR-12 | CloudFront shall serve static content from a private S3 bucket using Origin Access Control |
| FR-13 | AWS WAF shall be attached to CloudFront with managed rules and a rate-based rule |
| FR-14 | VPC Flow Logs shall be stored in S3 and queryable with Athena |
| FR-15 | CloudWatch alarms shall notify through SNS on defined events |
| FR-16 | The system shall support instance isolation by replacing its Security Groups, and EBS snapshots for evidence |
| FR-17 | The infrastructure shall be reproducible from Terraform |

## 5. Non-functional Requirements

| ID | Category | Requirement |
|---|---|---|
| NFR-01 | Availability | Application tier spans two AZs and tolerates loss of one instance or AZ (tested) |
| NFR-02 | Security | Least privilege on SGs, NACLs and IAM; defence in depth; no secrets in the repository |
| NFR-03 | Segmentation | Spoke A ↔ Spoke B traffic is blocked; tested |
| NFR-04 | Auditability | Network traffic and API activity are logged with a defined retention |
| NFR-05 | Cost | Total lab cost stays under the budget cap; paid resources are torn down after use |
| NFR-06 | Operability | No inbound SSH; changes are made through IaC where possible |
| NFR-07 | Recoverability | A compromised instance can be isolated, snapshotted and replaced following the runbook |
| NFR-08 | Documentation | Documentation reflects the real deployment and includes test evidence |
