---
title : "Công nghệ được sử dụng"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

#### Ngăn xếp công nghệ SmartHire-AI

SmartHire-AI được xây dựng trên một kiến trúc Serverless hiện đại sử dụng các dịch vụ AWS và công nghệ mã nguồn mở. Dưới đây là danh sách toàn diện tất cả các công nghệ được sử dụng.

---

#### Frontend

| Công nghệ | Mục đích | Phiên bản |
|-----------|---------|----------|
| **React** | Khung UI component | Mới nhất |
| **Vite** | Bundler & dev server | v3+ |
| **TypeScript** | JavaScript an toàn loại | v4.9+ |
| **Tailwind CSS** | Utility-first CSS framework | v3+ |
| **Apollo Client** | GraphQL client cho AppSync | v3+ |

**Hosting:**
- **Amazon S3**: Static website hosting
- **Amazon CloudFront**: CDN với edge caching
- **AWS Certificate Manager**: HTTPS/TLS certificates

---

#### Xác thực & Ủy quyền

| Công nghệ | Mục đích |
|-----------|---------|
| **Amazon Cognito** | User pools, sign-in, federation |
| **Google OAuth** | Social login integration |
| **JWT Tokens** | Ủy quyền API không trạng thái |
| **AWS IAM** | Role-based access control |

---

#### Backend API

| Công nghệ | Mục đích |
|-----------|---------|
| **AWS API Gateway** | HTTP API endpoints |
| **AWS Lambda** | Serverless .NET 8 runtime |
| **.NET 8** | C# language/framework |
| **AWS RDS** | PostgreSQL 14+ database |
| **Secrets Manager** | Credential storage |

---

#### Document Processing & AI

| Công nghệ | Mục đích |
|-----------|---------|
| **Amazon Textract** | Extract text từ PDF + OCR |
| **Amazon Comprehend** | NLP: entity & key phrase recognition |
| **Amazon Bedrock** | LLM access (Claude, Llama) |

---

#### Serverless Orchestration

| Công nghệ | Mục đích |
|-----------|---------|
| **AWS Step Functions** | Workflow orchestration |
| **AWS Lambda (Python)** | Processing functions |
| **Amazon ECR** | Container registry |
| **AWS SQS** | Message queue |

---

#### Real-time Data

| Công nghệ | Mục đích |
|-----------|---------|
| **AWS AppSync** | Managed GraphQL API |
| **GraphQL Subscriptions** | WebSocket real-time updates |

---

#### Data Storage

| Công nghệ | Mục đích |
|-----------|---------|
| **Amazon RDS** | Relational data |
| **Amazon DynamoDB** | NoSQL real-time tracking |
| **Amazon S3** | Object storage |

---

#### Infrastructure as Code

| Công nghệ | Mục đích |
|-----------|---------|
| **HashiCorp Terraform** | Infrastructure provisioning |
| **AWS SAM** | Serverless API template |
| **AWS CloudFormation** | Stack management |

---

#### CI/CD & Deployment

| Công nghệ | Mục đích |
|-----------|---------|
| **AWS CodePipeline** | Build & deploy orchestration |
| **AWS CodeBuild** | Builds Docker images |
| **GitHub** | Source repository |

---

#### Observability

| Công nghệ | Mục đích |
|-----------|---------|
| **Amazon CloudWatch** | Logs, metrics, dashboards |
| **CloudWatch Insights** | Log query & analysis |
| **X-Ray** | Request tracing |
| **CloudWatch Alarms** | Alert on anomalies |

---

#### Security

| Công nghệ | Mục đích |
|-----------|---------|
| **AWS WAF** | Web Application Firewall |
| **VPC Security Groups** | Network access control |
| **AWS KMS** | Encryption key management |
| **Secrets Manager** | Rotate DB credentials |

---

#### Tại sao Ngăn xếp này?

✅ **Serverless-First**: Auto-scale, pay-per-use

✅ **Event-Driven**: Step Functions + Lambda + SQS

✅ **Real-Time**: AppSync subscriptions

✅ **AI-Ready**: Bedrock, Textract, Comprehend

✅ **Secure**: IAM, KMS, Cognito, WAF

✅ **Cost-Effective**: Chỉ trả tiền cho compute được sử dụng

✅ **Scalable**: Multi-region ready

✅ **Observable**: CloudWatch built-in

✅ **Infrastructure as Code**: Terraform
