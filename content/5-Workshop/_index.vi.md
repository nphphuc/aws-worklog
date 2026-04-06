---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# SmartHire-AI: Nền tảng Tuyển Dụng Thông Minh trên AWS

#### Tổng quan

**SmartHire-AI** là nền tảng tuyển dụng thông minh được xây dựng trên AWS, cách mạng hóa quá trình tuyển dụng. Ứng viên tải lên CV và nhận được gợi ý công việc được cá nhân hóa, trong khi các nhà tuyển dụng quản lý các bài đăng và xem thứ hạp các ứng viên trong thời gian thực. Nền tảng này tận dụng AI và các dịch vụ serverless của AWS để có thể mở rộng quy mô và hiệu suất cao.

#### Các tính năng chính

- **Cho Ứng viên**: Đăng nhập, tải lên CV (PDF), và nhận gợi ý công việc với cập nhật thời gian thực
- **Cho Nhà tuyển dụng**: Tạo và quản lý các bài đăng việc làm, xem các ứng viên được xếp hạng khi họ ứng tuyển
- **Cập nhật Thời gian thực**: AWS AppSync GraphQL subscriptions đẩy kết quả ngay lập tức
- **Kết hợp được trang bị AI**: Bedrock, Comprehend, và Textract tạo nên kết hợp thông minh

#### Nội dung

1. [Giới thiệu Workshop](5.1-Workshop-overview/)
2. [Kiến trúc hệ thống](5.2-Prerequiste/)
3. [Luồng Ứng viên - Tải lên CV](5.3-S3-vpc/)
4. [Luồng Nhà tuyển dụng - Quản lý Việc làm](5.4-S3-onprem/)
5. [Công nghệ được sử dụng](5.5-Policy/)
6. [Cấu trúc Kho mã](5.6-Cleanup/)