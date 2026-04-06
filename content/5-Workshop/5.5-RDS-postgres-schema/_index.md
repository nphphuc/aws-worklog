---
title : "RDS Postgres Schema"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

#### SmartHire-AI Technology Stack

SmartHire-AI is built on a modern, serverless architecture using AWS services and open-source technologies. Below is a comprehensive breakdown of all technologies used.

---

#### Frontend Technologies

| Technology | Purpose | Version |
|-----------|---------|---------|
| **React** | UI component framework | Latest |
| **Vite** | Module bundler & dev server | v3+ |
| **TypeScript** | Type-safe JavaScript | v4.9+ |
| **Tailwind CSS** | Utility-first CSS framework | v3+ |
| **Apollo Client** | GraphQL client for AppSync | v3+ |

**Hosting:**
- **Amazon S3**: Static website hosting
- **Amazon CloudFront**: CDN with edge caching
- **AWS Certificate Manager (ACM)**: HTTPS/TLS certificates

---

#### Authentication & Authorization

| Technology | Purpose |
|-----------|---------|
| **Amazon Cognito** | User pools, sign-in, federation |
| **Cognito + Google OAuth** | Social login integration |
| **JWT Tokens** | Stateless API authorization |
| **AWS IAM** | Role-based access control (RBAC) |

---

#### Backend API

| Technology | Purpose |
|-----------|---------|
| **AWS API Gateway** | HTTP API endpoints |
| **AWS Lambda** | Serverless .NET 8 runtime |
| **.NET 8** | C# language/framework |
| **AWS RDS** | PostgreSQL 14+ database |
| **Secrets Manager** | Credential storage |

---

#### Document Processing & AI

| Technology | Purpose |
|-----------|---------|
| **Amazon Textract** | Extract text from PDFs + OCR |
| **Amazon Comprehend** | NLP: entity recognition, key phrases |
| **Amazon Bedrock** | LLM access (Claude, Llama) |

---

#### Serverless Orchestration & Processing

| Technology | Purpose |
|-----------|---------|
| **AWS Step Functions** | Workflow orchestration |
| **AWS Lambda (Python)** | Processing functions |
| **Amazon ECR** | Container registry |
| **AWS SQS** | Message queue |

---

#### Real-time Data

| Technology | Purpose |
|-----------|---------|
| **AWS AppSync** | Managed GraphQL API |
| **GraphQL Subscriptions** | WebSocket real-time updates |

---

#### Data Storage

| Technology | Purpose |
|-----------|---------|
| **Amazon RDS** | Relational data |
| **Amazon DynamoDB** | NoSQL real-time tracking |
| **Amazon S3** | Object storage |

---

#### Infrastructure as Code

| Technology | Purpose |
|-----------|---------|
| **HashiCorp Terraform** | Infrastructure provisioning |
| **AWS SAM** | Serverless API template |
| **AWS CloudFormation** | Stack management |

---

#### CI/CD & Deployment

| Technology | Purpose |
|-----------|---------|
| **AWS CodePipeline** | Build & deploy orchestration |
| **AWS CodeBuild** | Builds Docker images |
| **GitHub** | Source repository |

---

#### Observability

| Technology | Purpose |
|-----------|---------|
| **Amazon CloudWatch** | Logs, metrics, dashboards |
| **CloudWatch Insights** | Log query & analysis |
| **X-Ray** | Request tracing |
| **CloudWatch Alarms** | Alert on anomalies |

---

#### Security

| Technology | Purpose |
|-----------|---------|
| **AWS WAF** | Web Application Firewall |
| **VPC Security Groups** | Network access control |
| **AWS KMS** | Encryption key management |
| **Secrets Manager** | Rotate DB credentials |

---

#### Why This Stack?

✅ **Serverless-First**: Auto-scale, pay-per-use, no server management

✅ **Event-Driven**: Step Functions + Lambda + SQS for async processing

✅ **Real-Time**: AppSync subscriptions push updates instantly

✅ **AI-Ready**: Bedrock, Textract, Comprehend built-in

✅ **Secure**: IAM, KMS, Cognito, WAF layered security

✅ **Cost-Effective**: Pay only for compute used

✅ **Scalable**: Multi-region ready, CloudFront edge caching

✅ **Observable**: CloudWatch integrated across all services

✅ **Infrastructure as Code**: Terraform for reproducible deployments
