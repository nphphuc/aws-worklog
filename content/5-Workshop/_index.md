---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# SmartHire Matching System Architecture (v3.0 — Unified Pipeline)

#### Overview

**SmartHire-AI** is an intelligent hiring platform built on AWS that revolutionizes recruitment. Candidates upload CVs and receive personalized job suggestions, while recruiters manage postings and see ranked candidates in real-time. The platform leverages AI and serverless AWS services for scalability and performance.

#### Key Features

- **For Candidates**: Sign in, upload CV (PDF), and get job suggestions with real-time updates
- **For Recruiters**: Create and manage job postings, view ranked candidates as they apply
- **Real-time Updates**: AWS AppSync GraphQL subscriptions push results instantly
- **AI-Powered Matching**: Bedrock, Comprehend, and Textract power intelligent matching

### Key Changes

- **No JD PDF Upload**: Job Description text is read directly from RDS (Jobs.Description) instead of uploading PDF to S3.
- **Merged Processing Lambda**: text_processor and vector_ops merged into a single cv_jd_processor Lambda.
- **Direct JD Trigger**: .NET backend calls Step Functions StartExecution directly for JD processing (bypasses S3/SQS/IngestionTrigger).
- **Simplified State Machine**: Step Functions reduced from 4 states to 3 (CvJdProcessor → RoutingChoice → Engine).
- **Reduced Latency**: One fewer Lambda cold start per execution (2 Lambdas merged into 1).
- **Real-Time Contract via AppSync**: Matching engines publish updates through AppSync GraphQL mutations, and frontend receives filtered subscriptions.

### Quick Explanation (Easy View)
If you remember only 4 things, remember these:

1. Candidate upload CV -> system parses CV -> computes top matching jobs -> pushes realtime to candidate UI.
2. Recruiter creates job -> system embeds JD -> ranks top candidates -> pushes realtime to recruiter UI.
3. Step Functions always has 3 steps: `CvJdProcessor -> RoutingChoice -> Engine`.
4. AppSync is only the realtime delivery layer (publish from Lambda, subscribe from frontend), not the compute layer.

### Real-Time Contract (Current)
- Mutation: `publishJobSuggestions(candidateId, suggestions, updatedAt)`
- Mutation: `publishCandidateRanking(jobId, rankedCandidates, updatedAt)`
- Subscription: `onJobSuggestions(candidateId)`
- Subscription: `onCandidateRanking(jobId)`

Important: schema field is `updatedAt` (not `timestamp`).

#### Content

1. [System Workflows](5.1-System-workflows/)
2. [Detailed Data Flows](5.2-Detailed-data-flows/)
3. [Data Models & Entity Relationships](5.3-Data-models-entity-relationships/)
4. [Optimized Component Architecture](5.4-Optimized-component-architecture/)
5. [RDS Postgres Schema](5.5-RDS-postgres-schema/)
6. [DynamoDB Schema](5.6-DynamoDB-schema/)
7. [Infrastructure Components](5.7-Infrastructure-components/)
8. [Observability & Monitoring](5.8-Observability-monitoring/)