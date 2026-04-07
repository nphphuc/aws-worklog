---
title : "Kiến trúc thành phần tối ưu"
date : 2024-01-01 
weight : 4 
chapter : false
pre : " <b> 5.4. </b> "
---

### A. CvJdProcessor (Lambda xử lý hợp nhất)

Xử lý cả CV và JD trong cùng một Lambda với logic phân nhánh:

| Khía cạnh              | Luồng CV (Ứng viên)                              | Luồng JD (Nhà tuyển dụng)                           |
| ------------------- | ------------------------------------------------ | --------------------------------------------- |
| **Kích hoạt**         | Step Functions (qua S3 → SQS → IngestionTrigger) | Step Functions (gọi trực tiếp từ backend .NET) |
| **Nguồn văn bản**     | Textract (PDF từ S3)                                | RDS `Jobs.Description`                        |
| **Ẩn PII**     | Có (spaCy NER)                                  | Không                                            |
| **Phân tích Claude** | Có (Agent 1 + Agent 2)                          | Không                                            |
| **Embedding**       | Có (Cohere search_document)                     | Có (Cohere search_document)                  |
| **Cross-Encoder**   | Có (nếu có JD)                             | Không                                            |
| **Timeout**         | 300 giây                                             | 300 giây                                          |
| **Bộ nhớ**          | 1024 MB                                          | 1024 MB                                       |
| **VPC**             | Có (truy cập RDS để đọc JD)                     | Có (truy cập RDS để đọc JD)                  |

### B. Trách nhiệm của Engine (Tự điều khiển)

**JobSuggestionEngine (Luồng ứng viên)**

- Nhận: `profile_id`, `file_key`, `cv_vector`, `masked_cv_text`, `parsed_data`, `scoring_details`, `interview_guide`, `masking_report`
- Lưu: CV embedding → RDS (`candidate_embeddings`)
- Lưu: hồ sơ ứng viên + embedding → RDS (`candidate_profiles_ai`, `candidate_embeddings`)
- Thực hiện: tìm kiếm ANN pgvector cho top 20 công việc
- Xếp hạng lại: Cross-Encoder (top 5 công việc)
- Chấm điểm: Hybrid (35% BI-Encoder, 65% Cross-Encoder)
- Tạo: giải thích mức độ phù hợp cho từng công việc (Claude)
- Lưu kết quả: Top 5 gợi ý → DynamoDB (`SK=JOB_SUGGESTIONS`)

**CandidateRankingEngine (Luồng nhà tuyển dụng)**

- Nhận: `job_id`, `jd_vector`, `jd_text`, `job_title`, `masked_cv_text`
- Lưu: JD embedding → RDS (`job_embeddings`)
- Tái sử dụng: JD làm `search_query` cho embedding (tìm kiếm bất đối xứng)
- Tạo lại embedding: JD dưới dạng `search_query`
- Thực hiện: tìm kiếm ANN pgvector cho top 50 ứng viên
- Bổ sung dữ liệu: lấy hồ sơ ứng viên + interview guides từ DynamoDB
- Xếp hạng lại: Cross-Encoder (top 15 ứng viên)
- Chấm điểm: Hybrid (30% BI-Encoder, 70% Cross-Encoder)
- Tạo: tóm tắt ứng viên theo thứ hạng (Claude)
- Lưu kết quả: Top 15 ứng viên → DynamoDB (`SK=CANDIDATE_RANKING`)

### C. Điều phối: Step Functions (Pipeline hợp nhất)

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

**Các điểm cốt lõi:**

- Chỉ còn 3 state thay vì 4 (đã gộp TextProcessing + VectorOps vào CvJdProcessor)
- State Choice dùng để phân biệt luồng CV (ứng viên) và JD (nhà tuyển dụng)
- Các engine ghi dữ liệu trực tiếp vào RDS và DynamoDB (không cần Lambda riêng để lưu trữ)
- CvJdProcessor đọc JD trực tiếp từ RDS trong luồng nhà tuyển dụng (không dùng S3/Textract)