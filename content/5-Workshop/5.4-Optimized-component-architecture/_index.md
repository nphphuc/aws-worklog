---
title : "Optimized Component Architecture"
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
        ↓
React SPA sends POST request:
  Authorization: JWT (Cognito token)
  Content-Type: application/json
        ↓
API Gateway validates request
        ↓
Lambda (.NET 8) validates & stores data
```

---

#### Step 2: Job Storage in RDS

**RDS (PostgreSQL) Insert:**

Data saved with fields:
- ID, recruiter ID, title, description
- Required skills, location, salary
- Posted date, status

---

#### Step 3: Trigger Processing Pipeline (Lambda + Step Functions)

As soon as job is saved to RDS, the Lambda function:

```python
1. Extract job description & required skills
2. Call cv_jd_processor to:
   - Normalize job requirements
   - Generate job embedding using Bedrock
   - Extract key technical skills
3. Invoke Step Functions with job context:
   - Branches to candidate_ranking_engine
```

---

#### Step 4: Candidate Search & Ranking (candidate_ranking_engine Lambda)

**Processing Steps:**

```python
1. Query RDS for all candidate CVs with parsed skills
2. Query DynamoDB for stored candidate embeddings
3. For EACH candidate:
   - Calculate cosine similarity between CV and Job embeddings
   - Score components:
     * Semantic similarity: 40%
     * Exact skill match: 40%
     * Experience level match: 15%
     * Location match: 5%
4. Filter by minimum threshold (65%)
5. Sort by score descending
6. Limit to top 50 candidates
```

---

#### Step 5: Store Rankings in Data Layer

**RDS Update:**
- job_id, candidate_id, match_score
- rank, skill_match_percentage, semantic_similarity
- ranked_at timestamp

**DynamoDB Record:**
```
PK: "JOB#job-12345"
SK: "RANKING#1#can-555"
{
  "matchScore": 86.45,
  "candidateId": "can-555",
  "skills": [...]
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
        ↓
AppSync broadcasts via subscription
        ↓
Recruiter React dashboard receives update
        ↓
UI refreshes with ranked candidates list
```

---

#### Step 7: Recruiter Dashboard Experience

**Job Details Page Shows:**
- Job title, description, posting date
- **Live Candidate Rankings** (real-time updates):
  - 🏆 Top candidate with score 86
  - 🥈 Second candidate with score 81
  - 🥉 Third candidate with score 78

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
3. Auto re-ranked against ALL JOBS where candidate might fit
4. If score > threshold:
   - Candidate added to job's ranking list
   - Recruiter notified immediately
   - Dashboard updates in real-time

---

#### Performance & Monitoring

**CloudWatch Metrics:**
- Job creation latency (target: < 2 seconds)
- Candidate ranking time (target: < 3 seconds)
- Number of candidates per ranking
- AppSync subscription payload size

**Alarms:**
- Job creation fails → Notify Engineering
- Ranking > 5 minutes → Kill and requeue
- AppSync publish fails → Retry queue

---

#### Advantages of the Pipeline

✅ Full Ranking - All candidates compared to new job

✅ Real-time Updates - New candidates ranked immediately on apply

✅ Auto Re-trigger - When job requirements updated

✅ Sorted by Score - Best candidates listed first
