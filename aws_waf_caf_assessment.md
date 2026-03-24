# AWS Well-Architected Framework & Cloud Adoption Framework Assessment

## Project Overview

This assessment documents the migration of a two-tier on-premises web application to AWS. It evaluates the current state of the migrated workload using the **AWS Well-Architected Framework (WAF)**, examines organisational cloud readiness through the **AWS Cloud Adoption Framework (CAF)**, and proposes a fully improved architecture that addresses every identified risk and weakness.

---

## Task 1 – Review of the Existing Architecture

### 1.1 Context

The application has been migrated from on-premises to AWS using a basic **lift-and-shift** approach. This means the application now runs on AWS infrastructure, but it has not yet been re-architected to take advantage of cloud-native capabilities. The result is a workload that is running in the cloud but is not yet truly cloud-ready.

### 1.2 Current AWS Components

| AWS Resource | Current Configuration |
|---|---|
| **VPC** | Single VPC with public and private subnets in one Availability Zone |
| **EC2 Instances** | 1–2 instances (e.g. t3.medium) hosting the web/application tier, with public IP addresses assigned directly |
| **Amazon RDS** | Single-AZ MySQL or PostgreSQL instance hosted in a private subnet |
| **Security Groups** | Web SG: ports 22 (SSH), 80 (HTTP), and 443 (HTTPS) open to `0.0.0.0/0` (the entire internet) |
| **Internet Gateway** | Attached to the VPC, enabling direct internet access to EC2 instances |
| **Elastic IP / Public DNS** | Traffic routed directly to a single EC2 instance — no load balancer sits in front |
| **EBS Volumes** | Root and data volumes attached to EC2 with no snapshot schedule configured |
| **RDS Automated Backups** | Enabled with the default 7-day retention window; no manual snapshots or cross-region copies |
| **Monitoring** | Basic CloudWatch metrics only — no custom dashboards, alarms, or centralised logging |

### 1.3 Identified Risks and Weaknesses

**1. Single-AZ Deployment (Critical)**
Both the EC2 instances and the RDS database are deployed in a single Availability Zone. If that AZ experiences any disruption — a power failure, cooling issue, or AWS-side incident — the entire application goes offline. There is no redundancy and no automatic failover.

**2. No Load Balancer**
Traffic is sent directly to a single EC2 instance via an Elastic IP. There is no health checking, no traffic distribution, and no mechanism to route requests away from an unhealthy instance. A single instance failure takes down the entire application.

**3. No Auto Scaling**
The compute capacity is fixed regardless of traffic levels. During a sudden traffic spike, the instances will be overwhelmed and performance will degrade. During quiet periods, the instances run at full cost even when barely utilised. This wastes money and creates a reliability risk during peaks.

**4. Overly Permissive Security Groups**
Port 22 (SSH) is open to `0.0.0.0/0`, meaning any machine on the internet can attempt to connect to the server via SSH. This is a significant attack surface. Ports 80 and 443 are also open directly to EC2 rather than being fronted by a load balancer with controlled access rules.

**5. Direct SSH Key-Based Server Access**
Administrators connect to instances using SSH key pairs. This approach is difficult to audit, does not produce centralised access logs, and requires managing and rotating private keys across the team. A lost or stolen key can result in unauthorised server access with no automatic detection.

**6. Single-AZ RDS with No Cross-Region Backups**
If the RDS instance or its host AZ fails, the database is unavailable and no automatic promotion to a standby occurs. The default 7-day automated backup window is the only recovery mechanism. There are no manual snapshots and no cross-region copies, meaning a regional disaster could result in permanent data loss.

**7. No Web Application Firewall (WAF)**
The application is directly exposed to the internet without any web-layer filtering. This leaves it vulnerable to common attacks such as SQL injection, cross-site scripting (XSS), and HTTP floods. These threats require no special tools to exploit and are among the most common causes of web application breaches.

**8. No Content Delivery Network (CDN)**
All traffic — including requests for static assets like images, CSS files, and JavaScript — is served directly by the EC2 instances. Users located far from the AWS region experience high latency. The EC2 instances bear unnecessary load that could be offloaded to a CDN.

**9. Insufficient Monitoring and Alerting**
The current setup relies on basic CloudWatch metrics. There are no custom alarms, no centralised log aggregation, and no dashboards. Problems are typically discovered only after users report them. There is no visibility into errors, latency trends, or security events in real time.

**10. Always-On Fixed Compute at Full Cost**
The EC2 instances run 24 hours a day, 7 days a week, regardless of traffic patterns. During off-peak hours such as nights and weekends, this is pure waste. Without Auto Scaling or scheduling, there is no mechanism to reduce compute costs during periods of low demand.

**11. No Encryption at Rest or in Transit (Internal)**
While HTTPS may be configured on the public-facing side, there is no confirmation that data at rest in RDS and EBS volumes is encrypted using KMS. Internal traffic between application tiers may also be travelling in plaintext.

---

## Task 2 – AWS Well-Architected Framework Evaluation

For each of the five pillars, the table below documents a genuine strength of the current setup, a real weakness that must be addressed, a specific recommendation, and the AWS services that implement that recommendation.

| Pillar | Current Strength | Current Weakness | Recommendation | AWS Services |
|---|---|---|---|---|
| **Operational Excellence** | The workload is now in the cloud, which makes automation tools available and accessible | Infrastructure is managed manually. There is no IaC, no CI/CD pipeline, no centralised logging, and no operational runbooks | Replace manual processes with Infrastructure as Code and implement centralised observability across all tiers | AWS CloudFormation, AWS CloudWatch, AWS CloudTrail, AWS Systems Manager |
| **Security** | The workload has been moved off a potentially unsecured on-premises network onto AWS-managed infrastructure | Security groups are excessively open (port 22 to `0.0.0.0/0`). There is no WAF, no encryption enforced at rest, and no secrets management — credentials may be hardcoded or stored insecurely | Apply the principle of least privilege across all security groups and IAM roles. Enable encryption at rest and in transit. Replace SSH with Systems Manager Session Manager. Add WAF rules to filter malicious traffic | AWS IAM, AWS KMS, AWS WAF, AWS Secrets Manager, AWS Systems Manager Session Manager, VPC Security Groups |
| **Reliability** | AWS provides the underlying tools for Multi-AZ and automatic failover — the foundation is available | The entire application runs in a single Availability Zone. A single EC2 instance handles all traffic with no health checking, and the RDS instance has no standby replica | Distribute all compute across at least two Availability Zones behind an Application Load Balancer. Enable RDS Multi-AZ with automatic failover | Application Load Balancer, Auto Scaling Group, Amazon RDS Multi-AZ, Amazon Route 53 |
| **Performance Efficiency** | Cloud infrastructure can scale and adapt — the platform supports high performance natively | Static assets are served directly from EC2, adding unnecessary load. There is no caching layer and no CDN, causing high latency for geographically distributed users | Offload static content to S3 and serve it via CloudFront. Add an in-memory caching layer in front of the database to reduce read latency | Amazon CloudFront, Amazon S3, Amazon ElastiCache, Auto Scaling Group |
| **Cost Optimization** | Moving to AWS shifts the billing model from large upfront capital expenditure to variable operational expenditure, which is already an improvement | EC2 instances run at fixed capacity around the clock. There is no right-sizing analysis, no Reserved or Savings Plan pricing, and no mechanism to scale down during low traffic | Enable Auto Scaling to match compute to demand. Analyse instance utilisation and right-size. Purchase Savings Plans for baseline workloads | AWS Auto Scaling, AWS Compute Savings Plans, AWS Cost Explorer, AWS Trusted Advisor, S3 Intelligent-Tiering |

---

## Task 3 – AWS Cloud Adoption Framework (CAF) Assessment

The AWS Cloud Adoption Framework provides a structured way to assess an organisation's readiness for the cloud across six perspectives: Business, People, Governance, Platform, Security, and Operations. Each perspective below evaluates where the organisation currently stands and identifies the most important actions needed to advance successfully.

---

### Business Perspective

The organisation's motivation for migrating to AWS is clear: to reduce infrastructure costs, improve application availability, and accelerate the pace of product delivery. These are legitimate and achievable cloud benefits. However, the business case for cloud adoption remains incomplete. There is no formally documented cloud value roadmap, no Total Cost of Ownership (TCO) analysis comparing the old on-premises costs to the new AWS spend, and no defined Key Performance Indicators (KPIs) to measure whether the migration is actually delivering value.

The most important next step from a business perspective is to formalise the expected outcomes of cloud adoption. Leadership needs to agree on what success looks like — for example, a target reduction in infrastructure operating costs, a maximum acceptable downtime threshold, or a target reduction in time-to-market for new features. A TCO model should be built to quantify the financial benefit of moving from fixed capital expenditure to variable operational expenditure. Cloud investments must be tied directly to business outcomes, with clear reporting mechanisms so that stakeholders can see the return on investment over time. Without this foundation, cloud adoption risks becoming a technical project disconnected from the business objectives it was meant to serve.

---

### People Perspective

The organisation's IT team has deep expertise in managing traditional on-premises infrastructure — physical servers, local networking, and legacy deployment processes. These skills are valuable but are not directly transferable to a cloud-native operating model. Most team members are likely unfamiliar with managed AWS services, infrastructure automation, and the shared responsibility model that governs security in the cloud.

Bridging this skills gap is essential to a successful and sustainable migration. The organisation should invest in a structured upskilling programme built around AWS training pathways, including AWS Cloud Practitioner and AWS Solutions Architect certifications, as well as hands-on labs that mirror real-world scenarios. Beyond individual training, team roles need to evolve. Traditional "server administrator" roles should transition toward Cloud Engineer, DevOps Engineer, and Site Reliability Engineer (SRE) profiles — roles that prioritise automation, observability, and continuous improvement over manual maintenance. Change management is also critical: some team members may resist new tools and workflows. Establishing internal cloud champions, running lunch-and-learn sessions, and creating a Cloud Center of Excellence (CCoE) will help embed cloud-native thinking across the organisation and prevent the new architecture from being operated in the same manual, reactive way as the old one.

---

### Governance Perspective

At present, the organisation has no formal cloud governance policies in place. There are no enforced tagging standards for tracking resource ownership, no budget alerts to prevent unexpected cost overruns, no defined approval workflows for provisioning new cloud resources, and no process for periodic compliance reviews. This is a significant risk as the cloud footprint grows — without guardrails, costs can spiral, security configurations can drift, and accountability for resources becomes unclear.

Establishing a governance foundation should begin immediately. The first step is to implement a resource tagging strategy that captures at minimum the owning team, environment (development, staging, production), and cost centre for every resource. AWS Budgets should be configured with alerts that notify stakeholders before spending thresholds are breached. AWS Organizations and Service Control Policies (SCPs) should be used to enforce guardrails at the account level — for example, preventing the creation of resources in unapproved regions or blocking the disabling of CloudTrail logging. Role-Based Access Control (RBAC) must be defined clearly so that only authorised personnel can modify production infrastructure. If the organisation operates under any regulatory framework — such as GDPR for data privacy or PCI DSS for payment data — compliance requirements should be mapped to specific AWS controls and validated through automated auditing tools such as AWS Config and AWS Security Hub. Governance cannot be an afterthought; it must be built into the cloud operating model from the beginning.

---

### Platform Perspective

The current platform is technically functional but falls far short of cloud-native best practices. The architecture was designed for a lift-and-shift and has not been updated to take advantage of the resilience, scalability, and automation capabilities that AWS provides natively. Infrastructure is provisioned and managed manually, making it inconsistent between environments, difficult to reproduce, and slow to change.

The platform needs to be rebuilt around automation and resilience as first principles. Infrastructure as Code (IaC) using AWS CloudFormation should be adopted immediately, ensuring that every resource — VPC, subnets, security groups, EC2, RDS, load balancer — is defined in version-controlled templates that can be deployed consistently across development, staging, and production environments. CI/CD pipelines built with AWS CodePipeline, CodeBuild, and CodeDeploy should replace manual deployment processes, enabling faster and safer releases with automated testing at each stage. Networking should be restructured so that EC2 instances sit in private subnets with no direct internet exposure, behind an Application Load Balancer in the public subnets. NAT Gateways should provide outbound internet access for updates and patching without exposing instances inbound. The platform should also embrace managed services wherever possible — RDS Multi-AZ instead of self-managed databases, CloudFront instead of custom caching servers — to reduce operational overhead and improve reliability without increasing the team's maintenance burden.

---

### Security Perspective

The organisation's current security posture reflects an on-premises, perimeter-based mindset: assume the network boundary is the line of defence, and trust everything inside it. This model is inadequate for the cloud, where the perimeter is fluid and every component is potentially reachable from the internet if not explicitly protected. The open security groups, direct SSH access, and absence of encryption controls are direct consequences of this legacy thinking.

The transition to cloud-native security requires adopting a Zero Trust model — never trust by default, always verify, and enforce the minimum access required for every interaction. Security groups must be locked down so that the Application Load Balancer accepts traffic from the internet on ports 80 and 443, EC2 instances accept traffic only from the ALB, and RDS accepts connections only from EC2 — with no component directly exposed to `0.0.0.0/0`. SSH (port 22) should be removed entirely and replaced with AWS Systems Manager Session Manager, which provides audited, key-free server access through the AWS console without opening any inbound ports. IAM roles should replace any access keys or hardcoded credentials, following the principle of least privilege throughout. Encryption should be enforced at rest for all RDS instances and EBS volumes using AWS KMS, and in transit through TLS on the Application Load Balancer. Secrets such as database passwords and API keys should be stored and rotated automatically using AWS Secrets Manager. AWS WAF should be deployed in front of the Application Load Balancer to filter SQL injection, XSS, and other OWASP Top 10 threats. Finally, AWS CloudTrail, Amazon GuardDuty, and AWS Security Hub should be activated to provide continuous visibility into API activity, threat detection, and compliance posture — shifting the organisation from reactive incident response to proactive threat management.

---

### Operations Perspective

The current operational model is reactive. The team monitors the application by waiting for user complaints or checking basic CloudWatch graphs manually. There are no automated alarms, no defined incident response procedures, no documented recovery objectives, and no regular disaster recovery testing. In this model, the team is always behind — discovering problems after they have already impacted customers.

A cloud-native operations model must be proactive, automated, and measurable. Amazon CloudWatch should be configured with custom dashboards, metric alarms for CPU utilisation, error rates, and response times, and log groups that aggregate output from all application and infrastructure components in one place. Alarm thresholds should trigger automated responses where possible — for example, Auto Scaling should add capacity automatically when CPU crosses a defined threshold, rather than requiring a human to intervene. Incident response runbooks should be documented so that any team member can follow a clear procedure when an alarm fires, reducing mean time to recovery. Recovery objectives must be formally defined: the Recovery Time Objective (RTO — how quickly the application must be restored after failure) and Recovery Point Objective (RPO — how much data can be lost) should be agreed with business stakeholders and validated through regular backup and restore drills. Systems Manager Patch Manager should automate OS patching across EC2 instances, eliminating manual maintenance windows. By shifting from manual, reactive operations to automated, observable, and tested operations, the team can spend significantly less time keeping the application running and significantly more time improving it.

---

## Task 4 – Improved Architecture Design

### 4.1 Architecture Overview

The improved architecture eliminates every risk identified in Task 1 and aligns with all five pillars of the Well-Architected Framework. It is designed for high availability, security by default, automated scalability, and operational efficiency.

The key components of the new design are:

| Component | New Configuration |
|---|---|
| **VPC** | Single VPC spanning two Availability Zones (AZ-1 and AZ-2) |
| **Public Subnets** | One per AZ — host the Application Load Balancer only |
| **Private Subnets (App Tier)** | One per AZ — host EC2 instances in the Auto Scaling Group, no public IP |
| **Private Subnets (Data Tier)** | One per AZ — host the Multi-AZ RDS primary and standby replica |
| **Internet Gateway** | Attached to the VPC for inbound public traffic to the ALB |
| **NAT Gateway** | One per AZ in the public subnet — allows EC2 instances to make outbound requests (e.g. for updates) without being publicly reachable inbound |
| **Application Load Balancer** | Sits in public subnets across both AZs; accepts HTTPS traffic, performs health checks, distributes requests to EC2 |
| **Auto Scaling Group** | Minimum 2 EC2 instances (one per AZ), scales up under load, scales down during quiet periods |
| **Amazon RDS Multi-AZ** | Primary instance in AZ-1, synchronous standby replica in AZ-2; automatic failover in under 2 minutes |
| **Amazon S3** | Stores all static assets (images, CSS, JavaScript) |
| **Amazon CloudFront** | CDN that serves S3 static assets from edge locations globally; also fronts the ALB for dynamic requests |
| **AWS WAF** | Attached to CloudFront and/or ALB; filters OWASP Top 10 threats including SQLi and XSS |
| **AWS Systems Manager** | Session Manager replaces SSH for all administrative access — no port 22 open anywhere |
| **AWS KMS** | Encrypts RDS storage, EBS volumes, and S3 objects at rest |
| **AWS Secrets Manager** | Stores and auto-rotates database credentials and application secrets |
| **Amazon CloudWatch** | Custom dashboards, metric alarms, and centralised log groups across all tiers |
| **AWS CloudTrail** | Full API audit trail across the entire AWS account |
| **Amazon GuardDuty** | Continuous threat detection using ML analysis of network and API activity |
| **AWS CloudFormation** | All infrastructure defined as code in version-controlled templates |

---

### 4.2 How the New Design Addresses Each WAF Pillar

**Operational Excellence**

Every resource in the new architecture is defined in AWS CloudFormation templates stored in a version-controlled repository. This means the environment can be deployed, updated, or torn down consistently with no manual steps and no risk of configuration drift between environments. Deployments to production go through a CI/CD pipeline (CodePipeline + CodeBuild + CodeDeploy), meaning new application versions are released through an automated, tested process rather than a manual copy-and-restart. CloudWatch provides a single pane of glass for all metrics and logs, and alarms fire automatically when thresholds are crossed — enabling the team to detect and respond to issues before users are affected.

**Security**

Security groups are configured on the principle of least privilege throughout the stack. The ALB security group accepts inbound traffic from the internet on ports 443 (HTTPS) only — port 80 traffic is redirected to HTTPS. The EC2 security group accepts traffic only from the ALB security group on port 443, with no direct internet exposure and no port 22 open. The RDS security group accepts traffic only from the EC2 security group on the database port (3306 for MySQL). No component in the data tier is reachable from the internet under any circumstances. AWS WAF blocks common web exploits before they reach the application. Systems Manager Session Manager replaces SSH entirely. KMS encrypts all data at rest, TLS encrypts all data in transit, and Secrets Manager ensures no credentials are ever hardcoded in application code.

**Reliability**

The application is distributed across two Availability Zones at every tier. The Application Load Balancer spans both AZs and continuously health-checks the EC2 instances, routing traffic only to healthy targets. The Auto Scaling Group maintains a minimum of two instances — one in each AZ — so the loss of any single instance or even an entire AZ does not cause downtime. RDS Multi-AZ maintains a synchronous standby replica in the second AZ, and if the primary instance fails, AWS promotes the standby automatically in under two minutes with no manual intervention required. CloudWatch alarms and Auto Scaling ensure that capacity responds to demand automatically, preventing overload-induced outages.

**Performance Efficiency**

Static content — images, CSS, JavaScript, fonts, and any other assets that do not change per-request — is stored in Amazon S3 and served globally through CloudFront's edge network. Users receive these assets from the edge location nearest to them, dramatically reducing load times regardless of geographic location. This also removes significant load from the EC2 instances, which can then focus entirely on processing dynamic application logic. For workloads with a high volume of repetitive database reads, Amazon ElastiCache (Redis or Memcached) can be introduced in front of RDS to serve cached results in milliseconds rather than querying the database on every request. The Auto Scaling Group ensures that compute capacity always matches actual demand, so neither under-provisioning nor over-provisioning degrades performance.

**Cost Optimization**

The Auto Scaling Group scales compute capacity in real time based on actual demand. During off-peak hours, it can scale down to a single instance per AZ, and during high-traffic events, it scales out to meet demand — meaning the organisation pays only for what it uses. Static content delivery through CloudFront and S3 is significantly cheaper per-request than serving the same content from EC2. For the baseline EC2 and RDS capacity that runs continuously, Compute Savings Plans and RDS Reserved Instances can be purchased to reduce costs by up to 30–40% compared to on-demand pricing. AWS Cost Explorer and Trusted Advisor provide ongoing visibility into spending patterns and right-sizing recommendations, ensuring the architecture remains cost-efficient as it evolves.

---

### 4.3 How the New Design Supports the CAF

The improved architecture is built entirely from standard, well-documented AWS managed services. This means it can be codified into reusable CloudFormation templates — supporting the **Governance** and **Platform** teams by making the infrastructure reproducible, auditable, and enforceable through policy. The elimination of SSH and the adoption of IAM roles, Secrets Manager, WAF, GuardDuty, and CloudTrail directly advances the **Security** perspective's Zero Trust goals. The Multi-AZ design, health checking, and automated failover deliver the high availability outcomes that the **Operations** perspective demands, reducing unplanned downtime and enabling proactive incident response. By leaning on managed services — RDS Multi-AZ, CloudFront, Auto Scaling — rather than self-managed alternatives, the architecture reduces the operational burden on the **People** team, allowing them to build cloud skills incrementally rather than being overwhelmed by day-one complexity. Finally, the shift to variable, demand-driven compute directly supports the **Business** perspective's goal of moving from capital expenditure to operational expenditure and demonstrating a measurable return on the migration investment.

---

### 4.4 Mapping Improvements Back to Task 1 Risks

| Risk Identified in Task 1 | How the New Architecture Resolves It |
|---|---|
| Single-AZ deployment | Multi-AZ across all tiers: ALB, ASG (EC2), and RDS |
| No load balancer | Application Load Balancer distributes traffic and performs health checks |
| No Auto Scaling | Auto Scaling Group adjusts EC2 count based on real-time demand |
| Open security groups (port 22) | Port 22 removed; Systems Manager Session Manager replaces SSH |
| Direct SSH key-based access | Replaced entirely by Systems Manager — audited, key-free, no open ports |
| Single-AZ RDS, no cross-region backups | RDS Multi-AZ with automatic failover; AWS Backup for cross-region snapshot copies |
| No WAF / web application filtering | AWS WAF deployed on CloudFront and ALB with OWASP Top 10 rule sets |
| No CDN | CloudFront serves static assets globally from S3 |
| Insufficient monitoring | CloudWatch dashboards, metric alarms, centralised logs, GuardDuty, CloudTrail |
| Always-on fixed compute | Auto Scaling Group scales to demand; Savings Plans reduce baseline cost |
| No encryption at rest or in transit | KMS encrypts RDS, EBS, and S3; TLS enforced on ALB; Secrets Manager for credentials |

---

## Reflection

Working through this assessment in full — rather than only the surface-level tasks — made the connection between AWS services and real business risk genuinely clear. The most important shift in thinking was understanding that moving infrastructure to the cloud is not the same as designing for the cloud. A lift-and-shift gets workloads off on-premises hardware, but it carries all the same risks into a new environment without addressing any of them. True cloud readiness means designing for failure, automating everything that can be automated, enforcing security by default rather than as an afterthought, and measuring outcomes against business objectives — not just keeping servers online.

The CAF made it equally clear that the technology decisions are only half the challenge. An organisation can design a perfect Multi-AZ architecture and still fail to adopt the cloud successfully if the team lacks the skills to operate it, if governance guardrails are missing, and if leadership cannot see a return on the investment. Cloud adoption is an organisational transformation, not just a technical migration.
