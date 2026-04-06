---
title : "Detailed Data Flows"
date : 2024-01-01 
weight : 2 
chapter : false
pre : " <b> 5.2. </b> "
---

### A. CvJdProcessor Lambda — CV Path

**Trigger:** Step Functions (from IngestionTrigger via S3 → SQS)

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

**Processing Steps:**

| Step | Operation      | Service               | Description                                             |
| ---- | -------------- | --------------------- | ------------------------------------------------------- |
| 1    | Textract       | AWS Textract          | Extract raw text from S3 PDF (async)                    |
| 2    | PII Masking    | spaCy NER             | Replace PERSON, ORG, GPE, LOC, NORP with neutral tokens |
| 3    | Claude Agent 1 | Bedrock (Claude)      | Extract skills, seniority, experience, strengths/gaps   |
| 4    | Claude Agent 2 | Bedrock (Claude)      | Generate 3 targeted interview questions                 |
| 5    | Bi-Encoder     | Bedrock (Cohere)      | Embed masked CV text as `search_document` (1024-dim)    |
| 6    | Cross-Encoder  | sentence-transformers | Score CV chunks against JD text (if JD provided)        |
| 7    | Hybrid Score   | Local computation     | Blend BI (35%) + CE (65%) scores                        |

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

### B. CvJdProcessor Lambda — JD Path

**Trigger:** Step Functions (from .NET backend `StartExecution`)

**Input:**

```json
{
  "profile_id": "recruiter",
  "job_id": "job-uuid"
}
```

**Processing Steps:**

| Step | Operation  | Service          | Description                                             |
| ---- | ---------- | ---------------- | ------------------------------------------------------- |
| 1    | Read JD    | RDS (PostgreSQL) | `SELECT Title, Description FROM Jobs WHERE Id = job_id` |
| 2    | Bi-Encoder | Bedrock (Cohere) | Embed JD text as `search_document` (1024-dim)           |

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

### C. JobSuggestionEngine (Candidate Flow)

**Input:** Output from CvJdProcessor CV path (see above)

**Output (DynamoDB write + AppSync publish):**

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

### D. CandidateRankingEngine (Recruiter Flow)

**Input:** Output from CvJdProcessor JD path (see above)

**Output (DynamoDB write + AppSync publish):**

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