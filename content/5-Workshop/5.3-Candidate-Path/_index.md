---
title : "Data Models & Entity Relationships"
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
  ↓
[Step 2] Invoke job_suggestion_engine Lambda
          - Compare CV embeddings vs Job embeddings
          - Score & rank all open jobs
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

---

#### Step 4: CV & Document Processing (Lambda + AI)

**Lambda: cv_jd_processor (Python 3.12)**

Process:
1. Download PDF from S3
2. Amazon Textract → Extract text, tables, entities
3. Parse structure:
   - Name, email, phone
   - Work experience
   - Education
   - Skills list
4. Amazon Bedrock → Enrich data
5. Amazon Comprehend → NLP analysis
6. Generate embeddings using Bedrock

---

#### Step 5: Job Suggestion Engine

**Lambda: job_suggestion_engine**

Process:
1. Query RDS for all active jobs
2. For each job:
   - Calculate cosine similarity with CV embeddings
   - Score based on skill match, experience level, location
3. Sort by match score (descending)
4. Filter by threshold (>0.6 minimum match)
5. Select top 10 most relevant jobs

---

#### Step 6: Real-time UI Update (AWS AppSync)

Once matching is complete:

```
Lambda publishes AppSync GraphQL mutation:
"JobSuggestionsReady"
        ↓
AppSync broadcasts via subscription
        ↓
Candidate React SPA receives update
        ↓
Dashboard re-renders (NO page refresh needed)
```

---

#### Performance & Monitoring

**CloudWatch Metrics:**
- Lambda execution duration
- SQS queue depth
- Step Functions execution time
- Textract processing cost per document

**Alarms:**
- SQS queue > 100 messages
- Lambda error rate > 1%
- Processing time > 5 minutes
