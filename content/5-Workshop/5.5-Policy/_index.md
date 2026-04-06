---
title : "Technology Stack"
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
| **AWS Amplify UI** | Pre-built AWS UI components | Latest |
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
| **Amazon Cognito** | User pools, sign-in, sign-up, federation |
| **Cognito + Google OAuth** | Social login integration |
| **JWT Tokens** | Stateless API authorization |
| **AWS IAM** | Role-based access control (RBAC) |

**Flow:**
```
User Login → Cognito → Generate JWT → Store in localStorage
API Call → Pass JWT header → API Gateway validates → Route to Lambda
```

---

#### Backend API

| Technology | Purpose | Details |
|-----------|---------|---------|
| **AWS API Gateway** | HTTP API | REST endpoints, CORS, request throttling |
| **AWS Lambda** | Serverless compute | .NET 8 runtime, 15-minute timeout |
| **.NET 8** | API language/framework | C#, NuGet packages, Entity Framework |
| **AWS RDS** | Relational database | PostgreSQL 14+, Multi-AZ, backups |
| **Amazon Secrets Manager** | Secure credential storage | DB password rotation |

**API Endpoints:**
```
POST   /jobs              → Create job
GET    /jobs/{id}         → Get job details
PUT    /jobs/{id}         → Update job
POST   /candidates        → Register candidate
GET    /candidates/{id}   → Get candidate profile
```

---

#### Document Processing & AI

| Technology | Purpose |
|-----------|---------|
| **Amazon Textract** | Extract text from PDFs + OCR |
| **Amazon Comprehend** | NLP: entity recognition, key phrases |
| **Amazon Bedrock** | LLM access (Claude, Llama) for enrichment |
| **LangChain** | LLM orchestration (optional) |

---

#### Serverless Orchestration & Processing

| Technology | Purpose | Details |
|-----------|---------|---------|
| **AWS Step Functions** | Workflow orchestration | State machine, error handling, retries |
| **AWS Lambda** | Serverless functions | Python 3.12, 15-minute max duration |
| **Amazon ECR** | Container registry | Docker images for complex processing |
| **AWS SQS** | Message queue | FIFO queue for CV processing, DLQ |
| **Amazon SNS** | Notifications | Alerts, email notifications |

**Processing Languages:**
- **Python 3.12**: Data processing, ML libraries (pandas, numpy, scikit-learn)
- **Node.js 18** (optional): Alternative Lambda runtime for specific tasks

---

#### Real-time Data

| Technology | Purpose |
|-----------|---------|
| **AWS AppSync** | Managed GraphQL API |
| **GraphQL Subscriptions** | WebSocket-based real-time updates |
| **AppSync Data Sources** | Lambda, RDS, DynamoDB resolvers |

**GraphQL Schema Example:**
```graphql
type Query {
  getJobSuggestions(candidateId: ID!): [Job]!
  getCandidateRankings(jobId: ID!): [Candidate]!
}

type Subscription {
  onJobSuggestionsReady(candidateId: ID!): JobSuggestions
  onCandidateRankingReady(jobId: ID!): CandidateRanking
}
```

---

#### Data Storage

| Technology | Purpose | Data Type |
|-----------|---------|-----------|
| **Amazon RDS (PostgreSQL)** | Relational data | Users, jobs, candidates, matches |
| **Amazon DynamoDB** | NoSQL, real-time tracking | Match scores, processing status |
| **Amazon S3** | Object storage | CVs (PDFs), images, logs |
| **Amazon ElastiCache (optional)** | In-memory cache | Job/candidate embeddings cache |

**Data Flow:**
```
Write → RDS (primary source of truth)
Cache → DynamoDB (fast queries for rankings)
Files → S3 (PDFs, documents)
Search → ElastiCache (embedding lookups)
```

---

#### Infrastructure as Code

| Technology | Purpose |
|-----------|---------|
| **HashiCorp Terraform** | Infrastructure provisioning |
| **AWS SAM (Serverless Application Model)** | Lambda + API Gateway template |
| **AWS CloudFormation** | Stack management |

**Modules:**
```
iac/terraform/
├── vpc/              → VPC, subnets, security groups
├── rds/              → PostgreSQL database
├── cognito/          → User pool, identity provider
├── s3/               → Buckets, policies
├── lambda/           → Execution roles, permissions
├── step-functions/   → State machine definitions
├── appsync/          → GraphQL API
└── ci-cd/            → CodePipeline, CodeBuild
```

---

#### CI/CD & Deployment

| Technology | Purpose |
|-----------|---------|
| **AWS CodePipeline** | Orchestrates build & deploy stages |
| **AWS CodeBuild** | Builds Docker images, runs tests |
| **GitHub (CodeStar Connection)** | Source repository |
| **GitHub Actions** | Local CI/CD (alternative to CodePipeline) |

**Pipeline Stages:**
```
1. Source (GitHub commit)
   ↓
2. Build (CodeBuild)
   - npm install (frontend)
   - npm run build (frontend)
   - dotnet build (backend)
   - docker build (processing functions)
   ↓
3. Deploy (CloudFormation + SAM)
   - Update Lambda functions
   - Update container images in ECR
   - Deploy frontend to S3/CloudFront
   ↓
4. Test (automated smoke tests)
```

---

#### Observability & Monitoring

| Technology | Purpose |
|-----------|---------|
| **Amazon CloudWatch** | Logs, metrics, dashboards |
| **CloudWatch Insights** | Log query and analysis |
| **X-Ray** | Trace requests across services |
| **CloudWatch Alarms** | Alerts for anomalies |
| **Amazon SNS** | Email/SMS notifications |

**Key Metrics:**
- Lambda execution duration
- Error rates per function
- SQS queue depth
- RDS CPU & connection count
- AppSync subscription count
- Textract document processing cost

---

#### Security & Compliance

| Technology | Purpose |
|-----------|---------|
| **AWS WAF** | Web Application Firewall on CloudFront |
| **VPC Security Groups** | Network access control |
| **IAM Policies** | Least privilege access |
| **KMS** | Encryption key management |
| **Secrets Manager** | Rotate DB credentials |
| **VPC Endpoints** | Private access to AWS services |

**Encryption:**
- At-rest: S3 (SSE-S3/KMS), RDS (enabled)
- In-transit: HTTPS/TLS on all endpoints
- Database: Credentials in Secrets Manager

---

#### Optional/Advanced Technologies

| Technology | When to Use |
|-----------|-----------|
| **Amazon Bedrock** | Multi-model LLM access (Claude, Llama, Mistral) |
| **Amazon SageMaker** | Custom ML models for advanced matching |
| **AWS Glue** | ETL for batch processing, data catalog |
| **Athena** | SQL queries on S3 data (logs, archives) |
| **QuickSight** | Business analytics dashboard |
| **EventBridge** | Event-driven scheduler & integrations |

---

#### Development Tools

| Tool | Purpose |
|------|---------|
| **VS Code** | Editor with AWS Toolkit extension |
| **AWS SAM CLI** | Local testing of serverless functions |
| **LocalStack** | Local AWS emulation (development) |
| **Postman** | API testing |
| **AWS CLI** | Command-line interface |
| **Git** | Version control |

---

#### Summary: Why This Stack?

✅ **Serverless-First**: Auto-scale, pay-per-use, no server management

✅ **Event-Driven**: Step Functions + Lambda + SQS for async processing

✅ **Real-Time**: AppSync subscriptions push updates instantly

✅ **AI-Ready**: Bedrock, Textract, Comprehend built-in

✅ **Secure**: IAM, KMS, Cognito, WAF layered security

✅ **Cost-Effective**: Pay only for compute used, DynamoDB on-demand scaling

✅ **Scalable**: Multi-region ready, CloudFront edge caching

✅ **Observable**: CloudWatch integrated across all services

✅ **Infrastructure as Code**: Terraform for reproducible deployments


