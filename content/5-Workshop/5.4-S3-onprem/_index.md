---
title : "Recruiter Path"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

#### Recruiter Journey: Job Creation & Candidate Ranking

This section details the complete end-to-end flow when a recruiter creates or updates a job posting, from API call through intelligent candidate ranking and real-time dashboard updates.

---

#### Step 1: Job Creation via API (React + REST API)

When a recruiter creates a new job in the SmartHire-AI dashboard:

```
Recruiter fills form:
  - Job Title
  - Job Description
  - Required Skills
  - Location(s)
  - Salary Range
  - Department
        ↓
React SPA sends POST request:
  Authorization: JWT (Cognito token)
  Content-Type: application/json
        ↓
API Gateway validates request
        ↓
Lambda (.NET 8) validates & stores data
```

**Validation Check:**
- User is authenticated (JWT valid)
- User has recruiter role (IAM authorization)
- Required fields present (title, description)
- No duplicate job postings

---

#### Step 2: Job Storage in RDS

**RDS (PostgreSQL) Insert:**

```sql
INSERT INTO jobs (
  id, 
  recruiter_id, 
  title, 
  description, 
  required_skills, 
  location,
  salary_min,
  salary_max,
  posted_at,
  job_status
) VALUES (
  'job-12345',
  'recruiter-789',
  'Senior AWS Architect',
  'We are looking for...',
  '["AWS", "Terraform", "Python"]',
  'Remote, US',
  150000,
  180000,
  NOW(),
  'ACTIVE'
);

-- Also insert initial embedding (placeholder, will be computed):
INSERT INTO job_embeddings 
VALUES ('job-12345', vector_placeholder);
```

**Lambda Response to Recruiter:**
```json
{
  "jobId": "job-12345",
  "status": "CREATED",
  "processingStarted": true,
  "message": "Job posted successfully. Matching candidates is processing..."
}
```

---

#### Step 3: Trigger Processing Pipeline (Lambda + Step Functions)

As soon as job is saved to RDS, the Lambda function:

```python
1. Extract job description & required skills
2. Call cv_jd_processor to:
   - Normalize job requirements
   - Generate job embedding using Bedrock
   - Extract key technical skills
   - Estimate seniority level
3. Invoke Step Functions with job context:
   - Step Functions branches differently for JOB vs. CV path
   - Instead of job_suggestion_engine
   - Calls candidate_ranking_engine
```

**Difference from Candidate Path:**
```
Candidate Path:        Recruiter Path:
CV Upload  -----      Job Creation
   ↓                      ↓
Parse CV  -----      Parse Job
   ↓                      ↓
Generate  -----      Generate
Embeddings             Embeddings
   ↓                      ↓
Compare vs          Compare vs
All Jobs             All CVs
   ↓                      ↓
Suggest          Rank & Return
Jobs to           Candidates
Candidate          to Recruiter
```

---

#### Step 4: Candidate Search & Ranking (candidate_ranking_engine Lambda)

**Input from Step Functions:**
```python
{
  "jobId": "job-12345",
  "jobEmbeddings": [...vector...],
  "requiredSkills": ["AWS", "Terraform", "Python"],
  "minimumMatch": 0.65
}
```

**Processing Steps:**

```python
1. Query RDS for all candidate CVs with parsed skills
2. Query DynamoDB for stored candidate embeddings
3. For EACH candidate:
   - Calculate cosine similarity between:
     * CV embeddings vs Job embeddings
     * Candidate skills vs Required skills
   - Score components:
     * Semantic similarity: 40%
     * Exact skill match: 40%
     * Experience level match: 15%
     * Location/timezone match: 5%
   - Calculate final score (0-100)
4. Filter by minimum threshold (65%)
5. Sort by score descending
6. Limit to top 50 candidates
7. Enrich with additional data:
   - Apply date
   - Previous interviews
   - Candidate rating
```

**Scoring Example:**
```
Candidate: John Doe
  - CV Embeddings vs Job Embeddings: 0.88 (88 points × 0.40) = 35.2
  - Skill Match: 8/10 required skills = 0.80 (80 × 0.40) = 32.0
  - Experience Level: Senior role + 8 yrs exp = 0.95 (95 × 0.15) = 14.25
  - Location: Remote (matches) = 1.0 (100 × 0.05) = 5.0
  ─────────────────────────────────────────────────
  FINAL SCORE: 86.45 / 100 ✓ (Above 65% threshold)
```

---

#### Step 5: Store Rankings in Data Layer

**RDS Update:**
```sql
INSERT INTO candidate_rankings (
  job_id,
  candidate_id,
  match_score,
  rank,
  skill_match_percentage,
  semantic_similarity,
  ranked_at
) VALUES (
  'job-12345',
  'can-555',
  86.45,
  1,
  80,
  0.88,
  NOW()
);
-- Insert top 50 candidates...
```

**DynamoDB Record:**
```
PK: "JOB#job-12345"
SK: "RANKING#1#can-555"
{
  "matchScore": 86.45,
  "candidateId": "can-555",
  "skills": ["AWS", "Terraform", "Python"],
  "missingSkills": ["Kotlin"],
  "rankedAt": timestamp
}
```

---

#### Step 6: Real-time Dashboard Update (AppSync)

Once ranking completes:

```
Step Functions → Lambda finished
        ↓
Lambda publishes AppSync GraphQL mutation:
  "CandidateRankingReady"
  {
    "jobId": "job-12345",
    "totalCandidates": 47,
    "rankedCandidates": [
      {
        "rank": 1,
        "candidateId": "can-555",
        "name": "John Doe",
        "score": 86.45,
        "skills": [...]
      },
      // more candidates...
    ],
    "completedAt": timestamp
  }
        ↓
AppSync broadcasts via subscription
        ↓
Recruiter React dashboard receives update
        ↓
UI refreshes with ranked candidates list (NO page refresh)
```

---

#### Step 7: Recruiter Dashboard Experience

**Job Details Page Shows:**
- Job title, description, posting date
- **Live Candidate Rankings** (updated in real-time):
  - 🏆 Top candidate with score 86
  - 🥈 Second candidate with score 81
  - 🥉 Third candidate with score 78
  - ...and more

**Recruiter Actions:**
- Click candidate → View full CV + match details
- Send interview invitation
- Add notes / reject candidate
- Update job requirements → Re-trigger ranking

---

#### Step 8: Continuous Updates

**When New Candidates Apply:**
1. Candidate uploads CV
2. CV goes through processing pipeline
3. New embeddings created for candidate
4. **Auto re-ranked against ALL JOBS** where candidate might fit
5. If score > threshold and job is still active:
   - Candidate added to job's ranking list
   - Recruiter notified immediately
   - Dashboard updates in real-time

---

#### Performance & Monitoring

**CloudWatch Metrics:**
- Job creation latency (target: < 2 seconds)
- Candidate ranking time (target: < 3 seconds for full re-rank)
- Number of candidates per ranking
- AppSync subscription payload size

**Alarms:**
- Job creation fails → Notify Engineering
- Ranking > 5 minutes → Kill and requeue
- AppSync publish fails → Retry queue

---

#### Error Scenarios

| Scenario | Action |
|----------|--------|
| Job title blank | Return 400 Bad Request |
| Required skills empty | Generate default set from description |
| Bedrock API fails | Queue job for retry (max 3) |
| No candidates meet threshold | Show "Waiting for qualified candidates" |
| AppSync subscription fails | Store result in DB, push on recruiter next login |
| Duplicate job title + recruiter | Allow (different start dates may justify re-posting) |



