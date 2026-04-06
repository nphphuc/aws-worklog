---
title : "Luồng Ứng viên"
date : 2024-01-01 
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

#### Hành trình Ứng viên: Tải lên CV & Xử lý

Phần này chi tiết luồng hoàn toàn từ đầu đến cuối khi một ứng viên tải lên CV (resume) của họ lên SmartHire-AI, từ tải lên ban đầu đến kết hợp công việc thông minh.

---

#### Bước 1: Tải lên CV lên S3

Khi ứng viên nhấn "Tải lên CV" trong React SPA:

```
1. Ứng viên chọn tệp PDF
2. React app gửi yêu cầu PUT đã ký S3 qua JWT Cognito
3. Tệp tải lên trực tiếp vào bucket S3 dưới tiền tố "candidates/"
4. S3 tạo sự kiện object được tạo
```

---

#### Bước 2: Xếp hàng & Tiêu thụ (SQS → Lambda)

```
Sự kiện S3 Được tạo
        ↓
S3 → SNS → SQS (CV Queue)
        ↓
Lambda Trigger "ingestion_trigger"
```

**Hàm Lambda Tiêu thụ:**
- Tiêu thụ tin nhắn SQS chứa siêu dữ liệu đối tượng S3
- Xác nhận định dạng tệp (chỉ PDF)
- Trích xuất bucket S3, key, ID ứng viên từ siêu dữ liệu
- Khởi tạo thực thi **AWS Step Functions**
- Trả lại xác nhận SQS

---

#### Bước 3: Điều phối Pipeline (AWS Step Functions)

Máy trạng thái Step Functions quản lý toàn bộ workflow xử lý:

```
KHỞI ĐỘNG
  ↓
[Bước 1] Gọi Lambda cv_jd_processor
          - Trích xuất text từ PDF
          - Phân tích dữ liệu CV
          - Tạo embeddings
  ↓
[Bước 2] Gọi Lambda job_suggestion_engine
          - So sánh CV embeddings vs Job embeddings
          - Xếp hạng & xếp tất cả các công việc mở
  ↓
[Bước 3] Viết Kết quả
          - Lưu trữ theo dõi trong DynamoDB
          - Cập nhật RDS bằng kết quả kết hợp
  ↓
[Bước 4] Xuất bản Kết quả qua AppSync
          - Gửi đột biến GraphQL đến AppSync
          - Thông báo thời gian thực đến UI ứng viên
  ↓
KẾT THÚC (Thành công) hoặc LỖI
```

---

#### Bước 4: CV & Xử lý Tài liệu (Lambda + AI)

**Lambda: cv_jd_processor (Python 3.12)**

Xử lý:
1. Tải PDF từ S3
2. Amazon Textract → Trích xuất text, bảng, thực thể
3. Phân tích cấu trúc:
   - Tên, email, số điện thoại
   - Kinh nghiệm làm việc
   - Giáo dục
   - Danh sách kỹ năng
4. Amazon Bedrock → Làm giàu dữ liệu
5. Amazon Comprehend → Phân tích NLP
6. Tạo embeddings bằng Bedrock

---

#### Bước 5: Công cụ Gợi ý Việc làm

**Lambda: job_suggestion_engine**

Xử lý:
1. Truy vấn RDS cho tất cả các công việc hoạt động
2. Cho mỗi công việc:
   - Tính điểm kết hợp dựa trên:
     * Kỹ năng kết hợp
     * Căn chỉnh cấp độ kinh nghiệm
     * Sở thích vị trí
     * Khả năng tương thích lương
3. Sắp xếp theo điểm kết hợp (giảm)
4. Lọc theo ngưỡng (> 0.6 kết hợp tối thiểu)
5. Chọn 10 công việc phù hợp nhất

---

#### Bước 6: Cập nhật UI Thời gian thực (AWS AppSync)

Sau khi kết hợp hoàn tất:

```
Lambda xuất bản đột biến GraphQL AppSync:
"JobSuggestionsReady"
        ↓
AppSync phát qua subscription
        ↓
React SPA ứng viên nhận cập nhật
        ↓
Dashboard được re-render (KHÔNG cần làm mới trang)
```

---

#### Ưu điểm của Pipeline

✅ Xử lý không đồng bộ - Không chặn ứng viên

✅ Tự động retry với exponential backoff

✅ Bảo vệ timeout - Ngăn các quy trình bị treo

✅ Xử lý lỗi và trạng thái fallback

✅ Cập nhật thời gian thực qua AppSync
