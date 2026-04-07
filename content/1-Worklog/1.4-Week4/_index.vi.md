---
title: "Worklog Tuần 4"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.4. </b> "
---
<!-- {{% notice warning %}}
⚠️ **Lưu ý:** Các thông tin dưới đây chỉ nhằm mục đích tham khảo, vui lòng **không sao chép nguyên văn** cho bài báo cáo của bạn kể cả warning này.
{{% /notice %}} -->


### Mục tiêu tuần 4:

* Khám phá Amazon Lightsail để thiết lập VM và networking đơn giản.
* Triển khai ứng dụng container bằng Lightsail Containers và quản lý image.
* Thiết kế và cấu hình Auto Scaling group cho EC2 để tự động mở rộng ứng dụng.
* Giám sát tài nguyên và thiết lập cảnh báo bằng Amazon CloudWatch (metrics và logs).
* Cấu hình định tuyến DNS và kịch bản DNS hybrid bằng Amazon Route 53.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | Khám phá Amazon Lightsail: tạo instance, kiểm tra networking và snapshot.                                                                                                                     | 26/01/2026   | 26/01/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 3   | Triển khai ứng dụng container bằng Lightsail Containers; quản lý image và deployment.                                                                                                         | 27/01/2026   | 27/01/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 4   | Thiết kế và cấu hình EC2 Auto Scaling group (launch template/LC) cho ứng dụng demo.                                                                                                           | 28/01/2026   | 28/01/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 5   | Cấu hình Amazon CloudWatch: bật metrics, tạo log group và thiết lập alarm cơ bản.                                                                                                              | 29/01/2026   | 29/01/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 6   | Thiết lập Route 53 record để routing; thử scenario DNS hybrid nếu có tài nguyên on-prem/placeholder.                                                                                            | 30/01/2026   | 30/01/2026      | <https://cloudjourney.awsstudygroup.com/> |


### Kết quả đạt được tuần 4:

* Đã tạo và kiểm tra instance trên Amazon Lightsail; xác thực networking và tạo snapshot.
* Đã triển khai ứng dụng container bằng Lightsail Containers và quản lý image, deployment.
* Đã thiết kế và cấu hình Auto Scaling group cho EC2 (sử dụng launch template/launch configuration) cho ứng dụng demo và kiểm chứng hành vi scale.
* Đã bật CloudWatch metrics, tạo log group và thiết lập cảnh báo cơ bản để giám sát ứng dụng demo.
* Đã cấu hình record Route 53 để routing và xác minh hành vi DNS; thử kịch bản DNS hybrid khi có thể.
* Kiểm chứng triển khai và giám sát bằng cả Console và CLI.



