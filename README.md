
# 🚀 EC2 CPU Monitoring with n8n (AWS + Gmail Integration)

This project demonstrates an automated workflow using **n8n** that monitors the CPU usage of an Amazon EC2 instance using AWS CloudWatch. If the CPU usage exceeds a threshold (80%), it sends an alert email via Gmail. Otherwise, it notifies that the CPU is operating within normal limits.

---

## ✅ Workflow Overview

```
Schedule Trigger
  → GET EC2 CPU (CloudWatch via HTTP)
    → Set (Extract CPU Usage)
      → IF (cpu_usage > 80)
        ├─ true → Gmail (⚠️ High CPU Alert)
        └─ false → Gmail (✅ Normal Usage Notice)
```

---

## 📂 Files

- `EC2_Monitoring.json`: The exportable n8n workflow file
- `README.md`: You're reading it!

---

## ⚙️ Step-by-Step Setup

### 1. Prerequisites

- AWS Account with EC2 and CloudWatch enabled
- EC2 Instance ID
- AWS credentials with `cloudwatch:GetMetricStatistics` and `ec2:DescribeInstances`
- Gmail account for notifications

---

### 2. Run n8n Locally using Docker Desktop

1. Make sure **Docker Desktop** is installed and running
2. Pull and run the n8n Docker image:

```bash
docker run -it --rm   -p 5678:5678   -v ~/.n8n:/home/node/.n8n   n8nio/n8n
```

3. Open your browser and go to:
```
http://localhost:5678/
```

4. Sign up or log in with your **n8n account**
5. You’ll receive an **activation key** via email (to the email used in signup)
6. Enter the key when prompted to activate n8n

You now have a fully functional local n8n editor.

---

### 3. Import Workflow

1. Go to your n8n editor at `http://localhost:5678/`
2. Click on **Import Workflow**
3. Upload `EC2_Monitoring.json`

---

### 4. Configure Nodes

#### 🔸 HTTP Request Node
Sends a signed request to CloudWatch to retrieve average CPU usage over the past 5 minutes.

Key parameters:
- `StartTime`: `{{ (new Date(new Date($now).getTime() - 5 * 60000)).toISOString() }}`
- `EndTime`: `{{ (new Date($now)).toISOString() }}`

#### 🔸 Set Node
Extracts the `Average` CPU value from the CloudWatch response and stores it as `cpu_usage`.

#### 🔸 IF Node
Compares `cpu_usage` to 80.

- If **greater than 80** → sends alert via Gmail
- If **not** → sends normal status update

---

### ✉️ Setting Up Gmail Credentials in n8n

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a **new project** (or use existing)
3. Go to **APIs & Services > Library**
4. Search for **Gmail API** and click **Enable**
5. Go to **APIs & Services > OAuth consent screen**
   - Choose **External**, click **Create**
   - Fill in app name, support email, and add scopes:
     - `https://www.googleapis.com/auth/gmail.send`
6. Go to **Credentials > Create Credentials > OAuth Client ID**
   - Application type: **Web application**
   - Add `http://localhost:5678` as authorized redirect URI (for local n8n)
7. Download the JSON file
8. In n8n:
   - Go to **Credentials > Gmail OAuth2**
   - Enter Client ID, Secret, and do OAuth flow


---

### 🛰️ HTTP Request Node Configuration (GET EC2 CPU Metrics)

This node makes a signed API call to AWS CloudWatch to fetch the average CPU utilization for a specific EC2 instance over the last 5 minutes.

**Node Type:** HTTP Request  
**Authentication:** AWS (using Access Key & Secret)

**Request Method:** `POST`  
**URL:** `https://monitoring.<region>.amazonaws.com/`  
Example: `https://monitoring.us-east-2.amazonaws.com/`

**Body Content Type:** `Form Urlencoded`  
**Specify Body:** ✅ Yes (Use Fields Below)

---

#### 🧾 Body Parameters

| Name                             | Value                                                                 |
|----------------------------------|------------------------------------------------------------------------|
| Action                           | GetMetricStatistics                                                   |
| Version                          | 2010-08-01                                                            |
| Namespace                        | AWS/EC2                                                               |
| MetricName                       | CPUUtilization                                                        |
| Dimensions.member.1.Name         | InstanceId                                                            |
| Dimensions.member.1.Value        | `i-00a9364a01d1f6c28` *(Replace with your EC2 Instance ID)*            |
| StartTime                        | `{{ (new Date(new Date($now).getTime() - 5 * 60000)).toISOString() }}` |
| EndTime                          | `{{ (new Date($now)).toISOString() }}`                               |
| Period                           | 300 *(5 minutes)*                                                    |
| Statistics.member.1              | Average                                                               |

---

⏱️ This request calculates average CPU usage over the last 5 minutes and returns data for further evaluation in the workflow.


---

## 🧩 Challenges Faced

### ❌ Timestamp Not ISO8601
**Issue:** CloudWatch rejected timestamp with `"timestamp must follow ISO8601"` error  
**Fix:** Used `.toISOString()` and trimmed milliseconds if necessary

### ❌ Gmail “API not enabled” error
**Issue:** `"Gmail API has not been used..."`  
**Fix:** Enabled Gmail API in Google Cloud Console and reconnected account

### 🔐 Signature v4 Signing
Handled via built-in AWS credentials in n8n HTTP Request node (with `aws` auth selected)

---

## 💡 Ideas for Improvement

- Add Google Sheets logging
- Monitor multiple EC2 instances with `SplitInBatches`
- Add Slack/Discord alerts

---

## 📥 Upload to GitHub

Simply drag both `EC2_Monitoring.json` and this `README.md` into your GitHub repo.

Happy automating! 🤖
