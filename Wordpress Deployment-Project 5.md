#  Highly Available WordPress Deployment on AWS

##  Project Overview
This project started as a simple goal:
 “Deploy WordPress on AWS.”

It quickly turned into something much bigger. Building a **distributed, highly available system** that actually behaves correctly under load.

Instead of a single server, I ended up working with:
- Multiple EC2 instances
- A load balancer (ALB)
- A managed database (RDS)
  <img width="1895" height="1000" alt="Screenshot 2026-04-09 204732" src="https://github.com/user-attachments/assets/9f680cc6-9df5-43e5-9625-75a3aeee5276" />

- Shared storage (EFS)
- Cloudflare for DNS, SSL, and caching

What looked simple at first forced me to understand how real systems behave, not just how to install software.

---

##  Architecture
<img width="1536" height="1024" alt="ChatGPT Image Apr 14, 2026, 08_44_18 AM" src="https://github.com/user-attachments/assets/5657c8d0-5aa0-4e83-a381-6baf04f698a1" />


---

##  Implementation Journey

### Phase 1: Setting Up the Foundation

I started by setting up:
- VPC and subnets
  <img width="1875" height="1008" alt="Screenshot 2026-04-06 220627" src="https://github.com/user-attachments/assets/033279d8-e25d-41e0-b2f0-9e00991b957b" />
  <img width="1873" height="931" alt="Screenshot 2026-04-06 220602" src="https://github.com/user-attachments/assets/64f82302-3440-49b9-9245-41f5551b0a58" />

- Security groups
  <img width="1515" height="691" alt="Screenshot 2026-04-14 085613" src="https://github.com/user-attachments/assets/e13d706f-4687-42a6-ac82-16fecb92f3b6" />

- EC2 instances with Nginx, PHP, and WordPress
  <img width="1604" height="731" alt="Screenshot 2026-04-14 085937" src="https://github.com/user-attachments/assets/2c33116d-f2b8-4597-99f3-29dc1a376d59" />

**What went wrong:**
At first, things just didn’t connect. Services couldn’t talk to each other.
 
**What I realized:**
I was treating security groups like simple “open ports,” but they actually define **which services are allowed to communicate**.

---

### Phase 2: Introducing the Load Balancer

I added an Application Load Balancer and registered my EC2 instances.
<img width="1866" height="993" alt="Screenshot 2026-04-09 204641" src="https://github.com/user-attachments/assets/3ec541f9-1bdf-4a9b-a077-29a2137aee12" />

**What went wrong:**
- Targets showed as *unhealthy*
- The site didn’t respond through the ALB

**What fixed it:**
- Correcting the outbound rule of my ec2's security group
<img width="1871" height="1010" alt="Screenshot 2026-04-11 105628" src="https://github.com/user-attachments/assets/f43ab518-40d1-461b-8d9f-f3f162240e0b" />

- Making sure Nginx was actually serving the right content
<img width="1783" height="832" alt="Screenshot 2026-04-07 141649" src="https://github.com/user-attachments/assets/40261265-7911-43a4-a741-cd1347972eb5" />

**Lesson:**
If your health checks are wrong, your entire system *looks* broken—even when it isn’t.

---

## Phase 3: WordPress Installation & Configuration

What I did:
Installed and configured WordPress manually on the EC2 instances.

Steps:

# Navigate to web root
cd /var/www

# Download WordPress
sudo wget https://wordpress.org/latest.tar.gz

# Extract files
sudo tar -xvzf latest.tar.gz

# Set correct permissions
sudo chown -R nginx:nginx /var/www/wordpress
sudo chmod -R 755 /var/www/wordpress

Configured WordPress:

cd /var/www/wordpress
cp wp-config-sample.php wp-config.php
nano wp-config.php

Updated database details:

define('DB_NAME', 'your_db_name');
define('DB_USER', 'your_db_user');
define('DB_PASSWORD', 'your_db_password');
define('DB_HOST', 'your_rds_endpoint');

Nginx Configuration:

server {
    listen 80;
    server_name _;

    root /var/www/wordpress;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass unix:/run/php-fpm/www.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}

What went wrong:

Multiple WordPress directories caused confusion
Unsure which directory Nginx was serving

Fix:
Confirmed correct root:

root /var/www/wordpress;

Lesson learned:
Never assume your app path. Always verify what your web server is actually serving.
## Results
<img width="1920" height="1080" alt="Screenshot 2026-04-08 174306" src="https://github.com/user-attachments/assets/e2cd8f24-67a8-4cda-b03e-dca54a6487a7" />
<img width="1920" height="1080" alt="Screenshot 2026-04-09 191917" src="https://github.com/user-attachments/assets/fd36f27d-2499-4a3d-86dc-bce567ec7b35" />
<img width="1920" height="1080" alt="Screenshot 2026-04-09 195319" src="https://github.com/user-attachments/assets/c097daf3-6f70-407f-bb2a-f4baa5b7fb7e" />
<img width="1920" height="1080" alt="Screenshot 2026-04-09 195432" src="https://github.com/user-attachments/assets/f065a68d-7921-4417-8510-372c25c189ed" />


## Phase 4: EFS (Shared Storage)
What I did:

## Mounted EFS on EC2
 <img width="1895" height="985" alt="Screenshot 2026-04-09 204824" src="https://github.com/user-attachments/assets/91c9b738-e99f-45e8-8e75-f823038d1738" />
 
What went wrong:
Even after mounting EFS, WordPress wasn’t using it.

Fix:
Linked wp-content to EFS:

ln -s /mnt/efs /var/www/wordpress/wp-content

Lesson learned:

Mounting storage is not enough. Your application must actively use it.

## Phase 5: Cloudflare & SSL Issues

What I did:

Connected domain through Cloudflare
<img width="1432" height="613" alt="Screenshot 2026-04-14 104033" src="https://github.com/user-attachments/assets/db2561f6-7040-4b9a-9226-25491431cb17" />

What went wrong:

Redirect loops
Security warnings
<img width="1725" height="1060" alt="Screenshot 2026-04-14 104327" src="https://github.com/user-attachments/assets/ef94d5ff-bcf2-48d4-b5f0-960826bac31f" />

Fix:

Updated WordPress config:
if (isset($_SERVER['HTTP_X_FORWARDED_PROTO']) && $_SERVER['HTTP_X_FORWARDED_PROTO'] === 'https') {
    $_SERVER['HTTPS'] = 'on';
}

Lesson learned:
Reverse proxies require proper handling of forwarded headers.

## Phase 6: Login Issues

What went wrong:
Couldn’t log into /wp-admin
<img width="1714" height="1068" alt="Screenshot 2026-04-14 105716" src="https://github.com/user-attachments/assets/22d338a7-50bb-498f-8fa3-a1d6bf32e5dd" />

Root cause:
Session/cookie issues due to HTTPS handling

Fix:
define('FORCE_SSL_ADMIN', true);

Lesson learned:
Login issues are often session problems, not credential problems.

## Final Validation
Uploaded media 
Refreshed multiple times 
Tested across browsers 
<img width="1920" height="1080" alt="Screenshot 2026-04-14 004212" src="https://github.com/user-attachments/assets/18f76ee7-07e9-4255-a1f5-9e91f20b384e" />

Everything remained consistent.

Key Takeaways
Distributed systems fail inconsistently
Load balancers expose hidden issues
Shared storage is critical
Proxy layers must be handled carefully

## Achievements
Multi-instance high availability
Load balancing via ALB
Shared storage via EFS
Centralized database (RDS)
Domain + SSL via Cloudflare

This project started simple but became a lesson in:

handling state
managing distributed systems
ensuring consistency

There’s a big difference between “it works” and “it works reliably.”
