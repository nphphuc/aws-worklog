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

#### Data Flow Architecture

```mermaid
flowchart LR
  subgraph web [Frontend]
    SPA["React SPA"]
  end
  subgraph cdn [CDN & Auth]
    CF["CloudFront"]
    COG["Cognito"]
  end
  subgraph api [API]
    APIGW["API Gateway"]
    ApiFn["Lambda .NET8"]
  end
  subgraph ingest [CV Ingest]
    S3raw["S3 CV Bucket"]
    SQS["SQS Queue"]
    IngestFn["Lambda Ingestion"]
  end
  subgraph orchestration [Processing]
    SFN["Step Functions"]
    ProcFn["Lambda CV/JD Processor"]
    JS["Job Suggestion"]
    CR["Candidate Ranking"]
  end
  subgraph ai [AI Services]
    TX["Textract"]
    BR["Bedrock"]
    CM["Comprehend"]
  end
  subgraph data [Data]
    RDS[("RDS<br/>PostgreSQL")]
    DDB[("DynamoDB")]
  end
  subgraph realtime [Realtime]
    APS["AWS AppSync<br/>GraphQL"]
  end
  
  SPA → CF
  SPA → COG
  SPA → APIGW
  APIGW → ApiFn
  ApiFn → RDS
  ApiFn → SFN
  SPA → APS
  
  S3raw → SQS
  SQS → IngestFn
  IngestFn → SFN
  
  SFN → ProcFn
  ProcFn → TX
  ProcFn → BR
  ProcFn → CM
  
  SFN → JS
  SFN → CR
  
  ProcFn → RDS
  ProcFn → DDB
  
  JS → APS
  CR → APS
```

#### Optional Components

- **AWS WAF**: Web Application Firewall on CloudFront
- **Route 53**: Custom domain management
- **ACM**: SSL/TLS certificates
- **VPC Interface Endpoints**: Private access to AWS services
- **NAT Gateway**: Outbound internet access from Lambda
- **CloudWatch**: Logging and monitoring
- **SNS**: Alarms for pipeline health
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

#### Provision resources using CloudFormation

In this lab, we will use **N.Virginia region (us-east-1)**.

To prepare the workshop environment, deploy this **CloudFormation Template** (click link): [PrivateLinkWorkshop ](https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/quickcreate?templateURL=https://s3.us-east-1.amazonaws.com/reinvent-endpoints-builders-session/Nested.yaml&stackName=PLCloudSetup). Accept all of the defaults when deploying the template. 

![create stack](/images/5-Workshop/5.2-Prerequisite/create-stack1.png)

+ Tick 2 acknowledgement boxes
+ Choose **Create stack**

![create stack](/images/5-Workshop/5.2-Prerequisite/create-stack2.png)

The **ClouddFormation** deployment requires about 15 minutes to complete.

![complete](/images/5-Workshop/5.2-Prerequisite/complete.png)

+ **2 VPCs** have been created

![vpcs](/images/5-Workshop/5.2-Prerequisite/vpcs.png)

+ **3 EC2s** have been created

![EC2](/images/5-Workshop/5.2-Prerequisite/ec2.png)