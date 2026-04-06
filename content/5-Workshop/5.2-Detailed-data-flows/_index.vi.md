---
title : "Kiến trúc"
date : 2024-01-01 
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

#### Tổng quan Kiến trúc SmartHire-AI

SmartHire-AI tận dụng một kiến trúc Serverless tinh vi của AWS mà tự động mở rộng quy mô dựa trên nhu cầu. Dưới đây là thiết kế hệ thống hoàn chỉnh:

```
┌─────────────────────────────────────────────────────────────┐
│                     TẦNG WEB (Frontend)                      │
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

#### Các Thành phần Chính

**1. Frontend (Tầng Web)**
- **React SPA**: Xây dựng bằng Vite, TypeScript để kích thước bundle tối ưu
- **CDN**: CloudFront để phân phối toàn cầu
- **Hosting**: S3 static website hosting
- **Auth**: Cognito user pool với Google OAuth federation

**2. API Layer (.NET 8 Lambda + API Gateway)**
- HTTP REST API cho các hoạt động việc làm
- Ủy quyền JWT qua Cognito
- Truy cập VPC đến RDS cho dữ liệu ứng viên/việc làm
- Secrets Manager cho các thông tin nhạy cảm
- Kích hoạt Step Functions để xử lý không đồng bộ

**3. Pipeline Xử lý CV/JD**
- **Step Functions**: Điều phối workflow xử lý
- **AWS Lambda (Python 3.12)**: Thực thi các tác vụ xử lý
  - `cv_jd_processor`: Trích xuất và làm giàu dữ liệu
  - `job_suggestion_engine`: Xếp hạng việc làm cho ứng viên
  - `candidate_ranking_engine`: Xếp hạng ứng viên cho việc làm
- **Container Images**: Được lưu trữ trong Amazon ECR cho xử lý phức tạp
- **SQS Queue**: Đệm CV uploads, bao gồm Dead Letter Queue (DLQ)
- **S3 Bucket**: Lưu trữ CV thô với thông báo sự kiện

**4. AI & Document Analysis**
- **Amazon Textract**: Trích xuất text từ PDF resumes
- **Amazon Bedrock**: Large Language Model để làm giàu & kết hợp
- **Amazon Comprehend**: NLP để nhận dạng thực thể và cảm xúc

**5. Tầng Dữ liệu**
- **RDS (PostgreSQL)**: Việc làm, ứng viên, dữ liệu người dùng
- **DynamoDB**: Theo dõi ứng dụng thời gian thực & kết quả kết hợp
- **S3**: CV uploads, tài sản tĩnh

**6. Cập nhật Thời gian thực**
- **AWS AppSync**: GraphQL API với subscriptions
- WebSocket connections để cập nhật dashboard tức thì
- Tách riêng từ API chính để mở rộng quy mô độc lập

**7. Infrastructure as Code**
- **Terraform**: Cấp phát VPC, RDS, Cognito, mạng, CI/CD
- **AWS SAM**: Quản lý ngăn xếp API Gateway + Lambda
- **CodePipeline**: Tự động hóa triển khai từ GitHub

---

#### Các Thành phần Tùy chọn

- **AWS WAF**: Web Application Firewall trên CloudFront
- **Route 53**: Quản lý tên miền tùy chỉnh
- **ACM**: Chứng chỉ SSL/TLS
- **VPC Interface Endpoints**: Truy cập riêng tư vào dịch vụ AWS
- **NAT Gateway**: Truy cập internet đi ra từ Lambda
- **CloudWatch**: Ghi nhật ký và giám sát
- **SNS**: Cảnh báo cho tình trạng pipeline
