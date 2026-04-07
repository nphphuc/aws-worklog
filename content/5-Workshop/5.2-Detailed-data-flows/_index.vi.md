---
title : "Luồng dữ liệu chi tiết"
date : 2024-01-01 
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

### A. Lambda CvJdProcessor — Luồng CV

**Kích hoạt:** Step Functions (từ IngestionTrigger qua S3 → SQS)

**Input:**

```json
{
  "profile_id": "candidate-uuid",
  "job_id": "job-uuid | null",
  "file_key": "candidates/{candidateId}/cv.pdf",
  "bucket": "smarthire-raw-assets",
  "jd_text": "Job description text (optional, from DynamoDB cache)",
  "job_title": "Senior Backend Engineer (optional)",
  "required_skills": ["AWS", "Python"],
  "jd_vector": [0.12, 0.45, "...1024 floats (optional, cached)"]
}
```

**Các bước xử lý:**

| Bước | Thao tác      | Dịch vụ               | Mô tả                                             |
| ---- | -------------- | --------------------- | ------------------------------------------------------- |
| 1    | Textract                  | AWS Textract          | Trích xuất văn bản thô từ PDF trong S3 (bất đồng bộ)                    |
| 2    | Ẩn PII                    | spaCy NER             | Thay thế PERSON, ORG, GPE, LOC, NORP bằng token trung tính |
| 3    | Claude Agent 1            | Bedrock (Claude)      | Trích xuất kỹ năng, cấp độ, kinh nghiệm, điểm mạnh/yếu   |
| 4    | Claude Agent 2            | Bedrock (Claude)      | Tạo 3 câu hỏi phỏng vấn mục tiêu                 |
| 5    | Bi-Encoder                | Bedrock (Cohere)      | Nhúng CV đã ẩn danh thành vector `search_document` (1024 chiều)    |
| 6    | Cross-Encoder             | sentence-transformers | Chấm điểm các đoạn CV so với JD (nếu có JD)        |
| 7    | Điểm lai (Hybrid Score)   | Local computation     | Kết hợp điểm BI (35%) + CE (65%)                     |

**Output:**

```json
{
  "profile_id": "candidate-uuid",
  "file_key": "candidates/{candidateId}/cv.pdf",
  "bucket": "smarthire-raw-assets",
  "job_id": "job-uuid | null",
  "job_title": "Senior Backend Engineer",
  "masked_cv_text": "Masked CV text (PII removed, max 50K chars)",
  "cv_vector": [0.12, 0.45, "...1024 floats"],
  "parsed_data": {
    "seniority_estimate": "Senior",
    "frontend_skills": ["React", "TypeScript"],
    "backend_skills": ["Python", "AWS Lambda"],
    "devops_skills": ["Docker", "Terraform"],
    "soft_skills": ["Leadership"],
    "years_experience": 5,
    "matching_score": 82.5,
    "strengths": "Strong backend expertise...",
    "gaps": "Limited frontend experience..."
  },
  "scoring_details": {
    "bi_encoder_score": 78.5,
    "cross_encoder_score": 85.2,
    "hybrid_score": 82.5,
    "bi_encoder_weight": 0.35,
    "cross_encoder_weight": 0.65,
    "method": "hybrid_bi_cross_encoder"
  },
  "interview_guide": [
    {
      "question": "Explain event-driven architecture...",
      "skill_targeted": "System Design",
      "rationale": "Tests understanding of async patterns"
    }
  ],
  "masking_report": {
    "blind_screening": "applied",
    "total_entities_masked": 12,
    "by_label": { "PERSON": 3, "ORG": 5, "GPE": 4 }
  },
  "jd_text": "Job description text",
  "jd_vector": [0.12, 0.45, "...1024 floats | null"]
}
```

### B. Lambda CvJdProcessor — Luồng JD

**Kích hoạt:** Step Functions (từ backend .NET gọi `StartExecution`)

**Input:**

```json
{
  "profile_id": "recruiter",
  "job_id": "job-uuid"
}
```

**Các bước xử lý:**

| Bước | Thao tác  | Dịch vụ          | Mô tả                                             |
| ---- | ---------- | ---------------- | ------------------------------------------------------- |
| 1    | Đọc JD    | RDS (PostgreSQL) | `SELECT Title, Description FROM Jobs WHERE Id = job_id` |
| 2    | Bi-Encoder | Bedrock (Cohere) | Nhúng JD thành vector `search_document` (1024 chiều)          |

**Output:**

```json
{
  "profile_id": "recruiter",
  "job_id": "job-uuid",
  "job_title": "Senior Backend Engineer",
  "jd_text": "Raw JD description from RDS Jobs table",
  "jd_vector": [0.12, 0.45, "...1024 floats"],
  "masked_cv_text": "Same as jd_text (backward compat for CandidateRankingEngine)"
}
```

### C. JobSuggestionEngine (Luồng ứng viên)

**Input:** Output từ CvJdProcessor (luồng CV)

**Output (ghi vào DynamoDB + publish AppSync):**

```json
{
  "PK": "CANDIDATE#{profile_id}",
  "SK": "JOB_SUGGESTIONS",
  "type": "JOB_SUGGESTIONS",
  "candidateId": "candidate-uuid",
  "suggestions": [
    {
      "jobId": "job-uuid",
      "jobTitle": "Senior Backend Engineer",
      "biScore": 78.5,
      "crossScore": 82.1,
      "finalScore": 80.8,
      "matchExplanation": "Strong fit due to..."
    }
  ],
  "suggestionCount": 5,
  "updatedAt": "2026-03-30T10:30:00Z",
  "expiresAt": 1746000000
}
```

**AppSync publish payload:**

```json
{
  "candidateId": "candidate-uuid",
  "suggestions": "[ ...AWSJSON... ]",
  "updatedAt": "2026-03-30T10:30:00Z"
}
```

### D. CandidateRankingEngine (Luồng nhà tuyển dụng)

**Input:** Output từ CvJdProcessor (luồng JD)

**Output (ghi vào DynamoDB + publish AppSync):**

```json
{
  "PK": "JOB#{job_id}",
  "SK": "CANDIDATE_RANKING",
  "type": "CANDIDATE_RANKING",
  "jobId": "job-uuid",
  "rankedCandidates": [
    {
      "candidateId": "candidate-uuid",
      "rank": 1,
      "seniority": "Senior",
      "yearsExperience": 5,
      "biEncoderScore": 85.2,
      "crossEncoderScore": 88.7,
      "finalScore": 87.3,
      "candidateSnapshot": "The candidate brings 5 years of backend experience...",
      "topSkills": ["AWS", "Python", "Lambda"],
      "interviewGuide": [],
      "applicationStatus": "POOL"
    }
  ],
  "candidateCount": 15,
  "updatedAt": "2026-03-30T10:30:00Z"
}
```

**AppSync publish payload:**

```json
{
  "jobId": "job-uuid",
  "rankedCandidates": "[ ...AWSJSON... ]",
  "updatedAt": "2026-03-30T10:30:00Z"
}
```
