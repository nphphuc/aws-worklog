---
title : "Optimized Component Architecture"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

### A. CvJdProcessor (Merged Processing Lambda)

Handles both CV and JD processing in a single Lambda with branching logic:

| Aspect              | CV Path (Candidate)                              | JD Path (Recruiter)                           |
| ------------------- | ------------------------------------------------ | --------------------------------------------- |
| **Trigger**         | Step Functions (via S3 → SQS → IngestionTrigger) | Step Functions (via .NET backend direct call) |
| **Text Source**     | Textract (S3 PDF)                                | RDS `Jobs.Description`                        |
| **PII Masking**     | Yes (spaCy NER)                                  | No                                            |
| **Claude Analysis** | Yes (Agent 1 + Agent 2)                          | No                                            |
| **Embedding**       | Yes (Cohere search_document)                     | Yes (Cohere search_document)                  |
| **Cross-Encoder**   | Yes (if JD provided)                             | No                                            |
| **Timeout**         | 300s                                             | 300s                                          |
| **Memory**          | 1024 MB                                          | 1024 MB                                       |
| **VPC**             | Yes (RDS access for JD read)                     | Yes (RDS access for JD read)                  |

### B. Engine Responsibilities (Autonomous)

**JobSuggestionEngine (Candidate Flow)**

- Receives: `profile_id`, `file_key`, `cv_vector`, `masked_cv_text`, `parsed_data`, `scoring_details`, `interview_guide`, `masking_report`
- Stores: CV embedding → RDS (`candidate_embeddings`)
- Stores: Candidate profile + embedding → RDS (`candidate_profiles_ai`, `candidate_embeddings`)
- Performs: pgvector ANN search for top 20 jobs
- Re-ranks: Cross-Encoder (top 5 jobs)
- Scores: Hybrid (35% BI-Encoder, 65% Cross-Encoder)
- Generates: Match explanation per job (Claude)
- Persists: Top 5 suggestions → DynamoDB (`SK=JOB_SUGGESTIONS`)

**CandidateRankingEngine (Recruiter Flow)**

- Receives: `job_id`, `jd_vector`, `jd_text`, `job_title`, `masked_cv_text`
- Stores: JD embedding → RDS (`job_embeddings`)
- Reuses: JD text as `search_query` embedding input for asymmetric retrieval
- Re-embeds: JD as `search_query` (asymmetric search)
- Performs: pgvector ANN search for top 50 candidates
- Enriches: Fetch candidate profiles + interview guides from DynamoDB
- Re-ranks: Cross-Encoder (top 15 candidates)
- Scores: Hybrid (30% BI-Encoder, 70% Cross-Encoder)
- Generates: Candidate snapshot per rank (Claude)
- Persists: Top 15 ranked candidates → DynamoDB (`SK=CANDIDATE_RANKING`)

### C. Orchestration: Step Functions (Unified Pipeline)

**State Machine: `cv_processing`**

```json
{
  "Comment": "SmartHire CV/JD processing orchestration: unified processor -> route to matching engine",
  "StartAt": "CvJdProcessor",
  "States": {
    "CvJdProcessor": {
      "Type": "Task",
      "Resource": "arn:aws:states:::lambda:invoke",
      "Parameters": {
        "FunctionName": "${cv_jd_processor_arn}",
        "Payload.$": "$"
      },
      "OutputPath": "$.Payload",
      "Retry": [
        {
          "ErrorEquals": [
            "Lambda.ServiceException",
            "Lambda.AWSLambdaException",
            "Lambda.SdkClientException",
            "States.TaskFailed"
          ],
          "IntervalSeconds": 2,
          "MaxAttempts": 3,
          "BackoffRate": 2
        }
      ],
      "Next": "RoutingChoice"
    },
    "RoutingChoice": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.profile_id",
          "StringEquals": "recruiter",
          "Next": "CandidateRanking"
        }
      ],
      "Default": "JobSuggestion"
    },
    "JobSuggestion": {
      "Type": "Task",
      "Resource": "arn:aws:states:::lambda:invoke",
      "Parameters": {
        "FunctionName": "${job_suggestion_engine_arn}",
        "Payload.$": "$"
      },
      "OutputPath": "$.Payload",
      "Retry": [
        {
          "ErrorEquals": [
            "Lambda.ServiceException",
            "Lambda.AWSLambdaException",
            "Lambda.SdkClientException",
            "States.TaskFailed"
          ],
          "IntervalSeconds": 3,
          "MaxAttempts": 3,
          "BackoffRate": 2
        }
      ],
      "End": true
    },
    "CandidateRanking": {
      "Type": "Task",
      "Resource": "arn:aws:states:::lambda:invoke",
      "Parameters": {
        "FunctionName": "${candidate_ranking_engine_arn}",
        "Payload.$": "$"
      },
      "OutputPath": "$.Payload",
      "Retry": [
        {
          "ErrorEquals": [
            "Lambda.ServiceException",
            "Lambda.AWSLambdaException",
            "Lambda.SdkClientException",
            "States.TaskFailed"
          ],
          "IntervalSeconds": 3,
          "MaxAttempts": 3,
          "BackoffRate": 2
        }
      ],
      "End": true
    }
  }
}
```

**Key Points:**

- 3 states instead of 4 (merged TextProcessing + VectorOps into CvJdProcessor)
- Choice state differentiates CV (candidate) vs JD (recruiter) flow
- Engines write directly to RDS and DynamoDB (no separate persistence Lambda)
- CvJdProcessor reads JD text from RDS for recruiter flow (no S3/Textract)