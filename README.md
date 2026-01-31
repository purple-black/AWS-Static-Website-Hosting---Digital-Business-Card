# AWS Static Website Hosting - Digital-Business-Card
A beginner level cloud project where a simple static website (Digital Business Card) built using HTML and CSS is hosted in AWS using AWS S3 + CloudFront

📘 Project Documentation: AWS Static Website Hosting with CloudFront

Project Name: Digital Business Card
Services Used: Amazon S3, CloudFront
IAM Usage: Indirect (S3 bucket policy only)

1. Project Overview

This project hosts a fully static Digital Business Card website using AWS services.
It demonstrates a real-world static hosting architecture used for lightweight, global websites.

The deployment includes:

Amazon S3 for static website hosting

CloudFront CDN for HTTPS, caching, and global content delivery

S3 Bucket Policy (IAM-based) for public read permissions

No Lambda, no backend — purely static hosting with serverless AWS infrastructure.

2. Folder Structure
DigitalBusinessCard/
│── index.html
│── style.css


This structure ensures that the entry point (index.html) is placed at the bucket root as required by S3 website hosting.

3. Amazon S3 Configuration
3.1 Bucket Setup
Setting	Value	Reason
Bucket Name	digital-business-card-aswathy	Must be globally unique. Clear naming convention.
Region	ap-south-1 (Mumbai)	Low latency for India-based deployment.
Block Public Access	Disabled	Required for static website hosting.
Static Website Hosting	Enabled	Allows direct hosting of HTML, CSS, images.
Index Document	index.html	Entry point of the website.
3.2 S3 Website Endpoint

Your website is served from:

http://digital-business-card-aswathy.s3-website.ap-south-1.amazonaws.com


This endpoint only supports HTTP

3.3 S3 Bucket Policy (IAM-Based)

S3 bucket policies are part of IAM's permission system.

This policy allowed public read access:

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

Why needed?

S3 static websites require public GET access.

CloudFront retrieves objects using public GET requests.

Minimal and safe permission scope (read-only).

4. CloudFront Configuration

CloudFront acts as a secure, global CDN layer over your S3 website.

4.1 Origin Settings
Setting	Selected	Reason
Origin Domain	S3 Website Endpoint	allows index.html routing and static hosting behavior
Protocol	HTTP only	website endpoints do NOT support HTTPS
Port	80	standard HTTP
Origin Type	S3 Website	enables S3 redirect rules & website behavior
4.2 Default Behavior
Setting	Value	Reason
Viewer Protocol	Redirect HTTP to HTTPS	Users always access secure HTTPS version
Allowed Methods	GET, HEAD	Enough for static websites
Cache Policy	CachingOptimized	Reduces S3 cost and latency
Compression	Enabled	Faster HTML/CSS transfer
5. Challenges Faced & Their Resolutions

❌ Problem 1: 504 Gateway Timeout (CloudFront)
Symptoms

CloudFront displayed 504 Gateway Timeout

Browser console showed failed GET requests

Cause

CloudFront was configured with:

HTTPS Only
Port 443


But S3 website endpoints do NOT support HTTPS.

CloudFront tried to reach an HTTPS version of the endpoint → request failed → 504 error.

✔ Fix

Changed Origin Protocol to:

HTTP Only
Port 80


Then invalidated cache (/*).

Site began working immediately.

❌ Problem 2: Incorrect Folder Structure in S3

Initially uploaded:

/DigitalBusinessCard/src/index.html


S3 cannot serve an index file located inside a subfolder.

✔ Fix

Upload index.html and other assets directly to the S3 root, not inside src/.

6. IAM Usage
✔ IAM was used indirectly

S3 bucket policy is part of AWS IAM infrastructure.

When you allow public access to your bucket, it is done using IAM’s permission framework.

CloudFront also checks IAM-based permissions behind the scenes.

❌ IAM was NOT used directly

did NOT create:

IAM roles

IAM users

IAM policies

Origin Access Identity (OAI)

Origin Access Control (OAC)

This is because the website is public and uses S3 website hosting, not private S3 access.

7. Architecture Diagram
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
   
