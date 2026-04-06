---
title : "DynamoDB Schema"
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
│   ├── components/          # React components
│   ├── pages/              # Page components
│   ├── graphql/            # AppSync queries & subscriptions
│   ├── services/           # API services
│   ├── store/              # State management
│   ├── styles/             # CSS/Tailwind
│   ├── App.tsx
│   └── main.tsx
├── public/                 # Static assets
├── vite.config.ts
├── tsconfig.json
├── tailwind.config.js
├── package.json
└── .env.example
```

---

#### Backend Module (`backend/`)

**.NET 8 REST API**

```
backend/
├── SmartHire.Api/
│   ├── Controllers/        # REST endpoints
│   ├── Services/           # Business logic
│   ├── Models/             # Data models
│   ├── Data/               # Database context
│   ├── Middleware/
│   ├── Program.cs
│   └── appsettings.json
├── SmartHire.Tests/        # Unit tests
├── Dockerfile
└── smarthire-api.csproj
```

---

#### Python Lambda Functions (`iac/lambda/`)

**CV & JD Processor**

```
iac/lambda/
├── cv_jd_processor/        # CV processing
├── job_suggestion_engine/  # Job suggestions
├── candidate_ranking_engine/ # Candidate ranking
└── shared/                 # Shared utilities
```

---

#### Infrastructure as Code

**Terraform (`iac/terraform/`)**

```
iac/terraform/
├── main.tf                 # Provider config
├── variables.tf
├── outputs.tf
├── vpc.tf                  # VPC & networking
├── rds.tf                  # Database
├── cognito.tf              # Auth
├── s3.tf                   # Storage
├── lambda.tf               # Lambda
├── step_functions.tf       # Orchestration
├── appsync.tf              # GraphQL
├── cloudfront.tf           # CDN
└── modules/                # Reusable modules
```

**AWS SAM (`iac/sam/`)**

```
iac/sam/
├── template.yaml           # SAM template
├── parameters.json
└── packaged.yaml
```

---

#### CI/CD Configuration

**GitHub Actions (`.github/workflows/`)**

```
.github/workflows/
├── deploy-frontend.yml
├── deploy-backend.yml
├── deploy-infrastructure.yml
├── test.yml
└── security-scan.yml
```

---

#### Local Development

**Docker Compose**

```yaml
Includes:
- PostgreSQL database
- DynamoDB Local
- LocalStack (AWS services)
- Frontend dev server
- Backend dev server
```

---

#### Important Files

| File | Purpose |
|------|---------|
| `README.md` | Project overview |
| `docker-compose.yml` | Local dev environment |
| `iac/terraform/main.tf` | AWS resources |
| `iac/sam/template.yaml` | Serverless API |
| `frontend/src/App.tsx` | React root |
| `.github/workflows/` | CI/CD automation |

---

#### Deployment Process

**Development:**
```bash
docker-compose up
npm run dev        # Frontend
dotnet run         # Backend
```

**Staging/Production:**
```bash
terraform apply
sam deploy
```

---

#### Next Steps

You have completed the SmartHire-AI workshop! You now understand:

✅ Serverless architecture on AWS

✅ CV processing flow from upload to matching

✅ Candidate ranking flow for jobs

✅ How AWS services work together

✅ Project structure and deployment process

To deploy SmartHire-AI:
1. Clone the repository
2. Configure AWS credentials
3. Run Terraform to provision infrastructure
4. Deploy SAM template for API
5. Build and push frontend to S3
