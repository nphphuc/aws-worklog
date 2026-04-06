---
title : "RDS Postgres Schema"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

**candidate_embeddings** (pgvector)

```sql
CREATE TABLE candidate_embeddings (
  user_id UUID PRIMARY KEY,
  embedding VECTOR(1024) NOT NULL,
  raw_text TEXT,
  updated_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(user_id)
);
CREATE INDEX ON candidate_embeddings USING ivfflat (embedding vector_cosine_ops);
```

**job_embeddings** (pgvector)

```sql
CREATE TABLE job_embeddings (
  job_id UUID PRIMARY KEY,
  embedding VECTOR(1024) NOT NULL,
  raw_text TEXT,
  updated_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(job_id)
);
CREATE INDEX ON job_embeddings USING ivfflat (embedding vector_cosine_ops);
```

**Jobs** (EF Core managed — JD text source)

```sql
CREATE TABLE "Jobs" (
  "Id" UUID PRIMARY KEY,
  "RecruiterId" UUID NOT NULL,
  "Title" TEXT,
  "Description" TEXT,
  "CreatedAt" TIMESTAMP WITH TIME ZONE,
  "JdFileUrl" TEXT
);
```