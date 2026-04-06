---
title : "Cấu trúc Kho mã"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

#### Cấu trúc Kho mã SmartHire-AI

Dự án SmartHire-AI được tổ chức thành các mô-đun logic sử dụng cách tiếp cận monorepo. Bố cục này cho phép các đội làm việc độc lập trên frontend, backend và cơ sở hạ tầng trong khi duy trì các tiêu chuẩn chung.

---

#### Cấu trúc Thư mục Cấp cao

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

**React SPA với Vite**

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
Bao gồm:
- PostgreSQL database
- DynamoDB Local
- LocalStack (AWS services)
- Frontend dev server
- Backend dev server
```

---

#### Các Files Quan trọng

| File | Mục đích |
|------|---------|
| `README.md` | Project overview |
| `docker-compose.yml` | Local dev environment |
| `iac/terraform/main.tf` | AWS resources |
| `iac/sam/template.yaml` | Serverless API |
| `frontend/src/App.tsx` | React root |
| `.github/workflows/` | CI/CD automation |

---

#### Quy trình Triển khai

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

#### Tiếp theo

Bạn đã hoàn thành workshop SmartHire-AI! Bây giờ bạn hiểu:

✅ Kiến trúc serverless trên AWS

✅ Luồng xử lý CV từ upload đến matching

✅ Luồng xếp hạng ứng viên cho công việc

✅ Cách các dịch vụ AWS làm việc cùng nhau

✅ Cấu trúc dự án và quy trình triển khai

Để triển khai SmartHire-AI:
1. Clone repository
2. Cấu hình AWS credentials
3. Chạy Terraform để cấp phát cơ sở hạ tầng
4. Triển khai SAM template cho API
5. Build và đẩy frontend lên S3
