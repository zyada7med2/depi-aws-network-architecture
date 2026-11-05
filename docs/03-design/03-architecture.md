# Architecture Design

**Status:** Draft v1, target design (not yet deployed). Source diagram: `diagrams/overall-architecture.drawio` (to be redrawn in draw.io from this document).

## 1. Design Summary

- One Main VPC (3 tiers × 2 AZs) behind an edge layer (Route 53, CloudFront, WAF).
- A Transit Gateway connects Main, Spoke A, Spoke B and a Site-to-Site VPN to a simulated on-premises network.
- A separate peered VPC demonstrates VPC Peering.
- Monitoring: Flow Logs to S3 and Athena, CloudWatch alarms with SNS, an isolation Security Group for incident response.

## 2. Changes from Diagram v1

| # | Diagram v1 issue | Decision |
|---|---|---|
| 1 | Internet → ALB directly via the IGW (bypasses CloudFront/WAF) | ALB SG allows only the CloudFront origin-facing managed prefix list, and the ALB checks a secret custom header. The prefix list alone is not enough, because any CloudFront distribution can reach the ALB |
| 2 | Peering drawn from the TGW | Peering connects Main VPC ↔ Extra VPC directly |
| 3 | Client VPN drawn to the TGW | The Client VPN endpoint associates with a subnet in Main VPC. Spokes are reached through the TGW |
| 4 | Edge shown as a request chain | DNS (Route 53) resolves to CloudFront. WAF is attached to CloudFront. Shield Standard is automatic |
| 5 | Spokes were empty | One small test instance per spoke, with TGW route tables enforcing segmentation |
| 6 | On-premises shown as real | Simulated with strongSwan on EC2 in a separate VPC |
| 7 | Flow Logs to both CloudWatch and S3 | S3 + Athena by default. CloudWatch only if alarms need it |
| 8 | Traffic Mirroring shown as deployed | Lab or Concept. Shown only if built |

## 3. CIDR and Subnet Plan

| Network | CIDR | Purpose |
|---|---|---|
| Main VPC | 10.0.0.0/16 | Application environment |
| Public AZ-a / AZ-b | 10.0.1.0/24, 10.0.2.0/24 | ALB, NAT Gateway |
| Private app AZ-a / AZ-b | 10.0.11.0/24, 10.0.12.0/24 | EC2 Auto Scaling Group |
| Private data AZ-a / AZ-b | 10.0.21.0/24, 10.0.22.0/24 | RDS |
| Spoke VPC A | 10.1.0.0/16 (test subnet 10.1.1.0/24) | Segmentation test workload |
| Spoke VPC B | 10.2.0.0/16 (test subnet 10.2.1.0/24) | Segmentation test workload |
| Extra VPC (peered) | 10.3.0.0/16 (test subnet 10.3.1.0/24) | VPC Peering lab |
| Simulated on-premises | 192.168.0.0/24 | strongSwan VPN endpoint and test host |
| Client VPN client range | 172.16.0.0/22 | Must not overlap any network above |

No ranges overlap. Each /24 subnet loses 5 reserved addresses.

## 4. Routing Design

| Route table | Routes |
|---|---|
| Public | local; 0.0.0.0/0 → IGW |
| Private app (per AZ) | local; 0.0.0.0/0 → NAT in the same AZ; S3 prefix list → gateway endpoint; 10.1.0.0/16, 10.2.0.0/16, 192.168.0.0/24 → TGW; 10.3.0.0/16 → peering |
| Private data | local only. No default route |

**Transit Gateway route tables**

| TGW route table | Associated attachments | Routes (propagated) |
|---|---|---|
| `shared-rt` | Main, VPN | Main, Spoke A, Spoke B, on-premises |
| `spokes-rt` | Spoke A, Spoke B | Main, on-premises only |

Result: spokes reach Main and on-premises, but Spoke A and Spoke B have no route to each other. TODO(Hisham): decide whether on-premises should also reach the spokes. A third route table would restrict it to Main only.

## 5. Security Groups (planned)

| SG | Inbound | Outbound |
|---|---|---|
| `sg-alb` | 80/443 from the CloudFront origin-facing prefix list | App port to `sg-app` |
| `sg-app` | App port from `sg-alb` | 443 to endpoints/NAT, DB port to `sg-rds` |
| `sg-rds` | DB port from `sg-app` | None required |
| `sg-vpce` | 443 from `sg-app` | None required |
| `sg-isolation` | None | None |

TODO(Hisham): choose the DB engine (MySQL or PostgreSQL) and the app port.

NACLs (stateless): public subnets allow 80/443 inbound and ephemeral ports 1024–65535; private-app allows the app port from the public subnets; private-data allows the DB port from the app subnets only. A detailed rule matrix is produced in Phase 1.

## 6. Traffic Flows

1. User → Route 53 (DNS) → CloudFront (WAF evaluates) → S3 via OAC for static content.
2. CloudFront → ALB (public subnets, with the secret header) → EC2 in private-app → RDS in private-data.
3. Admin → SSM Session Manager → EC2 through interface endpoints (no inbound SSH).
4. On-premises → Site-to-Site VPN (2 IPsec tunnels) → TGW → Main VPC.
5. Remote user → Client VPN endpoint (Main VPC) → internal resources.
6. Private EC2 → NAT Gateway → internet for updates, and S3 via the gateway endpoint.

## 7. Monitoring and Incident Response

- VPC Flow Logs (Main VPC) → S3 → Athena queries (rejected traffic, top talkers, unexpected ports).
- CloudWatch alarms → SNS notification.
- Isolation procedure: tag the instance, replace its SGs with `sg-isolation`, snapshot EBS, preserve logs, then rebuild. TODO(Hisham): test whether existing connections survive an SG change (connection tracking) and document the finding.

## 8. Architectural Decisions

| Decision | Alternative | Reason | Trade-off |
|---|---|---|---|
| SSM instead of bastion | Bastion host | No inbound ports, IAM-based, audited | Needs interface endpoints or NAT |
| TGW hub-and-spoke | Full peering mesh | Scales, central routing, segmentation by route table | Hourly per-attachment cost |
| Data tier without a default route | NAT for the data tier | Smaller attack surface | Needs endpoints for AWS service access |
| 1 NAT for labs, 2 as target | 2 NATs always | Cost | Less AZ resilience during labs |
| Flow Logs to S3 | CloudWatch Logs | Cheaper for Athena analysis | No native alarms on the logs |

## 9. Cost Notes (approximate, verify in the AWS Pricing Calculator)

Hourly meters: NAT Gateway, TGW attachments, VPN connection, ALB, RDS, interface endpoints, Client VPN. WAF has monthly and per-request charges. Plan: build, test and tear down in sessions.

## 10. Official References

- VPC: https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html
- Transit Gateway: https://docs.aws.amazon.com/vpc/latest/tgw/what-is-transit-gateway.html
- Well-Architected: https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html
