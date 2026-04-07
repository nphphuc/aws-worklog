---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Kiến trúc hệ thống SmartHire Matching (v3.0 — Pipeline hợp nhất)

#### Tổng quan

**SmartHire-AI** là một nền tảng tuyển dụng thông minh được xây dựng trên AWS, giúp cách mạng hóa quy trình tuyển dụng. Ứng viên có thể tải CV lên và nhận được các gợi ý công việc cá nhân hóa, trong khi nhà tuyển dụng có thể quản lý tin đăng và xem danh sách ứng viên được xếp hạng theo thời gian thực. Nền tảng sử dụng AI và các dịch vụ serverless của AWS để đảm bảo khả năng mở rộng và hiệu năng cao.

#### Các tính năng chính

- **Dành cho ứng viên**: Đăng nhập, tải lên CV (PDF), và nhận gợi ý công việc với cập nhật theo thời gian thực
- **Dành cho nhà tuyển dụng**: Tạo và quản lý tin tuyển dụng, xem danh sách ứng viên được xếp hạng khi họ ứng tuyển
- **Cập nhật thời gian thực**: AWS AppSync GraphQL subscriptions gửi kết quả ngay lập tức
- **So khớp bằng AI**: Bedrock, Comprehend và Textract hỗ trợ việc so khớp thông minh

#### Các thay đổi chính

- **Không còn upload JD dạng PDF**: Nội dung mô tả công việc được đọc trực tiếp từ RDS (Jobs.Description) thay vì tải file PDF lên S3
- **Gộp Lambda xử lý**: text_processor và vector_ops được gộp thành một Lambda duy nhất là cv_jd_processor
- **Trigger JD trực tiếp**: Backend .NET gọi trực tiếp Step Functions StartExecution để xử lý JD (bỏ qua S3/SQS/IngestionTrigger)
- **Đơn giản hóa State Machine**: Step Functions giảm từ 4 bước xuống còn 3 (CvJdProcessor → RoutingChoice → Engine)
- **Giảm độ trễ**: Giảm một lần khởi động lạnh Lambda trong mỗi lần thực thi (do gộp 2 Lambda thành 1)
- **Hợp đồng realtime qua AppSync**: Các matching engine gửi cập nhật qua mutation GraphQL của AppSync, frontend nhận qua subscription đã lọc

#### Giải thích nhanh

1. Ứng viên tải CV → hệ thống phân tích CV → tính toán các công việc phù hợp nhất → đẩy kết quả realtime lên giao diện ứng viên
2. Nhà tuyển dụng tạo job → hệ thống embedding JD → xếp hạng ứng viên → đẩy kết quả realtime lên giao diện nhà tuyển dụng
3. Step Functions luôn có 3 bước: `CvJdProcessor -> RoutingChoice -> Engine`
4. AppSync chỉ là lớp truyền dữ liệu realtime (publish từ Lambda, subscribe từ frontend), không phải lớp xử lý tính toán

#### Hợp đồng realtime (hiện tại)
- Mutation: `publishJobSuggestions(candidateId, suggestions, updatedAt)`
- Mutation: `publishCandidateRanking(jobId, rankedCandidates, updatedAt)`
- Subscription: `onJobSuggestions(candidateId)`
- Subscription: `onCandidateRanking(jobId)`

Lưu ý: trường trong schema là `updatedAt` (không phải `timestamp`).

#### Nội dung

1. [Quy trình hệ thống](5.1-System-workflows/)
2. [Luồng dữ liệu chi tiết](5.2-Detailed-data-flows/)
3. [Mô hình dữ liệu & quan hệ thực thể](5.3-Data-models-entity-relationships/)
4. [Kiến trúc thành phần tối ưu](5.4-Optimized-component-architecture/)
5. [RDS Postgres Schema](5.5-RDS-postgres-schema/)
6. [DynamoDB Schema](5.6-DynamoDB-schema/)
7. [Các thành phần hạ tầng](5.7-Infrastructure-components/)
8. [Quan sát & giám sát hệ thống](5.8-Observability-monitoring/)