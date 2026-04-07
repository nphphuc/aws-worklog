---
title : "DynamoDB Schema"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

**Bảng: `ApplicationTracking`**

**Gợi ý công việc** (phía ứng viên)

```json
{
  "PK": "CANDIDATE#{profile_id}",
  "SK": "JOB_SUGGESTIONS",
  "GSI1PK": "JOB_SUGGESTIONS",
  "GSI1SK": "2026-03-30T10:30:00Z",
  "type": "JOB_SUGGESTIONS",
  "candidateId": "...",
  "suggestions": [
    {
      "jobId": "...",
      "jobTitle": "...",
      "biScore": 78.5,
      "crossScore": 82.1,
      "finalScore": 80.8,
      "matchExplanation": "..."
    }
  ],
  "suggestionCount": 5,
  "updatedAt": "2026-03-30T10:30:00Z",
  "expiresAt": 1746000000
}
```

**Xếp hạng ứng viên** (phía nhà tuyển dụng)

```json
{
  "PK": "JOB#{job_id}",
  "SK": "CANDIDATE_RANKING",
  "GSI1PK": "CANDIDATE_RANKING",
  "GSI1SK": "2026-03-30T10:30:00Z",
  "type": "CANDIDATE_RANKING",
  "jobId": "...",
  "rankedCandidates": [
    {
      "candidateId": "...",
      "rank": 1,
      "seniority": "Senior",
      "yearsExperience": 5,
      "biEncoderScore": 85.2,
      "crossEncoderScore": 88.7,
      "finalScore": 87.3,
      "candidateSnapshot": "...",
      "topSkills": ["AWS", "Python"],
      "interviewGuide": []
    }
  ],
  "candidateCount": 15,
  "updatedAt": "2026-03-30T10:30:00Z"
}
```

**Ghi chú**

- Nội dung JD (mô tả công việc) dùng cho bước cross-encoding được đọc trực tiếp từ bảng `Jobs.Description` trong RDS.
- Các dữ liệu kết quả cuối cùng cho nhà tuyển dụng và ứng viên vẫn được lưu trong DynamoDB (`CANDIDATE_RANKING`, `JOB_SUGGESTIONS`).