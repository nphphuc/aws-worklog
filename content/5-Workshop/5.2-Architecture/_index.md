---
title : "Architecture"
date : 2024-01-01 
weight : 2 
chapter : false
pre : " <b> 5.2. </b> "
---

#### SmartHire-AI Architecture Overview

SmartHire-AI leverages a sophisticated AWS serverless architecture that scales automatically based on demand. Below is the complete system design:

```
┌─────────────────────────────────────────────────────────────┐
│                     WEB TIER (Frontend)                      │
│  React SPA (Vite) → CloudFront CDN → S3 Static Hosting      │
│                  + Cognito Authentication                    │
└─────────────────────────────────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
┌───────▼──────┐  ┌─────────▼────────┐  ┌──────▼──────────┐
│  API Layer   │  │  Stream Layer    │  │  Data Layer    │
│              │  │                  │  │                 │
│ API Gateway  │  │   AWS AppSync    │  │ RDS PostgreSQL │
│ → Lambda     │  │   (GraphQL       │  │ DynamoDB       │
│ (.NET 8)     │  │    Subscriptions)│  │ S3 Buckets     │
└──────────────┘  └──────────────────┘  └─────────────────┘
```

#### Core Components

**1. Frontend (Web Tier)**
- **React SPA**: Built with Vite, TypeScript for optimal bundle size
- **CDN**: CloudFront for global distribution
- **Hosting**: S3 static website hosting
- **Auth**: Cognito user pool with Google OAuth federation

**2. API Layer (.NET 8 Lambda + API Gateway)**
- HTTP REST API for job operations
- JWT authorization via Cognito
- VPC access to RDS for candidate/job data
- Secrets Manager for sensitive credentials
- Triggers Step Functions for async processing

**3. CV/JD Processing Pipeline**
- **Step Functions**: Orchestrates the processing workflow
- **AWS Lambda (Python 3.12)**: Executes processing tasks
  - `cv_jd_processor`: Extracts and enriches data
  - `job_suggestion_engine`: Ranks jobs for candidates
  - `candidate_ranking_engine`: Ranks candidates for jobs
- **Container Images**: Stored in Amazon ECR for complex processing
- **SQS Queue**: Buffers CV uploads, includes Dead Letter Queue (DLQ)
- **S3 Bucket**: Raw CV storage with event notifications

**4. AI & Document Analysis**
- **Amazon Textract**: Extracts text from PDF resumes
- **Amazon Bedrock**: Large Language Model for enrichment & matching
- **Amazon Comprehend**: NLP for entity recognition and sentiment

**5. Data Layer**
- **RDS (PostgreSQL)**: Jobs, candidates, user data
- **DynamoDB**: Real-time application tracking & matching results
- **S3**: CV uploads, static assets

**6. Real-time Updates**
- **AWS AppSync**: GraphQL API with subscriptions
- WebSocket connections for instant dashboard updates
- Decoupled from main API for independent scaling

**7. Infrastructure as Code**
- **Terraform**: Provisions VPC, RDS, Cognito, networking, CI/CD
- **AWS SAM**: Manages API Gateway + Lambda stack
- **CodePipeline**: Automates deployment from GitHub

---

#### Optional Components

- **AWS WAF**: Web Application Firewall on CloudFront
- **Route 53**: Custom domain management
- **ACM**: SSL/TLS certificates
- **VPC Interface Endpoints**: Private access to AWS services
- **NAT Gateway**: Outbound internet access from Lambda
- **CloudWatch**: Logging and monitoring
- **SNS**: Alarms for pipeline health
