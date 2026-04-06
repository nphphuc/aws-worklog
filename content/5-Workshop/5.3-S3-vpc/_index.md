---
title : "Candidate Path"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

#### Candidate Journey: CV Upload & Processing

This section details the complete end-to-end flow when a candidate uploads their resume (CV) to SmartHire-AI, from initial upload through intelligent job matching.

---

#### Step 1: CV Upload to S3

When a candidate clicks "Upload CV" in the React SPA:

```
1. Candidate selects PDF file
2. React app sends signed S3 PUT request via Cognito JWT
3. File uploads directly to S3 bucket under "candidates/" prefix
4. S3 generates object created event
```

**S3 Configuration:**
- Enable S3 event notifications (SNS/SQS)
- Enable versioning for audit trail
- Enable encryption at rest (SSE-S3 or KMS)

---

#### Step 2: Queue & Ingestion (SQS → Lambda)

```
S3 Event Created
        ↓
S3 → SNS → SQS (CV Queue)
        ↓
Lambda Trigger "ingestion_trigger"
```

**Lambda Ingestion Function:**
- Consumes SQS message containing S3 object metadata
- Validates file format (PDF only)
- Extracts S3 bucket, key, candidate ID from metadata
- Initiates **AWS Step Functions** execution
- Returns SQS acknowledgment

**Error Handling:**
- Invalid file format → DLQ (Dead Letter Queue)
- Processing errors → SQS retry (3 attempts default)
- Failed retries → DLQ for manual review

---

#### Step 3: Pipeline Orchestration (AWS Step Functions)

The Step Functions state machine manages the entire processing workflow:

```
START
  ↓
[Step 1] Invoke cv_jd_processor Lambda
          - Extract text from PDF
          - Parse CV data
          - Generate embeddings
          - Branch to job matching
  ↓
[Step 2] Invoke job_suggestion_engine Lambda
          - Compare CV embeddings vs Job embeddings
          - Score & rank all open jobs
          - Select top N matching jobs
  ↓
[Step 3] Write Results
          - Store tracking in DynamoDB
          - Update RDS with match results
  ↓
[Step 4] Publish Results via AppSync
          - Send GraphQL mutation to AppSync
          - Real-time notification to candidate UI
  ↓
END (Success) or ERROR
```

**State Machine Benefits:**
- Automatic retries with exponential backoff
- Timeout protection (prevents hanging processes)
- Error handling and fallback states
- Visual state monitoring in AWS Console

---

#### Step 4: CV & Document Processing (Lambda + AI)

**Lambda: cv_jd_processor (Python 3.12)**

```python
Input: {
  "bucket": "candidates-bucket",
  "key": "candidates/john-doe.pdf",
  "candidateId": "can-12345"
}

Process:
1. Download PDF from S3
2. Amazon Textract → Extract text, tables, entities
3. Parse structure:
   - Name, email, phone
   - Work experience (company, duration, skills)
   - Education (degree, university)
   - Skills list
4. Amazon Bedrock (Claude) → Enrich data:
   - Normalize job titles
   - Extract hard skills vs soft skills
   - Categorize experience level
   - Generate career summary
5. Amazon Comprehend → NLP analysis:
   - Detect key phrases
   - Extract named entities
   - Sentiment of previous roles
6. Generate embeddings using Bedrock:
   - Convert CV summary to vector
   - Store in DynamoDB for similarity search
   
Output: {
  "parsed_cv": {...parsed data...},
  "embeddings": [...vector...],
  "skills": ["Python", "AWS", "React", ...],
  "experience_level": "SENIOR",
  "summary": "..."
}
```

---

#### Step 5: Job Suggestion Engine

**Lambda: job_suggestion_engine**

```python
Input: CV embeddings + candidate ID

Process:
1. Query RDS for all active jobs
2. For each job:
   - Retrieve job embeddings from cache/RDS
   - Calculate cosine similarity with CV embeddings
   - Score based on:
     * Skill match (exact vs. similar)
     * Experience level alignment
     * Location preferences
     * Salary compatibility
3. Sort by match score (descending)
4. Filter by threshold (>0.6 minimum match)
5. Select top 10 most relevant jobs
6. Store candidate_job_match records in RDS + DynamoDB

Output: [
  {"jobId": "job-1", "score": 0.92, "matchDetails": {...}},
  {"jobId": "job-5", "score": 0.87, "matchDetails": {...}},
  ...
]
```

---

#### Step 6: Real-time UI Update (AWS AppSync)

Once matching is complete:

```
Lambda publishes AppSync GraphQL mutation:
"JobSuggestionsReady"
{
  "candidateId": "can-12345",
  "suggestedJobs": [...],
  "processingStatus": "COMPLETED",
  "timestamp": "2024-01-15T10:30:00Z"
}
        ↓
AppSync broadcasts via subscription
        ↓
Candidate React SPA receives update
        ↓
Dashboard re-renders with suggested jobs (NO page refresh needed)
```

---

#### Data Storage

**RDS (PostgreSQL):**
```sql
candidates: {id, email, name, resume_text, skills, experience_level}
jobs: {id, title, description, required_skills, embeddings}
candidate_job_matches: {id, candidate_id, job_id, match_score, created_at}
```

**DynamoDB:**
```
candidate_matches: {
  PK: "CANDIDATE#can-12345",
  SK: "MATCH#job-1",
  match_score: 0.92,
  processed_at: timestamp
}
```

**S3:**
- `s3://candidates-bucket/candidates/` - Raw PDF uploads
- `s3://candidates-bucket/processed/` - Extracted text & structured data (optional)

---

#### Performance & Monitoring

**CloudWatch Metrics:**
- Lambda execution duration
- SQS queue depth
- Step Functions execution time
- Textract processing cost per document

**Alarms:**
- SQS queue > 100 messages (backlog)
- Lambda error rate > 1%
- Processing time > 5 minutes (timeout risk)

---

#### Error Scenarios

| Scenario | Action |
|----------|--------|
| Invalid PDF format | Return user error: "Please upload a valid PDF" |
| Processing timeout (>5 min) | Retry via Step Functions, max 3 attempts |
| Textract extraction fails | Fallback to simple text parsing |
| No job matches found | Show message: "No matching jobs at this time" |
| AppSync publish fails | Queue notification for retry, notify user on next login |