---
title: "Worklog Tuần 5"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.5. </b> "
---
<!-- {{% notice warning %}}
⚠️ **Lưu ý:** Các thông tin dưới đây chỉ nhằm mục đích tham khảo, vui lòng **không sao chép nguyên văn** cho bài báo cáo của bạn kể cả warning này.
{{% /notice %}} -->


### Mục tiêu tuần 5:

* Thực hiện các tác vụ tự động hóa và quản lý phổ biến bằng AWS CLI.
* Thiết kế mô hình dữ liệu NoSQL và vận hành Amazon DynamoDB (capacity, indexes, queries).
* Triển khai caching in-memory với Amazon ElastiCache (Redis/Memcached).
* Áp dụng các chủ đề networking nâng cao: VPC peering và Endpoints (workshop "Networking on AWS").
* Cấu hình CloudFront distribution để tăng tốc phân phối nội dung và caching.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | Viết script tự động hóa với AWS CLI (create/list/describe) cho workflow cơ bản.                                                                                                               | 02/02/2026   | 02/02/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 3   | Thiết kế mô hình NoSQL và tạo bảng DynamoDB; thử capacity, index và queries cơ bản.                                                                                                            | 03/02/2026   | 03/02/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 4   | Triển khai Amazon ElastiCache (Redis), kiểm thử pattern caching cho ứng dụng demo.                                                                                                             | 04/02/2026   | 04/02/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 5   | Thực hành networking nâng cao: VPC peering và tạo Endpoint (Interface/Private) cho dịch vụ cần thiết.                                                                                         | 05/02/2026   | 05/02/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 6   | Thiết lập CloudFront distribution cho S3 static site; kiểm tra behavior và invalidation.                                                                                                      | 06/02/2026   | 06/02/2026      | <https://cloudjourney.awsstudygroup.com/> |


### Kết quả đạt được tuần 5:

* Đã viết các script tự động hóa bằng AWS CLI cho các workflow create/list/describe.
* Đã thiết kế mô hình NoSQL và tạo bảng DynamoDB; thử nghiệm capacity, index và query cơ bản.
* Đã triển khai ElastiCache (Redis) và kiểm chứng pattern caching cho ứng dụng demo.
* Đã cấu hình networking nâng cao (VPC peering) và tạo các Endpoint (Interface/Private) cần thiết.
* Đã thiết lập CloudFront distribution cho S3 static site; kiểm tra behaviors và thực hiện invalidation.
* Kiểm chứng tự động hóa và mạng bằng cả Console và CLI.



