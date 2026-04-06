---
title : "Introduction"
date : 2024-01-01 
weight : 1 
chapter : false
pre : " <b> 5.1. </b> "
---

#### SmartHire-AI Overview

**SmartHire-AI** is an end-to-end intelligent hiring platform that uses AWS services and AI to match candidates with job opportunities. The system operates as a web application built with **React (Vite)** that runs behind **Amazon CloudFront** and **S3**, with **Amazon Cognito** for authentication.

#### How It Works

##### For Candidates
- Sign in to the platform via Cognito (supports Google federation)
- Upload a **PDF resume** to Amazon S3
- Receive **personalized job suggestions** based on CV analysis
- View real-time updates as the pipeline processes their CV through AWS AppSync GraphQL subscriptions

##### For Recruiters
- Create and manage **job postings** stored in a relational database (RDS PostgreSQL)
- When a job is created/updated, the backend automatically starts processing
- View **ranked candidates** on their dashboard
- Receive real-time updates as candidates apply via AppSync subscriptions

#### Key Architecture Highlights

🟢 **Frontend**: React SPA served via S3 + CloudFront with Cognito sign-in over JWT

🟢 **API**: .NET 8 Lambda functions behind API Gateway with Cognito authorization

🟢 **CV Processing Pipeline**: Serverless orchestration with:
- **AWS Step Functions** for workflow management
- **AWS Lambda** (Python) for document processing
- **Amazon Textract** for PDF text extraction
- **Amazon Bedrock** for AI-powered enrichment
- **Amazon Comprehend** for text analysis
- **AWS SQS** for job queuing
- **Amazon RDS (PostgreSQL)** for structured data
- **Amazon DynamoDB** for tracking

🟢 **Real-time Updates**: AWS AppSync for GraphQL subscriptions pushing results to UI instantly

🟢 **Infrastructure as Code**: Terraform for AWS resources, AWS SAM for serverless API

---

#### Workshop Structure

In this workshop, you will learn:
1. **Architecture** - Deep dive into components and data flows
2. **Candidate Path** - How CV uploads are processed end-to-end
3. **Recruiter Path** - How job postings trigger intelligent matching
4. **Technology Stack** - All AWS services and tools used
5. **Repository Layout** - Project structure and deployment
