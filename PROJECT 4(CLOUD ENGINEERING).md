#   Highly Available Nginx Server with SSL & Autoscaling

This project was a hands-on journey to build a secure, highly available web server on AWS using Nginx, an SSL certificate, and Auto Scaling. The goal was to create a production-ready environment that could handle traffic spikes while keeping the site secure.

---
## ARCHITECTURE
<img width="1408" height="768" alt="Gemini_Generated_Image_54hv6a54hv6a54hv" src="https://github.com/user-attachments/assets/2cd6b7f6-fb90-42fc-a952-10360d30b00a" />

## **Project Goals**

1. Launch a web server (EC2) running Nginx.
2. Install and configure an SSL certificate for HTTPS.
3. Purchase an SSL certificate from a trusted provider.
4. Ensure high availability using a Load Balancer.
5. Enable Auto Scaling to handle dynamic traffic.

---

## **Step-by-Step Breakdown**

### **Step 1: Launching the Server**
- **Action:** Created an EC2 instance with a launch template.
- **Decision:** Chose a private subnet for security but configured a NAT gateway for internet access.
- **Challenge:** Deciding between public and private subnet while maintaining connectivity for updates.
- **Success:** EC2 launched with all necessary ports (SSH for management, HTTP/HTTPS for web traffic) correctly configured in the security group.
- **Lesson Learned:** Planning your network architecture upfront avoids headaches later with Load Balancers and SSL.
  <img width="1855" height="994" alt="Screenshot 2026-04-03 145857" src="https://github.com/user-attachments/assets/52790303-bd44-4cdd-a271-d109d8ab02dc" />


---

### **Step 2: Installing Nginx**
- **Action:** Installed Nginx using `sudo apt install nginx` and configured default site.
- **Challenge:** Ensuring Nginx served content correctly over both HTTP and HTTPS.
- **Success:** Nginx served a test page successfully.
- **Lesson Learned:** Always test the web server locally before integrating with Load Balancer and SSL.
<img width="1883" height="1006" alt="Screenshot 2026-04-03 150054" src="https://github.com/user-attachments/assets/decaaaca-b8e0-4a3d-94e5-1286bb3433ef" />

---

### **Step 3: Purchasing & Installing SSL Certificate**
- **Action:** Bought SSL from a provider and installed it on Nginx.
- **Challenge:** Ensuring the certificate matched the domain and configuring Nginx to force HTTPS.
- **Success:** The site successfully redirected HTTP → HTTPS.
- **Lesson Learned:** Misconfigured SSL or mismatched certificates can break the site completely. Double-check the domain and certificate chain.
<img width="1917" height="1009" alt="Screenshot 2026-04-03 150436" src="https://github.com/user-attachments/assets/2a587e46-9512-4525-a53b-745245092d49" />

---

### **Step 4: Setting Up Load Balancer**
- **Action:** Created an AWS Application Load Balancer (ALB) to distribute traffic across multiple instances.
- **Challenge:** Properly configuring HTTPS listeners, target groups, and security groups.
- **Success:** Load Balancer accepted traffic and correctly routed it to healthy EC2 instances.
- **Lesson Learned:** Listener rules and health checks are critical—one wrong setting can make all instances appear unhealthy.
<img width="1876" height="1000" alt="Screenshot 2026-04-03 150823" src="https://github.com/user-attachments/assets/2841c760-e9ee-497b-a1fc-6207ca7f82ec" />

---

### **Step 5: Configuring Auto Scaling**
- **Action:** Set up an Auto Scaling Group (ASG) using the launch template.
- **Challenge:** Ensuring subnets, security groups, and health checks were correctly aligned.
- **Success:** The ASG automatically launched new instances under load and terminated unhealthy ones.
- **Lesson Learned:** Autoscaling only works reliably if your infrastructure (subnets, security groups, health checks) is correctly configured from the start.
<img width="1878" height="1006" alt="Screenshot 2026-04-03 150951" src="https://github.com/user-attachments/assets/148578ee-6c2f-42c0-8f61-50c21c5ed60b" />

---

## **Challenges Encountered**
1. SSL misconfigurations causing browsers to flag the site as insecure.
2. Security group misalignment preventing the Load Balancer from reaching EC2 instances.
3. Autoscaling misfires when health checks were improperly set.

---

## **Successes Achieved**
- Secure site with HTTPS enforced.
  <img width="1893" height="925" alt="Screenshot 2026-03-27 214741" src="https://github.com/user-attachments/assets/5f3040fd-ceb5-4606-94d1-291fc798e5ef" />

- High availability through Load Balancer.
- Auto Scaling handling dynamic traffic efficiently.
  <img width="1920" height="1080" alt="Screenshot 2026-03-27 210236" src="https://github.com/user-attachments/assets/857f754a-c1cd-4e31-ab7b-d201a9140c72" />

- Improved understanding of AWS networking, Nginx, and SSL management.

---

## **Key Lessons Learned**
1. **Network planning matters:** Subnets, NAT gateways, and security groups must align for high availability.
2. **Test often:** Validate each step (Nginx, SSL, Load Balancer) before moving to the next.
3. **Automation is king:** Using launch templates and ASG saves headaches during scaling events.
4. **Patience + debugging:** AWS can be tricky, but careful testing and logging prevent hidden failures.

---
