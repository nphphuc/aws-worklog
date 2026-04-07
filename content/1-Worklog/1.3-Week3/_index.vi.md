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

* Kết nối, làm quen với các thành viên trong First Cloud Journey.
* Hiểu dịch vụ AWS cơ bản, cách dùng console & CLI.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | Launch EC2 instance (t2.micro), tạo key pair và kết nối SSH để kiểm tra AMI/EBS.                                                                                                              | 19/01/2026   | 19/01/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 3   | Tạo và attach IAM role cho EC2 (instance profile); kiểm tra quyền truy cập tài nguyên.                                                                                                       | 20/01/2026   | 20/01/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 4   | Thiết lập AWS Cloud9 workspace; triển khai và chạy prototype nhỏ trong Cloud9.                                                                                                                | 21/01/2026   | 21/01/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 5   | Tạo S3 bucket cho static website; cấu hình hosting và bucket policy cơ bản.                                                                                                                     | 22/01/2026   | 22/01/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 6   | Provision RDS (ví dụ MySQL), cấu hình security group, backup và thử kết nối từ EC2/Cloud9.                                                                                                      | 23/01/2026   | 23/01/2026      | <https://cloudjourney.awsstudygroup.com/> |


### Kết quả đạt được tuần 3:

* Hiểu AWS là gì và nắm được các nhóm dịch vụ cơ bản: 
  * Compute
  * Storage
  * Networking 
  * Database
  * ...

* Đã tạo và cấu hình AWS Free Tier account thành công.

* Làm quen với AWS Management Console và biết cách tìm, truy cập, sử dụng dịch vụ từ giao diện web.

* Cài đặt và cấu hình AWS CLI trên máy tính bao gồm:
  * Access Key
  * Secret Key
  * Region mặc định
  * ...

* Sử dụng AWS CLI để thực hiện các thao tác cơ bản như:

  * Kiểm tra thông tin tài khoản & cấu hình
  * Lấy danh sách region
  * Xem dịch vụ EC2
  * Tạo và quản lý key pair
  * Kiểm tra thông tin dịch vụ đang chạy
  * ...

* Có khả năng kết nối giữa giao diện web và CLI để quản lý tài nguyên AWS song song.
* ...


