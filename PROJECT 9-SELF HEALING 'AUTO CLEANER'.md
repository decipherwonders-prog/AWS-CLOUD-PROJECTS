#  3-Tier VPC Macro + Custom Resource Project

##  Overview

This project demonstrates the design and implementation of a **CloudFormation Macro-driven infrastructure system**, evolving from simple automation into a **fully functional 3-tier VPC architecture**.

The journey covers:

* Macro-based infrastructure transformation
* Custom resource integration
* Real network validation using EC2
* Production-style VPC design (Public + Private + NAT)

---

##  How It All Started

The project began with a simple idea:

*Can we enforce infrastructure rules automatically instead of relying on developers to remember them?*

This led to building a **CloudFormation Macro** that:

* Automatically modifies templates
* Injects compliance configurations

From there, the scope expanded into:

* Dynamic infrastructure generation
* Real network architecture
* System-level validation

---
## ARCHITECTURE
<img width="1536" height="1024" alt="Cloudformation" src="https://github.com/user-attachments/assets/7e17e3e9-0f9f-47bd-bb2d-d74780d2be19" />

##  Phase Breakdown

###  Phase 1 — S3 Compliance Macro

Built a macro that:

* Detects `AWS::S3::Bucket`
* Injects:

  * Lifecycle rules (auto-delete after 7 days)
  * Tags
  * Ownership controls
  * Public access block
<img width="1899" height="996" alt="Screenshot 2026-04-29 193949" src="https://github.com/user-attachments/assets/8f31280e-617a-4199-9432-25f9d0c8778e" />
<img width="1883" height="835" alt="Screenshot 2026-04-29 194026" src="https://github.com/user-attachments/assets/27c37199-0a09-41ed-89f0-a9601262dab4" />
<img width="1898" height="828" alt="Screenshot 2026-04-30 112748" src="https://github.com/user-attachments/assets/e554fe6d-c7a9-430b-8b0e-f2e9a4d40f92" />
<img width="1377" height="926" alt="Screenshot 2026-04-30 120926" src="https://github.com/user-attachments/assets/58e91d24-9f48-42fd-a9d5-08376192e116" />

####  Outcome

* Developers could create buckets with **minimal config**
* Macro enforced compliance automatically
<img width="1250" height="961" alt="Screenshot 2026-04-30 174826" src="https://github.com/user-attachments/assets/6b9fe7dc-a5a2-444b-9806-722c4af44a2a" />
<img width="1892" height="906" alt="Screenshot 2026-04-30 234528" src="https://github.com/user-attachments/assets/65231012-575f-43cb-be76-d1347f034143" />

---

###  Phase 2 — Macro Debugging & Validation

Initial failures revealed critical issues:

####  Challenges

* Macro not being invoked (missing `Transform`)

* Name mismatches (`MyComplianceMacro` vs actual name)
<img width="1821" height="732" alt="Screenshot 2026-05-04 010848" src="https://github.com/user-attachments/assets/0424f0cf-8d32-4985-b653-19e24f9efdff" />

* Schema validation errors (`ID` vs `Id`)
<img width="1898" height="684" alt="Screenshot 2026-04-30 112748" src="https://github.com/user-attachments/assets/35224fb8-ccfe-4930-9db7-ad803ab58649" />

####  Fixes

* Correct macro naming
<img width="1894" height="1003" alt="Screenshot 2026-05-04 113417" src="https://github.com/user-attachments/assets/271c554f-8ae0-4c0f-a99d-5e92fbd290c3" />

* Proper template transformation usage
* Strict adherence to CloudFormation schema

####  Lesson

CloudFormation is extremely strict, small schema errors break everything.

---

###  Phase 3 — Custom Resource (Quote Fetcher)
<img width="1427" height="721" alt="Screenshot 2026-05-01 131641" src="https://github.com/user-attachments/assets/cb4a5912-ec5d-4c6d-ae66-5758633c10d8" />

Built a Lambda-backed custom resource that:
* Calls an external API
* Returns data into CloudFormation Outputs
<img width="882" height="641" alt="Screenshot 2026-05-04 003425" src="https://github.com/user-attachments/assets/50f4c603-8c55-42c8-8c39-eabc4bc770cb" />

####  Challenges

* Stack hanging (no response sent)
* Missing response schema fields
* API failures

####  Fixes

* Implemented reliable `send_response()`
* Ensured consistent response structure across all paths
* Added fallback handling

####  Lesson

 Custom Resources are contract-driven; consistency matters more than logic.

---

###  Phase 4 — Macro Expansion (VPC Generation)

Shifted from enforcement to **abstraction**.

Input:

```yaml
Type: Custom::SimpleVPC
```
<img width="1813" height="373" alt="Screenshot 2026-05-04 120632" src="https://github.com/user-attachments/assets/7b164cd8-088b-4f6f-83a6-fc8119216b82" />
<img width="1408" height="612" alt="Screenshot 2026-05-04 124646" src="https://github.com/user-attachments/assets/36cc39c3-9096-4736-803e-15928efaf81d" />

Output:

* VPC
* Subnets
* Internet Gateway
* Route Tables
<img width="1874" height="1019" alt="Screenshot 2026-05-04 125241" src="https://github.com/user-attachments/assets/2d6d9cc2-0ffc-4e0f-a9b2-2c081ef7a2f2" />

####  Challenges

* Modifying dictionary during iteration
* Macro not updating due to Lambda not redeployed

####  Fixes

* Used `list(resources.items())` for safe mutation
* Proper Lambda deployment workflow

---

###  Phase 5 — Public + Private Subnet Design

Expanded architecture into:

* Public subnet (internet-facing)
<img width="1864" height="999" alt="Screenshot 2026-05-05 070651" src="https://github.com/user-attachments/assets/1076f132-49a0-4b41-bb3b-4e4551163940" />
<img width="1896" height="517" alt="Screenshot 2026-05-05 070826" src="https://github.com/user-attachments/assets/a201a21d-910d-42d6-a9bb-02fbc094ff0a" />

* Private subnet (isolated)
<img width="1865" height="993" alt="Screenshot 2026-05-05 070716" src="https://github.com/user-attachments/assets/590b8e8f-c011-457c-953b-067d0fb7db12" />

####  Challenge

* Understanding correct routing separation

####  Outcome

* Public subnet → Internet Gateway
* Private subnet → No internet access

---

###  Phase 6 — NAT Gateway (Real 3-Tier)

Introduced:

* Elastic IP
<img width="1559" height="441" alt="Screenshot 2026-05-04 141328" src="https://github.com/user-attachments/assets/89df7ec8-59e9-4111-80dc-2d13bffa6658" />

* NAT Gateway
<img width="1906" height="802" alt="Screenshot 2026-05-04 141017" src="https://github.com/user-attachments/assets/c607ba43-8d7a-4581-afba-febd4e7381b9" />

* Private route via NAT
<img width="1893" height="502" alt="Screenshot 2026-05-05 070758" src="https://github.com/user-attachments/assets/1a67538b-9845-49db-9bdb-bc15cabc9ee7" />

####  Outcome

| Layer          | Behavior               |
| -------------- | ---------------------- |
| Public Subnet  | Direct internet access |
| Private Subnet | Outbound-only via NAT  |

####  Lesson

NAT enables functionality without sacrificing security.

---

###  Phase 7 — Real Validation with EC2

Launched EC2 in private subnet and tested:
<img width="1875" height="1012" alt="Screenshot 2026-05-04 171405" src="https://github.com/user-attachments/assets/2609bc50-47b6-4d97-8d20-eead996dd47b" />

```bash
curl https://api.github.com
```

####  Result

* Successful response confirmed:

  * NAT routing works
  * No direct exposure
  * Full network functionality
<img width="929" height="875" alt="Screenshot 2026-05-04 175529" src="https://github.com/user-attachments/assets/22cb6023-a481-4328-bf8c-084881df17b5" />

####  Lesson

Infrastructure is only valid when traffic flows correctly.

---

##  Architecture Summary

```
                Internet
                    │
            ┌───────▼───────┐
            │ Internet GW   │
            └───────┬───────┘
                    │
         ┌──────────▼──────────┐
         │ Public Subnet       │
         │ (NAT + IGW access)  │
         └──────────┬──────────┘
                    │
            ┌───────▼───────┐
            │ NAT Gateway   │
            └───────┬───────┘
                    │
         ┌──────────▼──────────┐
         │ Private Subnet      │
         │ (EC2, internal)     │
         └─────────────────────┘
```

---

##  Key Achievements

* Built a **CloudFormation Macro engine**
* Automated **S3 compliance enforcement**
* Implemented **Custom Resource integration**
* Designed a **3-tier VPC architecture**
* Validated real-world network behavior using EC2

---

##  Key Challenges Faced

* Macro not triggering due to missing Transform
* Lambda updates not reflecting in CloudFormation
* Schema validation failures (case sensitivity)
* Custom Resource response failures
* Debugging asynchronous CloudFormation behavior

---

##  Conclusion

This project demonstrates a complete journey from:

**Simple automation → Full infrastructure system design → Real-world validation**

The system is not just built—it’s **proven to work under real conditions**.
