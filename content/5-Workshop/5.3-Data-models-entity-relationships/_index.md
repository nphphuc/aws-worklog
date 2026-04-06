---
title : "Data Models & Entity Relationships"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

### A. Unified Database Schema

```mermaid
erDiagram
    Users ||--|| CandidateProfiles : "has_profile"
    Users ||--|| CandidateEmbeddings : "has_vector"
    Companies ||--|{ Jobs : "posts"
    Jobs ||--|| JobEmbeddings : "has_vector"
    Users ||--|{ Applications : "applies_to"
    Jobs ||--|{ Applications : "receives"

    Users {
        int id PK
        string email
        string cognito_sub
        string user_type
    }

    CandidateProfiles {
        int id PK
        int user_id FK
        jsonb extracted_profile "skills, exp, education, summary"
        timestamp updated_at
    }

    CandidateEmbeddings {
        int id PK
        int user_id FK
        vector(1024) embedding
        text raw_text "masked CV text"
    }

    Companies {
        int id PK
        string name
        string industry
    }

    Jobs {
        uuid id PK
        uuid recruiter_id FK
        text title
        text description "Raw JD text — source of truth for matching"
        timestamp created_at
    }

    JobEmbeddings {
        int id PK
        int job_id FK
        vector(1024) embedding
        text raw_text "JD text"
    }

    Applications {
        int id PK
        int user_id FK
        int job_id FK
        string status "APPLIED|REVIEWING|REJECTED"
        float match_score
        timestamp created_at
    }
```

### B. Data Storage Strategy

#### 1. Raw Assets (Amazon S3)

- **Bucket**: `smarthire-raw-assets`
- **Paths**: `candidates/{userId}/cv.pdf`
- **Purpose**: Immutable backup and audit trail for CV PDFs.
- **Note**: JD PDFs are no longer uploaded to S3. JD text is stored in RDS `Jobs.Description`.

#### 2. Relational & Vector Data (Amazon RDS PostgreSQL)

Serves as the **Source of Truth** for business entities and semantic search.

- **Business Tables**: `Users`, `Jobs`, `Applications`, `Companies`.
- **AI Tables**: `CandidateProfiles` (AI-extracted profile data).
- **Pgvector**: `CandidateEmbeddings`, `JobEmbeddings` (1024-dim Cohere vectors).
- **JD Source**: `Jobs.Description` column is read directly by `CvJdProcessor` Lambda.

#### 3. Hot Store / Cache (Amazon DynamoDB)

- **Table**: `ApplicationTracking` (Single Table Design)
- **Purpose**: High-frequency read/write operations for real-time UI updates (suggestions, rankings, parse results).
