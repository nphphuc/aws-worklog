---
title: "Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---
<!-- {{% notice warning %}}
⚠️ **Note:** The information below is for reference purposes only. Please **do not copy verbatim** for your report, including this warning.
{{% /notice %}} -->

<!-- In this section, you need to summarize the contents of the workshop that you **plan** to conduct. -->

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

<!-- ![IoT Weather Station Architecture](/images/2-Proposal/edge_architecture.jpeg)

![IoT Weather Platform Architecture](/images/2-Proposal/platform_architecture.jpeg) -->
![SmartHire architecture](https://media.discordapp.net/attachments/1227176240364916788/1490398906859786341/616856582_4425812177640627_3655938540521346192_n.jpg?ex=69d3e9a5&is=69d29825&hm=f0454329e251a7d7d41a528add45eb7362fef9bd7fd2b9873e80ca89ff930db8&=&format=webp)

### AWS Services Used
<!-- - **AWS IoT Core**: Ingests MQTT data from 5 stations, scalable to 15.
- **AWS Lambda**: Processes data and triggers Glue jobs (two functions).
- **Amazon API Gateway**: Facilitates web app communication.
- **Amazon S3**: Stores raw data in a data lake and processed outputs (two buckets).
- **AWS Glue**: Crawlers catalog data, and ETL jobs transform and load it.
- **AWS Amplify**: Hosts the Next.js web interface.
- **Amazon Cognito**: Secures access for lab users. -->
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
<!-- - **Edge Devices**: Raspberry Pi collects and filters sensor data, sending it to IoT Core.
- **Data Ingestion**: AWS IoT Core receives MQTT messages from the edge devices.
- **Data Storage**: Raw data is stored in an S3 data lake; processed data is stored in another S3 bucket.
- **Data Processing**: AWS Glue Crawlers catalog the data, and ETL jobs transform it for analysis.
- **Web Interface**: AWS Amplify hosts a Next.js app for real-time dashboards and analytics.
- **User Management**: Amazon Cognito manages user access, allowing up to 5 active accounts. -->
- **CV Upload Layer**: Frontend calls API Gateway → Lambda → generates S3 Presigned URL; candidate PUTs the file directly to S3.
- **Event-Driven Pipeline**: S3 Event Notification triggers SQS → Lambda Worker calls Textract and Bedrock → stores JSON in DynamoDB.
- **AI Evaluation Workflow**: Step Functions Express Workflow orchestrates Parse → ScoreParallel → Aggregate → GenerateReport.
- **Notification Layer**: Lambda updates processing status and delivers results to the Frontend via AppSync (GraphQL Subscription).
- **Auth & Security**: Cognito User Pool issues JWT tokens; API Gateway uses a Cognito Authorizer to authenticate every protected request.

### 4. Technical Implementation
**Implementation Phases**
<!-- This project has two parts—setting up weather edge stations and building the weather platform—each following 4 phases:
- Build Theory and Draw Architecture: Research Raspberry Pi setup with ESP32 sensors and design the AWS serverless architecture (1 month pre-internship)
- Calculate Price and Check Practicality: Use AWS Pricing Calculator to estimate costs and adjust if needed (Month 1).
- Fix Architecture for Cost or Solution Fit: Tweak the design (e.g., optimize Lambda with Next.js) to stay cost-effective and usable (Month 2).
- Develop, Test, and Deploy: Code the Raspberry Pi setup, AWS services with CDK/SDK, and Next.js app, then test and release to production (Months 2-3). -->
The project is delivered across 4 consecutive phases during the internship period:
- **Phase 1 — Storage Infrastructure & Upload API (Week 1):** Set up the S3 bucket, configure IAM Roles, and build the Lambda API to generate Presigned URLs for the Frontend. Candidates upload CVs directly to S3 without passing through the Backend.
- **Phase 2 — Event-Driven Pipeline & Textract (Week 2):** Configure S3 Event Notification → SQS trigger → Lambda Worker integrated with Amazon Textract to extract raw text from PDF/DOCX files.
- **Phase 3 — Bedrock Integration & DynamoDB (Week 3):** Design Prompt Engineering, call Amazon Bedrock (Claude 3.5 Sonnet) to parse raw text into JSON (`skills`, `yearsOfExperience`, `education`), and store results in DynamoDB.
- **Phase 4 — Step Functions & Finalization (Week 4):** Build the Step Functions Express Workflow with a Parallel State for concurrent interview scoring, generate PDF/HTML reports saved back to S3. Conduct security testing, optimization, and documentation.

**Technical Requirements**
<!-- - Weather Edge Station: Sensors (temperature, humidity, rainfall, wind speed), a microcontroller (ESP32), and a Raspberry Pi as the edge device. Raspberry Pi runs Raspbian, handles Docker for filtering, and sends 1 MB/day per station via MQTT over Wi-Fi.
- Weather Platform: Practical knowledge of AWS Amplify (hosting Next.js), Lambda (minimal use due to Next.js), AWS Glue (ETL), S3 (two buckets), IoT Core (gateway and rules), and Cognito (5 users). Use AWS CDK/SDK to code interactions (e.g., IoT Core rules to S3). Next.js reduces Lambda workload for the fullstack web app. -->
- **CV Parsing Pipeline:** S3 bucket (prefix `cvs/`), SQS Standard Queue with DLQ, Lambda runtime Python 3.12, Amazon Textract AnalyzeDocument API, Amazon Bedrock InvokeModel API (claude-3-5-sonnet), DynamoDB table with Single Table Design.
- **AI Evaluation Workflow:** Step Functions Express Workflow, Parallel State with up to 10 concurrent branches, Lambda integrated with Bedrock to generate overall summaries, S3 for report output storage.
- **API & Auth:** API Gateway REST API, Cognito User Pool + App Client, JWT Authorizer on all protected endpoints.

### 5. Timeline & Milestones
**Project Timeline**
<!-- - Pre-Internship (Month 0): 1 month for planning and old station review.
- Internship (Months 1-3): 3 months.
    - Month 1: Study AWS and upgrade hardware.
    - Month 2: Design and adjust architecture.
    - Month 3: Implement, test, and launch.
- Post-Launch: Up to 1 year for research. -->
- **Week 1:** Set up S3 bucket, build the Presigned URL API; configure IAM Roles for Lambda; test the direct upload flow from the Frontend.
- **Week 2:** Build the Event-Driven Pipeline: S3 Event Notification → SQS → Lambda; integrate and test Amazon Textract across various PDF and DOCX formats.
- **Week 3:** Design Prompt Engineering for Amazon Bedrock; standardize JSON output; store metadata in DynamoDB; evaluate extraction accuracy.
- **Week 4:** Build the Step Functions State Machine with Parallel scoring; generate PDF/HTML reports; conduct security testing (IAM, Cognito, S3 Bucket Policy) and finalize documentation.

### 6. Budget Estimation
<!-- You can find the budget estimation on the [AWS Pricing Calculator](https://calculator.aws/#/estimate?id=621f38b12a1ef026842ba2ddfe46ff936ed4ab01).  
Or you can download the [Budget Estimation File](../attachments/budget_estimation.pdf). -->
You can explore the cost estimate on the [AWS Pricing Calculator](https://calculator.aws/).

### Infrastructure Costs
<!-- - AWS Services:
    - AWS Lambda: $0.00/month (1,000 requests, 512 MB storage).
    - S3 Standard: $0.15/month (6 GB, 2,100 requests, 1 GB scanned).
    - Data Transfer: $0.02/month (1 GB inbound, 1 GB outbound).
    - AWS Amplify: $0.35/month (256 MB, 500 ms requests).
    - Amazon API Gateway: $0.01/month (2,000 requests).
    - AWS Glue ETL Jobs: $0.02/month (2 DPUs).
    - AWS Glue Crawlers: $0.07/month (1 crawler).
    - MQTT (IoT Core): $0.08/month (5 devices, 45,000 messages).

Total: $0.7/month, $8.40/12 months

- Hardware: $265 one-time (Raspberry Pi 5 and sensors). -->
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
<!-- - Network Outages: Medium impact, medium probability.
- Sensor Failures: High impact, low probability.
- Cost Overruns: Medium impact, low probability. -->
- **Inaccurate Prompt Engineering:** High impact, medium probability — CV extraction produces incorrect field values.
- **Lambda Timeout:** Medium impact, low probability — CV file too large, or Textract/Bedrock responds slowly.
- **Bedrock Cost Overrun:** Medium impact, low probability — if CV upload volume spikes unexpectedly.
- **DLQ Message Accumulation:** Low impact, medium probability — Lambda Worker repeatedly fails to process messages.

#### Mitigation Strategies
<!-- - Network: Local storage on Raspberry Pi with Docker.
- Sensors: Regular checks and spares.
- Cost: AWS budget alerts and optimization. -->
- **Prompt Engineering:** Build a test suite using diverse CVs (Vietnamese, English, multiple formats); evaluate accuracy per iteration before deploying.
- **Lambda Timeout:** Increase timeout as needed (up to 15 minutes); split the Textract and Bedrock steps into two separate Lambda functions if required.
- **Cost:** Set up an AWS Budget Alert to notify when spending exceeds $5/month; use Bedrock On-Demand pricing instead of Provisioned.
- **DLQ:** Configure a CloudWatch Alarm to monitor `ApproximateNumberOfMessagesNotVisible`; create a runbook for periodic DLQ reprocessing.

#### Contingency Plans
<!-- - Revert to manual methods if AWS fails.
- Use CloudFormation for cost-related rollbacks. -->
- If Bedrock is unavailable: fall back to Regex-based parsing to keep the pipeline operational.
- If Step Functions fails: manually re-trigger executions via the AWS Console or CLI.

### 8. Expected Outcomes
#### Technical Improvements: 
<!-- Real-time data and analytics replace manual processes.  
Scalable to 10-15 stations. -->
Fast, seamless CV upload and processing with no interruptions, thanks to the fully asynchronous architecture. The system can scale infinitely to handle large recruitment campaigns without requiring any server upgrades.
#### Long-term Value
<!-- 1-year data foundation for AI research.  
Reusable for future projects. -->
A reduction of **80%** in manual CV reading and classification time. Post-interview reports generated automatically in **under 90 seconds** through parallel processing. Consistent, objective evaluation criteria reusable across multiple job positions. The Serverless Pay-as-you-go model optimizes operational costs by charging only for actual usage.