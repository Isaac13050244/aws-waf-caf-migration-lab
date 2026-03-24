# AWS Well-Architected Framework & Cloud Adoption Framework Assessment

## Project Overview

This project documents the migration of a two-tier on-premises web application to AWS. It provides a full technical and organisational assessment using two of AWS's core frameworks:

- **AWS Well-Architected Framework (WAF)** — evaluates the migrated workload across the five pillars of operational excellence, security, reliability, performance efficiency, and cost optimisation.
- **AWS Cloud Adoption Framework (CAF)** — assesses the organisation's readiness for cloud adoption across six perspectives: business, people, governance, platform, security, and operations.

The project identifies the weaknesses of a basic lift-and-shift deployment, proposes concrete improvements for each, and presents a fully redesigned Multi-AZ architecture that addresses every identified risk.

---

## Repository Contents

| File | Description |
|---|---|
| `aws_waf_caf_assessment.md` | Full assessment covering all four tasks — architecture review, WAF evaluation, CAF analysis, and improved architecture design |
| `architecture.drawio.png` | Improved Multi-AZ AWS architecture diagram |
| `README.md` | This file |

---

## How to Review

1. Start with `aws_waf_caf_assessment.md` — it walks through the existing architecture, its risks, the WAF and CAF analysis, and the fully redesigned solution.
2. View `architecture.drawio.png` to see the improved architecture visually — it shows the VPC layout, subnets, ALB, Auto Scaling Group, Multi-AZ RDS, CloudFront, S3, and all traffic flows.

---

## Key AWS Services Used in the Improved Architecture

- **Compute** — EC2 with Auto Scaling Group (Multi-AZ)
- **Networking** — VPC, Public and Private Subnets, Application Load Balancer, NAT Gateway, Internet Gateway
- **Database** — Amazon RDS Multi-AZ with automatic failover
- **Content Delivery** — Amazon CloudFront, Amazon S3
- **Security** — AWS WAF, AWS IAM, AWS KMS, AWS Secrets Manager, AWS Systems Manager Session Manager
- **Monitoring** — Amazon CloudWatch, AWS CloudTrail, Amazon GuardDuty
- **Infrastructure as Code** — AWS CloudFormation

---

## Tasks Completed

- **Task 1** — Reviewed the existing post-lift-and-shift AWS architecture and documented all components and risks
- **Task 2** — Evaluated the workload against all five AWS Well-Architected Framework pillars, identifying strengths, weaknesses, and recommended improvements
- **Task 3** — Assessed cloud readiness across all six AWS Cloud Adoption Framework perspectives
- **Task 4** — Designed an improved Multi-AZ architecture and mapped every design decision back to the WAF pillars and CAF best practices
