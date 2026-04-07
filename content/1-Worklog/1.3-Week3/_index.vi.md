---
title: "Worklog Tuần 3"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.3. </b> "
---
<!-- {{% notice warning %}}
⚠️ **Lưu ý:** Các thông tin dưới đây chỉ nhằm mục đích tham khảo, vui lòng **không sao chép nguyên văn** cho bài báo cáo của bạn kể cả warning này.
{{% /notice %}} -->


### Mục tiêu tuần 3:

* Triển khai và quản lý các instance EC2: chọn loại instance, AMI, EBS và truy cập SSH.
* Tạo và gán IAM role cho EC2 và hiểu về instance profile.
* Sử dụng AWS Cloud9 để phát triển, kiểm thử và prototype nhanh trên cloud.
* Cấu hình Amazon S3 cho hosting trang tĩnh và thiết lập chính sách bucket cơ bản.
* Triển khai và kết nối tới Amazon RDS; hiểu về engine, backup và security group.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | Launch EC2 instance (t2.micro), tạo key pair và kết nối SSH để kiểm tra AMI/EBS.                                                                                                              | 19/01/2026   | 19/01/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 3   | Tạo và attach IAM role cho EC2 (instance profile); kiểm tra quyền truy cập tài nguyên.                                                                                                       | 20/01/2026   | 20/01/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 4   | Thiết lập AWS Cloud9 workspace; triển khai và chạy prototype nhỏ trong Cloud9.                                                                                                                | 21/01/2026   | 21/01/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 5   | Tạo S3 bucket cho static website; cấu hình hosting và bucket policy cơ bản.                                                                                                                     | 22/01/2026   | 22/01/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 6   | Provision RDS (ví dụ MySQL), cấu hình security group, backup và thử kết nối từ EC2/Cloud9.                                                                                                      | 23/01/2026   | 23/01/2026      | <https://cloudjourney.awsstudygroup.com/> |


### Kết quả đạt được tuần 3:

* Đã khởi tạo EC2 (t2.micro), tạo key pair và kiểm tra kết nối SSH, AMI và EBS.
* Đã tạo và gán IAM role cho EC2 (instance profile) và kiểm tra quyền truy cập.
* Đã thiết lập AWS Cloud9 workspace và triển khai prototype nhỏ.
* Đã tạo S3 bucket cho website tĩnh và cấu hình hosting cùng chính sách bucket cơ bản.
* Đã provision RDS (ví dụ MySQL), cấu hình security group và backup; kiểm tra kết nối từ EC2/Cloud9.
* Kiểm chứng tài nguyên và kết nối bằng cả Console và CLI.



