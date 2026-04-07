---
title : "Các thành phần hạ tầng"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.7. </b> "
---

### A. Các hàm Lambda

| Lambda                     | Nguồn                                 | Timeout | Bộ nhớ  | VPC | Mục đích                                              |
| -------------------------- | -------------------------------------- | ------- | ------- | --- | ---------------------------------------------------- |
| `ingestion-trigger`        | `iac/lambda/ingestion_trigger/`        | 30s     | 256 MB  | Không  | Xác thực file CV upload lên S3, khởi động Step Functions        |
| `cv-jd-processor`          | `iac/lambda/cv_jd_processor/`          | 300s    | 1024 MB | Có | Xử lý thống nhất CV/JD (Textract/PII/Claude/Embedding) |
| `job-suggestion-engine`    | `iac/lambda/job_suggestion_engine/`    | 90s     | 3008 MB | Có | Tìm các công việc phù hợp nhất cho ứng viên               |
| `candidate-ranking-engine` | `iac/lambda/candidate_ranking_engine/` | 90s     | 3008 MB | Có | Xếp hạng các ứng viên phù hợp nhất cho một công việc               |

### B. Các file Terraform

| Mục đích                                                              | Path                                                                    |
| -------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| Điều phối chính (S3 notification, module xử lý)              | `iac/terraform/main.tf`                                                 |
| Pipeline Lambdas + Step Functions + DynamoDB + AppSync API/Resolvers | `iac/terraform/modules/processing/main.tf`                              |
| Schema GraphQL của AppSync                                               | `iac/terraform/modules/processing/appsync_schema.graphql`               |
| ASL template                                                         | `iac/terraform/modules/processing/state_machine_cv_processing.asl.json` |
| Outputs của processing                                                   | `iac/terraform/modules/processing/outputs.tf`                           |
| Các hàng đợi SQS                                                           | `iac/terraform/modules/queue/main.tf`                                   |

### C. Các tài nguyên đã loại bỏ

- Lambda `text_processor` (đã gộp vào `cv_jd_processor`)
- Lambda `vector_ops` (đã gộp vào `cv_jd_processor`)
- S3 bucket notification cho prefix JD (`jobs/*.pdf`)