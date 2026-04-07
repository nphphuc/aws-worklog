---
title: "Worklog Tuần 6"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.6. </b> "
---
<!-- {{% notice warning %}}
⚠️ **Lưu ý:** Các thông tin dưới đây chỉ nhằm mục đích tham khảo, vui lòng **không sao chép nguyên văn** cho bài báo cáo của bạn kể cả warning này.
{{% /notice %}} -->


### Mục tiêu tuần 6:

* Triển khai các pattern edge computing sử dụng Amazon CloudFront và Lambda@Edge.
* Lên kế hoạch và triển khai workloads Windows trên AWS với sizing và lưu ý về licensing.
* Cấu hình AWS Managed Microsoft AD và tích hợp với các instance Windows và ứng dụng yêu cầu directory.
* Thiết kế và xây dựng ứng dụng web có độ sẵn sàng cao sử dụng load balancer, triển khai multi-AZ và kiến trúc chịu lỗi.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | Thử nghiệm Lambda@Edge với CloudFront (ví dụ rewrite header hoặc redirect nhẹ).                                                                                                              | 09/02/2026   | 09/02/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 3   | Lên kế hoạch và sizing cho workloads Windows trên AWS; thử deploy instance Windows mẫu.                                                                                                        | 10/02/2026   | 10/02/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 4   | Cấu hình AWS Managed Microsoft AD; thử join Windows instance vào domain và kiểm tra tích hợp.                                                                                                   | 11/02/2026   | 11/02/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 5   | Thiết kế kiến trúc web app highly-available: ELB, multi-AZ deployment và test failover.                                                                                                      | 12/02/2026   | 12/02/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 6   | Triển khai mẫu HA: ELB + Auto Scaling + multi-AZ; kiểm tra health checks và tính chịu lỗi.                                                                                                    | 13/02/2026   | 13/02/2026      | <https://cloudjourney.awsstudygroup.com/> |


### Kết quả đạt được tuần 6:

* Đã thử nghiệm Lambda@Edge cho CloudFront (ví dụ: rewrite header hoặc redirect đơn giản).
* Đã lên kế hoạch và triển khai workload Windows mẫu với sizing phù hợp.
* Đã cấu hình AWS Managed Microsoft AD và join instance Windows vào domain; kiểm tra tích hợp.
* Đã thiết kế và kiểm chứng kiến trúc ứng dụng web có độ sẵn sàng cao: ELB, Auto Scaling và triển khai multi-AZ; thử nghiệm failover.
* Đã triển khai mẫu HA (ELB + Auto Scaling + multi-AZ) và xác thực health checks, failover và khả năng chịu lỗi.
* Kiểm chứng triển khai, giám sát và hành vi failover bằng cả Console và CLI.



