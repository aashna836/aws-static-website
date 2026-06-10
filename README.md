# 🌐 Static Website Hosting on AWS

> A production-style static website deployed using **Amazon S3**, **CloudFront**, and **Route 53** — the same stack companies use to serve static content at scale. Built as part of my AWS learning journey.

[![AWS](https://img.shields.io/badge/AWS-Cloud-orange?logo=amazonaws&logoColor=white)](https://aws.amazon.com)
[![S3](https://img.shields.io/badge/Amazon_S3-Storage-yellow?logo=amazons3&logoColor=white)](https://aws.amazon.com/s3)
[![CloudFront](https://img.shields.io/badge/CloudFront-CDN-blue?logo=amazonaws&logoColor=white)](https://aws.amazon.com/cloudfront)
[![Free Tier](https://img.shields.io/badge/AWS_Free_Tier-Eligible-green)](https://aws.amazon.com/free)

---

## 🔴 Live Demo

🌍 **[View the live site →](https://du4l4ccgotjh.cloudfront.net)**

> The site itself explains the architecture — no need to read the whole README if you just want to see how it works.

---

## 📐 Architecture

```
User Browser
     │
     │  DNS lookup
     ▼
 Route 53  ──────────────────────────────┐
     │  (optional - skip to stay free)   │
     │  Alias record                     │
     ▼                                   │
 CloudFront  ◄──── HTTPS + SSL (ACM) ───┘
     │
     │  Cache hit → serve immediately
     │  Cache miss → fetch from origin
     ▼
 S3 Bucket  (private, OAC-only access)
     │
     └── index.html
     └── error.html
     └── assets/
```

**Request flow:**
1. Browser hits **Route 53** → resolves to CloudFront (not S3 directly)
2. **CloudFront** checks its edge cache — 400+ locations globally
3. Cache miss → CloudFront fetches from **S3** over private AWS backbone
4. S3 bucket stays private — only CloudFront can read it via **Origin Access Control (OAC)**
5. SSL handled entirely by **ACM** certificate on CloudFront

---

## ☁️ AWS Services Used

| Service | Purpose | Free Tier |
|---|---|---|
| **Amazon S3** | Stores all static files (HTML, CSS, JS) | 5 GB storage, 20K GET/month |
| **CloudFront** | Global CDN, SSL termination, caching | 1 TB transfer, 10M requests/month |
| **Route 53** | DNS routing to CloudFront | $0.50/hosted zone (optional) |
| **ACM** | Free SSL/TLS certificate | Always free |

---

## 💰 Monthly Cost Estimate

For a personal project with ~10,000 page views/month:

| Service | Usage | Cost |
|---|---|---|
| S3 Storage | ~50 MB | $0.00 |
| S3 Requests | CloudFront origin fetches | $0.00 |
| CloudFront | ~1 GB data transfer | $0.00 |
| ACM Certificate | SSL for custom domain | $0.00 |
| Route 53 | 1 hosted zone (optional) | $0.50 |
| **Total** | | **$0.00 – $0.50/month** |

---

## 🚀 How to Deploy This Yourself

### Prerequisites
- AWS account (free tier)
- AWS CLI installed and configured
- Your website files (HTML, CSS, JS)

### Step 1 — Create S3 bucket

```bash
aws s3 mb s3://your-bucket-name --region ap-south-1
```

### Step 2 — Upload files

```bash
aws s3 sync ./website s3://your-bucket-name
```

### Step 3 — Enable static website hosting

```bash
aws s3 website s3://your-bucket-name \
  --index-document index.html \
  --error-document error.html
```

### Step 4 — Create CloudFront distribution

Go to AWS Console → CloudFront → Create distribution:
- Origin domain: your S3 bucket
- Origin access: **Origin Access Control (OAC)** — create new OAC
- Viewer protocol policy: **Redirect HTTP to HTTPS**
- Default root object: `index.html`

### Step 5 — Apply bucket policy

After CloudFront creates the OAC, copy the generated bucket policy and apply it to your S3 bucket:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontOAC",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::ACCOUNT-ID:distribution/DIST-ID"
        }
      }
    }
  ]
}
```

### Step 6 — (Optional) Add custom domain via Route 53

1. Register or transfer domain in Route 53
2. Request ACM certificate in **us-east-1** (required for CloudFront)
3. Add the certificate to your CloudFront distribution
4. Create an A record in Route 53 → Alias → pointing to CloudFront

---

## 📁 Repo Structure

```
aws-static-website/
│
├── website/                  # Your actual website files
│   ├── index.html            # Main page (explains the project)
│   └── error.html            # Custom 404 page
│
├── docs/
│   └── architecture.png      # Architecture diagram screenshot
│
├── bucket-policy.json        # S3 bucket policy for CloudFront OAC
└── README.md
```

---

## 🔑 Key Concepts Demonstrated

- **Object storage** — using S3 as a static file origin, not as a web server
- **CDN fundamentals** — how CloudFront edge caching works and why it matters for performance
- **HTTPS enforcement** — SSL termination at the CDN layer, not the origin
- **Least privilege access** — S3 bucket stays private; only CloudFront can read it via OAC
- **DNS routing** — connecting a custom domain to a CDN distribution via alias records

---

## 🧹 Cleanup (avoid charges)

```bash
# Empty and delete S3 bucket
aws s3 rm s3://your-bucket-name --recursive
aws s3 rb s3://your-bucket-name

# Disable CloudFront distribution first, then delete it
# (can take up to 15 minutes to disable)
# Do this from the AWS Console: CloudFront → Distributions → Disable → Delete
```

> ⚠️ CloudFront distributions must be **disabled** before they can be deleted. Don't forget this step or you'll keep getting charged.

---

## 📚 What I Learned

- The difference between S3 website hosting and using S3 as a CloudFront origin
- Why you should use OAC instead of public bucket access (security best practice as of 2023)
- How CloudFront cache invalidation works and when you need it
- ACM certificates must be in `us-east-1` to work with CloudFront, regardless of your bucket region

---

## 🗺️ Next Project

**CI/CD Pipeline on AWS** — automatically deploy a Dockerized app to ECS on every git push using CodePipeline + CodeBuild + ECR.

---

*Part of my AWS learning series. Follow along on GitHub.*
