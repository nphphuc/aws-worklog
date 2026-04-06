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

#### Luồng Kiến trúc Dữ liệu

Kiến trúc này hỗ trợ hai luồng chính:

**Luồng Ứng viên:**
CV Upload → S3 → SQS → Lambda → Step Functions → Processing → AppSync → UI

**Luồng Nhà tuyển dụng:**
Job Creation → API Gateway → Lambda → RDS → Step Functions → Processing → AppSync → UI

---

#### Các Thành phần Tùy chọn

- **AWS WAF**: Web Application Firewall trên CloudFront
- **Route 53**: Quản lý tên miền tùy chỉnh
- **ACM**: Chứng chỉ SSL/TLS
- **VPC Interface Endpoints**: Truy cập riêng tư vào dịch vụ AWS
- **NAT Gateway**: Truy cập internet đi ra từ Lambda
- **CloudWatch**: Ghi nhật ký và giám sát
- **SNS**: Cảnh báo cho tình trạng pipeline
                "ec2:CreateTags",
                "ec2:CreateTransitGateway",
                "ec2:CreateTransitGatewayPeeringAttachment",
                "ec2:CreateTransitGatewayPrefixListReference",
                "ec2:CreateTransitGatewayRoute",
                "ec2:CreateTransitGatewayRouteTable",
                "ec2:CreateTransitGatewayVpcAttachment",
                "ec2:CreateVpc",
                "ec2:CreateVpcEndpoint",
                "ec2:CreateVpcEndpointConnectionNotification",
                "ec2:CreateVpcEndpointServiceConfiguration",
                "ec2:CreateVpnConnection",
                "ec2:CreateVpnConnectionRoute",
                "ec2:CreateVpnGateway",
                "ec2:DeleteCustomerGateway",
                "ec2:DeleteFlowLogs",
                "ec2:DeleteInternetGateway",
                "ec2:DeleteNetworkInterface",
                "ec2:DeleteNetworkInterfacePermission",
                "ec2:DeleteRoute",
                "ec2:DeleteRouteTable",
                "ec2:DeleteSecurityGroup",
                "ec2:DeleteSubnet",
                "ec2:DeleteSubnetCidrReservation",
                "ec2:DeleteTags",
                "ec2:DeleteTransitGateway",
                "ec2:DeleteTransitGatewayPeeringAttachment",
                "ec2:DeleteTransitGatewayPrefixListReference",
                "ec2:DeleteTransitGatewayRoute",
                "ec2:DeleteTransitGatewayRouteTable",
                "ec2:DeleteTransitGatewayVpcAttachment",
                "ec2:DeleteVpc",
                "ec2:DeleteVpcEndpoints",
                "ec2:DeleteVpcEndpointServiceConfigurations",
                "ec2:DeleteVpnConnection",
                "ec2:DeleteVpnConnectionRoute",
                "ec2:Describe*",
                "ec2:DetachInternetGateway",
                "ec2:DisassociateAddress",
                "ec2:DisassociateRouteTable",
                "ec2:GetLaunchTemplateData",
                "ec2:GetTransitGatewayAttachmentPropagations",
                "ec2:ModifyInstanceAttribute",
                "ec2:ModifySecurityGroupRules",
                "ec2:ModifyTransitGatewayVpcAttachment",
                "ec2:ModifyVpcAttribute",
                "ec2:ModifyVpcEndpoint",
                "ec2:ReleaseAddress",
                "ec2:ReplaceRoute",
                "ec2:RevokeSecurityGroupEgress",
                "ec2:RevokeSecurityGroupIngress",
                "ec2:RunInstances",
                "ec2:StartInstances",
                "ec2:StopInstances",
                "ec2:UpdateSecurityGroupRuleDescriptionsEgress",
                "ec2:UpdateSecurityGroupRuleDescriptionsIngress",
                "iam:AddRoleToInstanceProfile",
                "iam:AttachRolePolicy",
                "iam:CreateInstanceProfile",
                "iam:CreatePolicy",
                "iam:CreateRole",
                "iam:DeleteInstanceProfile",
                "iam:DeletePolicy",
                "iam:DeleteRole",
                "iam:DeleteRolePolicy",
                "iam:DetachRolePolicy",
                "iam:GetInstanceProfile",
                "iam:GetPolicy",
                "iam:GetRole",
                "iam:GetRolePolicy",
                "iam:ListPolicyVersions",
                "iam:ListRoles",
                "iam:PassRole",
                "iam:PutRolePolicy",
                "iam:RemoveRoleFromInstanceProfile",
                "lambda:CreateFunction",
                "lambda:DeleteFunction",
                "lambda:DeleteLayerVersion",
                "lambda:GetFunction",
                "lambda:GetLayerVersion",
                "lambda:InvokeFunction",
                "lambda:PublishLayerVersion",
                "logs:CreateLogGroup",
                "logs:DeleteLogGroup",
                "logs:DescribeLogGroups",
                "logs:PutRetentionPolicy",
                "route53:ChangeTagsForResource",
                "route53:CreateHealthCheck",
                "route53:CreateHostedZone",
                "route53:CreateTrafficPolicy",
                "route53:DeleteHostedZone",
                "route53:DisassociateVPCFromHostedZone",
                "route53:GetHostedZone",
                "route53:ListHostedZones",
                "route53domains:ListDomains",
                "route53domains:ListOperations",
                "route53domains:ListTagsForDomain",
                "route53resolver:AssociateResolverEndpointIpAddress",
                "route53resolver:AssociateResolverRule",
                "route53resolver:CreateResolverEndpoint",
                "route53resolver:CreateResolverRule",
                "route53resolver:DeleteResolverEndpoint",
                "route53resolver:DeleteResolverRule",
                "route53resolver:DisassociateResolverEndpointIpAddress",
                "route53resolver:DisassociateResolverRule",
                "route53resolver:GetResolverEndpoint",
                "route53resolver:GetResolverRule",
                "route53resolver:ListResolverEndpointIpAddresses",
                "route53resolver:ListResolverEndpoints",
                "route53resolver:ListResolverRuleAssociations",
                "route53resolver:ListResolverRules",
                "route53resolver:ListTagsForResource",
                "route53resolver:UpdateResolverEndpoint",
                "route53resolver:UpdateResolverRule",
                "s3:AbortMultipartUpload",
                "s3:CreateBucket",
                "s3:DeleteBucket",
                "s3:DeleteObject",
                "s3:GetAccountPublicAccessBlock",
                "s3:GetBucketAcl",
                "s3:GetBucketOwnershipControls",
                "s3:GetBucketPolicy",
                "s3:GetBucketPolicyStatus",
                "s3:GetBucketPublicAccessBlock",
                "s3:GetObject",
                "s3:GetObjectVersion",
                "s3:GetBucketVersioning",
                "s3:ListAccessPoints",
                "s3:ListAccessPointsForObjectLambda",
                "s3:ListAllMyBuckets",
                "s3:ListBucket",
                "s3:ListBucketMultipartUploads",
                "s3:ListBucketVersions",
                "s3:ListJobs",
                "s3:ListMultipartUploadParts",
                "s3:ListMultiRegionAccessPoints",
                "s3:ListStorageLensConfigurations",
                "s3:PutAccountPublicAccessBlock",
                "s3:PutBucketAcl",
                "s3:PutBucketPolicy",
                "s3:PutBucketPublicAccessBlock",
                "s3:PutObject",
                "secretsmanager:CreateSecret",
                "secretsmanager:DeleteSecret",
                "secretsmanager:DescribeSecret",
                "secretsmanager:GetSecretValue",
                "secretsmanager:ListSecrets",
                "secretsmanager:ListSecretVersionIds",
                "secretsmanager:PutResourcePolicy",
                "secretsmanager:TagResource",
                "secretsmanager:UpdateSecret",
                "sns:ListTopics",
                "ssm:DescribeInstanceProperties",
                "ssm:DescribeSessions",
                "ssm:GetConnectionStatus",
                "ssm:GetParameters",
                "ssm:ListAssociations",
                "ssm:ResumeSession",
                "ssm:StartSession",
                "ssm:TerminateSession"
            ],
            "Resource": "*"
        }
    ]
}

```

#### Khởi tạo tài nguyên bằng CloudFormation

Trong lab này, chúng ta sẽ dùng N.Virginia region (us-east-1).

Để chuẩn bị cho môi trường làm workshop, chúng ta deploy CloudFormation template sau (click link): [PrivateLinkWorkshop ](https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/quickcreate?templateURL=https://s3.us-east-1.amazonaws.com/reinvent-endpoints-builders-session/Nested.yaml&stackName=PLCloudSetup). Để nguyên các lựa chọn mặc định.

![create stack](/images/5-Workshop/5.2-Prerequisite/create-stack1.png)

+ Lựa chọn 2 mục acknowledgement 
+ Chọn Create stack

![create stack](/images/5-Workshop/5.2-Prerequisite/create-stack2.png)

Quá trình triển khai CloudFormation cần khoảng 15 phút để hoàn thành.

![complete](/images/5-Workshop/5.2-Prerequisite/complete.png)

+ 2 VPCs đã được tạo

![vpcs](/images/5-Workshop/5.2-Prerequisite/vpcs.png)

+ 3 EC2s đã được tạo

![EC2](/images/5-Workshop/5.2-Prerequisite/ec2.png)