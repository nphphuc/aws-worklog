---
title : "System workflows"
date : 2024-01-01 
weight : 1 
chapter : false
pre : " <b> 5.1. </b> "
---

### A. Candidate Flow (CV Upload → Job Suggestions)

![image](https://res.cloudinary.com/diahbkjog/image/upload/v1775491815/Mermaid_Chart_-_Create_complex_visual_diagrams_with_text.-2026-04-06-160926_ktpmft.png)

### B. Recruiter Flow (Job Create → Ranked Candidates)

```mermaid
sequenceDiagram
    autonumber
    participant R as Recruiter
    participant NET as .NET API (Backend)
    participant RDS_META as RDS (Jobs table)
    participant SF as Step Functions<br/>(cv_processing)
    participant PROC as CvJdProcessor
    participant CR as CandidateRankingEngine
    participant RDS as RDS Proxy<br/>(pgvector)
    participant DDB as DynamoDB
    participant APP as AppSync

    Note over R, APP: Recruiter opens dashboard and subscribes onCandidateRanking(jobId)
    R->>APP: GraphQL Subscription (Cognito JWT)

    Note over R, NET: 1. Job Creation
    R->>NET: POST /api/recruiter/jobs { title, description }
    NET->>RDS_META: INSERT INTO Jobs (Title, Description)
    NET->>SF: StartExecution { profile_id: "recruiter", job_id }
    NET-->>R: 201 Created { jobId }

    Note over SF, PROC: 2. Unified Processing (JD Path)
    SF->>PROC: Invoke CvJdProcessor
    rect rgb(240, 248, 255)
        PROC->>RDS_META: SELECT Title, Description FROM Jobs WHERE Id = job_id
        PROC->>PROC: Cohere: embed JD text (search_document)
        PROC-->>SF: job_title, jd_text, jd_vector
    end

    Note over SF, APP: 3. Routing & Ranking
    SF->>SF: Choice: profile_id == "recruiter"?
    SF->>CR: YES → Invoke CandidateRankingEngine

    rect rgb(255, 250, 240)
        Note right of CR: Recruiter-Side Engine
        CR->>RDS: UPSERT job_embeddings
        CR->>CR: Re-embed JD as search_query (asymmetric)
        CR->>RDS: pgvector ANN: top 50 candidates
        CR->>DDB: Fetch candidate profiles + interview guides
        CR->>CR: Cross-Encoder rerank<br/>Hybrid score (30% BI, 70% CE)
        CR->>CR: Claude: candidate snapshot per rank
        CR->>DDB: PUT CANDIDATE_RANKING<br/>(PK=JOB#{job_id})
        CR->>APP: publishCandidateRanking(jobId, rankedCandidates, updatedAt)
    end

    APP-->>R: WebSocket: Top 15 ranked candidates
```

### C. Application Management Flow (.NET Core CRUD)

```mermaid
sequenceDiagram
    autonumber
    participant C as Candidate (Frontend)
    participant R as Recruiter
    participant NET as .NET API (Backend)
    participant RDS as RDS (Metadata)

    Note over C, NET: 1. Candidate Applies
    C->>NET: POST /api/applications { jobId }

    rect rgb(240, 248, 255)
        NET->>RDS: Validate User & Job Exist
        NET->>RDS: Check for Duplicate Application
        NET->>RDS: INSERT INTO Applications (Status="APPLIED")
    end

    NET-->>C: 201 Created { applicationId, status: "APPLIED" }

    Note over R, NET: 2. Recruiter Review (CRUD)
    R->>NET: GET /api/jobs/{jobId}/applications
    NET->>RDS: SELECT Applications JOIN Users JOIN Profiles
    RDS-->>NET: Application List [ { id, candidateName, matchScore } ]
    NET-->>R: 200 OK (List)

    Note over R, NET: 3. Status Update
    R->>NET: PATCH /api/applications/{appId} { status: "INTERVIEWING" }
    NET->>RDS: UPDATE Applications SET Status="INTERVIEWING"
    NET-->>R: 200 OK
```

### D. Master End-to-End Flow (Single View)

```mermaid
sequenceDiagram
  autonumber
  participant C as Candidate UI
  participant R as Recruiter UI
  participant NET as .NET Backend
  participant S3 as S3 (CV)
  participant SQS as SQS
  participant ING as IngestionTrigger
  participant SF as Step Functions (cv_processing)
  participant PROC as CvJdProcessor
  participant JSE as JobSuggestionEngine
  participant CRE as CandidateRankingEngine
  participant RDS as RDS + pgvector
  participant DDB as DynamoDB
  participant APP as AppSync

  Note over C,APP: Candidate dashboard subscribes onJobSuggestions(candidateId)
  C->>APP: GraphQL Subscription (Cognito JWT)
  Note over R,APP: Recruiter dashboard subscribes onCandidateRanking(jobId)
  R->>APP: GraphQL Subscription (Cognito JWT)

  par Candidate trigger (CV upload)
    C->>S3: Upload CV.pdf
    S3->>SQS: ObjectCreated event
    SQS->>ING: Batch trigger
    ING->>ING: Validate PDF
    ING->>SF: StartExecution(profile_id, job_id, file_key, bucket)
  and Recruiter trigger (job create)
    R->>NET: POST /api/recruiter/jobs
    NET->>RDS: INSERT Jobs(title, description)
    NET->>SF: StartExecution(profile_id="recruiter", job_id)
  end

  SF->>PROC: Invoke CvJdProcessor
  PROC->>PROC: Extract/prepare text + embed
  SF->>SF: RoutingChoice(profile_id)

  alt profile_id != recruiter
    SF->>JSE: Invoke JobSuggestionEngine
    JSE->>RDS: Candidate vector search + rerank jobs
    JSE->>DDB: Save JOB_SUGGESTIONS
    JSE->>APP: publishJobSuggestions(candidateId, suggestions, updatedAt)
    APP-->>C: Realtime suggestions (Top jobs)
  else profile_id == recruiter
    SF->>CRE: Invoke CandidateRankingEngine
    CRE->>RDS: JD search + candidate rerank
    CRE->>DDB: Save CANDIDATE_RANKING
    CRE->>APP: publishCandidateRanking(jobId, rankedCandidates, updatedAt)
    APP-->>R: Realtime ranking (Top candidates)
  end
```