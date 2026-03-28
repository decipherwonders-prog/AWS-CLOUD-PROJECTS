# CLOUD ENGINEERING-PROJECT 1&2
 This repository contains two hands-on AWS cloud projects completed as part of my cloud engineering learning journey.

PROJECT 1: High-Availability Static Website (Production-Oriented Design)
Overview

Designed and deployed a globally distributed static website using Amazon S3 and CloudFront, with a strong focus on security, performance, and controlled content delivery.

**Architecture Decisions (and Why They Exist)**

The site assets (HTML/CSS) are stored in a private S3 bucket, deliberately blocking all public access. Instead of exposing S3 directly, CloudFront is used as the only entry point, enforcing a clean separation between storage and delivery.

Access between CloudFront and S3 is secured using Origin Access Control (OAC), ensuring that content can only be fetched through the CDN layer—not directly from the bucket. This eliminates a common misconfiguration risk where S3 objects are accidentally exposed.

**Performance & Caching Strategy**

Rather than relying on default behavior, caching was explicitly considered:

Configured CloudFront cache policies to control TTL (Time-To-Live)

Used cache versioning (e.g., app.v2.css) to avoid excessive invalidations

Reserved CloudFront invalidations for critical updates only

This approach reduces latency globally while maintaining control over content freshness.

**Security Enhancements**

Security was treated as a first-class concern:

S3 Block Public Access enabled at bucket level

CloudFront Viewer Protocol Policy set to enforce HTTPS

TLS configuration aligned with modern security standards

(Optional but recommended) AWS WAF integration for:

Rate limiting

Basic bot protection

Layer 7 filtering

This ensures the attack surface is minimized and traffic is controlled at the edge.

Observability & Debugging

To avoid blind spots:

Enabled CloudFront access logging

Configured S3 access logs

Integrated CloudWatch metrics and alerts for:

4xx/5xx error spikes

Unusual traffic patterns

This allows rapid diagnosis of delivery or configuration issues.

Failure Considerations

The system is designed with resilience in mind:

CloudFront edge caching reduces dependency on origin availability

Versioned deployments prevent partial rollout failures

Rollbacks can be executed by reverting object versions

Outcome

Achieved a secure, low-latency, globally distributed static site where:

The origin remains fully private

All access is funneled through a controlled CDN layer

Performance and security are both optimized

 **Project 2: Fortress Web Server (Hardened, Scalable Architecture)
Overview**

Built a secure, highly available web application architecture inside a custom VPC, ensuring compute resources remain isolated while still being reliably accessible through a controlled public interface.

Network Architecture

A custom VPC was designed with clear separation of concerns:

Public subnet: hosts the Application Load Balancer (ALB)

Private subnets: host EC2 instances (web servers)

The EC2 instances are intentionally not internet-facing. All inbound traffic flows through the ALB, enforcing a single controlled entry point.

Secure Access Strategy

Direct SSH access was eliminated in favor of:

AWS Systems Manager Session Manager (SSM)

No open port 22

IAM-based access control

Fully auditable sessions

This removes a major attack vector and aligns with modern cloud security practices.

Internet Access for Private Instances

To allow outbound access (e.g., updates, package installs):

Configured a NAT Gateway in the public subnet

Private subnet route tables direct outbound traffic through NAT

This ensures instances can reach external services without being exposed inbound.

**Load Balancing & TLS**

The Application Load Balancer (ALB) serves as the public interface:

HTTPS enforced using AWS Certificate Manager (ACM)

TLS termination handled at the ALB

HTTP traffic redirected to HTTPS

Health checks were tuned with:

Custom endpoint (/health)

Defined intervals, timeouts, and thresholds

This ensures only healthy instances receive traffic.

**Auto Scaling Strategy**

Auto Scaling was configured based on real demand signals, not defaults:

Scaling policies tied to:

CPU utilization

Request count per target

Minimum and maximum capacity defined to control cost and resilience

This allows the system to dynamically adapt to traffic spikes without overprovisioning.

Instance Hardening

Beyond just launching EC2:

Minimal packages installed

Web server (Nginx) configured via user data automation

Regular updates enabled via outbound access

Security groups strictly limited:

Only ALB can reach instances on HTTP/HTTPS

No public inbound access

Observability & Monitoring

To maintain visibility:

**CloudWatch metrics and alarms for:
**
CPU utilization

ALB 5xx errors

Target health status

Logs collected for troubleshooting and audit

**COST CONSIDERATIONS
**
Tradeoffs were acknowledged:

NAT Gateway introduces cost but improves security posture

ALB provides flexibility but may be overkill for low-traffic apps

Auto Scaling helps balance cost vs availability

Failure & Resilience Testing

The system was designed to handle:

Instance failure → Auto Scaling replaces unhealthy nodes

Traffic spikes → Scaling policies adjust capacity

Deployment issues → Instances can be replaced without downtime

Outcome

Delivered a production-style architecture where:

Compute resources are fully isolated

All access is controlled and encrypted

The system is resilient, scalable, and observable

**SCREENSHOTS**
For Project 1:
HTML/CSS files uploaded to an S3 bucket
<img width="1920" height="1080" alt="Screenshot 2026-03-18 175125" src="https://github.com/user-attachments/assets/03800f67-8048-49eb-ba54-e5f2253f770d" /> 

Cloudfront Distribution making the site load instantly
<img width="1920" height="1080" alt="Screenshot 2026-03-18 174311" src="https://github.com/user-attachments/assets/168491c1-2a87-476b-8b13-43d1feb56774" />

For Project 2:
A custom VPC with a public and private subnet
<img width="1920" height="1080" alt="Screenshot 2026-03-18 175018" src="https://github.com/user-attachments/assets/c0258cff-5f09-4486-8e46-665b37908f54" />

EC2 Instance launched in the public subnet
<img width="1920" height="1080" alt="Screenshot 2026-03-18 174748" src="https://github.com/user-attachments/assets/144a0138-495e-4f51-b6cf-20da9cc14580" />


Nginx web server installed
<img width="1920" height="1080" alt="Screenshot 2026-03-18 183124" src="https://github.com/user-attachments/assets/3245b935-b9bb-42d5-8da1-e6298bef3bf2" />

A Configured Security Group that only allows traffic on port 80(HTTP) and port 22(SSH)
<img width="1920" height="1080" alt="Screenshot 2026-03-18 184018" src="https://github.com/user-attachments/assets/a83bc073-fdfb-4378-a626-355c191ca9ef" />

Authors
@decipherwonders-prog on Github
https://www.linkedin.com/in/desmond-gyau-5a53382b4?utm_source=share_via&utm_content=profile&utm_medium=member_ios 
