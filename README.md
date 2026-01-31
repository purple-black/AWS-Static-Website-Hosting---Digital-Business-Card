# 🌐 Digital Business Card – AWS Static Website Hosting

This project is a fully hosted **static Digital Business Card website** deployed using **Amazon S3** and **CloudFront**.  
It follows production-grade architecture and industry best practices for scalable, secure, global static content hosting.

---

## 📌 **Project Architecture Overview**

This project uses:

- **Amazon S3** – Static website hosting  
- **CloudFront CDN** – HTTPS, caching, compression, global low-latency distribution  
- **S3 Bucket Policy (IAM-based)** – Public read permissions for static content  
- **Static HTML/CSS** – Lightweight, responsive digital business card website  

This architecture ensures:
✔ Global performance  
✔ HTTPS by default  
✔ Low cost  
✔ Highly scalable  
✔ Serverless infrastructure  

---

## 📁 **Folder Structure**

DigitalBusinessCard/
│── index.html
│── style.css


Everything is uploaded directly to the **root of the S3 bucket** (not inside `/src`).

---

## 🚀 **AWS Services Used**

### ### **1. Amazon S3 – Static Website Hosting**

| Setting | Value | Reason |
|--------|--------|--------|
| Bucket Name | `digital-business-card-aswathy` | Globally unique, descriptive |
| Region | `ap-south-1` | Low latency (India) |
| Static Website Hosting | Enabled | Required to serve index.html |
| Index Document | `index.html` | Landing page |
| Block Public Access | Disabled | Static sites must be public |


### **S3 Website Endpoint**

http://digital-business-card-aswathy.s3-website.ap-south-1.amazonaws.com


⚠️ **Important**: S3 *website endpoints do NOT support HTTPS* → you must use CloudFront for HTTPS.

---

## 🔐 **IAM Usage**

IAM was **not used directly**, but:

- S3 bucket permissions use **IAM-based resource policies**.
- CloudFront relies on IAM-managed permission checks.

### **S3 Bucket Policy applied:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::digital-business-card-aswathy/*"
    }
  ]
}
```

### **CloudFront CDN Configuration**

Origin Settings
create a cloud front distribution.
Origin Domain -	S3 website endpoint	Required for index.html redirect behavior
Protocol - HTTP only,	S3 website endpoints do not support HTTPS
Port -	80	Standard HTTP
Origin Type -	S3 Website Hosting	Supports S3 routing rules
Default Behavior
Setting	Value
Viewer Protocol	Redirect HTTP → HTTPS
Allowed Methods	GET, HEAD
Cache Policy	CachingOptimized
Compression	Enabled

### **Challenges Faced & Fixes**
❌ 1. 504 Gateway Timeout (CloudFront)

Cause:
CloudFront was incorrectly configured to use HTTPS origin access:

HTTPS only
Port 443


S3 website endpoints do not support HTTPS, so CloudFront timed out.

Fix:
Changed origin to:

HTTP only
Port 80


and invalidated cache.

❌ 2. Incorrect Folder Upload

Originally uploaded:

DigitalBusinessCard/src/index.html


CloudFront could not find index.html at root.

Fix:
Moved files to S3 root level.

### **Final Architecture Diagram**
         ┌────────────────────┐
         │     User Browser   │
         └─────────┬──────────┘
                   │  HTTPS
                   ▼
        ┌────────────────────────┐
        │      CloudFront CDN     │
        │ (SSL, caching, DDoS)    │
        └─────────┬──────────────┘
                   │  HTTP (port 80)
                   ▼
        ┌────────────────────────┐
        │ S3 Static Website Host │
        │ (index.html at root)   │
        └────────────────────────┘

