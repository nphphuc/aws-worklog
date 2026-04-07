---
title : "Mô hình dữ liệu & quan hệ thực thể"
date : 2024-01-01 
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

### A. Lược đồ cơ sở dữ liệu hợp nhất

![image](https://res.cloudinary.com/diahbkjog/image/upload/v1775493744/5_tckslm.png)

### B. Chiến lược lưu trữ dữ liệu

#### 1. Tài nguyên thô (Amazon S3)

- **Bucket**: `smarthire-raw-assets`
- **Paths**: `candidates/{userId}/cv.pdf`
- **Mục đích**: Lưu trữ bản sao bất biến và phục vụ kiểm toán cho các file CV dạng PDF.
- **Lưu ý**: File JD (Job Description) dạng PDF không còn được upload lên S3. Nội dung JD được lưu trong cột `Jobs.Description` của RDS.

#### 2. Dữ liệu quan hệ & vector (Amazon RDS Postgres)

Đóng vai trò là **nguồn dữ liệu chủ đạo** cho các thực thể nghiệp vụ và tìm kiếm ngữ nghĩa.

- **Bảng nghiệp vụ**: `Users`, `Jobs`, `Applications`, `Companies`.
- **Bảng AI**: `CandidateProfiles` (dữ liệu hồ sơ được AI trích xuất).
- **Pgvector**: `CandidateEmbeddings`, `JobEmbeddings` (vector 1024 chiều từ Cohere).
- **Nguồn JD**: Cột `Jobs.Description` được đọc trực tiếp bởi Lambda `CvJdProcessor`.

#### 3. Kho dữ liệu nóng / Cache (Amazon DynamoDB)

- **Bảng**: `ApplicationTracking` (thiết kế bảng đơn lẻ)
- **Mục đích**: Xử lý các thao tác đọc/ghi tần suất cao để cập nhật giao diện người dùng theo thời gian thực (gợi ý, xếp hạng, kết quả phân tích).