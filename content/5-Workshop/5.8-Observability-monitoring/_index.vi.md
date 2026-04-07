---
title : "Quan sát & giám sát hệ thống"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.8. </b> "
---

### A. Phạm vi truy vết

X-Ray instrumentation được áp dụng cho các luồng xử lý CV/JD:

1. **IngestionTrigger**: Truy vết quá trình kiểm tra file và việc gọi Step Functions.
2. **Step Functions Workflow**: Hiển thị luồng điều phối (CvJdProcessor → Engine).
3. **CvJdProcessor**: Truy vết các lời gọi Textract (CV), đọc dữ liệu từ RDS (JD), và tạo embedding.
4. **Engines**: Truy vết các truy vấn pgvector, ghi dữ liệu vào DynamoDB, và tạo snapshot bằng Claude.

### B. Các chỉ số quan trọng cần theo dõi

- **Độ trễ End-to-End**: Thời gian từ lúc upload/tạo job đến khi hoàn tất xếp hạng.
- **Tỷ lệ lỗi**: Lỗi trong Textract, giới hạn (throttling) của Bedrock, timeout của RDS.
- **Phụ thuộc dịch vụ**: Lambda → RDS, Lambda → Bedrock, Lambda → DynamoDB.
- **Cold Start**: Ảnh hưởng của thời gian khởi tạo Lambda đến thông lượng xử lý.

### C. Chi tiết triển khai

- **Quyền IAM**: `xray:PutTraceSegments` và `xray:PutTelemetryRecords` trên execution role của Lambda.
- **Biến môi trường**: `AWS_XRAY_CONTEXT_MISSING` được đặt là `LOG_ERROR`.
- **Cấu hình tracing**: Bật active tracing cho tất cả Lambda và state machine của Step Functions.