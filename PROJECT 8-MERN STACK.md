# MERN Task Manager – Full Project Breakdown

##  What This Project Is About

This project is a **cloud-ready MERN task management system** designed to simulate how modern applications are built, containerized, and prepared for deployment in real-world environments.
At its core, it allows users to manage tasks (create, read, update, delete). But beyond functionality, the real objective is architectural:

* Build a full-stack application that is **environment-independent**
* Ensure consistency using **Docker containerization**
* Design infrastructure using **AWS VPC principles** (isolation, availability, security)
* Separate concerns between **application logic and infrastructure**

In simple terms:

> This is not just a task manager, it is a controlled experiment in building software that can survive outside a local machine.

---
##   Architecture
<img width="1536" height="1024" alt="Mern-Stack" src="https://github.com/user-attachments/assets/5cd7f767-bdea-4b75-acce-7f5ad2e7d20f" />

##  Phase 1: The Application (MERN Stack)

Before touching infrastructure, the **core engine** had to be built and validated.

### MongoDB (Database Layer)

* Used **MongoDB Atlas (DB-as-a-Service)**
* Avoided local database dependency
* Enabled cloud-native architecture from the start
<img width="1896" height="1060" alt="Screenshot 2026-04-22 161115" src="https://github.com/user-attachments/assets/293eebdb-ea42-46d9-80d9-70252733f8cf" />
<img width="1886" height="1012" alt="Screenshot 2026-04-22 161135" src="https://github.com/user-attachments/assets/0fd7a902-f7a5-4727-af41-e8b6733d5d01" />

**Key Insight:** Externalizing the database early removes migration pain later.

---

### Backend (Node.js + Express)

A REST API was implemented with the following endpoints:

```
POST   /tasks
GET    /tasks
PUT    /tasks/:id
DELETE /tasks/:id
```
<img width="1820" height="948" alt="Screenshot 2026-04-24 113448" src="https://github.com/user-attachments/assets/1ce31862-ad48-421d-ae96-7e4cf699a3e7" />
<img width="1914" height="846" alt="Screenshot 2026-04-24 113918" src="https://github.com/user-attachments/assets/c865a402-765e-4474-9a24-105bef0b3b5b" />

Responsibilities:

* Handle business logic
* Validate incoming data
* Interface with MongoDB

---

### Frontend (React + Vite)

* Built using **React + Vite**
* Communicates with backend via HTTP
* Dynamically renders task data

Core features:

* Create tasks
* View tasks
* Update tasks
* Delete tasks
<img width="1343" height="940" alt="Screenshot 2026-04-25 164345" src="https://github.com/user-attachments/assets/aa5bbeb8-a1b7-4af9-933a-b9c2d7fdcb25" />
<img width="1908" height="766" alt="Screenshot 2026-04-25 170424" src="https://github.com/user-attachments/assets/5e2806a5-9a4c-4df4-b0fb-0fbd72d87911" />
<img width="1600" height="934" alt="Screenshot 2026-04-26 115547" src="https://github.com/user-attachments/assets/0d29e091-5b3d-41bc-a515-c62de5d2641e" />
<img width="1677" height="1004" alt="Screenshot 2026-04-27 185644" src="https://github.com/user-attachments/assets/dbce457a-38a3-41c5-b7c1-f9ef7f42b49e" />

---

### Containerization (Critical Layer)

Both frontend and backend were dockerized.
<img width="1400" height="712" alt="Screenshot 2026-04-26 155205" src="https://github.com/user-attachments/assets/29b1d023-e75f-4889-a7a2-7140a72e9ca1" />
<img width="1607" height="692" alt="Screenshot 2026-04-26 155225" src="https://github.com/user-attachments/assets/6303fa41-fed8-4f9a-8d93-b4031bd34371" />
<img width="953" height="547" alt="Screenshot 2026-04-27 094104" src="https://github.com/user-attachments/assets/82011e20-b4f2-4c28-925c-5424b286dc53" />
<img width="1856" height="808" alt="Screenshot 2026-04-27 100641" src="https://github.com/user-attachments/assets/0ec99881-ac9e-4892-8565-1c45b4bac2c7" />
<img width="1883" height="886" alt="Screenshot 2026-04-27 104621" src="https://github.com/user-attachments/assets/37e8b977-fb70-4ce8-b91d-44e89a43c429" />
<img width="765" height="602" alt="Screenshot 2026-04-27 102251" src="https://github.com/user-attachments/assets/b637b84a-028f-41c9-bc47-632be8a50c04" />
<img width="1901" height="908" alt="Screenshot 2026-04-27 120008" src="https://github.com/user-attachments/assets/eb0a2086-f4fd-4600-89f2-7019ef2165af" />
<img width="1904" height="1005" alt="Screenshot 2026-04-27 120112" src="https://github.com/user-attachments/assets/96fe10d9-6342-42bd-889e-acd2540f09e4" />
<img width="1230" height="780" alt="Screenshot 2026-04-27 120857" src="https://github.com/user-attachments/assets/0782c202-c27c-41b2-954c-264c691bf6f7" />

```
docker compose up --build
```
<img width="1906" height="976" alt="Screenshot 2026-04-27 183151" src="https://github.com/user-attachments/assets/9f728dda-77d7-49ec-9331-ec238a34acd4" />
<img width="1901" height="967" alt="Screenshot 2026-04-27 183128" src="https://github.com/user-attachments/assets/f56df344-3abb-4d52-a75a-3b153f1071f5" />
<img width="1907" height="953" alt="Screenshot 2026-04-27 184059" src="https://github.com/user-attachments/assets/91cca887-c798-45a9-9473-782399fd616f" />
<img width="1902" height="997" alt="Screenshot 2026-04-27 184933" src="https://github.com/user-attachments/assets/96bb8899-3601-4ac6-a875-32cd6396b3b2" />
<img width="1898" height="895" alt="Screenshot 2026-04-27 185045" src="https://github.com/user-attachments/assets/8ba33d6c-d7dc-42f9-bc7b-5912c288fbdf" />

**Why this matters:**

* Eliminates environment inconsistencies
* Standardizes runtime
* Makes deployment reproducible

If it’s not containerized, it’s not reliable.

---

##  Phase 2: The Network Fortress (AWS VPC)

This phase answers a different question:

> Where does the system live, and how is it protected?

---

### VPC Setup

* Custom VPC (e.g., `10.0.0.0/16`)
* Full control over network boundaries
<img width="1863" height="1004" alt="Screenshot 2026-04-27 231416" src="https://github.com/user-attachments/assets/b783c5b5-266e-4d48-86ab-0aef597aade9" />

---

### Subnet Architecture

**Public Subnets (2):**

* Internet-facing components
* NAT Gateway placement
<img width="1887" height="1000" alt="Screenshot 2026-04-28 001911" src="https://github.com/user-attachments/assets/b8f3d6d7-18dd-4769-9759-9a432cef3e51" />

**Private Subnets (2):**

* Backend layer (secure)
* No direct internet access
<img width="1876" height="726" alt="Screenshot 2026-04-28 001941" src="https://github.com/user-attachments/assets/d7b8ffad-bd1b-4217-b2a9-7286bc6fe695" />

---

### High Availability

* Subnets distributed across **2 Availability Zones**
* Reduces single points of failure

---

### NAT Gateway

* Located in public subnet
* Enables outbound internet access for private resources

**Use cases:**

* Installing dependencies
* Pulling updates
* External API communication

---

### Security Model

* Public access restricted to controlled entry points
* Backend isolated in private subnets
* Outbound traffic routed through NAT

---

##  Challenges

### 1. Docker Networking Misconception

Incorrect:

```
mongodb://localhost:27017/tasks
```

Correct:

```
mongodb://mongo:27017/tasks
```

**Problem:** Containers don’t share localhost.

---

### 2. Port Misconfiguration

* Used wrong port (`5173` instead of `3000`)
* Result: App failed to load
<img width="1673" height="990" alt="Screenshot 2026-04-25 165847" src="https://github.com/user-attachments/assets/f47be9d5-8468-40b5-beff-bdbfb7305c2f" />

---

### 3. Cloud Networking Complexity

Understanding relationships between:

* Subnets
* Route tables
* Internet Gateway
* NAT Gateway

required deeper architectural thinking.

---

##  Successes

### 1. Fully Functional MERN Stack

* CRUD operations working
* Clean frontend-backend integration
<img width="1677" height="1004" alt="Screenshot 2026-04-27 185644" src="https://github.com/user-attachments/assets/f899d2a6-b48d-4f97-ae3d-0bf11d54db49" />

---

### 2. Stable Containerized Environment

* Consistent builds
* Isolated services

---

### 3. Working Service Communication

```
mongodb://mongo:27017/tasks
```
<img width="1882" height="908" alt="Screenshot 2026-04-28 115700" src="https://github.com/user-attachments/assets/5b734bb6-c4f7-49ef-bfb9-ec9f1f4ed33f" />

Backend successfully connects to database.

---

### 4. Production-Oriented Design

* Network isolation
* High availability mindset
* Managed database usage

---

##  Lessons Learned

### 1. Scope Expansion Changes Everything

A simple app becomes complex when designed for real-world deployment.

---

### 2. Most Issues Are Configuration Problems

Not code-related:

* Ports
* Hostnames
* Networking

---

### 3. Infrastructure Is Part of Development
If it can’t be deployed, it’s incomplete.

---

### 4. Isolation Improves Security

Private subnets + NAT Gateway = controlled exposure.

---

### 5. Docker Is Foundational

Without it:

* Environments drift
* Bugs become inconsistent
* Deployment breaks

---

##  Final Takeaway

This project represents a shift in thinking:

```
From: Writing code that works
To: Designing systems that can run anywhere
```

That shift is what separates basic development from real engineering.
