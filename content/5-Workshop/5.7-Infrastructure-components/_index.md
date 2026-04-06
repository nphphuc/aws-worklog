---
title : "Infrastructure Components"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.7. </b> "
---

### A. Lambda Functions

| Lambda                     | Source                                 | Timeout | Memory  | VPC | Purpose                                              |
| -------------------------- | -------------------------------------- | ------- | ------- | --- | ---------------------------------------------------- |
| `ingestion-trigger`        | `iac/lambda/ingestion_trigger/`        | 30s     | 256 MB  | No  | Validate S3 CV uploads, start Step Functions         |
| `cv-jd-processor`          | `iac/lambda/cv_jd_processor/`          | 300s    | 1024 MB | Yes | Unified CV/JD processing (Textract/PII/Claude/Embed) |
| `job-suggestion-engine`    | `iac/lambda/job_suggestion_engine/`    | 90s     | 3008 MB | Yes | Find top matching jobs for a candidate               |
| `candidate-ranking-engine` | `iac/lambda/candidate_ranking_engine/` | 90s     | 3008 MB | Yes | Rank top matching candidates for a job               |

### B. Terraform Files

| Purpose                                                              | Path                                                                    |
| -------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| Root orchestration (S3 notification, processing module)              | `iac/terraform/main.tf`                                                 |
| Pipeline Lambdas + Step Functions + DynamoDB + AppSync API/Resolvers | `iac/terraform/modules/processing/main.tf`                              |
| AppSync GraphQL schema                                               | `iac/terraform/modules/processing/appsync_schema.graphql`               |
| ASL template                                                         | `iac/terraform/modules/processing/state_machine_cv_processing.asl.json` |
| Processing outputs                                                   | `iac/terraform/modules/processing/outputs.tf`                           |
| SQS queues                                                           | `iac/terraform/modules/queue/main.tf`                                   |

### C. Removed Resources

- `text_processor` Lambda (merged into `cv_jd_processor`)
- `vector_ops` Lambda (merged into `cv_jd_processor`)
- S3 bucket notification for JD prefix (`jobs/*.pdf`)