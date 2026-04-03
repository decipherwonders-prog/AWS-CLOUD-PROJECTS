#  AWS Hands-On Cloud Projects

I recently completed two hands-on AWS cloud projects focused on building and securing real-world infrastructure. These projects helped me strengthen my understanding of networking, security, load balancing, and deployment troubleshooting.

---

## **Project 1: High-Availability Static Website**
##    ARCHITECTURE
<img width="1536" height="1024" alt="ChatGPT Image Apr 3, 2026, 05_05_20 PM" src="https://github.com/user-attachments/assets/3f09fdc6-99ea-4300-ab89-2cf713dd74e1" />


**Objective:** Host a static website on AWS that is highly available and secure.

### **Step-by-Step Breakdown**

1. **Create an S3 Bucket**
   - Set up an S3 bucket to store website files.
   - Enabled versioning to keep track of changes.
   - Initially set the bucket to private to prevent unauthorized access.
     <img width="1901" height="990" alt="Screenshot 2026-04-03 171102" src="https://github.com/user-attachments/assets/5ee19d05-b1bb-444c-ab2c-37e2d7475285" />

2. **Upload Website Content**
   - Uploaded HTML, CSS, and assets into the bucket.
   - Verified access locally using the S3 endpoint.
     <img width="1891" height="979" alt="Screenshot 2026-04-03 171328" src="https://github.com/user-attachments/assets/e823bc32-4cf4-47e7-9421-17212b4f894d" />


3. **Configure CloudFront Distribution**
   - Created a CloudFront distribution with the S3 bucket as the origin.
   - Enabled **Origin Access Control (OAC)** so the bucket remained private, but content could still be served globally.
   - Configured caching and edge locations for optimal performance.
     <img width="1898" height="1008" alt="Screenshot 2026-04-03 171627" src="https://github.com/user-attachments/assets/f56170b3-e33f-492a-a8bf-ef9838a1a96a" />


4. **Set up HTTPS**
   - Configured SSL using Amazon-provided certificates for secure HTTPS delivery.
   - Tested the site via CloudFront domain to ensure proper encryption.
     <img width="1920" height="1080" alt="Screenshot 2026-03-18 174311" src="https://github.com/user-attachments/assets/e981ce52-3222-4a0d-b7d0-f6515855384b" />


5. **Testing & Verification**
   - Verified content was delivered globally via CloudFront.
   - Confirmed bucket remained private—direct S3 URL returned an access denied error.
     <img width="1920" height="1080" alt="Screenshot 2026-03-18 174401" src="https://github.com/user-attachments/assets/69dea749-0d6e-4ffa-b4d5-6f504d8e6a93" />


**Key Learning Points:**
- How to integrate S3 with CloudFront for high availability.
- The importance of OAC for security in static site hosting.
- End-to-end HTTPS configuration for static websites.

---

## **Project 2: Fortress Web Server**

**Objective:** Build a secure, highly available web server using EC2, Nginx, and a Load Balancer.
##   ARCHITECTURE
<img width="1536" height="1024" alt="ChatGPT Image Apr 3, 2026, 05_25_01 PM" src="https://github.com/user-attachments/assets/973231d6-60d4-4764-815d-db09de81dfa9" />

### **Step-by-Step Breakdown**

1. **VPC & Subnet Design**
   - Created a custom VPC for network isolation.
   - Configured private and public subnets to separate internet-facing and internal resources.
   - Set up a NAT gateway to allow private instances to access the internet for updates.
<img width="1902" height="1009" alt="Screenshot 2026-03-18 175018" src="https://github.com/user-attachments/assets/f4c475e9-a941-45b0-b7ad-ce0814d0b2e9" />

2. **Launch EC2 Instance**
   - Launched an EC2 instance in a private subnet using a launch template.
   - Installed Nginx for serving web content.
     <img width="1920" height="1020" alt="Screenshot 2026-03-18 183124" src="https://github.com/user-attachments/assets/c4fe2295-3b30-4c41-b02a-3826eb5335dc" />

   - Initially forgot to start the web service, which caused the target group in the ALB to mark the instance unhealthy.
<img width="1900" height="991" alt="Screenshot 2026-03-18 174748" src="https://github.com/user-attachments/assets/c9f73202-cda7-4311-9acb-8626c088648e" />

3. **Configure Security Groups**
   - Created security groups for EC2 to allow only HTTP/HTTPS traffic from the ALB.
   - ALB security group allowed traffic from the public internet on ports 80 and 443.
   - Ensured SSH access was restricted to my IP.
<img width="1893" height="1008" alt="Screenshot 2026-04-03 173522" src="https://github.com/user-attachments/assets/f8cb2533-3c2e-4a66-8c82-37655901a701" />

4. **Set up Application Load Balancer (ALB)**
   - Created an ALB to route traffic to EC2 instances.
   - Configured target groups pointing to the EC2 instance.
   - Set up listeners for HTTP → HTTPS redirection.
<img width="1875" height="995" alt="Screenshot 2026-04-03 173810" src="https://github.com/user-attachments/assets/2cf8462e-3928-4153-a0a5-802f0bab1f1c" />

5. **Fix Deployment Issue**
   - Detected unhealthy targets in the ALB.
   - Realized Nginx was not running on EC2.
   - Installed and configured Nginx properly → target group became healthy.
<img width="1872" height="1010" alt="Screenshot 2026-04-03 174202" src="https://github.com/user-attachments/assets/99ccfee1-132c-4cc9-a06a-aacbb78462db" />

6. **Enable Auto Scaling**
   - Configured an Auto Scaling Group (ASG) for EC2 instances.
   - ASG automatically launched additional instances under load and terminated unhealthy ones.
     <img width="1920" height="1017" alt="Screenshot 2026-03-21 172339" src="https://github.com/user-attachments/assets/b3bee633-a4da-495b-aee1-ca927e253acb" />


**Key Learning Points:**
- Hands-on experience with VPC design and subnetting.
- Load balancer configuration and traffic flow understanding.
- Security group management for secure access.
- Troubleshooting real-world deployment issues under time constraints.

---

**Conclusion:**  

Both projects reinforced my understanding of AWS infrastructure, security, high availability, and deployment best practices. The experience of troubleshooting and debugging real issues was particularly valuable.
