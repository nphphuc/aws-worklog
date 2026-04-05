---
title: "Event 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

# Sự kiện đã tham gia: Báo cáo tổng kết

---

## Báo cáo tổng kết: “AWS re:Invent Recap HCMC”

### Mục tiêu sự kiện
* Tìm hiểu sâu về các bản phát hành sản phẩm mới nhất, bản cập nhật dịch vụ và các best practice về kiến trúc trực tiếp từ sự kiện AWS re:Invent.
* Cung cấp các phiên kỹ thuật tập trung vào việc mở rộng hạ tầng, hiện đại hóa ứng dụng và bảo mật đám mây.
* Giúp các lập trình viên và quản lý kỹ thuật khám phá các chiến lược nâng cấp hệ thống cũ bằng các công cụ cloud-native hiện đại.
* Thúc đẩy kết nối và giao lưu trong cộng đồng các kỹ sư và kiến trúc sư cloud tại địa phương.

### Các điểm nhấn chính
Sự kiện bao quát một loạt các cập nhật cốt lõi của AWS, tập trung vào Cơ sở hạ tầng & Hiện đại hóa ứng dụng (Track 1):
* **Chiến lược Hybrid & Multi-Cloud:** Xây dựng môi trường cloud linh hoạt, nhanh nhẹn và có khả năng phục hồi, trải dài trên nhiều cơ sở hạ tầng khác nhau.
* **Tiến bộ trong Container hóa:** Đơn giản hóa việc triển khai, mở rộng và quản lý các ứng dụng dựa trên container trên AWS (ECS, EKS, Fargate).
* **Cơ sở dữ liệu chuyên dụng (Purpose-Built Databases):** Dịch chuyển khỏi cơ sở dữ liệu quan hệ nguyên khối bằng cách tối ưu hóa workload và chọn đúng loại cơ sở dữ liệu cho đúng bài toán (NoSQL, Graph, Time-Series).
* **Hiện đại hóa bằng AI:** Khám phá các công cụ như AWS Transform để tự động nâng cấp và refactor codebase của các hệ thống cũ.
* **Bảo mật đám mây:** Tăng cường bảo vệ hệ thống bằng các tính năng phát hiện mối đe dọa tự động, thông minh và các framework tuân thủ.
* **Cơ sở hạ tầng nội địa:** Công bố điểm kết nối Vietnam DirectConnect POP mới giúp giảm thiểu đáng kể độ trễ và mang lại kết nối an toàn cho các doanh nghiệp Việt Nam.

### Bài học rút ra
* **Tư duy thiết kế**
    * **Đúng công cụ cho đúng việc:** Ngừng ép mọi dữ liệu vào cơ sở dữ liệu quan hệ; áp dụng các cơ sở dữ liệu chuyên dụng dựa trên các pattern truy xuất dữ liệu cụ thể của ứng dụng.
    * **Nhanh nhẹn & Linh hoạt:** Các kiến trúc hiện đại phải được thiết kế để mở rộng tự động và có thể chịu tải qua các đợt tăng lưu lượng đột biến bằng cách sử dụng kiến trúc container hóa và serverless.
* **Kiến trúc kỹ thuật**
    * **Hệ thống Decoupled (Tách rời):** Sử dụng nhiều container để đảm bảo các ứng dụng luôn portable (dễ dàng di chuyển) và dễ quản lý trên các môi trường hybrid.
* **Chiến lược hiện đại hóa**
    * **Refactor với sự hỗ trợ của AI:** Việc áp dụng các công cụ AI (như AWS Transform) giờ đây là một phần cốt lõi của vòng đời hiện đại hóa ứng dụng, giúp giảm đáng kể thời gian cần thiết để nâng cấp code cũ.

### Áp dụng vào công việc
* **Tối ưu hóa việc sử dụng Database:** Xem xét lại kiến trúc hiện tại của các dự án để đảm bảo chúng ta đang tận dụng các cơ sở dữ liệu chuyên dụng thay vì dựa vào phương pháp "one-size-fits-all" (một giải pháp cho mọi vấn đề).
* **Cải thiện Pipeline Container:** Áp dụng các best practice mới nhất cho EKS và ECS để cải thiện hiệu quả của quá trình triển khai CI/CD.
* **Đánh giá DirectConnect:** Đánh giá xem điểm DirectConnect POP mới tại Việt Nam có thể giúp giảm độ trễ cho người dùng nội địa và các dịch vụ backend của chúng ta như thế nào.

### Trải nghiệm sự kiện
Việc tham gia **“AWS re:Invent Recap HCMC”** mang lại giá trị cực kỳ lớn, vì nó mang quy mô khổng lồ và những công bố mới nhất từ hội nghị toàn cầu đến trực tiếp cộng đồng địa phương. Các trải nghiệm chính bao gồm:
* **Học hỏi từ các chuyên gia trong ngành:** Có được góc nhìn rộng hơn về cách các công ty toàn cầu đang thiết kế kiến trúc hạ tầng của họ để đạt quy mô cực lớn.
* **Tiếp cận kỹ thuật chuyên sâu:** Việc tập trung vào các cơ sở dữ liệu chuyên dụng và chuyển đổi code với sự hỗ trợ của AI đã cung cấp những insight thực tế để giải quyết technical debt (nợ kỹ thuật).
* **Giao lưu và thảo luận:** Sự kiện mang đến những cơ hội tuyệt vời để trao đổi ý tưởng với những người đam mê cloud và các kiến trúc sư AWS tại địa phương, củng cố tầm quan trọng của cộng đồng trong ngành công nghệ.
* **Bài học kinh nghiệm:** Hành trình hiện đại hóa là một quá trình liên tục. Tận dụng AI không chỉ cho các tính năng, mà để refactor và bảo mật chính cơ sở hạ tầng nền tảng, đang trở thành tiêu chuẩn mới.

### Một số hình ảnh sự kiện
*[Thêm hình ảnh sự kiện của bạn vào đây]*

**Nhìn chung, sự kiện không chỉ cung cấp những cập nhật kỹ thuật quan trọng mà còn giúp tôi định hình lại tư duy về việc mở rộng hạ tầng đám mây, hiện đại hóa ứng dụng cũ và lựa chọn các pattern kiến trúc phù hợp.**
