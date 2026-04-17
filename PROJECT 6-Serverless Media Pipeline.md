#   Smart Media Pipeline 

## Overview

This project is a serverless, event-driven image processing pipeline built on AWS. It automates the ingestion, analysis, transformation, and notification of uploaded images using fully managed cloud services.

This project was designed to demonstrate real-world cloud architecture principles: event-driven workflows, IAM security, service integration, and debugging across distributed systems.

---

## How It Started

The goal was straightforward:

Build a system where uploading an image automatically triggers processing, AI analysis, storage, and notification without manual intervention.

---

## Architecture Flow

1. Image uploaded to **S3 Input Bucket**
2. S3 triggers **Lambda function**
3. Lambda:

   * Extracts file info
   * Sends image to Rekognition
4. Rekognition returns labels
5. Lambda copies image to **Output Bucket**
6. Lambda publishes message to **SNS Topic**
7. SNS sends email notification
<img width="1536" height="1024" alt="AWS image recognition architecture diagram" src="https://github.com/user-attachments/assets/681f7b8d-deb7-46d9-86d3-52ad9d42fada" />
     
## Resources Created

This project involved multiple AWS resources working together:

###  1. Amazon S3 Buckets

Two buckets were created to avoid recursive triggers:

* **Input Bucket** 

  * Stores uploaded images
  * Configured with **event notification**
<img width="1878" height="985" alt="Screenshot 2026-04-16 075630" src="https://github.com/user-attachments/assets/31e5b140-29d4-4a9f-95ab-9aecfe7c3da1" />
<img width="1889" height="997" alt="Screenshot 2026-04-14 232020" src="https://github.com/user-attachments/assets/fc6edfff-7e1e-48a4-b063-0a0dd83e807f" />

* **Output Bucket** 

  * Stores processed images
  * Prevents infinite loop by separating from input
<img width="1890" height="1009" alt="Screenshot 2026-04-16 075612" src="https://github.com/user-attachments/assets/cf85a0cc-258b-4971-b773-7b644d1c0ace" />


---

###  2. AWS Lambda Function

* Runtime: **Python**
* Acts as the **core processing engine**
* Triggered automatically by S3 events
<img width="1894" height="669" alt="Screenshot 2026-04-14 202501" src="https://github.com/user-attachments/assets/c64cc34b-dfce-478d-84e3-5fead04db01e" />

Responsibilities:

* Extract object metadata
* Validate file type
* Call Rekognition
* Copy processed file
* Send SNS notification
<img width="1877" height="942" alt="Screenshot 2026-04-14 202932" src="https://github.com/user-attachments/assets/c7ea33cd-8f35-4535-928c-20a13dc43eb3" />

---

###  3. IAM Role (Lambda Execution Role)

A dedicated IAM role was created and attached to Lambda with:

* `AmazonS3FullAccess`
* `AmazonRekognitionFullAccess`
* `CloudWatchLogsFullAccess`
<img width="1874" height="994" alt="Screenshot 2026-04-14 200053" src="https://github.com/user-attachments/assets/d140b6bf-12ed-4b42-9edb-fe8700c4b817" />

---

###  4. Amazon Rekognition

Used for:

* Image analysis
* Label detection (e.g., objects, scenes)
  
Integrated directly inside Lambda.
<img width="1885" height="827" alt="Screenshot 2026-04-17 075607" src="https://github.com/user-attachments/assets/1b061158-52e8-4128-8374-34ef4d3efc1d" />

---

###  5. Amazon SNS (Simple Notification Service)

* Topic created: `ImageSuccess`
<img width="1894" height="1009" alt="Screenshot 2026-04-14 200448" src="https://github.com/user-attachments/assets/528a1589-cc9d-442e-8137-f35dc1cbc728" />

* Email subscription configured
<img width="1890" height="1003" alt="Screenshot 2026-04-14 200851" src="https://github.com/user-attachments/assets/cfdf60ce-83f1-43d1-874c-a535fcca4b89" />

Used for:
* Sending real-time notifications after processing

---

###  6. CloudWatch Logs
<img width="1903" height="813" alt="Screenshot 2026-04-17 075705" src="https://github.com/user-attachments/assets/12325da6-5ebd-4b8a-a2ba-76c0c26ed05a" />

* Automatically logs Lambda execution
* Used heavily for debugging and tracing failures
  
---

## Key Challenges Faced

### 1. Misleading Debugging Signals

Initial assumption:

Lambda was not working

Reality:

* Lambda was executing correctly
* Issue existed in downstream service interaction

---

### 2. SNS Notification Failure

Even after:

* Successful uploads
* Valid label detection
* Correct S3 output

 No email was received

This created a false assumption that:

The Lambda function or code was broken

---

## Root Cause Analysis
The real issue was:
**AWS permissions were blocking Lambda from publishing to SNS**
<img width="921" height="398" alt="Screenshot 2026-04-16 195903" src="https://github.com/user-attachments/assets/8a357a69-5d62-4117-b252-3f3bf0828fc6" />

This was not a code problem, it was an **IAM misconfiguration**.

---

## Resolution

The fix was applied at the IAM level:

### Action Taken:

* Created an **inline policy** in the Lambda execution role
* Granted:

  ```
  sns:Publish
  ```
* Scoped to the specific SNS topic ARN
<img width="1920" height="1017" alt="Screenshot 2026-04-16 221632" src="https://github.com/user-attachments/assets/fe5b6048-d8f3-4640-9147-ee8fcc07e40f" />


---

## Final Outcome

After applying the fix:

* Image uploads trigger Lambda reliably
* Rekognition processes images successfully
* Output images appear in the output bucket
* SNS sends notifications instantly
<img width="1453" height="713" alt="Screenshot 2026-04-16 215453" src="https://github.com/user-attachments/assets/cfe9face-5847-4260-ad4b-44bce78021b5" />

The system now operates as a **fully functional event-driven pipeline**

---

## Key Learnings

* Most cloud failures are **permission-related, not code-related**
* Debugging requires isolating each service layer:
* CloudWatch logs are critical for tracing execution flow
* Event-driven systems must be validated step-by-step, not as a whole
* Small configuration mistakes (IAM, ARNs, naming) can break entire pipelines

---

## Technologies Used

* Amazon S3
* AWS Lambda (Python)
* Amazon Rekognition
* Amazon SNS
* IAM Roles & Policies

---

## Project Status

 Completed:
 
 Fully functional,
  debugged and production-stable architecture, 
  that demonstrates real-world AWS integration and troubleshooting.
