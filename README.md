
# 🔐 SecMon Product Feed Validator

A serverless system that validates uploaded XML product feeds to S3, calculates a SHA256 hash, and sends a notification via SNS — all built with AWS Lambda, S3, CloudWatch, and Terraform.

---

## 📦 Architecture

```
S3 Bucket (feed uploads)
       │
       ▼
Amazon EventBridge (ObjectCreated)
       │
       ▼
AWS Lambda (XML Validator)
       │
       ├── SHA256 Hash
       ├── XML Parsing
       └── SNS Notification
```

---

## 🚀 Features

- ✅ Triggered by S3 `ObjectCreated` events
- ✅ Parses uploaded XML feeds
- ✅ Computes SHA256 hash for integrity
- ✅ Publishes status to an SNS topic
- ✅ Logs to CloudWatch
- ✅ Fully managed via Terraform

---

## 🛠 Prerequisites

- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
- Terraform v1.3+ configured with access to:
  - IAM
  - S3
  - Lambda
  - KMS
  - SNS
- Python 3.12 (for local Lambda testing)
- An S3 bucket named `secmon-product-feed`
- A customer-managed KMS key with correct permissions
- An SNS topic named `secmon-alerts`

---

## 📁 File Structure

```
terraform/
├── main.tf
├── lambda.tf
├── iam_policy.tf
└── variables.tf

lambda/
└── main.py
```

---

## 📜 Lambda Logic (main.py)

- Downloads the uploaded S3 file
- Computes SHA256 hash
- Validates the XML structure using `ElementTree`
- Sends a notification via SNS
- Logs key actions to CloudWatch

---

## ✅ Deployment

```bash
cd terraform/
terraform init
terraform apply
```

> Ensure your environment has access to create IAM roles, Lambda functions, and SNS topics.

---

## 🔄 Testing the Flow

### Manual Event Test via AWS Console

```json
{
  "version": "0",
  "id": "test-event-id-1234",
  "detail-type": "Object Created",
  "source": "aws.s3",
  "account": "824385116033",
  "time": "2025-05-28T00:00:00Z",
  "region": "us-west-1",
  "resources": [],
  "detail": {
    "bucket": {
      "name": "secmon-product-feed"
    },
    "object": {
      "key": "feed-test.xml",
      "size": 1234,
      "etag": "abcdef1234567890",
      "sequencer": "0055AED6DCD90281E5"
    }
  }
}
```

Or upload a file manually:

```bash
aws s3 cp feed-test.xml s3://secmon-product-feed/
```

---

## 📊 Viewing Logs

```bash
aws logs tail /aws/lambda/secmon-xml-validator --follow --region us-west-1
```

You can also view logs via the **CloudWatch Console > Log Groups > /aws/lambda/secmon-xml-validator**

---

## 🔐 IAM and Security

- Lambda assumes `secmon-lambda-role`, which has:
  - Read access to S3
  - Decrypt access to KMS
  - Publish access to SNS
- The KMS key policy grants `kms:Decrypt` to this role
- Environment variables encrypted with the customer-managed KMS key

---

## 📬 Notifications

SNS Topic: `arn:aws:sns:us-west-1:824385116033:secmon-alerts`

Example message:

> Feed `feed-test.xml` uploaded and parsed OK. SHA256: `f40b43f1...`

---

## 📌 License

MIT License — customize this as needed for your project.
