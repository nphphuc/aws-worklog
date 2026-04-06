---
title : "Giới thiệu"
date : 2024-01-01 
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

#### Giới thiệu SmartHire-AI

**SmartHire-AI** là một nền tảng tuyển dụng thông minh toàn diện, sử dụng các dịch vụ AWS và AI để kết nối ứng viên với các cơ hội việc làm. Hệ thống hoạt động như một ứng dụng web được xây dựng bằng **React (Vite)** chạy phía sau **Amazon CloudFront** và **S3**, có **Amazon Cognito** để xác thực.

#### Cách thức hoạt động

##### Cho Ứng viên
- Đăng nhập vào nền tảng thông qua Cognito (hỗ trợ Google federation)
- Tải lên **CV dạng PDF** lên Amazon S3
- Nhận **gợi ý việc làm được cá nhân hóa** dựa trên phân tích CV
- Xem cập nhật thời gian thực khi pipeline xử lý CV của họ thông qua AWS AppSync GraphQL subscriptions

##### Cho Nhà tuyển dụng
- Tạo và quản lý **các bài đăng việc làm** được lưu trữ trong cơ sở dữ liệu quan hệ (RDS PostgreSQL)
- Khi một công việc được tạo/cập nhật, backend sẽ tự động bắt đầu xử lý
- Xem **ứng viên được xếp hạng** trên dashboard của họ
- Nhận cập nhật thời gian thực khi ứng viên ứng tuyển thông qua AppSync subscriptions

#### Các điểm nổi bật về Kiến trúc

🟢 **Frontend**: React SPA được phục vụ qua S3 + CloudFront với đăng nhập Cognito thông qua JWT

🟢 **API**: Các hàm Lambda .NET 8 phía sau API Gateway với ủy quyền Cognito

🟢 **Pipeline Xử lý CV**: Điều phối Serverless bao gồm:
- **AWS Step Functions** để quản lý workflow
- **AWS Lambda** (Python) để xử lý tài liệu
- **Amazon Textract** để trích xuất text từ PDF
- **Amazon Bedrock** để tự giàu hóa được hỗ trợ AI
- **Amazon Comprehend** để phân tích text
- **AWS SQS** để xếp hàng công việc
- **Amazon RDS (PostgreSQL)** để lưu dữ liệu có cấu trúc
- **Amazon DynamoDB** để theo dõi

🟢 **Cập nhật Thời gian thực**: AWS AppSync cho GraphQL subscriptions đẩy kết quả tức thì đến UI

🟢 **Infrastructure as Code**: Terraform cho tài nguyên AWS, AWS SAM cho Serverless API

---

#### Cấu trúc Workshop

Trong workshop này, bạn sẽ tìm hiểu:
1. **Kiến trúc** - Đi sâu vào các thành phần và luồng dữ liệu
2. **Luồng Ứng viên** - Cách CV uploads được xử lý từ đầu đến cuối
3. **Luồng Nhà tuyển dụng** - Cách các bài đăng việc làm kích hoạt kết hợp thông minh
4. **Công nghệ được sử dụng** - Tất cả các dịch vụ AWS và công cụ được sử dụng
5. **Cấu trúc Kho mã** - Cấu trúc dự án và triển khai
