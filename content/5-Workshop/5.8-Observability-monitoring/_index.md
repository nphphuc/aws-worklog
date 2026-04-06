---
title : "Observability & Monitoring"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.8. </b> "
---

### A. Tracing Scope

X-Ray instrumentation applies to the CV/JD processing flows:

1. **IngestionTrigger**: Trace file validation and Step Functions invocation.
2. **Step Functions Workflow**: Visualize orchestration path (CvJdProcessor → Engine).
3. **CvJdProcessor**: Trace Textract calls (CV), RDS reads (JD), embedding generation.
4. **Engines**: Trace pgvector queries, DynamoDB writes, Claude snapshot generation.

### B. Key Metrics to Monitor

- **End-to-End Latency**: Time from upload/job creation to ranking completion.
- **Error Rates**: Failures in Textract, Bedrock throttling, RDS timeouts.
- **Service Dependencies**: Lambda → RDS, Lambda → Bedrock, Lambda → DynamoDB.
- **Cold Starts**: Lambda initialization impact on processing throughput.

### C. Implementation Details

- **IAM Permissions**: `xray:PutTraceSegments` and `xray:PutTelemetryRecords` on Lambda execution roles.
- **Environment Variables**: `AWS_XRAY_CONTEXT_MISSING` set to `LOG_ERROR`.
- **Tracing Config**: Active tracing enabled on all Lambdas and Step Functions state machine.