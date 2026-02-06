**Task 1 & 2: AWS Well-Architected Framework (WAF) Assessment**

| Pillar | Observation | Improvement | Supporting AWS Service |
| :---- | :---- | :---- | :---- |
| **Operational Excellence** | Manual deployments are prone to human error. | Implement Infrastructure as Code (IaC) for consistent environments. | **AWS CloudFormation**  |
| **Security** | Open security groups increase the attack surface. | Implement the Principle of Least Privilege for network traffic. | **AWS Security Groups / IAM**  |
| **Reliability** | Single-AZ deployment creates a single point of failure. | Distribute the workload across multiple Availability Zones. | **Multi-AZ Deployment / ELB**  |
| **Performance Efficiency** | Static resources served from the web server can slow down load times. | Use a content delivery network to serve assets closer to users. | **Amazon CloudFront**  |
| **Cost Optimization** | Fixed-size servers may lead to overprovisioning and wasted spend. | Scale resources automatically based on demand. | **AWS Auto Scaling**  |

**Task 3: AWS Cloud Adoption Framework (CAF) Summary**

### **Business Perspective**

The Business perspective focuses on ensuring that cloud investments accelerate digital transformation and deliver clear business value. For this migration, the primary enabler is aligning the technical move with business KPIs such as reduced operational costs and increased service availability. We must move from a "capital expenditure" model of buying servers to an "operational expenditure" model. By leveraging AWS, the organization can pivot faster to market demands without waiting for physical hardware procurement. Success will be measured by how much we reduce downtime and how quickly we can deploy new features to customers compared to our previous on-premises environment.

### 

### **People Perspective**

The People perspective bridges the gap between technology and personnel, focusing on culture, training, and change management. A successful migration requires a workforce that is comfortable with cloud-native tools. Our key action is to implement a comprehensive upskilling program, focusing on AWS certifications and hands-on labs. We need to evolve traditional IT roles into cloud-focused roles, such as Cloud Engineers and Site Reliability Engineers (SREs). By fostering a culture of continuous learning and addressing potential skill gaps early, we ensure the team can effectively manage the new Multi-AZ environment and utilize automated scaling tools.

### 

### **Governance Perspective**

Governance in the cloud focuses on minimizing risk and maximizing the business value of cloud investments. For this workload, we must establish automated guardrails using services like AWS Organizations and Service Control Policies (SCPs). Key actions include implementing a strict tagging strategy for cost allocation and setting up AWS Budgets to prevent cost overruns. We will also define clear roles and responsibilities through RBAC (Role-Based Access Control) to ensure that only authorized personnel can modify critical infrastructure. This perspective ensures that as we scale, we maintain compliance with internal policies and external regulations without slowing down development.

**Platform Perspective**

The Platform perspective focuses on the technical architecture and delivery of cloud services. Our goal is to move away from manually managed servers toward a scalable, automated platform. This involves implementing Infrastructure as Code (IaC) to ensure our environments are reproducible and consistent across development and production. By utilizing the Multi-AZ architecture shown in our design, we are building a resilient platform that can withstand infrastructure failures automatically. We will focus on optimizing our "landing zone" to provide a secure and high-performance environment for the two-tier application, ensuring the network topology supports both speed and security.

### 

### **Security Perspective**

The Security perspective is critical for protecting the organization's data and infrastructure in the cloud. We are shifting from a perimeter-based security model to a "Zero Trust" approach. Key enablers include the implementation of fine-grained Security Groups and IAM roles that follow the principle of least privilege. We will also automate security monitoring using services like AWS CloudTrail and GuardDuty to detect and respond to threats in real-time. By encrypting data at rest in our Amazon RDS instances and data in transit through SSL/TLS on our Application Load Balancer, we ensure that our migration aligns with modern security best practices and compliance requirements.

### 

### **Operations Perspective**

The Operations perspective focuses on ensuring that cloud services are delivered at a level that meets the needs of the business. This involves a shift from reactive troubleshooting to proactive observability. We will implement Amazon CloudWatch to monitor system health and set up automated alerts for any performance degradation. A key action for this migration is automating our incident response—for example, using Auto Scaling to handle traffic spikes without manual intervention. By focusing on operational excellence, we ensure that the application remains highly available and that our team spends less time on "keeping the lights on" and more time on innovation.

