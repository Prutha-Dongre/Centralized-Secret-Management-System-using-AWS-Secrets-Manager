# 🔐 Centralized Secret Management System using AWS Secrets Manager

## 📌 Project Overview

This project demonstrates how to securely manage database credentials in a cloud environment by eliminating hardcoded secrets and implementing centralized secret storage using AWS services.

Initially, the application uses hardcoded database credentials (insecure approach). Later, it is enhanced to securely fetch credentials dynamically from AWS Secrets Manager using IAM roles and SDK integration, along with automatic secret rotation.

---

## 🎯 Objectives

* Eliminate hardcoded credentials from application code
* Implement centralized secret storage using AWS Secrets Manager
* Control access using IAM roles (no access keys)
* Enable automatic secret rotation using AWS Lambda
* Ensure application works seamlessly after credential rotation

---

## 🏗️ Architecture Overview

* **EC2** → Hosts Flask application
* **RDS (MySQL)** → Database service
* **Secrets Manager** → Stores DB credentials securely
* **IAM Role** → Grants EC2 access to secrets
* **Lambda** → Handles automatic secret rotation

---

## ⚙️ Technologies Used

* AWS EC2
* AWS RDS (MySQL)
* AWS Secrets Manager
* AWS IAM
* AWS Lambda
* Python (Flask, boto3, pymysql)

---

# 🚀 Implementation Steps

---

## 🔹 Step 1: RDS Setup (Database Creation)

1. Go to AWS Console → RDS → Create database
2. Select:

   * Engine: MySQL
   * Template: Free tier
3. Configure:

   * DB name: `mydb`
   * Username: `admin`
   * Password: `TempPass123`
4. Enable **Public Access (for testing)**
5. Create database

### 🔐 Security Group Configuration

* Allow inbound rule:

  * Type: MySQL (3306)
  * Source: EC2 security group

---

## 🔹 Step 2: Connect to RDS & Create Table

Connect from EC2:

```bash
mysql -h <RDS-ENDPOINT> -u admin -p
```

Run:

```sql
CREATE DATABASE mydb;
USE mydb;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100)
);
```

---

## 🔹 Step 3: EC2 Setup & Application Deployment

1. Launch EC2 instance (Ubuntu)
2. Install dependencies:

```bash
sudo apt update
sudo apt install python3-pip python3-venv mysql-client -y
```

3. Create virtual environment:

```bash
python3 -m venv myenv
source myenv/bin/activate
```

4. Install Python libraries:

```bash
pip install flask pymysql boto3
```

---

## 🔹 Step 4: Insecure Application (Hardcoded Credentials ❌)

A Flask application is created with hardcoded database credentials.

### Example:

```python
DB_HOST = "<RDS-ENDPOINT>"
DB_USER = "admin"
DB_PASS = "TempPass123"
DB_NAME = "mydb"
```

### Features:

* Web form to input user name
* Stores data in RDS
* Displays stored records

### Run Application:

```bash
python app.py
```

Access:

```
http://<EC2-IP>:5000
```

👉 This demonstrates the **security risk of hardcoded credentials**

---

## 🔹 Step 5: Store Credentials in AWS Secrets Manager

1. Go to AWS Secrets Manager
2. Click **Store a new secret**
3. Choose:

   * Credentials for RDS database
4. Enter:

```json
{
  "username": "admin",
  "password": "TempPass123",
  "host": "<RDS-ENDPOINT>",
  "port": 3306
}
```

5. Secret name:

```
mydb_secret
```

---

## 🔹 Step 6: Enable Automatic Rotation 🔄

* Enable **Automatic rotation**
* Use default Lambda function
* Select rotation interval (e.g., 30 days or demo: 1 hour)

👉 AWS creates a Lambda function automatically to:

* Rotate DB password
* Update RDS
* Update secret

---

## 🔹 Step 7: IAM Role Configuration

1. Go to IAM → Create Role
2. Trusted entity → EC2
3. Attach policy:

   * `SecretsManagerReadWrite` (or custom least privilege)
4. Name role:

```
EC2-SecretsManager-Role
```

5. Attach role to EC2 instance

👉 This removes need for access keys

---

## 🔹 Step 8: Secure Application (Using SDK 🔐)

Modify application to fetch secrets using **boto3 SDK**:

```python
import boto3
import json

def get_secret():
    client = boto3.client("secretsmanager", region_name="us-west-1")

    response = client.get_secret_value(
        SecretId="mydb_secret"
    )

    return json.loads(response["SecretString"])
```

Update DB connection:

```python
def connect_db():
    creds = get_secret()

    return pymysql.connect(
        host=creds["host"],
        user=creds["username"],
        password=creds["password"],
        database="mydb"
    )
```

### Key Improvements:

* No credentials in code
* Secrets fetched dynamically
* IAM-based authentication

---

## 🔹 Step 9: Test Secure Application

* Run app
* Insert data
* Verify DB entries

👉 Application works without hardcoded credentials

---

## 🔹 Step 10: Test Secret Rotation (Final Validation 🔥)

1. Go to Secrets Manager
2. Click:

   * **Rotate secret immediately**
3. Wait for completion

### Verify:

* Password updated in RDS
* Secret updated
* Application still works

👉 This proves:
✔ Dynamic secret retrieval
✔ Zero downtime
✔ Secure architecture

---

# 🔐 Why Secret Rotation Matters

* Prevents long-term credential exposure
* Reduces risk of security breaches
* Ensures compliance with best practices
* Automates credential lifecycle

---

# 🚀 Security Improvements

| Before ❌              | After ✅           |
| --------------------- | ----------------- |
| Hardcoded credentials | Secrets Manager   |
| Exposed passwords     | Encrypted secrets |
| Static credentials    | Auto-rotated      |
| Access keys required  | IAM roles         |
| Manual updates        | Automated         |

---

# ✅ Final Outcome

* Successfully implemented centralized secret management
* Eliminated hardcoded credentials
* Enabled secure IAM-based access
* Achieved zero-downtime secret rotation
* Built a production-level DevSecOps solution

---

# 📸 Suggested Screenshots

* RDS configuration
* Secrets Manager secret
* IAM role
* Flask application UI
* Rotation success

---

# 🧠 Key Learnings

* Importance of secure secret management
* IAM role-based access control
* SDK integration using boto3
* Automatic credential rotation
* Real-world DevSecOps practices

---
