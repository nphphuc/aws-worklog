---
title : "Repository Layout"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

#### SmartHire-AI Repository Structure

The SmartHire-AI project is organized into logical modules using a monorepo approach. This layout enables teams to work independently on frontend, backend, and infrastructure while maintaining shared standards.

---

#### High-Level Directory Structure

```
smart-hire-ai/
├── frontend/                 # React SPA
├── backend/                  # .NET 8 API
├── iac/                      # Infrastructure as Code
│   ├── terraform/           # AWS infrastructure
│   ├── sam/                 # Serverless API template
│   └── lambda/              # Python Lambda functions
├── docs/                    # Architecture & design
├── .github/                 # GitHub Actions workflows
├── docker-compose.yml       # Local development environment
├── README.md                # Project overview
└── LICENSE
```

---

#### Frontend Module (`frontend/`)

**React SPA with Vite**

```
frontend/
├── src/
│   ├── components/
│   │   ├── Auth/               # Login, sign-up, OAuth
│   │   ├── Dashboard/          # Main dashboard layouts
│   │   ├── Candidate/          # CV upload, job suggestions
│   │   ├── Recruiter/          # Job creation, rankings
│   │   └── Common/             # Header, sidebar, footer
│   ├── pages/
│   │   ├── CandidateDashboard.tsx
│   │   ├── RecruiterDashboard.tsx
│   │   ├── JobDetails.tsx
│   │   └── CandidateProfile.tsx
│   ├── graphql/
│   │   ├── queries.ts          # AppSync queries
│   │   └── subscriptions.ts    # Real-time subscriptions
│   ├── services/
│   │   ├── api.ts              # REST API calls
│   │   ├── auth.ts             # Cognito integration
│   │   └── appsync.ts          # GraphQL client setup
│   ├── store/
│   │   └── useStore.ts         # Zustand or Redux state
│   ├── styles/
│   │   └── tailwind.css        # Tailwind configuration
│   ├── App.tsx
│   └── main.tsx
├── public/                     # Static assets
├── vite.config.ts
├── tsconfig.json
├── tailwind.config.js
├── package.json
└── .env.example                # Environment template
```

**Key Dependencies:**
```json
{
  "react": "^18.2.0",
  "vite": "^4.5.0",
  "typescript": "^5.3.0",
  "tailwindcss": "^3.4.0",
  "@apollo/client": "^3.8.0",
  "aws-amplify": "^6.0.0"
}
```

---

#### Backend Module (`backend/`)

**.NET 8 REST API**

```
backend/
├── SmartHire.Api/
│   ├── Controllers/
│   │   ├── JobsController.cs
│   │   ├── CandidatesController.cs
│   │   └── AuthController.cs
│   ├── Services/
│   │   ├── JobService.cs
│   │   ├── CandidateService.cs
│   │   └── StepFunctionsService.cs
│   ├── Models/
│   │   ├── Job.cs
│   │   ├── Candidate.cs
│   │   └── MatchResult.cs
│   ├── Data/
│   │   ├── ApplicationDbContext.cs
│   │   └── Migrations/
│   ├── Middleware/
│   │   ├── CognitoAuthMiddleware.cs
│   │   └── ErrorHandling.cs
│   ├── Program.cs            # Dependency injection, routing
│   └── appsettings.json      # Configuration
├── SmartHire.Tests/          # Unit tests
├── Dockerfile
├── .gitignore
└── smarthire-api.csproj
```

**Project File:**
```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="AWSSDK.CognitoIdentityProvider" />
    <PackageReference Include="AWSSDK.StepFunctions" />
    <PackageReference Include="Amazon.Lambda.AspNetCoreServer" />
  </ItemGroup>
</Project>
```

---

#### Python Lambda Functions (`iac/lambda/`)

**CV & JD Processor**

```
iac/lambda/
├── cv_jd_processor/
│   ├── handler.py            # Lambda entry point
│   ├── processor.py          # Main logic
│   ├── requirements.txt       # Python dependencies
│   └── Dockerfile            # Container image
├── job_suggestion_engine/
│   ├── handler.py
│   ├── ranker.py
│   ├── requirements.txt
│   └── Dockerfile
├── candidate_ranking_engine/
│   ├── handler.py
│   ├── scorer.py
│   ├── requirements.txt
│   └── Dockerfile
└── shared/
    ├── embedding_utils.py    # Bedrock integration
    ├── textract_utils.py     # Amazon Textract wrapper
    └── db_utils.py           # RDS queries
```

**Example handler.py:**
```python
import json
import boto3
from processor import CVProcessor

textract = boto3.client('textract')
bedrock = boto3.client('bedrock-runtime')

def lambda_handler(event, context):
    """Process CV and generate embeddings"""
    bucket = event['bucket']
    key = event['key']
    
    processor = CVProcessor(textract, bedrock)
    result = processor.process_cv(bucket, key)
    
    return {
        'statusCode': 200,
        'body': json.dumps(result)
    }
```

**Dependencies:**
```
boto3==1.28.0           # AWS SDK
pdf2image==1.16.0       # PDF processing
langchain==0.1.0        # LLM orchestration
anthropic==0.7.0        # Bedrock client
pandas==2.1.0
numpy==1.24.0
```

---

#### Infrastructure as Code

**Terraform Structure (`iac/terraform/`)**

```
iac/terraform/
├── main.tf               # Provider, backend config
├── variables.tf          # Input variables
├── outputs.tf            # Output values
├── locals.tf             # Local values
├── terraform.tfvars      # Variable values (git-ignored)
├── vpc.tf                # VPC, subnets, security groups
├── rds.tf                # RDS PostgreSQL
├── cognito.tf            # User pool, identity provider
├── s3.tf                 # S3 buckets, policies
├── iam.tf                # IAM roles, policies
├── lambda.tf             # Lambda function configurations
├── step_functions.tf     # State machine definitions
├── appsync.tf            # GraphQL API
├── cloudfront.tf         # CDN distribution
├── dynamodb.tf           # DynamoDB tables
├── cloudwatch.tf         # Logs, alarms, dashboards
└── modules/              # Reusable Terraform modules
    ├── vpc/
    ├── rds/
    ├── cognito/
    └── lambda/
```

**SAM Template (`iac/sam/template.yaml`)**

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31

Parameters:
  CognitoPoolId:
    Type: String

Globals:
  Function:
    Timeout: 30
    Runtime: dotnet8

Resources:
  SmartHireApiFunction:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: SmartHireApi
      CodeUri: ../backend/
      Handler: SmartHire.Api::SmartHire.Api.LambdaEntryPoint::FunctionHandlerAsync
      Events:
        ApiEvent:
          Type: Api
          Properties:
            RestApiId: !Ref SmartHireApi
            Path: /{proxy+}
            Method: ANY
      Environment:
        Variables:
          COGNITO_POOL_ID: !Ref CognitoPoolId
```

---

#### AWS SAM Layer Structure

```
iac/sam/
├── template.yaml         # Main SAM template
├── parameters.json       # Parameter overrides
├── build/               # Build output (generated)
└── packaged.yaml        # Packaged template (generated)
```

---

#### Documentation (`docs/`)

```
docs/
├── architecture-smart-matching.md    # Detailed architecture
├── api-specification.md              # REST API contracts
├── graphql-schema.md                 # AppSync schema
├── deployment-guide.md               # How to deploy
├── development-setup.md              # Local development
├── security-model.md                 # Auth & RBAC
├── cost-estimation.md                # AWS cost breakdown
└── troubleshooting.md                # Common issues
```

---

#### CI/CD Configuration

**GitHub Actions Workflows (`.github/workflows/`)**

```
.github/workflows/
├── deploy-frontend.yml       # Build & deploy React SPA
├── deploy-backend.yml        # Build & deploy .NET Lambda
├── deploy-infrastructure.yml # Terraform apply
├── test.yml                  # Run automated tests
└── security-scan.yml         # CodeQL, dependency check
```

**Example deploy-frontend.yml:**
```yaml
name: Deploy Frontend
on:
  push:
    branches: [main]
    paths: ['frontend/**']

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install dependencies
        run: cd frontend && npm install
      - name: Build
        run: cd frontend && npm run build
      - name: Deploy to S3
        run: aws s3 sync frontend/dist s3://${{ secrets.FRONTEND_BUCKET }}/
      - name: Invalidate CloudFront
        run: aws cloudfront create-invalidation --distribution-id ${{ secrets.CF_DIST_ID }} --paths "/*"
```

---

#### Local Development Setup

**Docker Compose (`docker-compose.yml`)**

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: local_password
      POSTGRES_DB: smarthire_db
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  dynamodb-local:
    image: amazon/dynamodb-local
    ports:
      - "8000:8000"

  localstack:
    image: localstack/localstack:latest
    ports:
      - "4566:4566"
    environment:
      SERVICES: s3,sqs,sns,lambda
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock

  frontend:
    build: ./frontend
    ports:
      - "5173:5173"
    environment:
      VITE_APP_API_URL: http://localhost:3000
      VITE_APP_REGION: us-east-1

  backend:
    build: ./backend
    ports:
      - "3000:3000"
    depends_on:
      - postgres
    environment:
      DATABASE_URL: postgresql://postgres:local_password@postgres:5432/smarthire_db

volumes:
  postgres_data:
```

---

#### Environment Variables

**Frontend `.env.example`:**
```
VITE_APP_API_URL=https://api.smarthire.com
VITE_APP_REGION=us-east-1
VITE_APP_COGNITO_DOMAIN=smarthire.auth.us-east-1.amazoncognito.com
VITE_APP_COGNITO_CLIENT_ID=xxxxxxxxx
VITE_APP_APPSYNC_ENDPOINT=https://xxxxxxxxx.appsync-api.us-east-1.amazonaws.com/graphql
```

**Backend `appsettings.Development.json`:**
```json
{
  "Cognito": {
    "UserPoolId": "us-east-1_xxxxxxxxx",
    "ClientId": "xxxxxxxxx",
    "Authority": "https://cognito-idp.us-east-1.amazonaws.com/us-east-1_xxxxxxxxx"
  },
  "Database": {
    "ConnectionString": "postgresql://localhost:5432/smarthire_db"
  },
  "StepFunctions": {
    "StateMachineArn": "arn:aws:states:us-east-1:123456789012:stateMachine:CVProcessingStateMachine"
  }
}
```

---

#### Deployment Workflow

**Development:**
```bash
cd frontend && npm install && npm run dev
cd backend && dotnet run
docker-compose up   # Local PostgreSQL, DynamoDB
```

**Staging:**
```bash
terraform apply -var-file=staging.tfvars
sam package --s3-bucket staging-artifacts
sam deploy --guided
```

**Production:**
```bash
terraform apply -var-file=prod.tfvars
sam package --s3-bucket prod-artifacts
sam deploy --guided
```

---

#### Key Files to Know

| File | Purpose |
|------|---------|
| `README.md` | Project overview & quick start |
| `docker-compose.yml` | Local dev environment |
| `iac/terraform/main.tf` | AWS resource definitions |
| `iac/sam/template.yaml` | Serverless API template |
| `backend/Program.cs` | .NET startup & DI config |
| `frontend/src/App.tsx` | React root component |
| `.github/workflows/` | CI/CD automation |