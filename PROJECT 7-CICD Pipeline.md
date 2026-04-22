##  CI/CD Pipeline (Production-Style)

##  Objective

Build a complete CI/CD pipeline:

Code Push → Jenkins Build → Docker Image → Deploy (EC2 → ECS Fargate)

The goal was not just automation, but **consistency and reliability across environments**.

---
 ## Architecture
<img width="1536" height="1024" alt="Jenkins Pipeline Image Apr 21, 2026, 12_40_57 PM" src="https://github.com/user-attachments/assets/9a65fb55-6e77-48c6-8c60-ceebbd6ce6d6" />

## Phase 1 — Infrastructure

### Setup
- EC2 Instance:
  - Type: `t3.medium`
  - OS: `Amazon Linux`
<img width="1879" height="999" alt="Screenshot 2026-04-18 093809" src="https://github.com/user-attachments/assets/39751351-c27d-4317-8da2-e4a25ccf6906" />

- Security Group:
  - `22` → SSH  
  - `8080` → Jenkins  
  - `80` → Application  
<img width="1884" height="1004" alt="Screenshot 2026-04-21 231534" src="https://github.com/user-attachments/assets/bb458181-2bc4-4584-aa5d-1436947ddf08" />

### Challenges
- Initial port misconfigurations blocked access
- Underestimating resource needs for Jenkins + Docker

### Successes
- Stable base environment
- Proper network exposure

### Lessons Learned
- Infrastructure mistakes cascade into bigger problems later
- Always verify ports early

---

##  Phase 2 — Core Installation

### Setup (Strict Order)
1. Install Java
<img width="1906" height="979" alt="Screenshot 2026-04-18 163644" src="https://github.com/user-attachments/assets/d14a3fa9-7649-478d-90e0-11faf01eca29" />

3. Install Jenkins
<img width="1898" height="945" alt="Screenshot 2026-04-18 164053" src="https://github.com/user-attachments/assets/cc652d3f-a732-4c37-80f6-5adc8f791b9b" />

5. Install Docker  
<img width="1899" height="963" alt="Screenshot 2026-04-19 171051" src="https://github.com/user-attachments/assets/3dfca0ee-b641-4063-aacc-12bf692119ee" />

Additional:
- Add Jenkins to Docker group
- Restart services

### Challenges
- Permission issues running Docker from Jenkins
- Order dependency caused setup failures

### Successes
- Jenkins successfully running Docker commands
<img width="1898" height="787" alt="Screenshot 2026-04-18 170202" src="https://github.com/user-attachments/assets/3cc036dd-a776-4b05-96a7-fa03d2e3860b" />

### Lessons Learned
- Installation order matters more than expected
- Permissions are silent blockers in CI/CD systems

---

##  Phase 3 — Jenkins Setup

### Setup
- Access Jenkins via browser
<img width="965" height="905" alt="Screenshot 2026-04-18 172241" src="https://github.com/user-attachments/assets/30bc59e7-ecc0-4596-8a43-2f8eb1be0e38" />

- Unlock using initial admin password
<img width="970" height="913" alt="Screenshot 2026-04-18 182252" src="https://github.com/user-attachments/assets/c8f2127b-5512-4fb3-a860-06188fd8565b" />

- Install plugins
<img width="962" height="905" alt="Screenshot 2026-04-18 182658" src="https://github.com/user-attachments/assets/e24d4397-fb23-420d-ae72-f31a9a3e2206" />

- Create admin user
<img width="936" height="906" alt="Screenshot 2026-04-18 182743" src="https://github.com/user-attachments/assets/a530618c-abbe-4e02-aace-e1470df59418" />
<img width="1898" height="1079" alt="Screenshot 2026-04-18 183640" src="https://github.com/user-attachments/assets/808305fa-5da5-4821-b7a9-3e52ddcbd752" />
<img width="1895" height="1058" alt="Screenshot 2026-04-18 183729" src="https://github.com/user-attachments/assets/87eb339b-ca73-4b75-9a67-e76125cb3963" />

### Challenges
- Plugin delays

### Successes
- Fully operational Jenkins environment
<img width="1897" height="885" alt="Screenshot 2026-04-19 160037" src="https://github.com/user-attachments/assets/e0e6c3ec-4d33-4330-be36-51851d47bcee" />

### Lessons Learned
- Always verify service status during setup

---

##  Phase 4 — GitHub Integration

### Setup
- Add GitHub credentials in Jenkins
<img width="1908" height="923" alt="Screenshot 2026-04-18 194223" src="https://github.com/user-attachments/assets/a8d35e36-fb58-435e-8b02-fdc1da4b9d42" />

- Configure webhook:
http://<EC2-IP>:8080/github-webhook/
<img width="1893" height="983" alt="Screenshot 2026-04-19 200113" src="https://github.com/user-attachments/assets/706a77fa-952c-4958-8168-93c811202c12" />

### Challenges
- Webhook not triggering builds
- Credential misconfiguration
<img width="1185" height="653" alt="Screenshot 2026-04-19 001109" src="https://github.com/user-attachments/assets/a6e0f97d-d83f-4b7d-a38a-d508b923b23a" />

### Successes
- Automatic builds on code push
<img width="1849" height="933" alt="Screenshot 2026-04-19 201332" src="https://github.com/user-attachments/assets/c05f4a1c-86d3-4bed-8130-f8e066011027" />

### Lessons Learned
- Webhooks fail silently, always check delivery logs

---

##  Phase 5 — Pipeline (Jenkinsfile)

### Setup
Pipeline stages:
- Checkout
- Build (Docker)
- Test
- Deploy (run container on EC2)
<img width="1868" height="1073" alt="Screenshot 2026-04-21 224757" src="https://github.com/user-attachments/assets/6a76f44a-99c1-4d9b-9dd4-6a8fb2304a93" />

### Challenges
- Jenkinsfile syntax errors
- `server.js` missing inside container
- Incorrect Docker CMD (`npm` vs `node`)
<img width="1863" height="885" alt="Screenshot 2026-04-21 234315" src="https://github.com/user-attachments/assets/8452cef0-f143-4a6b-b1af-59603515320b" />

### Successes
- Successful Docker builds
- Application runs locally in container

### Lessons Learned
- Build success ≠ runtime success
- Docker context must be verified
- Always test containers before deployment

---

##  Phase 6 — Production Upgrade (ECR + ECS)

### Setup
- Create ECR repository
<img width="1886" height="1003" alt="Screenshot 2026-04-19 204017" src="https://github.com/user-attachments/assets/11df5228-643f-4af2-b3e8-55224388ac2c" />

- Configure IAM permissions
<img width="1872" height="972" alt="Screenshot 2026-04-19 210336" src="https://github.com/user-attachments/assets/110c30ca-a0e3-4985-b46c-132284975c5e" />

- Push Docker images to ECR
 <img width="1889" height="948" alt="Screenshot 2026-04-20 113216" src="https://github.com/user-attachments/assets/342f0a46-6f41-46d0-9be1-4182abba4714" />

- Deploy via ECS (Fargate)
<img width="1885" height="858" alt="Screenshot 2026-04-20 120216" src="https://github.com/user-attachments/assets/fa7e8522-f074-4d27-9996-a9013d5ec1d9" />

### Challenges

#### 1. ECS Task Failures
- Tasks stopped immediately
- Error:
Essential container exited (exitCode: 1)
<img width="1874" height="549" alt="Screenshot 2026-04-21 125837" src="https://github.com/user-attachments/assets/4e7c8acc-2a3f-4159-9f15-6d0168bb3375" />


#### 2. Missing Files at Runtime
- `server.js` existed in repo but not in container
<img width="1919" height="889" alt="Screenshot 2026-04-21 094441" src="https://github.com/user-attachments/assets/3d5b8d60-e397-4ca8-9cc7-870ff9e37844" />

#### 3. Version Drift (Critical)
- Jenkins built new images
- ECS ran old image (`my-app:27`)

#### 4. False Confidence
- CI pipeline passed
- Deployment still failed

### Successes
- Images successfully pushed to ECR
- Local container execution validated runtime
- Root cause identified (image mismatch)

### Lessons Learned
- ECS does NOT automatically use latest images
- Image tagging strategy is critical
- CI must include runtime validation

---

##  Debugging Breakthrough

Added runtime validation inside Jenkins:


docker run --rm my-app:latest node server.js


### Impact
- Verified container behavior before deployment
- Shifted debugging focus to ECS configuration

### Lesson
If runtime isn’t tested in CI, deployment is guesswork.

---

##  Final Architecture


GitHub → Jenkins → Docker → ECR → ECS (Fargate)


---

##  Key Takeaways

- CI/CD is about **consistency**, not just automation  
- Version control of images is critical  
- Runtime validation is mandatory  
- Debugging must be done layer by layer  

---
