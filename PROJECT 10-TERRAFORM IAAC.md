#  Enterprise-Grade Multi-Environment Web Cluster (Terraform)

##  Overview

This project demonstrates how to design and deploy a **production-style, scalable, and fault-tolerant web infrastructure** on AWS using **Terraform**.

Instead of building a single EC2 instance, this system evolves into a **modular, multi-environment architecture** with:

* Remote state management (S3 + DynamoDB)
* Reusable Terraform modules
* Multi-AZ networking
* Auto Scaling (self-healing infrastructure)
* Application Load Balancer (ALB)

---
##   ARCHITECTURE
<img width="1536" height="1024" alt="Terraform-Infrastructure As Code" src="https://github.com/user-attachments/assets/ac957071-4a6c-4499-a029-889f8fcd95b2" />

##  Objective

To build a system capable of:

* Deploying **Dev, Staging, and Production environments**
* Scaling automatically under load
* Recovering from failures without manual intervention
* Maintaining clean, reusable Terraform code

---

##   Project Structure

```
terraform-enterprise-cluster/
├── modules/
│   ├── vpc/
│   ├── ec2/
│   ├── security/
│   └── alb/
├── environments/
│   ├── dev.tfvars
│   └── prod.tfvars
├── main.tf
├── variables.tf
├── backend.tf
└── outputs.tf
```
---

##  Step 1 — Remote State (Foundation)

### Problem

Local Terraform state is fragile and unsafe for teams.

### Solution

* **S3 Bucket** → Stores Terraform state
<img width="1318" height="213" alt="Screenshot 2026-05-07 114552" src="https://github.com/user-attachments/assets/cb3d2112-a103-4e5c-92b2-72e647412d00" />
<img width="1330" height="87" alt="Screenshot 2026-05-07 114619" src="https://github.com/user-attachments/assets/f048a520-347f-4ed4-938f-5c17da313472" />
<img width="1918" height="976" alt="Screenshot 2026-05-07 115336" src="https://github.com/user-attachments/assets/34f93dc1-40d7-4794-9f63-04c847711dbf" />
<img width="1299" height="709" alt="Screenshot 2026-05-07 115543" src="https://github.com/user-attachments/assets/0aade150-eee3-4bb8-bb3a-eee73fa507e1" />
<img width="1466" height="528" alt="Screenshot 2026-05-08 101621" src="https://github.com/user-attachments/assets/4839256a-25bd-4631-94f1-4f7bbb9636a5" />

* **DynamoDB Table** → Enables state locking
<img width="958" height="277" alt="Screenshot 2026-05-07 172034" src="https://github.com/user-attachments/assets/9fd5975e-03dd-443a-81a0-163bc6feb68a" />
<img width="1919" height="592" alt="Screenshot 2026-05-08 101702" src="https://github.com/user-attachments/assets/a362a894-842c-41aa-b1f8-9fd31dfb184e" />

### Result

* Prevents state corruption
* Enables team collaboration

---

##  Step 2 — VPC Module (Networking Layer)

### Components

* VPC
<img width="1917" height="1007" alt="Screenshot 2026-05-07 180055" src="https://github.com/user-attachments/assets/a1cf7cbf-c36e-4b80-92e3-0ce589fd4617" />
<img width="1707" height="928" alt="Screenshot 2026-05-07 181047" src="https://github.com/user-attachments/assets/15c023cd-fd97-42c8-a5f0-3077cfb02aff" />
<img width="1868" height="1015" alt="Screenshot 2026-05-07 182427" src="https://github.com/user-attachments/assets/538440c3-941a-4e91-9fde-5276c6d0e152" />

* Public Subnets (Multi-AZ)
* Internet Gateway
* Route Tables
<img width="1842" height="671" alt="Screenshot 2026-05-07 182455" src="https://github.com/user-attachments/assets/209496cb-9de3-4e2f-a667-1f052793af85" />

### Key Concept

Dynamic subnet creation using:

```
count = length(var.public_subnets)
```

### Outcome

* Multi-AZ network
* Internet connectivity

---

##  Step 3 — Security Module

### Security Group Rules

| Port | Access      | Purpose      |
| ---- | ----------- | ------------ |
| 80   | 0.0.0.0/0   | HTTP traffic |
| 22   | Your IP /32 | SSH access   |
<img width="1919" height="969" alt="Screenshot 2026-05-07 183528" src="https://github.com/user-attachments/assets/c2649229-40f4-4063-8ab5-dd6240367d17" />
<img width="1221" height="496" alt="Screenshot 2026-05-07 183631" src="https://github.com/user-attachments/assets/69f32526-73ec-4c98-805d-19842399db6b" />

### Key Insight

Restricting SSH access prevents:

* Brute force attacks
* Unauthorized access

---

##  Step 4 — EC2 (Initial Deployment)

### First Approach

* Single EC2 instance
<img width="953" height="374" alt="Screenshot 2026-05-07 215055" src="https://github.com/user-attachments/assets/0b3b5abd-e3a5-4385-ac8b-fa09545dda11" />
<img width="1204" height="550" alt="Screenshot 2026-05-07 215330" src="https://github.com/user-attachments/assets/0507b0f8-5de6-4e08-b202-3147cef1f1fc" />
<img width="1160" height="485" alt="Screenshot 2026-05-07 215921" src="https://github.com/user-attachments/assets/2e6442d2-672d-461d-af92-2fcc078faba0" />
<img width="1173" height="484" alt="Screenshot 2026-05-07 220208" src="https://github.com/user-attachments/assets/e41c37d0-287e-47cd-97f5-5888d9e7f35e" />

* Apache web server via `user_data`
<img width="949" height="1008" alt="Screenshot 2026-05-07 221557" src="https://github.com/user-attachments/assets/509e0753-edb8-4395-a92b-57b98e73e2a1" />

### Result

* Application accessible via public IP
<img width="1706" height="932" alt="Screenshot 2026-05-07 222804" src="https://github.com/user-attachments/assets/21adb1cd-7878-4892-8329-007a319f7f6f" />

### Limitation

* Single point of failure
* No scalability

---

##  Step 5 — Auto Scaling Transformation

### Upgrade

Replaced EC2 with:

* **Launch Template**
<img width="1225" height="544" alt="Screenshot 2026-05-08 080921" src="https://github.com/user-attachments/assets/ebe01ba8-81ff-47b9-9b5e-726be926d574" />
<img width="1880" height="963" alt="Screenshot 2026-05-08 083327" src="https://github.com/user-attachments/assets/3c659e1e-af50-44da-8ffc-90035fc3bb14" />

* **Auto Scaling Group (ASG)**
<img width="961" height="822" alt="Screenshot 2026-05-08 082424" src="https://github.com/user-attachments/assets/b260033b-2aad-4aa4-8bfb-5c43aab007f5" />
<img width="957" height="871" alt="Screenshot 2026-05-08 082528" src="https://github.com/user-attachments/assets/3b3baaa0-da3f-490d-a8b6-9c7830a1f5d3" />
<img width="960" height="989" alt="Screenshot 2026-05-08 083145" src="https://github.com/user-attachments/assets/b2d06ddf-cf25-448d-a049-048e300ac1b8" />

### Configuration

* Desired capacity: 2
* Min: 1
* Max: 3

### Outcome

* Self-healing system
<img width="1876" height="972" alt="Screenshot 2026-05-08 083353" src="https://github.com/user-attachments/assets/24f47e30-11fd-4ef4-859a-48832b5761e2" />

* Automatic instance replacement
<img width="1496" height="441" alt="Screenshot 2026-05-08 083457" src="https://github.com/user-attachments/assets/647b3905-4e7c-40db-b769-ff3367458001" />

### Trade-off

* No single public IP

---

##  Step 6 — Load Balancer (Critical Upgrade)

### Problem

Multiple instances → no stable access point

### Solution

Application Load Balancer (ALB)
<img width="954" height="564" alt="Screenshot 2026-05-08 083905" src="https://github.com/user-attachments/assets/2403de02-2557-49da-920c-85cbc1b1c34f" />

### Components

* ALB
<img width="1899" height="778" alt="Screenshot 2026-05-08 092435" src="https://github.com/user-attachments/assets/08070086-92f7-4def-97a3-c88a76da6fb2" />
<img width="967" height="1002" alt="Screenshot 2026-05-08 091242" src="https://github.com/user-attachments/assets/bf421543-ed02-4a50-9fea-e1dc1ee1b401" />

* Target Group
<img width="1887" height="773" alt="Screenshot 2026-05-08 092713" src="https://github.com/user-attachments/assets/eba71810-b244-4901-ae4b-2cac35525d8d" />
<img width="1896" height="612" alt="Screenshot 2026-05-08 092732" src="https://github.com/user-attachments/assets/3a36a6cb-5a47-4760-99f2-9c6fe4546dd8" />

* Listener (HTTP:80)
<img width="1888" height="562" alt="Screenshot 2026-05-08 092449" src="https://github.com/user-attachments/assets/7d3d9f9b-dbbf-491e-b46f-55c916bcee0c" />


### Traffic Flow

```
User → ALB → Target Group → ASG → EC2 Instances
```

### Result

* Single entry point
* Load distribution
* High availability
<img width="1702" height="891" alt="Screenshot 2026-05-08 092105" src="https://github.com/user-attachments/assets/e1350e07-5151-4584-b4b8-b08c99616210" />

---

##  Key Concepts Learned

### 1. Modular Architecture

* Reusable components
* Clean separation of concerns

### 2. Infrastructure as Code

* Version-controlled infrastructure
* Repeatable deployments

### 3. High Availability

* Multi-AZ deployment
* Auto recovery

### 4. System Design Thinking

Shift from:

```
Single Server → Distributed System
```

---

##  Challenges Faced

### 1. Missing Module Outputs

* Could not reference VPC/subnets
* Fixed by exposing outputs

### 2. Incorrect Variable Wiring

* Root vs module confusion
* Learned input/output chaining

### 3. Silent Terraform Failures

* "No changes" due to empty modules
* Emphasized verification over assumptions

---

