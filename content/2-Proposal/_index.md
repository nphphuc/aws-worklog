---
title: "Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# SmartHire AI
## An AWS Serverless Solution for Automated CV Extraction and Candidate Evaluation

### 1. Executive Summary
SmartHire AI is a recruitment support platform built on a 100% Serverless architecture on AWS, aimed at automating the two most time-consuming core processes in HR operations: receiving and parsing candidate resumes (CVs), and aggregating and evaluating interview results. The system leverages an Event-Driven architecture combined with Generative AI (Amazon Bedrock) services to deliver high performance, reduced bandwidth costs, and fair, consistent candidate scoring.

### 2. Problem Statement
### What’s the Problem?
Uploading large CV files (PDF, DOCX) through an intermediate server frequently causes bandwidth bottlenecks, degrading the candidate experience and increasing infrastructure costs. HR teams and Technical Leaders must manually read each CV to filter skills, consuming significant manhours and risking overlooking promising candidates. After interviews, consolidating feedback and comparing scores across candidates often lacks consistency and relies heavily on individual interviewer judgment.

### The Solution
The platform uses S3 Presigned URLs so candidates can upload CVs directly to Amazon S3 without going through the Backend, completely eliminating bandwidth bottlenecks. The S3 Event Notification → SQS → Lambda → Amazon Textract pipeline automatically extracts raw text asynchronously as soon as a CV is uploaded. Amazon Bedrock (Claude 3.5 Sonnet) analyzes the raw text into structured JSON stored in DynamoDB. Meanwhile, AWS Step Functions orchestrates a parallel interview scoring workflow based on the STAR method, automatically generating reports for HR.

### Benefits and Return on Investment
The solution allows HR to focus on high-value tasks rather than manually reading each resume, reducing CV screening time by 80%. Post-interview reports are generated automatically in under 90 seconds through parallel processing, ensuring consistent and objective evaluation criteria across all candidates. The Serverless Pay-as-you-go architecture optimizes operational costs and scales infinitely to support large-scale recruitment drives.

### 3. Solution Architecture
The platform uses a Serverless architecture with a clear separation between the API layer and heavy background processing pipelines. All data processing is performed asynchronously to ensure a seamless user experience:

![SmartHire architecture](https://res.cloudinary.com/diahbkjog/image/upload/v1775413223/smarthire_architecture_cugoh4.png)

### AWS Services Used
- **Amazon S3**: Stores original CVs, generates Presigned URLs for direct candidate uploads, and stores final PDF/HTML reports.
- **Amazon SQS**: Message queue that prevents the system from being overwhelmed when thousands of CVs are uploaded simultaneously.
- **AWS Lambda**: Worker functions that execute AI call logic, process data, and store results.
- **Amazon Textract**: Optical Character Recognition (OCR) to extract text from PDF and DOCX file formats.
- **Amazon Bedrock**: Provides the LLM (Claude 3.5 Sonnet) for semantic CV analysis and interview answer scoring.
- **AWS Step Functions**: Orchestrates multi-step workflows for parallel candidate scoring.
- **Amazon DynamoDB**: Stores candidate metadata, extracted skill structures, and processing status.
- **Amazon API Gateway**: Communication gateway between the Frontend and Backend Lambda functions.
- **Amazon Cognito**: Authentication and authorization for users (candidates, HR, recruiters).

### Component Design
- **CV Upload Layer**: Frontend calls API Gateway → Lambda → generates S3 Presigned URL; candidate PUTs the file directly to S3.
- **Event-Driven Pipeline**: S3 Event Notification triggers SQS → Lambda Worker calls Textract and Bedrock → stores JSON in DynamoDB.
- **AI Evaluation Workflow**: Step Functions Express Workflow orchestrates Parse → ScoreParallel → Aggregate → GenerateReport.
- **Notification Layer**: Lambda updates processing status and delivers results to the Frontend via AppSync (GraphQL Subscription).
- **Auth & Security**: Cognito User Pool issues JWT tokens; API Gateway uses a Cognito Authorizer to authenticate every protected request.

### 4. Technical Implementation
**Implementation Phases**
The project is delivered across 4 consecutive phases during the internship period:
- **Phase 1 — Storage Infrastructure & Upload API (Week 1):** Set up the S3 bucket, configure IAM Roles, and build the Lambda API to generate Presigned URLs for the Frontend. Candidates upload CVs directly to S3 without passing through the Backend.
- **Phase 2 — Event-Driven Pipeline & Textract (Week 2):** Configure S3 Event Notification → SQS trigger → Lambda Worker integrated with Amazon Textract to extract raw text from PDF/DOCX files.
- **Phase 3 — Bedrock Integration & DynamoDB (Week 3):** Design Prompt Engineering, call Amazon Bedrock (Claude 3.5 Sonnet) to parse raw text into JSON (`skills`, `yearsOfExperience`, `education`), and store results in DynamoDB.
- **Phase 4 — Step Functions & Finalization (Week 4):** Build the Step Functions Express Workflow with a Parallel State for concurrent interview scoring, generate PDF/HTML reports saved back to S3. Conduct security testing, optimization, and documentation.

**Technical Requirements**
- **CV Parsing Pipeline:** S3 bucket (prefix `cvs/`), SQS Standard Queue with DLQ, Lambda runtime Python 3.12, Amazon Textract AnalyzeDocument API, Amazon Bedrock InvokeModel API (claude-3-5-sonnet), DynamoDB table with Single Table Design.
- **AI Evaluation Workflow:** Step Functions Express Workflow, Parallel State with up to 10 concurrent branches, Lambda integrated with Bedrock to generate overall summaries, S3 for report output storage.
- **API & Auth:** API Gateway REST API, Cognito User Pool + App Client, JWT Authorizer on all protected endpoints.

### 5. Timeline & Milestones
**Project Timeline**
- **Week 1:** Set up S3 bucket, build the Presigned URL API; configure IAM Roles for Lambda; test the direct upload flow from the Frontend.
- **Week 2:** Build the Event-Driven Pipeline: S3 Event Notification → SQS → Lambda; integrate and test Amazon Textract across various PDF and DOCX formats.
- **Week 3:** Design Prompt Engineering for Amazon Bedrock; standardize JSON output; store metadata in DynamoDB; evaluate extraction accuracy.
- **Week 4:** Build the Step Functions State Machine with Parallel scoring; generate PDF/HTML reports; conduct security testing (IAM, Cognito, S3 Bucket Policy) and finalize documentation.

### 6. Budget Estimation
You can explore the cost estimate on the [AWS Pricing Calculator](https://calculator.aws/).

### Infrastructure Costs
- **AWS Lambda:** $0.00/month (within Free Tier — first 1 million requests free).
- **Amazon S3 Standard:** ~$0.05/month (CV and report storage, ~2 GB).
- **Amazon SQS:** $0.00/month (under 1 million requests/month — Free Tier).
- **Amazon Textract:** ~$0.015/page (AnalyzeDocument API).
- **Amazon Bedrock (Claude 3.5 Sonnet):** ~$0.10–$0.30/month depending on CV processing volume.
- **AWS Step Functions:** $0.00/month (first 4,000 state transitions free).
- **Amazon DynamoDB:** $0.00/month (25 GB storage + 25 WCU/RCU — Free Tier).
- **Amazon API Gateway:** ~$0.01/month (under 1 million calls).

**Total Estimated Cost: ~$0.50–$1.00/month** (depending on actual CV traffic volume)

### 7. Risk Assessment
#### Risk Matrix
- **Inaccurate Prompt Engineering:** High impact, medium probability — CV extraction produces incorrect field values.
- **Lambda Timeout:** Medium impact, low probability — CV file too large, or Textract/Bedrock responds slowly.
- **Bedrock Cost Overrun:** Medium impact, low probability — if CV upload volume spikes unexpectedly.
- **DLQ Message Accumulation:** Low impact, medium probability — Lambda Worker repeatedly fails to process messages.

#### Mitigation Strategies
- **Prompt Engineering:** Build a test suite using diverse CVs (Vietnamese, English, multiple formats); evaluate accuracy per iteration before deploying.
- **Lambda Timeout:** Increase timeout as needed (up to 15 minutes); split the Textract and Bedrock steps into two separate Lambda functions if required.
- **Cost:** Set up an AWS Budget Alert to notify when spending exceeds $5/month; use Bedrock On-Demand pricing instead of Provisioned.
- **DLQ:** Configure a CloudWatch Alarm to monitor `ApproximateNumberOfMessagesNotVisible`; create a runbook for periodic DLQ reprocessing.

#### Contingency Plans
- If Bedrock is unavailable: fall back to Regex-based parsing to keep the pipeline operational.
- If Step Functions fails: manually re-trigger executions via the AWS Console or CLI.

### 8. Expected Outcomes
#### Technical Improvements: 
Fast, seamless CV upload and processing with no interruptions, thanks to the fully asynchronous architecture. The system can scale infinitely to handle large recruitment campaigns without requiring any server upgrades.
#### Long-term Value
A reduction of **80%** in manual CV reading and classification time. Post-interview reports generated automatically in **under 90 seconds** through parallel processing. Consistent, objective evaluation criteria reusable across multiple job positions. The Serverless Pay-as-you-go model optimizes operational costs by charging only for actual usage.