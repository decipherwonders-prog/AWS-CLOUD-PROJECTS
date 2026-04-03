  # RESUME API (SERVERLESS)

 # ARCHITECTURE
  <img width="1135" height="944" alt="Gemini_Generated_Image_wg0cnpwg0cnpwg0c(1)" src="https://github.com/user-attachments/assets/a8c87f74-b0c6-479c-9d0b-835750bb6517" />

The goal was simple on paper: build an API that serves my resume and tracks views. In reality, it forced me to understand how serverless systems actually behave.


# STEP 1: DynamoDB — The Data Vault

## What I Did

* Created a table: `CloudResumeTable`
  <img width="1876" height="992" alt="Screenshot 2026-04-01 164918" src="https://github.com/user-attachments/assets/68cd7ea9-7893-42ff-ab0f-93ea525eba9e" />

* Partition key: `ID (String)`
* Inserted initial record:

  * `ID: 1`
  * `Name`
  * `Views: 0`
  * (Optionally: Skills, Education, Experience)
    <img width="1873" height="1004" alt="Screenshot 2026-04-01 183302" src="https://github.com/user-attachments/assets/42bb6255-7901-4f0c-9f83-6b51f63ebd9b" />

## The Real Challenge

DynamoDB looks simple until you realize it’s not relational. There’s no joins, no structure safety net. If your data model is trash, everything downstream becomes painful.

Also, the idea of using a single item (`ID = 1`) felt too basic at first, but that’s the point. This isn’t about complexity, it’s about efficiency.

### What Worked

* Using a single partition key kept reads and writes extremely fast
* No schema enforcement meant I could evolve the resume structure easily

### Lessons Learned

* NoSQL forces you to think in **access patterns**, not tables
* Overengineering at this stage is a mistake, keep it tight and intentional
* DynamoDB is brutally efficient if you respect its design philosophy


## Step 2: AWS Lambda — The Brain

### What I Did

* Created a Lambda function (Python 3.12)
* Wrote logic to:

  1. Increment the `Views` count
  2. Fetch the resume data
  3. Return it as JSON

### The Code (Core Logic)

```python
import json
import boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('CloudResumeTable')

def lambda_handler(event, context):
    response = table.update_item(
        Key={'ID': '1'},
        UpdateExpression='ADD Views :inc',
        ExpressionAttributeValues={':inc': 1},
        ReturnValues="UPDATED_NEW"
    )

    resume_data = table.get_item(Key={'ID': '1'})

    return {
        'statusCode': 200,
        'headers': {
            'Content-Type': 'application/json',
            'Access-Control-Allow-Origin': '*'
        },
        'body': json.dumps(resume_data['Item'])
    }
```
<img width="1874" height="1004" alt="Screenshot 2026-04-01 165126" src="https://github.com/user-attachments/assets/0ecebe8a-adb2-46bd-b058-46430948fb41" />


### The Real Challenge

Lambda forces you to think stateless. There is no “server running in the background.” Everything must execute fast and clean in one shot.

Also, the `update_item` expression is not intuitive at first. If you mess that up, your counter either won’t increase or will break silently.

### What Worked

* Using `ADD Views :inc` avoids race conditions for simple increments
* Keeping logic minimal reduced execution time and cost

### Lessons Learned

* Serverless rewards simplicity; the less code, the better
* You don’t “manage servers,” you manage **execution flow**
* If your function is doing too much, you’re already doing it wrong

---

## Step 3: IAM — Permissions (Where Most People Mess Up)
<img width="1878" height="615" alt="Screenshot 2026-04-03 124414" src="https://github.com/user-attachments/assets/c738a45c-c67b-4484-93cd-05f357e3d84d" />

### What I Did

* Attached `AmazonDynamoDBFullAccess` to the Lambda execution role

### The Real Challenge

This is where things silently fail. Your code can be perfect, but if IAM is wrong, nothing works. And AWS won’t hold your hand.

### What Worked

* Granting full access made testing fast and frictionless

### Lessons Learned

* IAM is not optional knowledge; it’s core infrastructure
* In production, full access is reckless. Principle of least privilege is the real move
* If something isn’t working, always suspect permissions early

---

## Step 4: API Gateway — The Front Door

### What I Did

* Created a Http API
* Added `/get` resource
* Connected GET method to Lambda
* Enabled CORS
* Deployed to `$default` (auto-deploy)

  <img width="1895" height="997" alt="Screenshot 2026-04-03 125934" src="https://github.com/user-attachments/assets/e0302939-7499-4af4-8bb3-d34e6ea8cb67" />


### The Real Challenge

API Gateway is powerful but not beginner-friendly. Small misconfigurations break everything:

* Forget CORS → browser blocks requests
* Wrong integration → endpoint fails silently

### What Worked

* Clean resource structure (`/get`)
* Proper deployment stage (`$default`) for stable endpoint

### Lessons Learned

* API Gateway is glue; if it’s misconfigured, your entire system looks broken
* CORS is not optional if you plan to call this from a frontend
* Deployment stages matter, don’t just click buttons blindly

---

## Step 5: The Problem — Raw JSON Isn’t a Product

### What Happened

When I hit the API URL in the browser, I got my resume… but in raw JSON format.

That’s technically correct but practically useless for any real user.

No structure. No design. No readability. Just data.
<img width="1918" height="1003" alt="Screenshot 2026-04-01 195301" src="https://github.com/user-attachments/assets/6ae6721c-b4b7-4335-990d-7eee67497793" />

### The Realization

This is where most people stop and think they’re done. They’re not.

An API is not a user experience. It’s just a data pipe.

If a non-technical person opens your “resume” and sees JSON, you’ve already failed the usability test.

---

## Step 6: S3 — Turning Data Into a Real Interface

### What I Did

* Created an S3 bucket 
* Enabled static website hosting
* Built a simple HTML/CSS frontend
* Used JavaScript (fetch API) to call the API Gateway endpoint
* Dynamically rendered the resume data on the page
<img width="1893" height="1011" alt="Screenshot 2026-04-03 131254" src="https://github.com/user-attachments/assets/e59e03d8-b61b-4111-9000-eff45f0d82da" />

### The Real Challenge

This is where things connect, and break:

* If CORS wasn’t enabled earlier, the browser blocks the API call
* If the API response isn’t clean JSON, rendering fails
* If your JS logic is weak, the page loads but shows nothing

Also, S3 permissions can still mess you up with “Access Denied” if not configured properly.

### What Worked

* Separating frontend (S3) from backend (API + Lambda) kept the architecture clean
* Using `fetch()` made the integration straightforward

### Lessons Learned

* Raw data means nothing without presentation
* S3 + API Gateway is a powerful pattern for modern web apps
* Frontend is not optional — it’s what people actually interact with

---

## Step 7: Testing — The Reality Check

### What I Did

* Opened the S3-hosted resume site
* Verified data loads dynamically from API
* Refreshed page and confirmed `Views` increments
<img width="1920" height="1080" alt="Screenshot 2026-04-02 121445" src="https://github.com/user-attachments/assets/f9e04b48-118a-45cf-9559-7130cd31a228" />

### The “Aha” Moment

Now everything clicks as a system:

* S3 serves the frontend
* API Gateway handles requests
* Lambda executes logic
* DynamoDB stores state

No EC2. No always-on backend. No wasted cost.

### Lessons Learned

* This is what scalability actually looks like in practice
* You only pay for execution — not idle time
* Frontend + API separation is clean and production-relevant


